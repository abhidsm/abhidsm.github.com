---
layout: post
title: Blackberry app using phonegap and jquery mobile
categories: blackberry phonegap issues html5 webworks
description: Blackberry app using phonegap and jquery mobile. Issues I faced while developing blackberry app using phonegap and jquery mobile.
---

Last week I was working on a cross mobile application, which is
developed to work on all the mobile platforms including Android,
Blackberry, iPhone and Windows mobile. We developed the app using
phonegap and jquery mobile, and it worked well in Android mobile. Then I
moved on to Blackberry. In Blackberry it took sometime to setup the
environment and run the application. Here I am going to explain the
issues I faced and the steps I taken to fix the issues.

1. #Setting up environment

      We were using eclipse for developing Android application. So I thought
      to install eclipse plugin for Blackberry by following the instructions
      given in the [phonegap wiki](http://wiki.phonegap.com/w/page/25653281/Getting%20Started%20with%20PhoneGap-BlackBerry%20with%20the%20Latest%20Environment). And
      that didn't work. Then I tried [this link](http://wiki.phonegap.com/w/page/31930982/Getting%20Started%20with%20PhoneGap%20BlackBerry%20WebWorks) which is for
      BlackBerry OS 5.0 and higher. Here the instructions under the
      heading <small markdown="1">***'Install the BlackBerry Web and
      WebWorks SDK Plug-ins for Eclipse'***</small> won't work because there is a small
      message in italics <small markdown="1">***'End Of Life Notice: The BlackBerry WebWorks
      Plug-in will no longer be available for download as of Oct. 18, 2011. Support for the plug-in will be discontinued on Dec. 31, 2011'***</small>. 

      So I stopped searching for an Eclipse plugin. I started
      setting up environment for the console. I setup Ant by following the instructions
      from the phonegap wiki. Then I downloaded and installed the
      webworks sdk from [this link](https://bdsc.webapps.blackberry.com/html5/download/sdk). So at last the
      Environment setup for console is ready. 

2. #The commands to build, load simulator and load device

      In phonegap doc the commands to build the blackberry phonegap app from
      console is given as 'ant build'. But this won't work. Because for
      phonegap 1.3.0 it is 'ant blackberry build'. Similarly for load the
      app in simulator 'ant blackberry load-simulator' and load app in
      device 'ant blackberry load-device'.

3. #Code signing

      To load the app in device you have to sign the code using the key
      provided by Blackberry. You can order the keys from [this link](https://www.blackberry.com/SignedKeys/). It will take around 2
      hours to process the request and get the keys. Now you have to [install/register the keys](https://bdsc.webapps.blackberry.com/html5/documentation/ww_publishing/signing_setup_smartphone_apps_1920010_11.html) using your webworks sdk. After installing the signing keys you can run 'ant blackberry load-device'
      to install the app in the device.

4. #Testing app in the device

      In the phonegap doc it is mentioned that the installed app will be
      available in the Downloads section of Blackberry. But I couldn't find
      the installed app there. So I opened the app by searching in
      blackberry search. 

5. #Back button in Blackberry exits the application

      In Android the back button loads the previous screen of the app. But
      in Blackberry if you press the back button it exits the app even if
      you have visited multiple pages. I fixed this issue by overriding the
      back button event click from the phonegap. The phonegap code for
      listening the back button click and show the history is given below:

         blackberry.system.event.onHardwareKey(blackberry.system.event.KEY_BACK, 
            function() { 
                history.back();
                return false;
            }); 

6. #jquery loading message breaks in blackberry

      We are showing the jquery mobile loading message while loading the
      data using ajax. But it is breaking in Blackberry. And I found [this link](http://forum.jquery.com/topic/mobile-showpageloadingmsg-appearing-in-wrong-position-on-blackberry) which saying "It is apparently known and won't be fixed on the current
      devices". This one we didn't fix. But we will do some work around like
      showing our own message or something else.

So this is the Blackberry phonegap experience I got from the last
week. Hope this will help you to setup an environment for your
Blackberry webworks app.
