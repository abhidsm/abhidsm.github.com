---
layout: post
title: Vapor Custom headers in the Response using Middleware
categories: Vapor 
tags: Vapor custom headers in the response using Middleware
description: Vapor custom headers in the response using Middleware
---
We can create a new Middleware to add custom headers to the response:
```swift
// CustomHeaderMiddleware.swift 

import Vapor

class CustomHeaderMiddleware: Middleware {
    
    func respond(to request: Request, chainingTo next: Responder) -> EventLoopFuture<Response> {
        return next.respond(to: request).map { response in
            response.headers.add(name: "Secure", value: "true")
            return response
        }
    }

}
```
Then we should use this middleware from Configure:
```swift
// Configure.swift

let customHeaderMiddleware = CustomHeaderMiddleware()
app.middleware.use(customHeaderMiddleware)
```
These changes will add the custom header to all the responses from the application.