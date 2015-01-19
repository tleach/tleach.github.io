---
layout: post
title: Refactoring your monolithic API
subtitle: The indirection of an orchestration layer can buy you wiggle room you need
categories: architecture
tags: architecture orchestration service-oriented soa microservices monolithic
type: post
header-img: "img/orchestra.jpg"
comments: true
---

Like a lot of quickly-growing startups, GameChanger has gone through it's fair share of growing pains and I'm sure we'll continue to do so. That's one of the great challenges of software engineering.

One specific area of growth which has been challenging has been the private API which devices running the GameChanger app use to sync data to/from our servers.

Like many startups, GameChanger's API stack started out life as a small, synchronous Python-based application doing a few things. Over time as we've added more and more features, our API has incrementally grown into a gargantuan monolithic mountain of Python code which does a whole bunch of different things, not all of which are necessarily logically related to one another.

This presents a number of challenges:

 - as our engineering organisation has scaled, we've necessarily structured our teams around specific areas of the business. Having a number of teams jointly own





Jez Humble

Netflix blog post








## Transcoding legacy message formats



## Buying asynchronous patterns for synchronous services

## Paginating large or unbounded requests

Problem:

In times gone by we implemented some API endpoints which either accept of return payloads with large or unbounded bodies. For example, getting all the teams a user follows.

## Implementing a "batch" endpoint

## Moving an endpoint to a new service
