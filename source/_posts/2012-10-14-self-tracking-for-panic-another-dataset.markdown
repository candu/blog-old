---
layout: post
title: "Self-Tracking for Panic: Another Dataset"
date: 2012-10-14 21:23
comments: true
categories: [Panic, Technical]
---

In this post, I perform the same analyses presented in
[my last post](LINK) using data from my second panic tracking period.
I then test whether my average alcohol and sugar consumption changed
measurably between the two tracking periods.

During the second tracking period, I gathered data using
[qs-counters](LINK), a simple utility I built for reducing friction in
the recording process.

<!-- more -->

## The Usual Suspects

### Linear Regression

During the second tracking period, alcohol consumption remained
relatively constant:

TODO: graph (Alcohol Consumption)

Sugar consumption is a different story, with a pronounced negative trend:

TODO: graph (Sugar Consumption)

The evidence to suggest that alcohol and sugar consumption are
[comorbid](LINK) is also much stronger now:

TODO: graph (Alcohol vs. Sugar Consumption)

On the other hand, the previous-day alcohol effect seems to be
non-existent:

TODO: graph (Alcohol: Previous vs. Current)

### Fast Fourier Transform

With more data points, the FFT frequency amplitude plot is more muddled:

TODO: graph (FFT frequencies)

The 2-day and 7-day effects previously "discovered" are nowhere to be
found.

### Maximum Entropy Modelling

I didn't record panic attacks during this tracking period. My previous
efforts reduced the severity and frequency of these attacks drastically,
enough so that the data here would have been extremely sparse.

In the absence of that data, I asked a different question:

{% blockquote %}
What features best predict my exercise patterns?
{% endblockquote %}

Here are the top features from `MaxentClassifier`:

TODO: insert data here

## Student's t-test

### What?

TODO: write this

### Why?

In a self-tracking context, you might ask the following questions:

- Did I improve significantly across tracking periods?
- Is my behavior consistent across tracking periods?

Student's t-test can help address both questions.

### The Data

You can see the code for this analysis [here](LINK).

My alcohol consumption was significantly lower during the second
tracking period than during the first, but my sugar consumption remained
relatively stable:

{% codeblock %}
TODO: insert data
{% endcodeblock %}

## Conclusions

TODO: write this

## Thought Experiments

### Repeat Yourself: A Reflection On Self-Tracking And Science

One criticism often launched at the Quantified Self community is that
self-tracking is not *scientific* enough:

TODO: find quote

To be fair, there are a host of valid concerns here. For starters,
it's very difficult to impose a controlled environment when self-tracking.
In a Heisenbergian twist, being mindful of your behavior modifies the
behavior you're trying to measure. (TODO: get link)

Additionally, a sample population of one is meaningless. Will your
approaches work for others? Did you gather the data in a consistent
manner? Are your sensors working properly? The usual antidote is to
increase the sample population, but then you have another set of problems.
Are all participants using the same sensors in the same way? Are they all
running the same analyses?

From watching several presentations about self-tracking, there is a
curious pattern: like any other habit, the tracking habit is hard
to maintain. As a corollary, many tracking experiments consist of multiple
tracking periods, these punctuated by relapses of the tracking habit.

Many people interpret these relapses as failures, but they're actually
amazing scientific opportunities! These are chances to re-run the same
experiment, verifying or confounding results from your earlier tracking
periods.

### The Predictive Modelling Game

Predictive modelling could be an interesting component of a habit
modification system. Suppose I want to exercise more regularly. First, 
I select several features that seem likely to influence my exercise
patterns, such as:

- Am I travelling? Where am I?
- What foods did I eat? When? How much?
- How positive or negative is my mood?
- Did I schedule time today to exercise?
- Did I exercise yesterday? How much?

Next, I gather some baseline data by tracking these features along with
my exercise patterns. I then use that data to train a classifier.
Finally, I keep tracking the features, ask the classifier to predict my
exercise activity, and play a simple game with myself:

{% blockquote %}
Can I beat the classifier?
{% endblockquote %}

That is, can I exercise more often than my existing behavior patterns
suggest I should?
