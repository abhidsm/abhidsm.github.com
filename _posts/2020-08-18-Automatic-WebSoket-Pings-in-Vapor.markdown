---
layout: post
title: Automatic WebSocket pings in Vapor
categories: Vapor 
tags: Automativ WebSocket pings in Vapor pingInterval connection still active
description: Automatic WebSocket pings in Vapor. Adds a pingInterval property on WebSocket which will trigger automatic pings to the peer to ensure that the connection is still active.
---

> WebSockets allow for two-way communication between a client and server. Unlike HTTP, which has a request and response pattern, WebSocket peers can send an arbitrary number of messages in either direction. Vapor's WebSocket API allows you to create both clients and servers that handle messages asynchronously.

We have a messaging application implemented using WebSocket deployed on AWS. We were facing some intermittent connection termination issues. When looked at the issue, we found that Amazon's Application Load Balancers are configured to shut down connections that haven't had traffic. There is already an issue reported in the [websoclet-kit github repo](https://github.com/vapor/websocket-kit/issues/66).
<!--more-->
There is already a [PR exist](https://github.com/vapor/websocket-kit/pull/68) which solves this problem. The PR adds a `pingInterval` property on WebSocket which will trigger automatic pings to the peer to ensure that the connection is still active. These changes are merged to the latest version of websocket-kit 2.1.0.

But the latest version of [Vapor:master](https://github.com/vapor/vapor/blob/7d1e0b162f61fd4b34c4a7e575cfaef14e65484a/Package.swift#L52) is using websocket-kit 2.0.0.

Right now we don't have a solution apart from manually implementing the ping, so we implemented the `"ping"` command in the `onText` event and returned `"pong"`. The client will send `"ping"` on a specified interval.
```swift
ws.onText { (_, data) in
    if data == "ping" {
        ws.send("pong")
    }
}
```
