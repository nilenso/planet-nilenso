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

### Why we needed a custom SB?
We looked at Wufoo, Google Forms, Survey Monkey before deciding to write our hand-rolled Survey Builder.

- No offline mode
- Infinite nesting of questions and sub-questions
- Very targeted question types


---
### This is what it can do

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
- The Mobile apps, survey-mobile: https://github.com/nilenso/ashoka-survey-mobile) and survey-mobile-native](https://github.com/nilenso/ashoka-survey-mobile-native)


Here's a figure explaining their roles and interfaces:

![Architecture Diagram](/images/architecture.png)

The user-owner app was built for Ashoka to serve as the auth provider for all of it's web services. It is implemented using the [doorkeeper](https://github.com/applicake/doorkeeper) gem.

The clients Survey Mobile and Survey Web Response, use the API that is thrown by the Survey Web app. The API specification for this can be found here: [insert link here]

### This is what we plan to do next
- redesign of builder - v2
- redesign of native app
- native app
- data centric application
