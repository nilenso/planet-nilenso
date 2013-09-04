---
layout: post
title: "Android Development - Harnessing powers of MVP"
date: 2013-09-04 18:33
author: [Jithu Gopal, Akshay Gupta]
comments: true
published: false
---

As we set out to develop our very first native android app for [Ashoka Survey](https://github.com/nilenso/ashoka-survey-mobile-native), discussions kicked up a notch about how should the architecture pan out. Literature in the internet was not promising for almost every hurdle we jumped through - setting up the IDE, prominent libraries that could come handy, better ways to do testing. After three to four repo reboots, we finally decided on these -

- IntelliJ Idea CE
- [Robolectric](https://github.com/robolectric/robolectric)
- [RoboGuice]()
- FEST for android

### Structure

Maven dictated the basic skeleton to work out from. Once we had the project setup sorted out we ventured into getting out a basic login screen. 


###Problems

Identifying boundaries is paramount for writing good tests. Our `LoginActivity` started out to do way too many things - talking to network boundary and manipulating the view along with it. Testing proved to be a challenge at this point. What we wanted at this point was -

1. Being able to test the view behaviour in isolation.
2. Testing if appropriate handlers are triggered according to responses from network layer.
3. Identify a service layer which deals with IO, so that mocks can be cheap and painless. 

The idea is that we should be able to test in isolation what view decisions are being made due to different interactions on our application, without actually caring about what the view would look like.

That essentially breaks down our testing strategies into:

- Testing how the view behaves
- Testing how the view interacts with the boundaries

