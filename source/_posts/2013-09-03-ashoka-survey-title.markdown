---
layout: post
title: "Ashoka Survey Title"
date: 2013-09-03 18:21
comments: true
published: false

---

At Nilenso, we've helped [Ashoka](http://india.ashoka.org/) accomplish their initiative in building infrastructure for the [citizen sector](https://www.ashoka.org/citizensector).


Infiniti is a suite of products envisioned for social entrepreneurs, the fellows at Ashoka and other citizen sector organizations to get a variety of quality data through crowd sourcing, organized surveys and market research. It is currently very much in use at Ashoka around these verticals:

- Nutrition and Wellness – for addressing malnutrition problem among children in India
- Affordable Housing – for enabling access to housing in the informal sector
- Rural Innovation and Farming – to understand the role of women in Agriculture

The Infiniti suite is largely composed of these attributes:

![Infiniti Suite](http://cl.ly/image/3u2E0I0M1A0Z/Image%202013.09.05%206_40_04%20PM.png)

As a part of this, we built Ashoka Survey Web––one of the arms that takes care of data collection, validation, integrity and reporting.

# Survey Web

It's basically a collection of these 4 applications:

- the auth provider, [user-owner](https://github.com/nilenso/ashoka-user-owner).
- the survey builder, [survey-web](https://github.com/nilenso/ashoka-survey-web) 
- response collection from the web, [survey-web](https://github.com/nilenso/ashoka-survey-web).
- the mobile applications:
  [survey-mobile](https://github.com/nilenso/ashoka-survey-mobile),
  [survey-mobile-native](https://github.com/nilenso/ashoka-survey-mobile-native).

Here's a figure explaining their roles and interfaces:
  
![Architecture Diagram](http://cl.ly/image/3a0n2g0Q1A16/architecture.png)

#### Process

Once the fellows at Ashoka are done identifying the problem sets and designing the questions for the survey, they use the survey builder to quickly digitise them. These surveys can be shared with different registered organisations or shared publicly.

The responses are then collected by the field agents of the respective organizations using the mobile application that can take validated answers client-side and store them offline. These can be synced to survey-web, once the field agents get internet connectivity.

We created a quick one-minute screencast that describes a simple workflow on how a survey is created, how a response is taken on that survey on the web and some basic reports you can see around it.

<video width="640" height="360" controls>
  <source src="http://cl.ly/0f0u0P0B2N21/ashoka-survey-screencast.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

Unfortunately, we don't have anything to show for the field work that Ashoka does using these apps, but it is definitely something worth experiencing!

#### Need for a separate auth app

The user-owner app was built to serve as a centralized auth provider for all of Ashoka's web services. It is implemented using the [doorkeeper](https://github.com/applicake/doorkeeper) gem.

#### Survey Builder amongst the alternatives available

[Wufoo](http://wufoo.com) and [Google Forms](http://forms.google.com) aren't designed for long surveys. They work well for short forms/questionnaires that can be answered on the web, but they do not have support for nesting questions or collecting responses from a mobile app offline.

[OpenDataKit](http://opendatakit.org) doesn't have a good interface for building long surveys or nested questions. It requires working with XLS files for many complex operations.

[SurveyMonkey](http://surveymonkey.com) is a good tool, but we couldn't use it since we needed complete ownership of the survey and response data.

# Future

#### Survey Builder V2
The current survey builder, although built with a lot of thought on usability, fails on a few aspects. Accenture woked on the designs for a better survey builder. This is a work in progress, and can be viewed from the create-v2 menu. The new VDs are [here](https://github.com/nilenso/ashoka-survey-web/commit/a5aeb01fadedf43311a779412ef49c0c28081d92)

#### The Native Android App
The currently funtional survey-mobile is built with [Titanium](http://www.appcelerator.com/platform/titanium-platform/). It works, but we certainly want to move away from this, to a native app.
Here is where we are with the native app as of now: [insert link to jithu's post]

#### Mobile app redesign
Here are the mockups for the new mobile app: [insert link here]

#### Data Portal
Each survey conducted by ashoka typically has about 300 responses. We currently have some basic reports built with google charts. But we don't have the ability to say, interpret data of similar/same respondants across multiple surveys.
Here are the mockups for what is planned in this regard: [insert link to mockups]
