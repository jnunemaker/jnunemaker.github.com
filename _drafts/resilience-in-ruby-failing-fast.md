---
title: 'Resilience in Ruby: Failing Fast'
layout: post
---

Earlier this year, I [wrote in depth about handling failure](/resilience-in-ruby/) in Ruby apps. Handling failure is step one, but by itself will not make your app resilient. Imagine a scenario where all the failures are handled, but an operation that was taking 100ms is now taking 1 second. An order of magnitude or more increase in duration will nearly always lead to utilization saturation. In other words, your app will be unavailable.

Rather than fail gracefully and slowly until your service is saturated, you need to shed load and fail fast. One tool to aid in that is a circuit breaker and that is what I am going to talk to you about today.

## Resilient
