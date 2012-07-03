---
layout: post
title: WhoIsNext
categories: whoisnext android application for keep in touch with friends or relatives 
description: whoisnext android application for keep in touch with friends or relatives
---

[WhoIsNext](http://market.android.com/details?id=com.abhidsm.whoisnext) is an android application which helps us to keep in touch
with friends and relatives. This application I started implementing
when I faced a situation where I was thinking of to whom I want to
make a call next. From my call log it is very difficult to check whom
I haven't contacted in the recent past and when I contacted them. And
I searched in the Android market for an app which helps me to keep in
touch with my friends. But I couldn't find a simple one. So I created
one [WhoIsNext](http://market.android.com/details?id=com.abhidsm.whoisnext).

User can add contacts to this app and the app will sort the contacts
based on the LAST_TIME_CONTACTED. So the app will show the contacts
which are recently contacted on bottom, and contacts which the user
hasn't contacted recently will show up on top. Along with the contact
name user can see when he contacted that person. 

I am using the [LAST_TIME_CONTACTED](http://developer.android.com/reference/android/provider/ContactsContract.Contacts.html) property of a contact to sort the
contacts. But the problem is some of the Android devices don't update
this property. So this application won't work in those devices for
now. For more details visit [this
link](http://stackoverflow.com/questions/5009072/problem-with-times-contacted-value-in-android-contacts-data?rq=1).

So for the next version(1.1) I am planning to create a [BroadcastReceiver](http://www.vogella.com/articles/AndroidBroadcastReceiver/article.html)
which will update the LAST_TIME_CONTACTED property of the contact once
a call ends. This will helps me to fix the problem without much
rework. But there is one problem with this approach as both the device and the
application uses the same property LAST_TIME_CONTACTED. If the
device updates this property for SMS and email then the application
can't use this property as the application concentrates on calls.

In another version(1.2) I am planning to save the LAST_TIME_CONTACTED
value to a custom variable. And update that using the
BroadcastReceiver. In that way the application is not dependent on
device specific properties.

You can check the source code of this application from my [GitHub repo](https://github.com/abhidsm/who-is-next).

