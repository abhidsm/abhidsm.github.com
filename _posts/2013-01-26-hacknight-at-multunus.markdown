---
layout: post
title: Hacknight @ Multunus
categories: hacknight multunus fun develop product
description: We had a hacknight at multunus. It was very fun and enjoyed a lot creating a small application
---

We had a very successful hacknight here at [multunus](http://www.multunus.com) yesterday(Jan 25, 2012). This was
our 5th or 6th hacknight. And why it is very successful because we
could complete all the 3 applications which we planned to do. Last
time our team couldn't finish the app completely.

We started by 10:30 PM, and there were 8 participants. We came up with
ideas and selected 3 out of them, 2 team with 3 members and one team
with 2 members. The ideas were:
1. An obstacle detection app based on Arduino board \[Akshay, Anup, Vimal\]
2. A Chrome extension for broadcasting URL \[Abhilash, Sandeep, Manoj\]
3. Scrollee - an app which scrolls the content across the screens
(Multiple monitors) \[Midhun, Ernest\]

So I form a pair with Sandeep to work on the BroadcastIt app. We cameup
with the tasks and spikes. We have selected node.js to implement the
server part. We worked on the spikes and we were confident that it can be done. By 1:30 AM we are done with the basic
app. That is the server part and client using the normal page. So we
were able to show the demo of broadcasting URL to others. The next
task was to integrate the client with the extension.

Then we faced the first problem, that is the extension is not able
load the socket.io.js file. The chrome extension can load js files
only from inside the package or from an external URL which is
secured(https). We have tried to include the socket.io.js onside the
extension package, that didn't work. Then searched for socket.io cdn
URLs. We found a [cdn URL for socket.io](http://cdnjs.com/), and that problem got fixed.

So almost done with the app. On click of the extension it will popup a small dialog with
Broadcast button, on cllick of that will broadcast the current tab URL
to all other browsers which installed this extension. When we have
done with this it was around 2:30 AM. But then the
next problem, the extension will work only when the
popup is showing. When the popup is closed the extension can't listen
to the server event.

For chrome extension we can have [background scripts and pages](http://developer.chrome.com/extensions/background_pages.html). We
spend around 1 hr learning these and found a solution to fix this
problem. We removed showing the popup and started running the script
in background. So it will listen to the server in background. So now
when we install the extension, it can broadcast the current tab URL to
other browsers which are installed the extension. It was 4:30 AM.

Then we thought of extending the app by introducing channel, by adding
options to the extension. The options page will ask the user to enter
a channel, and the URL will broadcast only to those browsers who are
selected the same channel. We were able to create the options page and
saving the channel for a browser. But we couldn't complete the
broadcast only to the same channel browsers, because it was morning 6
Am and we had to create a final video of the demo. 

So it was a very nice experience having successfully completing an app
that we started with the idea, implementation and finally showing the
demo. It was very fun too, we didn't feel asleep at all. Once I get
the demo videos link, I will add it here :)
