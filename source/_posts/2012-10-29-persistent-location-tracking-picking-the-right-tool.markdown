---
layout: post
title: "Persistent Location Tracking: Picking The Right Tool"
date: 2012-10-29 18:37
comments: true
categories: [Location, Technical]
---

In this post, I compare Google Latitude and InstaMapper, two popular services
for persistent location tracking. I walk through installation and data
extraction via API for each service, then provide some subjective first
impressions as to which one better suits my location-tracking needs.

<!-- more -->

## Evaluating Tools

As the ecosystem of self-tracking tools [grows exponentially](http://quantifiedself.com/2011/03/the-state-of-quantified-self-a-year-of-growth/),
choosing the right tool for the job becomes an increasingly daunting task.
To add to the complexity of this decision, self-tracking tools are
*highly personal.*

{% blockquote %}
How do I pick the best tool for me?
{% endblockquote %}

This question is not monolithic, and entails a series of considerations:

- Do I want *manual* or *automatic* tracking?
  - How much *time and effort* am I willing to spend on self-tracking?
  - Is a hybrid *automatic raw data with manual annotation* approach workable?
- Do I want *constant (persistent)* tracking?
  - *All day?* Or only at predetermined times?
  - *Every day?* What if I'm on vacation?
- Do I want *raw data access?*
  - How do I want to access that data? Through Excel CSV files?
    Via a developer-friendly API?
  - What *granularity* do I want? Sub-second? Hourly? Variable based
    on activity?
  - What *parameters* do I want? For location, is `(lat, long)` enough?
    Do I want altitude as well? GPS fix accuracy? Number of satellites?

Without a *searchable database of self-tracking tools*, these questions can be
difficult to answer. The main [Quantified Self website](LINK) includes a
[Guide to Self-Tracking Tools](LINK), but their implementation is subject to
criticism:

{% blockquote Robert Wood Johnson Foundation http://www.rwjf.org/content/dam/farm/reports/program_results_reports/2012/rwjf400733 %}
In a report to RWJF, Project Director Alexandra C. Carmichael noted that the guide 
was more a catalog of tools than a useful manual for people wanting to choose and 
use these tools.
{% endblockquote %}

This is a point worth repeating. Simply listing tools is *not enough*; a
database of tools *must answer these basic questions* to be useful. A quick
search on the Guide for [location-related tools](http://quantifiedself.com/guide/tag/location)
comes up short:

TODO: screenshot

Why is [MoodPanda](LINK) listed? I suppose it must location-tag mood entries,
but that isn't made explicit in the description.
[Momento](LINK) makes a bit more sense, but it's primarily a journalling app.
[Foursquare](LINK) is definitely location-based, but anyone unfamiliar with
it must read its description closely to realize that it relies on
manual check-ins.

In lieu of a useful tool database, the only effective option is direct
evaluation. By investigating two popular location tracking tools, I'll
demonstrate how such an evaluation might be carried out.

## The Tools

### Google Latitude

Google Latitude bills itself primarily as a social location sharing service:

TODO: screenshot

Social capacities aside, the [Location History](LINK)
functionality can be used as a persistent location-tracking tool.

### InstaMapper

I first heard of InstaMapper from
[Ted Power's talk on geo-tracking](http://vimeo.com/8545134). Unlike Google
Latitude, InstaMapper focuses more on personal tracking:

TODO: screenshot

### The Criteria

My ideal location tracking tool is:

- **Android-compatible:** It should work with my [RAZR M](LINK) running
  [Android 4.0.4 (Ice Cream Sandwich)](LINK).
- **Battery-friendly:** It should allow me to get through the day without
  recharging.
- **Automatic:** It should track my location without requiring check-ins
  or other manual input.
- **Persistent:** It should track my location 24/7.
- **Fine-grained:** It should be capable of per-minute resolution or better.
- **Developer-friendly:** It should provide an API for fetching location
  history, and the data offered through that API should be as complete as
  possible.

## Installation

### Google Latitude

Enabling automatic location tracking on Android is easy:

TODO: device screenshots

### InstaMapper

Since I'm using the Android-based [RAZR M](LINK), I followed the Android
installation directions [here](http://www.instamapper.com/android_howto.html).

## Data Export

### Google Latitude

Exporting data from the [Location History dashboard](LINK) is relatively
intuitive:

TODO: screenshots

Data is exported in KML format:

TODO: sample of KML

### InstaMapper

Head to the data page for your device:

TODO: screenshots

Here's where this process gets weird. To export your data, you first have to
define a track:

TODO: screenshots 

Once the track is created, you'll be taken to the Track Manager where you can
export your track data in a variety of formats:

TODO: screenshots

After this needlessly cumbersome process, you finally have some workable data:

TODO: sample of data

### Comparison

This one goes to *Google Latitude*. Aside from the terrible UI flow,
InstaMapper has some other problems:

- The `accuracy` field is missing, making it harder to filter out noisy
  readings.
- As stated in the [InstaMapper FAQ](LINK), data access is limited to the
  previous 30 days or 100 000 locations.

## API Fetching

### Tailers and Streams

Many real-time APIs provide [REST](LINK) endpoints for fetching
time-bounded chunks of data:

{% codeblock %}
https://www.googleapis.com/latitude/v1/location?key=INSERT-YOUR-KEY&min-time=1111&max-time=2222&max-results=10
{% endcodeblock %}

By keeping track of a `since` time to fetch after, we can easily turn this
into a stream:

{% codeblock lang:py %}
def request(since):
  # TODO: actually fetch data
  pass

def sleep(since, freq):
  elapsed = time.time() - since
  wait = freq - elapsed
  if wait < 0:
    return
  time.sleep(wait)

def poll(freq):
  since = time.time()
  while True:
    sleep(since, freq)
    locations = request(since)
    if not locations:
      continue
    doSomething(locations)
    since = locations[-1].timestamp
{% endcodeblock %}

This pattern is often referred to as a *tailer.* Why? Suppose we have a simple
implementation of `doSomething()`:

{% codeblock lang:py %}
def doSomething(location):
  print location
{% endcodeblock %}

This prints out locations as they are received, similar to UNIX `tail -F`. By
adjusting `freq` I can make different real-time guarantees, although at some
point the upstream API will start throttling my requests.

### Google Latitude

You can see a working tailer implementation [here](LINK).

To access the [Google Latitude API](LINK), you first need to
[register an application](LINK). This gives you the necessary
parameters `YOUR_KEY`, `YOUR_SECRET` for stepping through the OAuth flow.

With the Python library [oauthclient2](LINK), retrieving OAuth credentials is
relatively painless:

{% codeblock lang:py %}
from oauth2client.client import OAuth2WebServerFlow
from oauth2client.file import Storage
from oauth2client.tools import run

def getCredentials(key, secret):
  flow = OAuth2WebServerFlow(
    client_id=key,
    client_secret=secret,
    scope='https://www.googleapis.com/auth/latitude.all.best',
    redirect_uri='http://localhost:8080/oauth2callback'
  )
  storage = Storage('.creds')
  return run(flow, storage)
{% endcodeblock %}

`oauth2client.tools.run()` invokes a browser window and starts an HTTP server
to receive the OAuth callback. With the credentials, we can make a signed API
request:

{% codeblock lang:py %}
import httplib2
import urllib
import json

def request(self, since):
  http = httplib2.Http()
  credentials = getCredentials(YOUR_KEY, YOUR_SECRET)
  credentials.authorize(http)
  url = 'https://www.googleapis.com/latitude/v1/location?%s' % urllib.urlencode({
    'max-results': 10,
    'min-time': since,
    'max-time': since + 10 * (15 * 1000),
    'granularity': 'best'
  })
  resp, content = http.request(url)
  data = json.loads(content)
  if data.get('error') is not None:
    return None
  return data['data'].get('items', [])
{% endcodeblock %}

There are some minor details:

- The API uses milliseconds for its timestamps, so my `since` values take
  this into account.
- Without `max-time`, the API returns the most recent `max-results` locations.
  I supply a 150-second window.
- If there are no locations within the given time range, the API does not
  populate `data['data']['items']`. I use `get()` to work around the resulting
  `KeyError`.
- In the event of an error, the API populates `data['error']`. I use `None` as
  a sentinel value to indicate that an error has occurred.

### InstaMapper

You can see a working tailer implementation [here](LINK).

InstaMapper doesn't use OAuth. Instead, you enable API access on a device:

TODO: screenshot

This gives you a key `YOUR_KEY` that can be used to make API requests:

{% codeblock lang:py %}
import httplib
import json
import urllib

def request(self, since):
  params = {
    'action': 'getPositions',
    'key': YOUR_KEY,
    'num': 10,
    'from_ts': since,
    'format': 'json',
  }
  url = 'http://www.instamapper.com/api?{1}'.format(APIHOST, urllib.urlencode(params))
  conn = httplib.HTTPConnection('www.instamapper.com')
  conn.request('GET', url)
  resp = conn.getresponse()
  if resp.status != 200:
    raise Exception('HTTP {0} {1}'.format(resp.status, resp.reason))
  data = json.loads(resp.read())
  conn.close()
  return data['positions']
{% endcodeblock %}

### Comparison

Although InstaMapper's API is arguably simpler to use, I'll award this one to
*Google Latitude:*

- **Security:** InstaMapper uses unencrypted HTTP GET requests, so anyone
  running a [packet sniffer](LINK) on my network has complete access to my
  location data. Google Latitude uses HTTPS and OAuth. No contest.
- **Support:** I can leverage the community of Google API users to help resolve
  any issues I encounter.
- **Data:** again, InstaMapper is missing location accuracy.
- **Request Volume:** InstaMapper permits one request every 10 seconds. Google
  Latitude allows 1 000 000 requests per day, or one request every 0.0864
  seconds.

## Battery Usage

To find out how battery-friendly the two Android apps are, I check the
Battery Manager:

TODO: device screenshots

### Google Latitude

TODO: more device screenshots

### Instamapper

InstaMapper's GPS Tracker is definitely drawing more power. Why is that?

TODO: more device screenshots

### Comparison

*Google Latitude* wins this one as well. InstaMapper keeps the GPS radio
running almost constantly, whereas Google Latitude manages to sip radio
access. I'm guessing that it uses WiFi, cell towers, and other non-GPS sources
where possible.

## First Impressions

After a day of persistent location tracking with both Google Latitude and
InstaMapper, *Google Latitude wins hands-down.* It's easy to install, it provides
simple and secure data access via `oauth2client`, and it preserves battery life
nicely.

## Next Steps
