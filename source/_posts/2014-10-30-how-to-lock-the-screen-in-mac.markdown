---
layout: post
title: "how to lock the screen in Mac"
date: 2014-10-30 16:46:20 +0800
comments: true
categories: 
---

* use "Automator" create a service, then set the input as "no input".
* drag "Run shell script" to the editor in right side, then
    
        '/System/Library/CoreServices/Menu Extras/User.menu/Contents/Resources/CGSession' -suspend
        
* save the service as "lockscreen"
* set the shortcut key for the service in keyboard->shortcut keys






