---
title: Never Query the Same Thing More Than Once
layout: post
---

In a recent post, I talked about how I used [Postgres DISTINCT ON](/postgres-distinct-on/) to remove a pesky N+1 query. That was actually one of the final touches in immproving the `Graphics#index` action for Box Out. Prior to that, I whipped together a pretty good amount of custom data preloading and I thought it might be worth discussing that as well.

## The Background

Our `Graphics#index` action shows two things -- templates and graphics. Templates are what you use to create new graphics (below Create New Graphic header in image). Graphics are all the graphics you have created from templates thus far (below Recently Created header).

<img src="{{site.url}}/images/posts/preload-data/graphics-index.jpg" alt="Box Out Graphics Index Action" />

### The Models

To help you understand the data model, the tldr is org -> team -> template -> graphic. Here is an extremely slimmed down but more verbose than the tldr version of the models:

```ruby
class Org < ApplicationRecord
  has_many :teams
end

class Team < ApplicationRecord
  belongs_to :org
  has_many :templates, as: :owner
  has_many :graphics
end

# source of the contents of the template
class DropboxTemplate < ApplicationRecord
  has_many :templates
end

# data for the template to use and link to dropbox template
class Template < ApplicationRecord
  # owner is polymorphic because of stuff that doesn't matter for this
  # post, so I left it that way but don't worry about why
  belongs_to :owner, polymorphic: true
  belongs_to :dropbox_template
  has_many :graphics
end

# jpg end result of the dropbox template rendered with the template data
class Graphic < ApplicationRecord
  belongs_to :team
  belongs_to :template
end
```

### The Action

The data for the controller action is scoped to a team belonging to an org. This is what the controller looked like when I started on it (abbreviated for clarity).

```ruby
class GraphicsController < ApplicationController
  def index
    @org = Org.find(params[:org_id])
    @team = @org.teams.find(params[:team_id])

    if @team.templates.count == 0
      redirect_to [@org, @team, :store]
    else
      @graphics = @team.graphics.
        includes(team: :org, template: [:dropbox_template, :owner => :org]).
        paginate(page: params[:page])
      @templates = @team.templates.includes(:dropbox_template, :owner => :org)
    end
  end
end
```

Pretty innocent and standard rails, right? If you look at this code and you know Rails, your assumption would be that it is efficiently loading the data to use in the view. When I peaked under the covers though, I found a different story entirely.

### The Sample Data

To show you what was happening, we need a bit of sample data and a view. First the sample data:

```ruby
org = Org.create(name: "Box Out")
team = org.teams.create(name: "Athletics")
gameday = DropboxTemplate.create(name: "Gameday")
score_update = DropboxTemplate.create(name: "Score Update")
gameday_1 = team.templates.create({
  name: "Gameday 1",
  dropbox_template: gameday,
})
gameday_2 = team.templates.create({
  name: "Gameday 2",
  dropbox_template: gameday,
})
gameday_3 = team.templates.create({
  name: "Gameday 3",
  dropbox_template: gameday,
})
score_update_1 = team.templates.create({
  name: "Score Update 1",
  dropbox_template: score_update,
})
score_update_2 = team.templates.create({
  name: "Score Update 2",
  dropbox_template: score_update,
})
score_update_3 = team.templates.create({
  name: "Score Update 3",
  dropbox_template: score_update,
})

team.graphics.create({
  name: "Gameday 1",
  template: gameday_1,
})
team.graphics.create({
  name: "Gameday 2",
  template: gameday_2,
})
team.graphics.create({
  name: "Gameday 3",
  template: gameday_3,
})
team.graphics.create({
  name: "Score Update 1",
  template: score_update_1,
})
team.graphics.create({
  name: "Score Update 2",
  template: score_update_2,
})
team.graphics.create({
  name: "Score Update 3",
  template: score_update_3,
})
```

So we have an org, a team, a few source dropbox templates and 6 templates which each have one graphic.

### The View

Finally, here is a view, which looks nothing like the beautiful image above, but invokes all the associations and thus performs all the queries necessary for me to maybe drop some knowledge.

```erb
<h1><%= @org.name %> - <% @team.name %></h1>

<h2>Templates</h2>
<ul>
  <% @templates.each do |template| %>
    <li>
      <strong><%= template.name %></strong> owned by <%= template.owner.org.name %> - <%= template.owner.name %>
    </li>
  <% end %>
</ul>

<h2>Graphics</h2>
<ul>
  <% @graphics.each do |graphic| %>
    <li>
      <%= graphic.template.dropbox_template.name %> - <%= graphic.template.name %> - <%= graphic.name %>
      <%= graphic.template.owner.org.name %>
      <%= link_to 'View', [graphic.team.org, graphic.team, graphic] %>
    </li>
  <% end %>
</ul>
```

