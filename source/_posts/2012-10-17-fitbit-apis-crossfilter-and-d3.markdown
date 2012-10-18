---
layout: post
title: "Fitbit: APIs, crossfilter, and d3.js"
date: 2012-10-17 08:30
comments: true
categories: [Panic, Technical]
---

In this post, I present
[fitbit-crossfilter](https://github.com/candu/fitbit-crossfilter), which uses
the [Fitbit API](http://dev.fitbit.com/), [crossfilter](http://square.github.com/crossfilter/) and [d3.js](http://d3js.org/)
to provide an interactive visualization for exploratory analysis.

<!-- more -->

## The Inspiration

It was early April 2012. By this point, I'd been through a stint of pen-and-paper
self-tracking for [panic recovery](/blog/2012/10/03/panic/).
I'd [just received my Fitbit in the mail](/blog/2012/10/16/fitbit-my-brief-experience/). 

Earlier that year, I applied to the [EECS PhD program at UC Berkeley](http://www.eecs.berkeley.edu/Gradadm/) with
[this statement of purpose](https://docs.google.com/document/d/10PupOF0RLa54o6y9_xBGnj7VbjtQNPab0-HdoVfT6gA/edit). I was fascinated by this idea that *pervasive
gameplay really could make us all better*, that somewhere beyond the rat wheel
of gamification was hidden a Shangri-La of game-driven awesome.

That unfortunately didn't pan out, and I was left with the age-old question:

{% blockquote %}
What do I do with this idea?
{% endblockquote %}

It was around this time that, in a moment of exquisite
digital serendipity, [Meetup](http://www.meetup.com/) suggested I check out the
[Bay Area Quantified Self Meetup Group](http://www.meetup.com/quantifiedself/).

Quantified Self? [What's that?](/blog/2012/10/02/welcome-to-quantified-savagery/). As I explored the group page, I felt
a rush of clarity: *this was exactly what I'd been doing!* There's a whole
community of people turning their lives into games in the name of
self-betterment!

I bit the bullet and forked over hard cash to sign up for
[QS Show&Tell #25](http://www.meetup.com/quantifiedself/events/58370532/) at the
[California College of the Arts](http://goo.gl/maps/fn8H4). It was everything I'd hoped for.
One presenter dissected 30 years of medical data and correlated it with
his marital status. Another showed off a cyclist threat detection system
cobbled together by mounting a webcam and sonar unit to his handlebars.
There was a *rich vein of inquiry into awesome here.* I was hooked.

[Beau Gunderson](http://www.beaugunderson.com/) of
[Singly](https://singly.com/) presented [zeo-crossfilter](https://github.com/beaugunderson/zeo-crossfilter).
That was the turning point. I saw what he had done and said

{% blockquote %}
Hey, I can build that!
{% endblockquote %}

And so [fitbit-crossfilter](https://github.com/candu/fitbit-crossfilter) was born.

## The Tools

As mentioned, [fitbit-crossfilter](https://github.com/candu/fitbit-crossfilter) is a mashup between
the [Fitbit API](http://dev.fitbit.com/),
[crossfilter](http://square.github.com/crossfilter/),
and [d3.js](http://d3js.org/).
I'll go over each part with examples.

### Fitbit API

The Fitbit API uses [OAuth](http://oauth.net/) for authentication. If you've never
confronted OAuth before, it can be confusing. To compound the confusion, *every
API provider seems to do it slightly differently.* The
[official Fitbit docs](https://wiki.fitbit.com/display/API/OAuth+Authentication+in+the+Fitbit+API) are opaque, the
[OAuth specs](http://tools.ietf.org/html/rfc5849) are even more opaque, and
the [unofficial apis.io listing](http://apis.io/Fitbit) is just wrong:

{% codeblock lang:bash %}
$ curl -X GET -u '<username>:<password>' http://api.fitbit.com/1/user/-/profile.json 2>/dev/null | jsonpp
{
  "errors": [
    {
      "errorType": "oauth",
      "fieldName": "n/a",
      "message": "No Authorization header provided in the request. Each call to Fitbit API should be OAuth signed"
    }
  ]
}
{% endcodeblock %}

I turned to [oauth2](https://github.com/simplegeo/python-oauth2), a Python library that makes it easier to carry out
this handshake. First, we get a *temporary access token:*

{% codeblock lang:py %}
# Fill in your app parameters here.
FITBIT_APP_KEY = '<app key>'
FITBIT_APP_SECRET = '<app secret>'

import oauth2
consumer = oauth2.Consumer(key=FITBIT_APP_KEY, secret=FITBIT_APP_SECRET)
client = oauth2.Client(consumer)
resp, content = client.request('http://api.fitbit.com/oauth/request_token, 'GET')
token = oauth2.Token.from_string(content)
# NOTE: the auth URL uses www.fitbit.com as the domain, NOT api.fitbit.com
auth_url = 'http://www.fitbit.com/oauth/authorize?oauth_token={0}'.format(token.key)
print auth_url
{% endcodeblock %}

Now we need an [OAuth verifier](http://wiki.oauth.net/w/page/12238555/Signed%20Callback%20URLs). This will be used to retrieve the real
access credentials. Visit `auth_url` in your browser,
log into Fitbit, and click Allow. You'll be redirected to the OAuth callback
specified in your app. Use the value of the `oauth_verifier` GET param on your
`token` from before to keep going:

{% codeblock lang:py %}
token.set_verifier('<oauth_verifier>')
client = oauth2.Client(consumer, token)
resp, content = client.request('http://api.fitbit.com/oauth/access_token', 'POST')
access_token = oauth2.Token.from_string(content)
{% endcodeblock %}

With this, we can now *retrieve useful information:*

{% codeblock lang:py %}
request_url = 'http://api.fitbit.com/1/user/-/profile.json'
oauth_request = oauth2.Request.from_consumer_and_token(consumer, token=access_token, http_url=request_url)
# Despite what the docs say, you need to generate a plaintext signature.
oauth_request.sign_request(oauth2.SignatureMethod_PLAINTEXT(), consumer, access_token)
headers = oauth_request.to_header(realm='api.fitbit.com')

import httplib
connection = httplib.HTTPSConnection('api.fitbit.com')
connection.request('GET', request_url, headers=headers)
resp = connection.getresponse()

import json
data = json.loads(resp.read())
{% endcodeblock %}

I encountered a few difficulties in figuring this out:

- For the authorize step, you need to use `www.fitbit.com` as the URL domain.
  `api.fitbit.com` will NOT work.
- You need to *sign all requests with the access token.*
- No, `oauth2.SignatureMethod_HMAC_SHA1` will **NOT** work. Yes, they explicitly
  claim to use HMAC-SHA1 in the documentation. Don't believe everything you
  read. Use [plaintext signatures](http://oauth.net/core/1.0/#anchor35) instead.
- Fitbit expects both the URI and Authorization header to be set, but
  `oauth2` will only set **ONE** of them properly.
  See [this commit message](https://github.com/candu/fitbit-crossfilter/commit/1d094cecaa6c78bc8d5c295a797d96b7e1687493)
  for more details.
  
You can see the full implementation [here](https://github.com/candu/fitbit-crossfilter/blob/master/fitbit_crossfilter/foo/lib/fitbit.py), along with
[an example of its use](https://github.com/candu/fitbit-crossfilter/blob/master/fitbit_crossfilter/foo/views/__init__.py).

### crossfilter

Square's [crossfilter](http://square.github.com/crossfilter/) is a JavaScript library for efficiently performing
*multidimensional range queries.* I've included an interactive example
[below](#quick-demo).

crossfilter uses two types of objects to *represent a multidimensional dataset:*

- **dimension:** a map function that returns totally-ordered *dimension values*
  (e.g. numbers, dates);
- **group:** a reduce function on those dimension values.

The *totally-ordered* part is essential, since that makes it possible to
perform range queries. A quick code snippet might help explain this further:

{% codeblock lang:js %}
var L = [], N = 10, M = 2;
for (var i = 0; i < N; i++) {
  L.push([i, Math.floor(M * (N - i - 1) / N)]);
}
var c = crossfilter(L);
var d0 = c.dimension(function(x) { return x[0]; });
var g0 = d0.group();
var d1 = c.dimension(function(x) { return x[1]; });
var g1 = d1.group();
d0.filterRange([3, 8]);
{% endcodeblock %}

At this point, we can *inspect the dimensions and groups* to understand the
effect of `filterRange()`:

{% codeblock lang:js %}
> JSON.stringify(d1.top(Infinity))
'[[4,1],[3,1],[7,0],[6,0],[5,0]]'
> JSON.stringify(g1.all())
'[{"key":0,"value":3},{"key":1,"value":2}]'
{% endcodeblock %}

Note that the range `[3, 8]` is actually interpreted as the semi-open interval
$ [3, 8) $. Note also that the elements of `g1.all()` are of the form
`{key: k, value: v}` where `v` is the number of elements `x` with
`3 <= x[0] && x[0] < 8 && x[1] == k`.

### d3.js

{% blockquote http://d3js.org/ %}
D3.js is a JavaScript library for manipulating documents based on data.
{% endblockquote %}

Using HTML, SVG, CSS, and JavaScript, you can build some pretty stunning
visualizations.
Again, check out the interactive example [below](#quick-demo). For more
examples, the [D3 Gallery](https://github.com/mbostock/d3/wiki/Gallery) is
many kinds of awesome.

### A Quick Demo

<div id="quick-demo" markdown="0">
  <style type="text/css">
    .chart {
      display: inline-block;
      height: 151px;
      margin-bottom: 20px;
    }
    
    .reset {
      margin-left: 1em;
      font-size: smaller;
    }
    
    .background.bar {
      fill: #ccc;
    }
    
    .foreground.bar {
      fill: steelblue;
    }
    
    .axis path, .axis line {
      fill: none;
      stroke: #000;
      shape-rendering: crispEdges;
    }
    
    .axis text {
      font: 10px sans-serif;
    }
    
    .brush rect.extent {
      fill: steelblue;
      fill-opacity: .125;
    }
    
    .brush .resize path {
      fill: #eee;
      stroke: #666;
    }

    #chartA {
      width: 610px;
    }
    
    #chartB {
      width: 610px;
    }
  </style>
  <script src="https://raw.github.com/candu/quantified-savagery-files/master/Panic/fitbit-crossfilter/crossfilter.js"></script>
  <script src="https://raw.github.com/candu/quantified-savagery-files/master/Panic/fitbit-crossfilter/d3.js"></script>
  <script src="https://raw.github.com/candu/quantified-savagery-files/master/Panic/fitbit-crossfilter/chart.js"></script>
  <script src="https://raw.github.com/candu/quantified-savagery-files/master/Panic/fitbit-crossfilter/demo.js"></script>
  <div id="charts">
    <div id="chartA" class="chart">
      <div class="title">A</div>
    </div>
    <div id="chartB" class="chart">
      <div class="title">B</div>
    </div>
  </div>
</div>

## Insights From My Data

You can see the live dashboard [here](http://fitbit.savageevan.com/). Some of the highlights:

- During this tracking period, I was *most active during the 8-10 am and 6-9 pm
  timeslots.* (The former was my morning walk to the employee shuttle; the
  latter was the evening walk back plus [Soccer Fours](http://soccerfours.com/).
- The more sleep I get, the more bipolar my exercise habits become.
- Unlike Beau Gunderson, I'm not seeing a correlation between number of times
  awoken and duration of sleep.
- There is, however, a clear positive correlation between steps per minute
  and calories burned per minute, as expected.

Again, you can play around with the dashboard [here](http://fitbit.savageevan.com/)
to find patterns in my Fitbit data.

## How To Use fitbit-crossfilter

I've placed my live fitbit-crossfilter dashboard into demo mode, but *you can
fetch and view your data as follows.*

First, you will need a Fitbit app *with Partner API access*; see
[this page](https://wiki.fitbit.com/display/API/Fitbit+Partner+API) for more details on setting that up. Use the following
application settings:

- **Application Type:** Website
- **Callback URL:** `http://localhost:9001/oauth`
- **Default Access Type:** Read-Only

Now copy `settings.py.nopasswd` to create your settings file:

{% codeblock lang:bash %}
$ cp settings.py.nopasswd settings.py
{% endcodeblock %}

Edit the bottom of `settings.py`:

{% codeblock lang:py %}
SYNC_ENABLED = True
DEFAULT_USER = None
FITBIT_CONSUMER_KEY = <your app key>
FITBIT_CONSUMER_SECRET = <your app secret>
{% endcodeblock %}

Start the server, login, and sync your data:

{% codeblock lang:bash %}
$ python manage.py runserver 9001
# visit localhost:9001/login in the browser to do the OAuth handshake
# visit localhost:9001/sync-user-data in the browser to sync data
{% endcodeblock %}

When the syncing completes, you'll be redirected to your dashboard.
