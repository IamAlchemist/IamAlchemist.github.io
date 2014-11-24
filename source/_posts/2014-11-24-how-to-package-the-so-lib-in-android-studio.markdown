---
layout: post
title: "how to package the .so lib in Android Studio"
date: 2014-11-24 11:09:59 +0800
comments: true
categories: Android Development
---

change build.Gradle like this:  
  
    sourceSets {  
        main {  
            jniLibs.srcDirs = ['libs']  
        }  
    }   
