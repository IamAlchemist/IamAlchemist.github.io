---
layout: post
title: "Process in Android"
date: 2014-11-25 10:24:54 +0800
comments: true
categories: Android Development
---

there are four types of components for an App, <activity>, <service>, <receiver> and <provider>, which support "android:process".

one App can have multi-processes via "android:process", and multi apps can share one process via share the same Linux user ID and signed with the same

###Process lifecycle

* Foreground process
A process that is required for what the user is currently doing. A process is considered to be in the foreground if any of the following conditions are true:

  > It hosts an Activity that the user is interacting with (the Activity's onResume() method has been called).
    
  > It hosts a Service that's bound to the activity that the user is interacting with.
  
  > It hosts a Service that's running "in the foreground"—the service has called startForeground().
  
  > It hosts a Service that's executing one of its lifecycle callbacks (onCreate(), onStart(), or onDestroy()).
  
  >It hosts a BroadcastReceiver that's executing its onReceive() method.

* Visible Process

A process that doesn't have any foreground components, but still can affect what the user sees on screen. A process is considered to be visible if either of the following conditions are true:

  >It hosts an Activity that is not in the foreground, but is still visible to the user (its onPause() method has been called). This might occur, for example, if the foreground activity started a dialog, which allows the previous activity to be seen behind it.
  
  >It hosts a Service that's bound to a visible (or foreground) activity.
  
A visible process is considered extremely important and will not be killed unless doing so is required to keep all foreground processes running.

* Sevice process

A process that is running a service that has been started with the startService() method and does not fall into either of the two higher categories. Although service processes are not directly tied to anything the user sees, they are generally doing things that the user cares about (such as playing music in the background or downloading data on the network), so the system keeps them running unless there's not enough memory to retain them along with all foreground and visible processes.

* Background process

A process holding an activity that's not currently visible to the user (the activity's onStop() method has been called). These processes have no direct impact on the user experience, and the system can kill them at any time to reclaim memory for a foreground, visible, or service process. Usually there are many background processes running, so they are kept in an *LRU (least recently used)* list to ensure that the process with the activity that was most recently seen by the user is the last to be killed. If an activity implements its lifecycle methods correctly, and saves its current state, killing its process will not have a visible effect on the user experience, because when the user navigates back to the activity, the activity restores all of its visible state. See the Activities document for information about saving and restoring state.

* Empty process
A process that doesn't hold any active application components. The only reason to keep this kind of process alive is for caching purposes, to improve startup time the next time a component needs to run in it. The system often kills these processes in order to balance overall system resources between process caches and the underlying kernel caches.

###Threads

* How to access UI thread from other threads
  > Handler

  > AsyncTask

  > Others

    >> Activity.runOnUiThread(Runable)

    >> View.post(Runnable)

    >> View.postDelayed(Runnable)


* Caution 1: need to consider *runtime configuration change*

* Caution 2: need to consider therad-safe


