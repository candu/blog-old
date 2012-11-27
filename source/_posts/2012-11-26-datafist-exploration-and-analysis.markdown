---
layout: post
title: "datafist: Exploration And Analysis"
date: 2012-11-26 21:25
comments: true
categories: [datafist, Non-Technical]
---

{% blockquote %}
Are we analyzing data? Then we should be manipulating the data themselves; or if we are designing an analysis of data, we should be manipulating the analytic structures themselves.
{% endblockquote %}
- Direct Manipulation Interfaces, Hutchins et al.

<!-- more -->

Self-tracking is a complex process. It can be broken down into stages:

- *Intent:* our initial goals or motivations in deciding to self-track.
- *Tools:* the devices or methods we use for tracking.
- *Measurement:* collecting our data using those tools.
- *Analysis:* extracting insights from our data.
- *Interpretation:* creating personal meaning from those insights.
- *Action:* responding to that personal meaning.

Thanks to smartphones and cheap sensors, many of us already have the tools
necessary for measurement. The psychology of intent and action is rapidly
being explored through [Habit Design](LINK) and [Captology](LINK), with
startups like [Lift](LINK) and [Beeminder](LINK) harnessing the findings
to great effect.

What are the technologies of analysis and interpretation?

### Visualization

Visualizations, even interactive ones, are usually designed to answer
specific questions such as [TODO: link to a d3.js/mbostock example](LINK).
Within the context of those questions, they help us understand our data
intuitively. Outside that context, however, they are often useless.

### Statistical/Mathematical Software

Environments like [R](LINK) and [Mathematica](LINK) allow you to explore
your data in meticulous detail. In the hands of people like
[Stephen Wolfram](LINK), they are the holy grail of data analysis. For the
less technically inclined, they remain hopelessly unintuitive.

## A Middle Road

What we need is something between the two, a hybrid that exposes the
exploratory power of the latter through the intuitive interface of the former.
Such a tool would give us the opportunity to explore our data as we see fit.
We could ask our data questions, iterating quickly on those questions until
we reach useful insights.

Until we can converse with our data like [Hans Rosling](LINK), the technologies
of analysis and interpretation will lag behind the rest of the self-tracking
ecosystem.

## datafist

In an attempt to demonstrate what this might look like, I've started
working on a tool I call [datafist](LINK). Rather than explain datafist in
words, I'll walk through a sample flow in screenshots and design mocks.

First, you import your personal data:

TODO: screenshot

Your data is broken into channels, which appear in the command palette:

TODO: screenshot

You drag the channels from the palette to the viewer:

TODO: screenshot

You then connect the channels to a view:

TODO: screenshot

By clicking `show`, you see the channel data rendered as a sparkline graph:

TODO: screenshot

You decide to zoom in on a specific time range:

TODO: screenshot

Switching back to the box-and-arrow diagram, you see that it has changed to
include the time range filter:

TODO: screenshot

You swap the view for a different view:

TODO: screenshot

This view renders your data as a linear regression:

TODO: screenshot

You're curious about the data at top-right, so you drag a region around them:

TODO: screenshot

With your analysis finished, you send its textual representation to a friend.

TODO: screenshot

## Inspiration

- [AudioMulch](LINK) is an awesome graphical audio synthesis tool.
- [PureData](LINK) and [Max/MSP](LINK) are visual signal processing languages.
- [Direct Manipulation Interfaces](LINK) is a classic paper on the design of
  natural-seeming interfaces.
- [FAKEGRIMLOCK](LINK) is a fountain of poorly-Englished wisdom on
  entrepreneurship.

## Data Expo!

TODO: shoutout for data expo (where?)

