---
title: Postgres Distinct On
layout: post
---

By day, I am a mild mannered owner/programmer at <a href="https://boxoutsports.com">Box Out Sports</a>. By no means do you need to understand all that we do there or how we do it, but recently I fixed a tricky N+1 query and thought I should write it up.

The need arises when you have a parent child relationship and you need the first child matching some criteria. Simple enough. The tricky part is when you want to bulk load a single child for several parents to avoid an N+1 query.

## The Background

On our graphics page, which lists all the available templates for a team, we show the templates along with a preview. An elegant touch we added is that the preview is greyscale if you have never used the template to create a graphic, but in color if you have.

In addition to showing the template preview in color when used, we show the latest graphic you created using that template as the preview image instead of the stock preview that comes in our template store.

<img src="{{site.url}}/images/posts/postgres-distinct-on/graphics.png" alt="Box Out Graphics Page" />

## The Problem

Seems like a nice touch and innocent enough, right? Yes and no. To determine which preview to show, we were doing a latest graphic query for each template. The `latest_graphic` method looked something like this:

```ruby
def latest_graphic
  owner.graphics.where(template_id: id).order("created_at desc").first
end
```

If any graphics had been created for the template, we used the most recent one as the preview. If there were no latest graphics, we fell back to the store preview. To accomplish that, `preview_url` looked something like this:

```ruby
def preview_url
  if latest_graphic.present?
    latest_graphic.preview_url
  else
    # plain old template store preview
  end
end
```

Many teams have many templates, which meant many queries for the latest graphic of each template each time the page loaded. The more templates our customers had, the slower this page was. You might think to eager load the association, but this is not a simple `includes` in Rails.

My goal was to get rid of the N+1 in favor of a single query that would return the latest graphic for each template. From there, I could preload the latest graphic to avoid the N+1 lookup. The part I was stuck on was how to do a bulk query for all the latest graphics of multiple templates, while only returning the most recent graphic over the wire.

## The Solution

In order to understand the solution, a little background on the schema would likely be useful, so let's pretend it is as simple as this:

* `teams` has `id` and `name`.
* `templates` has `id`, `team_id` and `title`.
* `graphics` has `id`, `template_id`, and `created_at`.

### The SQL

From the [Postgres documentation on the DISTINCT clause](https://www.postgresql.org/docs/9.4/sql-select.html#SQL-DISTINCT):

> SELECT DISTINCT ON ( expression [, ...] ) keeps only the first row of each set of rows where the given expressions evaluate to equal. The DISTINCT ON expressions are interpreted using the same rules as for ORDER BY (see above).

Boom. That is exactly what I needed &mdash; a distinct graphic based on `template_id` ordered by `created_at`. That led me to the following query:

```sql
  SELECT DISTINCT on (graphics.template_id) graphics.*
    FROM graphics
   WHERE team_id = :team_id AND template_id IN (:template_ids)
ORDER BY template_id, created_at DESC
```

### The Ruby

From there, I plopped it in Ruby land. Along with the query, I added memoization to the reader method (`latest_graphic`) that supports memoizing nil values and a writer method (`latest_graphic=`) that can be used to preload the instance variable used for memoization.

```ruby
class Template
  def self.preload_latest_graphics(team, templates)
    binds = {
      team_id: team.id,
      template_ids: templates.map(&:id),
    }
    statement = <<-SQL
        SELECT DISTINCT on (graphics.template_id) graphics.*
          FROM graphics
         WHERE team_id = :team_id AND template_id IN (:template_ids)
      ORDER BY template_id, created_at DESC
    SQL
    graphics = Graphic.find_by_sql([statement, binds])

    # Index by template_id for fast lookup.
    by_template_id = graphics.index_by(&:template_id)

    # Loop through templates and preload the latest_graphic ivar.
    templates.each do |template|
      template.latest_graphic = by_template_id.fetch(template.id, nil)
    end
  end

  # Reader method that is memoized. defined? lets us memoize nil.
  def latest_graphic
    return @latest_graphic if defined?(@latest_graphic)

    @latest_graphic = owner.graphics.where(template_id: id).
      order("created_at desc").first
  end

  # Writer method used above to populate the latest_graphic ivar.
  def latest_graphic=(graphic)
    @latest_graphic = graphic
  end
end
```

With that in place, all I needed to avoid the N+1 call was to use that in the controller.

```ruby
class GraphicsController < TeamAdminController
  def index
    @team = Team.find(params[:team_id])
    @templates = @team.templates
    Template.preload_latest_graphics(@team, @templates)
  end
end
```

Just like that my server logs went from N+1 queries to the graphics table to 1 query from the graphics table. This dramatically reduced the load time for this page, especially for some larger customers that have 100's of templates.

Another place I used this pattern was on <a href="https://speakerdeck.com">Speaker Deck</a>. I used it for the cover slide. A deck has_many slides. Given an array of decks, I can use `DISTINCT ON` to query the first slide for each deck and show the image for that as a preview for the deck. We call it the cover image.

## The Downside

The only real downside of this approach is that postgres itself still has to filter through all the matching records to do the distinct part. If you really can't afford that then your best bet is likely to denormalize some field that you can index and use as your criteria.

In the Speaker Deck example, I could add a column named `slide_number` or something. Then, I could query all the slides `WHERE slide_number = 0 and deck_id IN (:deck_ids)` in a quick, indexed fashion.

Super handy and super easy. Go forth and distinct on whatever your heart desires.
