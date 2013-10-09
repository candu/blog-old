---
layout: post
title: "A Tale of Two Trips"
date: 2013-10-09 09:32
comments: true
categories: [Non-Technical, Location, Text Processing, Visualization]
---

This summer, [Valkyrie](http://www.eecs.berkeley.edu/~valkyrie/) and I travelled
through Eastern Asia.  We started with the Great Barrier Reef and rainforests near
Cairns, then went onwards to Singapore, Vietnam, Cambodia, Malaysia, and Korea
before spending three typhoon-stricken days holed up in Manila, Philippines.

As with our [bike trip](http://fearlesstost.github.io/biketotheearth/), we kept
a [daily journal](http://ramblelust.savageinter.net/) of our travels in blog
form.  In this post, I visualize the two trips from our blog entries.

<!-- more -->

First of all, it feels good to write another post!  The rapidly upcoming [Quantified Self 2013 Global Conference](http://quantifiedself.com/conference/San-Francisco-2013/)
seemed like a good excuse to break seven months of blog neglect, so here I am.

I'll refer to these trips as *Bike to the Earth* and *Ramblelust* respectively,
following the names of each trip blog.  Let's start with a look at the broadest
of broad textual measures, word count.

## Word Count

How many words did we write for each trip?  How many per post, on average?

{% codeblock %}
$ python word_count.py < BikeToTheEarth/posts_normalized.json 
141329 words in 197 posts (717 words/post)

$ python word_count.py < Ramblelust/posts_normalized.json 
61010 words in 86 posts (709 words/post)
{% endcodeblock %}

Not much difference here.  What happens if we graph the word counts over time?

TODO: create graph here

TODO: offer insight

## Word Cloud

[Word clouds are considered harmful](http://www.niemanlab.org/2011/10/word-clouds-considered-harmful/),
but let's make some anyways.  We'll use some [small multiples](LINK) here to
put these word clouds in context.

First, let's get a look at the two trips side-by-side:

TODO: build word clouds

### Authorship

The [Bike to the Earth post data](LINK) has authorship info, but it's unreliable:
it reflects who uploaded the post, not who wrote it.  A better measure of
authorship is the haikus Valkyrie almost always puts at the head of her posts:

TODO: offer example

Using these haikus, we can split the corpus by author:

TODO: build word clouds

### Countries

Thanks to our [Ramblelust categories](LINK) and [Bike to the Earth country flag icons](LINK),
we know which posts were written in which countries.  Let's break down the
text by country:

TODO: build word clouds

## Wrapping Up

TODO: write this