## The Problem

Whew, that was a lot of background. Let's get to the good part now -- tailing a log file. I fired up a `rails server` and hit the route to invoke the `Graphics#index` action (http://localhost:3000/1/1/graphics). When I head back over to my rails server log, I expect to see several queries.

We get the org and team, so that is two queries. We count the templates to see if we need to redirect them to the store to add templates, so that is another query. Lastly, if we make it past the template count conditional, we query for the templates and the graphics. Maybe you expect five queries, but if you note the `includes`, you have to assume at least one more bulk query for each included thing. I count nine included things, so maybe 14 queries.

```
Org Load (0.3ms)  SELECT "orgs".* FROM "orgs" WHERE "orgs"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
Team Load (0.3ms)  SELECT "teams".* FROM "teams" WHERE "teams"."org_id" = $1 AND "teams"."id" = $2 LIMIT $3  [["org_id", 1], ["id", 1], ["LIMIT", 1]]
  (0.4ms)  SELECT COUNT(*) FROM "templates" WHERE "templates"."owner_id" = $1 AND "templates"."owner_type" = $2  [["owner_id", 1], ["owner_type", "Team"]]
Template Load (0.5ms)  SELECT "templates".* FROM "templates" WHERE "templates"."owner_id" = $1 AND "templates"."owner_type" = $2  [["owner_id", 1], ["owner_type", "Team"]]
DropboxTemplate Load (0.4ms)  SELECT "dropbox_templates".* FROM "dropbox_templates" WHERE "dropbox_templates"."id" IN ($1, $2)  [["id", 1], ["id", 2]]
Graphic Load (0.3ms)  SELECT "graphics".* FROM "graphics" WHERE "graphics"."team_id" = $1 LIMIT $2 OFFSET $3  [["team_id", 1], ["LIMIT", 30], ["OFFSET", 0]]
Template Load (0.3ms)  SELECT "templates".* FROM "templates" WHERE "templates"."id" IN ($1, $2, $3, $4, $5, $6)  [["id", 1], ["id", 2], ["id", 3], ["id", 4], ["id", 5], ["id", 6]]
CACHE DropboxTemplate Load (0.0ms)  SELECT "dropbox_templates".* FROM "dropbox_templates" WHERE "dropbox_templates"."id" IN ($1, $2)  [["id", 1], ["id", 2]]
Team Load (0.3ms)  SELECT "teams".* FROM "teams" WHERE "teams"."id" = $1  [["id", 1]]
Org Load (0.3ms)  SELECT "orgs".* FROM "orgs" WHERE "orgs"."id" = $1  [["id", 1]]
```

10 queries and one of them was a cached load, which is not as bad as going over the network, but we still want zero of those. The reason it was not 14 queries is because several of the queries were scoped so Rails smartly set the associations on those for us (e.g. each graphic set :team to `@team`). Still, 10 queries to get 5 different types of data (most of which are nested) is too many queries.

## The Solution

The first thing that struck me was the count of templates followed by a query for templates. Why not query for the templates first and use that instead of the count? Now our controller performs one fewer query and looks something like this:

```ruby
class GraphicsController < ApplicationController
  def index
    @org = Org.find(params[:org_id])
    @team = @org.teams.find(params[:team_id])
    @templates = @team.templates.includes(:dropbox_template, :owner => :org).load

    if @templates.size == 0
      redirect_to [@org, @team, :store]
    else
      @graphics = @team.graphics.
        includes(team: :org, template: [:dropbox_template, :owner => :org]).
        paginate(page: params[:page])
    end
  end
end
```

Note the use of load to kick the query and the use of size to check the in memory array size instead of count which would perform another count query.

### Assign Team Association

Next, if you look at the log above, you will notice that we are querying for org and team because of the includes, even though we already have those in the `Org.find` and `@org.teams.find` calls. Rails models have an association method which has a target. You can assign the target manually to load the data. There might be a newer, better way of doing this, so let me know, but here is how I ensured that org and team were preloaded without the includes.

```ruby
@graphics = @team.graphics.
  includes(template: [:dropbox_template]).
  paginate(page: params[:page])

# iterate the graphics and load the team so we can
# avoid the includes in the graphics query
@graphics.each do |graphic|
  graphic.association(:team).target = @team
  if graphic.template.owner_type == "Team" &&
    graphic.template.owner_id == @team.id
    graphic.template.association(:owner).target = @team
  end
end
```

