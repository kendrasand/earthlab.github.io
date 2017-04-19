---
layout: single
title: "Social Media Data"
excerpt: "This lesson will discuss how social media data are used in science."
authors: ['Carson Farmer', 'Leah Wasser']
modified: '2017-04-18'
category: [course-materials]
class-lesson: ['social-media-r']
permalink: /course-materials/earth-analytics/week-12/social-media/
nav-title: "Social media intro"
module-title: "Intro to Social Media Data in R"
module-description: "Coming soon. "
module-nav-title: 'twitter APIs'
module-type: 'class'
week: 12
sidebar:
  nav:
author_profile: false
comments: true
order: 1
lang-lib:
  r: ['rtweet']
tags2:
  social-science: ['social-media']
---


{% include toc title="In This Lesson" icon="file-text" %}

<div class='notice--success' markdown="1">

## <i class="fa fa-graduation-cap" aria-hidden="true"></i> Learning Objectives

After completing this tutorial, you will be able to:

*

## <i class="fa fa-check-square-o fa-2" aria-hidden="true"></i> What you need

You will need a computer with internet access to complete this lesson.

</div>


## Social media data in science

Social media data typically describes information created and curated by
individual users and collected using public platforms. These public platforms
include social media networks like Twitter, Facebook, Snapchat and Instagram but
also could be crowd sourced data including Yelp, Zillow and others.

Social media data can be a powerful source of information given it *can* provide
a near real-time outlook on both social processes such as politics, and current
day events and also natural processes including weather events (tornados, rainfall,
snow), disturbances (floods, and other natural disasters) and more.

There are many challenges associated with working social media data.

1. Text mining: because social media data are often a combination of text, graphics and videos, there is a significant text mining component involved - where you are trying to find information about something in non-standard text.
2. Geolocation: Not all social media is geolocated so it's often tricky to figure
out where the data are coming from.

###
Something about the volumne of data produced by social media platforms and twitter.


### twitter

This week, we are going to look at the use of <a href="http://twitter.com" target="_blank">Twitter</a> as a source of information
to better understand the impacts of weather and disturbance events on people.

> Twitter is an online news and social networking service where users post and interact with messages, "tweets," restricted to 140 characters. Registered users can post tweets, but those who are unregistered can only read them. - Source: wikipedia


Remember that last week we explored data access using API's. Twitter has an API
which allows you to access everyone's tweets. The API has certain limitations including:

1. **You can only access tweets from the last 6-9 days:** This means that you need
to think ahead if you want to collect tweets for a particular event.
2. others...


## Something about hashtags and how information is organized.

usernames
hashtops
text searches..
geolocation
what else???

### Twitter data access in R

Lucky for us, there are several R packages that can be used to collect tweets
from the twitter API. These include:

* `twitteR`
* `rtweet`

`rtweet` is a newer package that facilitates importing twitter data into data.frames.
This package is becoming the standard tool to access twitter data and thus we will
use it in our class this week!