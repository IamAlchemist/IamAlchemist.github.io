---
layout: post
title: "android preformace 2: detect memory leak"
date: 2015-05-05 20:42:43 +0800
comments: true
categories: android
---

###the common way to detect memory leak
<!--more-->

* use Heap tool and Allocation tracing  
* start a empty activity  
* jump to the activity in question  
* trigger gc  
* watch the heap  

{% img center /images/android_performace/memory_leak_detection.png 600 %}  