I removed the `team: :org` includes in favor of assigning the target for each graphic. I also removed the `owner: :org` includes in favor of preloading the owner association. Because I know that we are only getting graphics for one team and the owner of all the templates is the team I can do this. In reality, I wrote some preloading that grouped the templates by owner_type and ensured that any missing owner records (owners other than team) were correctly loaded, but we don't really need to go into that here.

```
Org Load (0.3ms)  SELECT "orgs".* FROM "orgs" WHERE "orgs"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
Team Load (0.3ms)  SELECT "teams".* FROM "teams" WHERE "teams"."org_id" = $1 AND "teams"."id" = $2 LIMIT $3  [["org_id", 1], ["id", 1], ["LIMIT", 1]]
Template Load (0.3ms)  SELECT "templates".* FROM "templates" WHERE "templates"."owner_id" = $1 AND "templates"."owner_type" = $2  [["owner_id", 1], ["owner_type", "Team"]]
DropboxTemplate Load (0.3ms)  SELECT "dropbox_templates".* FROM "dropbox_templates" WHERE "dropbox_templates"."id" IN ($1, $2)  [["id", 1], ["id", 2]]
Graphic Load (0.2ms)  SELECT "graphics".* FROM "graphics" WHERE "graphics"."team_id" = $1 LIMIT $2 OFFSET $3  [["team_id", 1], ["LIMIT", 30], ["OFFSET", 0]]
Template Load (0.3ms)  SELECT "templates".* FROM "templates" WHERE "templates"."id" IN ($1, $2, $3, $4, $5, $6)  [["id", 1], ["id", 2], ["id", 3], ["id", 4], ["id", 5], ["id", 6]]
CACHE DropboxTemplate Load (0.0ms)  SELECT "dropbox_templates".* FROM "dropbox_templates" WHERE "dropbox_templates"."id" IN ($1, $2)  [["id", 1], ["id", 2]]
```

### Preload All Dropbox Templates

At this point, we are down to 7 queries, but we can do better. You can see above that the templates and dropbox templates tables are both hit twice. If we get rid of those then we can get down to 5. If I wait to load dropbox templates until I know all the templates, then I can query them all for `@templates` and `graphic#template` at the same time.

I removed the `:dropbox_template` from the includes of both the templates and graphics queries and added the following lines.

```ruby
# get all the templates in an array, we've already loaded template
# for graphics so we can just map it
all_templates = @templates + @graphics.map(&:template)
# get array of all the uniq dropbox_template_id's, could have used Set, also compact it so we don't have nils
dropbox_template_ids = all_templates.map(&:dropbox_template_id).
  uniq.compact
# get a hash of {id => DropboxTemplate, id2 => DropboxTemplate, etc...}
dropbox_templates = DropboxTemplate.where(id: dropbox_template_ids).
  index_by(&:id)
# loop through all templates and load the target
all_templates.each do |template|
  target = dropbox_templates[template.dropbox_template_id]
  template.association(:dropbox_template).target = target
end
```

Also, I should stop and warn you, if you have not noticed already, that this controller is getting real ugly, real quick. If you were worrying about that, stop. If you are worrying about it now, stop. We can clean that all up later. **First, we will get it fast, then we will make it pretty**.

Ok, back to killing queries. At this point, we are down to 6 queries.

```
Org Load (0.1ms)  SELECT "orgs".* FROM "orgs" WHERE "orgs"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
Team Load (0.2ms)  SELECT "teams".* FROM "teams" WHERE "teams"."org_id" = $1 AND "teams"."id" = $2 LIMIT $3  [["org_id", 1], ["id", 1], ["LIMIT", 1]]
Template Load (0.2ms)  SELECT "templates".* FROM "templates" WHERE "templates"."owner_id" = $1 AND "templates"."owner_type" = $2  [["owner_id", 1], ["owner_type", "Team"]]
Graphic Load (0.3ms)  SELECT "graphics".* FROM "graphics" WHERE "graphics"."team_id" = $1 LIMIT $2 OFFSET $3  [["team_id", 1], ["LIMIT", 30], ["OFFSET", 0]]
Template Load (0.4ms)  SELECT "templates".* FROM "templates" WHERE "templates"."id" IN ($1, $2, $3, $4, $5, $6)  [["id", 1], ["id", 2], ["id", 3], ["id", 4], ["id", 5], ["id", 6]]
DropboxTemplate Load (0.3ms)  SELECT "dropbox_templates".* FROM "dropbox_templates" WHERE "dropbox_templates"."id" IN ($1, $2)  [["id", 1], ["id", 2]]
```

### Preload graphic#template From Already Loaded Templates

The last tweak is that I know all the templates are loaded for the team and the graphics were created using those templates. This means I can try to preload the `graphic#template` association from the already loaded template data. Prior to looping through the graphics to set the team target, I get another nice hash where the key is the template id and the value is the template.

