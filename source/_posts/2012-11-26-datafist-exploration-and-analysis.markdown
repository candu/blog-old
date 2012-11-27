---
layout: post
title: "datafist: Exploration And Analysis"
date: 2012-11-26 21:25
comments: true
categories: [datafist, Non-Technical]
---

In this post I introduce datafist, a tool for interactive data analysis.
datafist provides visual and gestural actions for building analysis
pipelines, making the process of analysis more accessible to
non-programmers.

<!-- more -->

## First, A Quote

{% blockquote Direct Manipulation Interfaces http://cleo.ics.uci.edu/teaching/Winter10/231/readings/1-HutchinsHollanNorman-DirectManipulation-HCI.pdf %}
Are we analyzing data? Then we should be manipulating the data themselves; or if we are designing an analysis of data, we should be manipulating the analytic structures themselves.
{% endblockquote %}

## The Sixfold Path Of Self-Tracking

Self-tracking is a complex process. It can be broken down into stages:

- *Intent:* our initial goals or motivations in deciding to self-track.
- *Tools:* the devices or methods we use for tracking.
- *Measurement:* collecting our data using those tools.
- *Analysis:* extracting insights from our data.
- *Interpretation:* creating personal meaning from those insights.
- *Action:* responding to that personal meaning.

Thanks to smartphones and cheap sensors, many of us already have the tools
necessary for measurement. The psychology of intent and action is rapidly
being explored through [Habit Design](http://www.meetup.com/habitdesign/) and [Captology](http://captology.stanford.edu/), with
startups like [Lift](http://lift.do/) and [Beeminder](https://www.beeminder.com/) harnessing the findings
to great effect.

*What are the technologies of analysis and interpretation?*

### Visualization

Visualizations, even interactive ones, are usually designed to *answer
specific questions* such as [how did Obama win re-election?](http://www.nytimes.com/interactive/2012/11/07/us/politics/obamas-diverse-base-of-support.html)
Within the context of those questions, they help us understand our data
intuitively. Outside that context, however, they are often useless.

### Statistical/Mathematical Software

Environments like [R](http://www.r-project.org/) and [Mathematica](http://www.wolfram.com/mathematica/) allow you to *explore
your data in meticulous detail.* In the hands of people like
[Stephen Wolfram](http://blog.stephenwolfram.com/2012/03/the-personal-analytics-of-my-life/), they are the holy grail of data analysis. For the
less technically inclined, they remain hopelessly unintuitive.

## A Middle Road

What we need is *something between the two*, a hybrid that exposes the
exploratory power of the latter through the intuitive interface of the former.
Such a tool would give us the opportunity to *explore our data as we see fit.*
We could ask our data questions, iterating quickly on those questions until
we reach useful insights.

Until we can all converse with our data with the fluency of [Hans Rosling](http://www.ted.com/talks/hans_rosling_the_good_news_of_the_decade.html),
there's still room for improvement!

## datafist

In an attempt to demonstrate what this might look like, I've started
working on an HTML5-based in-browser tool called
[datafist](https://github.com/candu/datafist). Rather than explain datafist in
words, I'll *walk through a sample flow* in screenshots and design mocks.

First, you import your personal data:

{% img https://lh3.googleusercontent.com/-JCtNtNaRc2w/ULRaPWjpW1I/AAAAAAAAAPA/-2dDxgnwRiI/s800/1-channels-import.jpg %}

Your data is broken into channels, which appear in the palette:

{% img https://lh5.googleusercontent.com/-m2bQ50wZyYQ/ULRaJnLSCdI/AAAAAAAAAOI/rTk3IFCWhYQ/s800/2-channels-appear.jpg %}

You drag the channels from the palette to the viewer:

{% img https://lh4.googleusercontent.com/-n4rHxwwQyBc/ULRaLBPT8XI/AAAAAAAAAOQ/yuaCkCsK0cw/s800/3-channels-dragged.jpg %}

You then connect the channels to a view:

{% img https://lh3.googleusercontent.com/-zc8CaVt66dI/ULRaLsqUkYI/AAAAAAAAAOY/KdLY8mT7mjI/s800/4-channels-connected-to-view.jpg %}

By clicking `show`, you see the channel data rendered as a sparkline graph:

{% img https://lh3.googleusercontent.com/-zc8CaVt66dI/ULRaLsqUkYI/AAAAAAAAAOY/KdLY8mT7mjI/s800/4-channels-connected-to-view.jpg %}

You decide to zoom in on a specific time range:

{% img https://lh6.googleusercontent.com/-BcQxYKzQg6E/ULRaO0kqszI/AAAAAAAAAPE/1WNp8Jdw-dg/s800/6-view-sparkline-zooming.jpg %}

Switching back to the box-and-arrow diagram with `hide`, you see that it has
changed to include the time range filter. You swap `view-channel` for a different
view, `view-regress`:

{% img https://lh3.googleusercontent.com/-Fk75BTYvEZY/ULRaOQkBY8I/AAAAAAAAAOo/dOiNIYwz-5g/s800/8-view-changed.jpg %}

This view renders your data as a linear regression. You're curious about the
data near top-right, so you drag a region around them:

{% img https://lh3.googleusercontent.com/-7qipp4xp3mU/ULRaRHbm_PI/AAAAAAAAAPM/IZFkSMML7OY/s800/10-region-select.jpg %}

As you use datafist, *you're constantly modifying the analysis itself.*
This modification takes place through *visual and gestural actions:* you
move channels to the viewer, drag out ranges of time to zoom in on, and draw
regions around interesting clusters. As you do this, you can *see the effects* of
those actions on the analysis diagram.

## Inspiration

- [AudioMulch](http://www.audiomulch.com/) is an awesome graphical audio synthesis tool.
- [PureData](http://puredata.info/) and [Max/MSP](http://cycling74.com/products/max/) are visual signal processing languages.
- [Direct Manipulation Interfaces](http://cleo.ics.uci.edu/teaching/Winter10/231/readings/1-HutchinsHollanNorman-DirectManipulation-HCI.pdf) is a classic paper on the design of
  natural-seeming interfaces. You'll recognize some of the interface concepts
  from the analysis package design mocks at the beginning.
- [FAKEGRIMLOCK](http://www.startuplessonslearned.com/2011/11/startup-is-vision.html) is a fountain of poorly-Englished wisdom on
  entrepreneurship.
