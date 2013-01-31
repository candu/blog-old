---
layout: post
title: "What I'll Look Like In 50 Years"
date: 2013-01-31 12:43
comments: true
categories: [Non-Technical, Technical, Computer Vision]
---

I spent a few weeks in the not-so-frozen Canadian northlands over the winter
holidays. While there, I had the chance to visit an old childhood favorite:
the [Ontario Science Centre](LINK), six floors of science-based awesomeness.
One of their current exhibits, the [Amazing Aging Machine](LINK), uses a
computer vision software package called [APRIL](LINK) to predict how your
face will change over 50 years.

In this post, I explore my results from that exhibit alongside a customized
aging I performed using the [APRIL API](LINK).

<!-- more -->

## Present Me

It's not the most flattering photo, but here I am at 26:

TODO: image

## Future Me

### Take One: Amazing Aging Machine

First, my face balloons out massively:

Next, my cheek bones set downwards:

Finally, my face leans up and wrinkles a tiny bit:

### Take Two: APRIL API

For this run, I had access to the raw aging metadata, so I could see
exactly how old APRIL thought I was at each point in the aging sequence.

From 26 to 28, there's not much change:

Then, by age 35, my face elongates slightly:

I while away the next couple of decades in relative facial stasis. The
most pronounced change is in my skin, which pales gradually with age:

Finally, age catches up with me, and I wrinkle into a severely troubled
septuagenarian:

A few changes, each very minor, contribute to my forlorn expression over
these last three photos.

- The eyes get slightly rounder, as though they're welling up.
- At the top of the eyes, wrinkling gives the impression of a furrowed brow.
  (Note the lack of deep forehead and upper nose creases which normally
  accompany this expression. The mere suggestion of it on the eyes is enough
  to trigger our recognition of facial expressions!)
- The face elongates yet again, creating a drawn expression.
- As part of the elongation of the face, the mouth corners sag downwards
  into the merest hint of a frown.

Looking at this, it's surprising how little variation we need to recognize
a profoundly different facial expression. Hopefully I look happier than
this at 72!

### Summary

These images provide two divergent visions for my distant future:

TODO: images

For comparison, here's my father in his late 50s:

{% img http://farm4.staticflickr.com/3177/2828216200_5846e31c4a_z.jpg %}

## Why Were Those So Different?

TODO: quote from Amazing Aging Machine

The Amazing Aging Machine is calibrated for ages 8-12, likely to match 
the Ontario Science Centre's target demographic. (Sadly, I couldn't find
detailed visitor demographic data!) In my case, this creates an awkward
puffy look: it's applying changes in facial structure through adolescence,
when much of our bone growth occurs.

By contrast, the APRIL API asks for your current age, allowing it to more
correctly calibrate its models. As a result, the second set of faces exhibits
relatively little change in shape.

## What Does This Tell Me?

Although my face is unlikely to match either of these faces at 72, this
experiment provides a window into how our appearance changes with age.

## Appendix: How To Use The APRIL API

