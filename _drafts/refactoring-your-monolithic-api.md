---
layout: post
title: Refactoring your monolithic API
subtitle: The indirection of an orchestration layer can buy you wiggle room you need
categories: architecture
tags: architecture orchestration service-oriented soa microservices monolithic
type: post
header-img:
comments: true
---

Like a lot of quickly-growing startups, [GameChanger](http://gc.com) has gone through it's fair share of growing pains and I'm sure we'll continue to do so. That's one of the exciting challenges that comes with software engineering.

One specific area of growth which has been challenging has been the private API which devices running the GameChanger app use to sync data to/from our servers.

## From morsel to monolith

Like many startups, GameChanger's API stack started out life as a small, synchronous Python-based application doing a small number of things. Over time as we've added more and more features, our API has incrementally grown into a gargantuan monolithic mountain of Python code which does a whole bunch of different things, not all of which are necessarily logically related to one another.

This presents a number of challenges:

### 1. Lack of ownership boundaries
As our engineering organisation has scaled, we've necessarily structured our teams around specific areas of the business. Having a number of teams jointly own a single component like out API is problematic as the ownership boundaries are fuzzy and no one team truly owns it, which means no one owns it.

When your business relies on the SLA of a component like this being maintained, the propensity for things to fall through the cracks of unowned components like this is high.

Another way to look at this is that it's a violation of [Conway's Law](http://en.wikipedia.org/wiki/Conway%27s_law).

### 2. Technical debt which needs to be tackled
Like most fast growing technology companies, much of our API was created in an era when fast prototyping to obtain product-market fit was prioritized ahead of technical quality. As a result there are some areas of our API which are a little quirky, do things in a non-standard way, or just straight up have some design flaws.

Another term for this is _technical debt_. We all have it, so let's be grown-ups and admit to it. These design flaws are something we would dearly like to refactor out wherever possible.

### 3. APIs live forever
If you maintain an API whose consumers are not under your direct control, then you essentially have to regard every endpoint you publish as immortal.

At GameChanger we typically release new version of our iOS app every 3-4 weeks, but we have no control over how long it takes our users to upgrade to that version. If an albeit small population of our users are still using a version of our iOS app from a year ago which happens to call an API endpoint we'd dearly love to change, we're duty bound to maintain that endpoint nonetheless.

The best you can really do is just create a version 2 of the same endpoint which is used by all new versions of the client going forward and then eventually pull the plug on version 1 when the population of users has reached an acceptable level to "end of life" that iOS app version.

### 4. Tests and builds take longer to run
As a once-small component like our API grows larger and larger, the time it takes to run test suites and builds grows with it. This is a tax on developer time and ultimately profitability. The same small code change which used to take me a small amount of time to get merged and released now takes 2-4 times as long as the suite of  tests I must run in unrelated product areas mounts up.

### 5. Product areas with different SLAs become indirectly coupled
Perhaps the least obvious side effects of a monolithic API like this are indirect coupling effects. I'm not talking about coupling in the traditional programming/OO sense where various modules are more tightly interdependent than they should be (though that is also something worth factoring out). I'm talking about coupling effects which derive from the fact who independent product areas with different operational characteristics and SLAs are simply running together inside the same _process_.

For example, one part of our API deals with user sign-in. It is extremely important for our business that this feature is fast, responsive and available because if it isn't the user immediately notices. Sign-in is a relatively low throughput activity so maintaining a high level of availability should not be too tricky.

Another part of our API is responsible for receiving game events from all the iOS devices being used to score games using the GameChanger app at any given point in time. This traffic has very different operational characteristics to sign-in. This tends to be an extremely high throughput activity. Moreover throughput can spike very quickly is a number of games happen to start together. For traffic like this, which is generated in the background from the user's perspective, we also ideally want our API to be highly available. But when traffic like this can suddenly surge by two orders of magnitude without warning over the course of 5 minutes, it may become necessary to tolerate some slow requests or 503s until the autoscaler has done it's work and more machines have been provisioned. The impact from the user's perspective is tolerable - the iOS app will simply retry any failed calls in the background in due course.

So the problem here is that maintaining these very different SLAs for different classes of traffic with different operational characteristics is nigh-on impossible in a single API component, especially one written with blocking code (more on that in a moment). The game event SLA which allows us to 503 a few requests as we scale will deliver a horrible experience for users attempting to sign-in. The sign-in SLA which ensures we are always available no-matter-what requires us to run far more infrastructure than we generally need at much greater cost.

Different product areas with different SLAs need to be scaled independently.


### 6. Synchronous code is a painful and inefficient to scale
Our API, being over 5 years old, runs on top of Django which is a blocking Python framework. Unlike hot new hipster languages like Go and node.js which use goroutines and callback models respectively to maximise utilization, we spend most of our time in Django just sitting waiting for IO to complete on a fixed number of threads.

Finding the ideal number of threads to run on a given server is tricky as at any given point in time you might be serving a bunch of requests which require a lot of IO but hardly any CPU (in which case you want a lot of threads to maximise your utilization) or bunch of requests which require minimal IO but do a lot processing (in which case you want a smaller number of threads to minimise context switching).

Typically you'll settle for a number of threads which is a compromise between these two extremes.

The really nice thing about non-blocking languages like Go and node.js from a scaling perspective is that the relationship between CPU Utilization and response time is fairly linear. This makes them a breeze to auto-scale.

The problem with blocking code is that knowing when to scale up is not obvious as you'll start to hit performance bottlenecks before you head towards 80% utilization.


## Enter the orchestration layer

![Before](/img/network1.png)

Jez Humble

Netflix blog post



### The plan




### Transcoding legacy message formats



### Buying asynchronous patterns for synchronous services

### Paginating large or unbounded requests

Problem:

In times gone by we implemented some API endpoints which either accept of return payloads with large or unbounded bodies. For example, getting all the teams a user follows.

### Implementing a "batch" endpoint

### Moving an endpoint to a new service
