---
title: Database Performance Simplified
layout: post
---

I have been fortunate enough to work on a few high throughput applications (thousands of requests per second) in my life where a database was the bottleneck. I am far from an expert on any database, but I have a pretty good model for how to use databases in a way that they get the job done, assuming realistic expectations of hardware/networking.

Most of the database performance articles I have read involve hardware specs, configuration changes, or are too specific about a data set and access patterns to build an overall model to live by. I guess it is best said that they are really focused on a specific problem, which is good if you have that specific problem, but if you just want to learn in general, you have to gather together a lot of scraps.

My hope for this post is to provide a general model or guidelines to live by that won't solve all your problems, but will keep you from hanging yourself overly quickly.

## Balance

This is an overly simplistic model, but indulge me. Imagine you have 100 units of work and only 100. If you use all 100 for writes, writes are fast, but reads are painfully slow. If you use all 100 for reads, reads are fast, but writes are painfully slow.

The goal is to balance the appropriate number of units toward reads and writes where both are fast enough, but neither is perfect. It is worth noting that a plethora of web applications are read heavy, often to the tune of 95% reads (or more). GitHub.com is probably 97% reads.

If your application is 97% reads, does that mean you should be liberal with indexes and apply 97 of the 100 units towards reads? Definitely not. Balance. Use the appropriate number of indexes to get performance that is fast enough.

## Fast Writes

Fast writes are easy. Create a table and insert rows into it. Never update or delete rows. Do not add any indexes. Boom. Your writes will be pretty fast using whatever database you choose. To go even faster, do bulk inserts on the client side when possible, so you don't have network overhead for every insert.

If you want to then randomly read that data by any columns other than the primary key, good luck. With no indexes, you will end up with a lot of full table scans. The larger your table grows the slower your reads will become.

## Fast Reads

Fast reads are a bit trickier (hard to beat doing nothing for fast writes). There are several solutions, but they basically all involve reducing the amount of data the database has to read to fulfill a query. Most of the ideas below can be used in combination as well, if necessary.

### Add Indexes

The simplest way to improve reads is to add indexes. When indexing, the goal is to get to a point where the queries you need to answer don't have to sift through more than a few hundred or at most thousand rows. Ideally, the only data the database has to sift through is exactly the data you need to answer the query.

Let's use GitHub's Gist as an example. Each gist is owned by one user. This means we have a `users` table and we have a `gists` table with a `user_id` column. In order to show the most recent 50 of a user's gists, we might do a query like so:

```sql
SELECT
  *
FROM
  gists
WHERE
  user_id = ?
ORDER BY
  created_at desc;
LIMIT
  0, 50
```

Without any indexes, the database will need to read the entire gists table to satisfy a query for an individual user. As more users create gists, each query for a user's latest gists will get slower and slower.

The first step to make this query faster would be to index `user_id`. Doing so means that the database now only needs to hit the index to get all the gists for a user instead of the entire table. Additionally, the database will need to read the rows for those index entries, order them all by created_at desc (since we have the order by) and then discard anything after the first 50 (since we have the limit).

The problem with the `user_id` index is two fold. First, the more gists an individual user has, the more data the database will need to read and the slower the query will be. Second, databases can rarely only read a single row from disk. They typically work more with pages, which may hold 4-16k of rows per page. This means that even though you only asked for `user_id = 1`, you are asking the database to actually read far more data than just for that user.

To fix problem the first problem with our index, we can do a composite index on `user_id` and `created_at`. In MySQL, that would look like this:

```sql
CREATE INDEX index_user_created ON gists (user_id, created_at);
```

I picture composite indexes like a nested hash. This is far from accurate, but fits the bill for a usage/performance mental model.

```ruby
index = {
  1 => {
    2015-09-8 00:00:00 => [1, 2, 3],
    2015-09-9 00:00:00 => [4, 5, 6],
    2015-09-10 00:00:00 => [7, 8, 9],
  },

  2 => {
    2015-09-10 00:00:00 => [10, 11, 12],
  },
  # etc, etc et
}
```

In the structure above, 1 and 2 would be user ids, the times would be sorted ascending and the values for a given user id and date would be rows in the gists table. Picturing the composite index like this is useful because composite indexes are only useful left to right.

If you need to query for all the gists of two users, this index will help. If you need to query for all the gists of a single user on at a given time, this index will help. Additionally, if you need to query for all gists of a single user for a given time range or ordered by time, this will help. If you want to query for all gists (across all users) order by time, this will not help.

This means if you have a composite index on `user_id` and `created_at`, you do not need an index on `user_id` as the composite will cover you. Column order in composite indexes is important. As far as a rule of thumb, you should order by whatever will satisfy the most queries or when only dealing with common queries, whatever will reduce the rows your database has to read.

### More Buckets

Put data in separate databases, tables, buckets, or whatever the database of choice calls a collection of records that are related.