For the more technically-minded, I'll run through a transcript of the APRIL
API aging pipeline. For all the gritty details, consult the [API docs](http://www.aprilage.com/AprilAPI_V2.pdf).

Before starting, I highly recommend installing a tool like [jsonpp](https://github.com/jmhodges/jsonpp);
it makes it much easier to read API results.

The first step is manual: you need to register at [ageme.com](http://www.ageme.com/), then
click the confirmation link in your email.

TODO: screenshot

The next step is uploading an image, but let's check first that the API
works by retrieving our user info:

{% codeblock %}
$ curl http://www.ageme.com/AprilAPI/users/savage.evan@gmail.com/userInfo
{"result_code":0,"message":"Unauthorized"}
{% endcodeblock %}

Oops! We haven't authenticated ourselves. The Authorization header uses a
brain-dead and highly insecure `base64` encoding:

{% codeblock %}
$ python -c "import base64; print base64.encodestring('username:password')"
dXNlcm5hbWU6cGFzc3dvcmQ=
{% endcodeblock %}

(Obviously this isn't my real username/password. Substitute yours to get it
working, and use that `base64`-encoded string in the `Authorization` headers
below.)

With the correct header, we can try fetching the user info again:

{% codeblock %}
$ curl -H "Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=" http://www.ageme.com/AprilAPI/users/savage.evan@gmail.com/userInfo | jsonpp 
{
  "uri": "http://www.ageme.com/AprilAPI/users/savage.evan@gmail.com",
  "email": "savage.evan@gmail.com",
  "tokens": 0,
  "numOfAgings": 1,
  "role": "user"
}
{% endcodeblock %}

Great! Now we can POST an image to the uploading endpoint with `curl`:

{% codeblock %}
$ curl -F 'filename=aging1.jpg' -F 'image=@/Users/candu/Desktop/aging1.jpg' -H "Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=" http://www.ageme.com/AprilAPI/images
{% endcodeblock %}

Another manual step: before proceeding, you'll need to purchase a token on
the [ageme.com](http://www.ageme.com/) site. At time of writing, this cost $3.99; I looked for
active promotion codes, but couldn't find any.

TODO: screenshot

With your aging token purchased, you can now create an aging document. This
lets APRIL know your age and ethnicity, which helps it to select the
appropriate models for your particular aging sequence. It also identifies the
starting image of that sequence via the `imageId` returned during image upload.

{% codeblock %}
$ curl -H 'Content-Type: application/json' -d '{"document": {"gender": "male", "age": 26, "name": "Evan", "ethnicity": "Caucasian"}, "imageId": 2371944}}' -H "Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=" http://www.ageme.com/AprilAPI/documents
{% endcodeblock %}

We're ready to run the aging process. There's a single method `detectMatchAge`
for performing all three steps, but I'll break it down into the component
steps here:

{% codeblock %}
$ curl -X POST -H "Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=" http://www.ageme.com/AprilAPI/users/savage.evan@gmail.com/documents/2371973/pointDetection
$ while true; do curl -H "Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=" http://www.ageme.com/AprilAPI/users/savage.evan@gmail.com/documents/2371973/status; done
$ curl -X POST -H "Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=" http://www.ageme.com/AprilAPI/users/savage.evan@gmail.com/documents/2371973/match
$ while true; do curl -H "Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=" http://www.ageme.com/AprilAPI/users/savage.evan@gmail.com/documents/2371973/status; done
$ curl -H 'Content-Type: application/json' -d '{"sequenceType": "Max72", "sequences": [{"smoking": 0, "sunExposure": 0, "multiplier": 1}]}' -H "Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=" http://www.ageme.com/AprilAPI/users/savage.evan@gmail.com/documents/2371973/aging
$ while true; do curl -H "Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=" http://www.ageme.com/AprilAPI/users/savage.evan@gmail.com/documents/2371973/status; done
{% endcodeblock %}

Note the `while` loops, which wait for each step to complete. Once all steps
are completed, we retrieve the aging results:

{% codeblock %}
$ curl -H "Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=" http://www.ageme.com/AprilAPI/users/savage.evan@gmail.com/documents/2371973/results > aging_results.json
$ cat aging_results.json | jsonpp | head -15
[
  {
    "uri": "http://www.ageme.com/AprilAPI/users/savage.evan@gmail.com/documents/2371973/results/76647",
    "status": "done",
    "sequenceType": "Max72",
    "sequences": [
      {
        "smoking": 0.0,
        "sunExposure": 0.0,
        "multiplier": 1.0,
        "images": [
          {
            "age": 26,
            "uri": "http://www.ageme.com/AprilAPI/images/IhLAo8Sp"
          },
{% endcodeblock %}

Finally, I wrote a bit of [Python glue](LINK) to fetch the URLs and name
them by age:

{% codeblock lang:py %}
import json
import sys
import urllib2

data = json.loads(sys.stdin.read())
images = data[0]['sequences'][0]['images']
for image in images:
  url = urllib2.urlopen(image['uri'])
  path = 'age{0}.jpg'.format(image['age'])
  with open(path, 'w') as f:
    f.write(url.read())
{% endcodeblock %}

With this, we can fetch the images:

{% codeblock %}
$ python fetch_aging.py < aging_results.json
{% endcodeblock %}

And that's it! Most of the process uses `curl`, with minimal leaning
on Python for its `base64` module.