```ruby
templates_by_id = @templates.index_by(&:id)
```

Then, in the iteration of the graphics, prior to using the template, I attempt to load the target:

```ruby
@graphics.each do |graphic|
  graphic.association(:team).target = @team

  if template = templates_by_id[graphic.template_id]
    graphic.association(:template).target = template
  end

  if graphic.template.owner_type == "Team" &&
    graphic.template.owner_id == @team.id
    graphic.template.association(:owner).target = @team
  end
end
```

A refresh of the page and peak at the log shows that we are now down to 5 queries.

```
Org Load (0.3ms)  SELECT "orgs".* FROM "orgs" WHERE "orgs"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
Team Load (0.2ms)  SELECT "teams".* FROM "teams" WHERE "teams"."org_id" = $1 AND "teams"."id" = $2 LIMIT $3  [["org_id", 1], ["id", 1], ["LIMIT", 1]]
Template Load (0.2ms)  SELECT "templates".* FROM "templates" WHERE "templates"."owner_id" = $1 AND "templates"."owner_type" = $2  [["owner_id", 1], ["owner_type", "Team"]]
Graphic Load (0.3ms)  SELECT "graphics".* FROM "graphics" WHERE "graphics"."team_id" = $1 LIMIT $2 OFFSET $3  [["team_id", 1], ["LIMIT", 30], ["OFFSET", 0]]
DropboxTemplate Load (0.3ms)  SELECT "dropbox_templates".* FROM "dropbox_templates" WHERE "dropbox_templates"."id" IN ($1, $2)  [["id", 1], ["id", 2]]
```

## The Conclusion

Booyeah! Of course the code I wrote in real life is not exactly what was written above, but I hope it illustrates the point -- **never query data that you have already queried** and **consider the order that you query the data to maximize efficiency**. Also, Rails is not going to do all of that for you, so watch your logs, think and investigate.

That said, here is the final result of cutting our queries in half:

```ruby
class GraphicsController < ApplicationController
  def index
    @org = Org.find(params[:org_id])
    @team = @org.teams.find(params[:team_id])
    @templates = @team.templates.load

    if @templates.size == 0
      redirect_to [@org, @team, :store]
    else
      @graphics = @team.graphics.paginate(page: params[:page])
      templates_by_id = @templates.index_by(&:id)

      # iterate the graphics and load the team so we can
      # avoid the includes in the graphics query
      @graphics.each do |graphic|
        graphic.association(:team).target = @team

        if template = templates_by_id[graphic.template_id]
          graphic.association(:template).target = template
        end

        if graphic.template.owner_type == "Team" &&
          graphic.template.owner_id == @team.id
          graphic.template.association(:owner).target = @team
        end
      end

      all_templates = @templates + @graphics.map(&:template)
      dropbox_template_ids = all_templates.
        map(&:dropbox_template_id).uniq.compact
      dropbox_templates = DropboxTemplate.
        where(id: dropbox_template_ids).index_by(&:id)
      all_templates.each do |template|
        target = dropbox_templates[template.dropbox_template_id]
        template.association(:dropbox_template).target = target
      end
    end
  end
end
```

As I said to the cleanliness worriers above, I would never ship code in a controller like that. I wrote a custom data preloader that can use already loaded data and efficiently load anything not already loaded ala Rails `includes` mechanism. It is quite specific to my use case, but the end result of the controller looks something like this:

```ruby
@templates = @team.templates.load
@graphics = @team.graphics.paginate(page: params[:page]).load

DataPreloader.call(@graphics, {
  association_name: :template,
  loaded: @templates,
})
DataPreloader.call(@graphics, {
  association_name: :team,
  loaded: @team,
})
DataPreloader.call(@templates, {
  association_name: :parent,
  loaded: @templates,
})

templates_and_parents = @templates + @templates.map(&:parent).compact
DataPreloader.call(templates_and_parents, {
  association_name: :dropbox_template,
})
Template.preload_owners(templates_and_parents, @team) # owners used in latest graphics
Template.preload_latest_graphics(@team, templates_and_parents)
```

That feels a little better, right? I mean that one even loads more data than the simplified example I have been working on and still feels cleaner.

Ok, well that was long. Sorry about that. I felt like my last post was less verbose than it could have been and perhaps went overboard on this one, but hopefully there is a tidbit of goodness in here for you.

By all means, if there are better ways to do anything I did above, I am all ears. I spent several years in the dark corners of GitHub where we had many patterns like this, but were not on modern Rails, so I am far from up to date on these matters.

**Go forth and kill network calls**!
