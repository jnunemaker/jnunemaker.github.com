---
title: All Requests Are Not Equal
layout: post
---

The norm for rate limiting these days seems to be based on the number of requests a consumer can make in a given time window. The problem with this is that all requests are not equal. **Some requests are more expensive than other requests and should be treated as such.**

## A Concrete Example

To demonstrate that all requests are not equal, let us imagine paginating all of the stargazers for [twbs/bootstrap](https://github.com/twbs/bootstrap) using the GitHub API. As of the time of this writing, a little over 90k GitHub users have starred that repository.

A quick curl of the GitHub API shows the pagination and rate limiting information (non-essential headers omitted):

```
$ curl -I https://api.github.com/repos/twbs/bootstrap/stargazers
HTTP/1.1 200 OK
Date: Sun, 13 Dec 2015 15:57:37 GMT
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 59
X-RateLimit-Reset: 1450025857
Link: <https://api.github.com/repositories/2126244/stargazers?page=2>; rel="next", <https://api.github.com/repositories/2126244/stargazers?page=1334>; rel="last"
```

Based on that information, if we wanted to fetch all the stargazers for twbs/bootstrap, we would have to make 1334 requests. As we paginate through those stargazers, GitHub removes a token for each request (one page of stargazers) until we burn through our rate limit or fetch all the stargazers and discontinue requests.

## The Problem

The problem for GitHub is that **each page is a bit more expensive to build** than the previous. The tl;dr of why each page is more expensive is that MySQL has to check and count each row. MySQL cannot skip directly to the offset you provide. For more on this, you can read [MySQL ORDER BY / LIMIT performance: late row lookups](http://explainextended.com/2009/10/23/mysql-order-by-limit-performance-late-row-lookups/).

If the query to get the stargazers for the first page takes 100ms, **the query for the last page of stargazers could take a second or even multiple seconds**.  As I said before, all requests are not equal.

Multiple this effect by all the actions in your application and this means that request based rate limiting does not really protect you. All it takes is for enough consumers to request your slowest resources (page one thousand of anything) and your service will become saturated.

## The Solution

Instead of counting all requests as equal, **make more expensive requests consume more tokens**. There are several ways one could do this. The solution I will talk about here is based on the duration of the request. Using the same framework/code as request based rate limiting, you can simply increment by the duration of the request rounded up to the nearest second.

The rounding up means you are not giving away exact server side timing. Additionally, I would think the goal for most apps is sub-second response times (probably sub 100-250ms). This means that most requests will use the same rate limit as in a request based rate limit system as 100 milliseconds rounds up to 1 second and therefore 1 token.

What is neat about this idea is where the request and cost based solutions diverge. Imagine one of your services/databases starts returning responses slower than usual. With the request based limiting, consumers can consume at the same rate. With the cost based limiting, they cannot.

Instead, **when the timing slows down, the consumers burn through tokens more quickly**, which means they get rate limited more quickly. Rate limited responses are dramatically faster to generate, which means as timing slows down, you can effectively shed load and lessen the burden on the downstream service. The opposite is also true. If you improve response times, you free up more tokens, which means consumers can make more requests.

## The Solution's Problem

Everything is about tradeoffs. No solution is perfect and this solution comes with some problems as well. One problem with this solution is that consumers do not definitively know how many requests they can make in a given time window. Personally, I do not think this is a real problem.

Any API with rate limiting provides information about those rate limits in the response headers and that is what should be programmed for, not a number documented on a website. Also, it would be fairly easy to track cost and include typical costs for endpoints in the docs (updating it a few times a year; could even be based on parameters like page).

A more legitimate problem is concurrency. In order to increment based on duration, you need to wait for the request to complete. This means if enough concurrent requests were issued, you could DOS the application for a period of time.

A way around this perhaps would be to track typical costs and remove tokens based on typical cost at the beginning of a request, rather than waiting for the requeset to complete. This would, unfortunately, slow down the auto-balancing that cost based solution provides as it would take time for the rolling numbers to increase, but could help against the concurrency issue.

## The Truth

The truth is concurrency is a far better way to rate limit than both request and cost based, but is also far more difficult &mdash; both to implement and  configure. **I feel that cost (duration) based rate limiting is a nice balance between request and concurrency based**. It provides a lot more protection than request based, but is far easier to implement and configure than concurrency based.
