---
layout: page
header-img: "img/microphone.jpg"
title: Talks
---

On this page I post details of talks I've given to the software engineering community.

************

# Scaling Engineering with Docker

[Travis Thieman](http://twitter.com/thieman) and I delivered a talk to the [CTO School meetup](http://www.meetup.com/ctoschool) in NYC covering our experience migrating [GameChanger](http://gc.com)'s build and deploy pipelines from being heavily based on [Chef](https://www.chef.io/) to one based around [Docker](https://www.docker.com).

<iframe src="//www.slideshare.net/slideshow/embed_code/44505333" width="425" height="355" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe>

This presentation is split in to two main sections. The first section covers the motivations for why GameChanger, as a fast-growing startup, identified a need to replace it's existing Chef-based deploy model with a model which reduces deploy-time risk and allows its engineering team to scale.

The second section is a high-level walkthrough of the new GameChanger deploy pipeline based around Docker.

<iframe src="//player.vimeo.com/video/119260316" width="500" height="333" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>

******************

<br/>

# Monufacture: Effortless Test Data for MongoDB
One of the biggest selling points of [MongoDB](http://www.mongodb.com) is its ability to directly persist arbitrary object structures without requiring the developer to navigate issues like building an ORM layer. However, this flexibility comes at a price - creating meaningful test data which adheres to these more complex structures can be much more involved.

At GameChanger we observed that developers typically had to write large amounts of test data setup boilerplate to perform an effective test against a MongoDB-dependent function, dis-incentivizing them from writing rigorous tests. So we created [Monufacture](http://github.com/gamechanger/monufacture) - a Python test data generation framework for MongoDB that makes setting up test data a breeze.

<iframe src="//www.slideshare.net/slideshow/embed_code/44712551" width="476" height="400" frameborder="0" marginwidth="0" marginheight="0" scrolling="no"></iframe>

In this talk (originally given at the [MongoDB Meetup in NYC](http://www.meetup.com/New-York-MongoDB-User-Group/events/212225672/)) I break down some of motivations and design decisions behind Monufacture, demoing its functionality and giving some tips on how to write effective tests of your MongoDB-dependent code.

