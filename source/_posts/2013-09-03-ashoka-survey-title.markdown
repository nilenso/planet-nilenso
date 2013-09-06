---
layout: post
title: "Ashoka Survey Title"
date: 2013-09-03 18:21
comments: true
published: false

---

At Nilenso, we've helped [Ashoka](http://india.ashoka.org/) accomplish their initiative in building infrastructure for the [citizen sector](https://www.ashoka.org/citizensector).


Infiniti, is a suite of products envisioned for social entrepreneurs, the fellows at Ashoka and other citizen sector organizations to get a variety of quality data through crowd sourcing, organized surveys and market research. It is currently very much in use at Ashoka around these verticals:

- Nutrition and Wellness – for addressing malnutrition problem among children in India
- Affordable Housing – for enabling access to housing in the informal sector
- Rural Innovation and Farming – to understand the role of women in Agriculture

The Infiniti suite is largely composed of these attributes:

![Infiniti Suite](http://cl.ly/image/3u2E0I0M1A0Z/Image%202013.09.05%206_40_04%20PM.png)

As a part of this, we built Ashoka Survey Web––one of the arms that takes care of data collection, validation, integrity and reporting.

## Survey Web
---
<Explain a bit of the architecture here>

We created a quick one-minute screencast that describes a simple workflow on how a survey is created, how a response is taken on that survey on the web and some basic reports you can see around it.

<video width="640" height="360" controls>
  <source src="http://cl.ly/0f0u0P0B2N21/ashoka-survey-screencast.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

## Why we needed a custom SB?
We looked at Wufoo, Google Forms, Survey Monkey before deciding to write our hand-rolled Survey Builder.

- No offline mode
- Infinite nesting of questions and sub-questions
- Very targeted question types


---
## This is what it can do

After identifying the problem and designing a set of questions around it, it's useful to have it digitized quickly.


So, Having designed a questionnaire, one can quickly digitize these questions using the survey builder. We currently have the ability to build these types of questions: [list]

 this how the application actually solves these problems.

- Create surveys
- Collect responses
- Show reports

- Organizations can register themselves
- Surveys can be shared between orgs
- Can be shared publicly

---


## This is how it is built

The survey application is 4 parted:

- The auth provider, user-owner: https://github.com/nilenso/ashoka-user-owner
- The survey builder, survey-web: https://github.com/nilenso/ashoka-survey-web
- The response collector, survey-web: https://github.com/nilenso/ashoka-survey-web
- The Mobile apps:
  survey-mobile: https://github.com/nilenso/ashoka-survey-mobile
  survey-mobile-native: https://github.com/nilenso/ashoka-survey-mobile-native


Here's a figure explaining their roles and interfaces:

![Architecture Diagram](/images/architecture.png)

The user-owner app was built for Ashoka to serve as the auth provider for all of it's web services. It is implemented using the [doorkeeper](https://github.com/applicake/doorkeeper) gem.


---


## This is what we plan to do next

### Survey Builder V2
The current survey builder, although built with a lot of thought on usability, fails on a few aspects. Accenture woked on the designs for a better survey builder. This is a work in progress, and can be viewed from the create-v2 menu. The new VDs are [here](https://github.com/nilenso/ashoka-survey-web/commit/a5aeb01fadedf43311a779412ef49c0c28081d92)

### The Native Android App
The currently funtional survey-mobile is built with [Titanium](http://www.appcelerator.com/platform/titanium-platform/). It works, but we certainly want to move away from this, to a native app.
Here is where we are with the native app as of now: [insert link to jithu's post]

### Mobile app redesign
Here are the mockups for the new mobile app: [insert link here]

### Data Portal
Each survey conducted by ashoka typically has about 300 responses. We currently have some basic reports built with google charts. But we don't have the ability to say, interpret data of similar/same respondants across multiple surveys.
Here are the mockups for what is planned in this regard: [insert link to mockups]
