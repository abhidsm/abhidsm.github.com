---
layout: post
title: Sendgrid integration with Vapor
categories: Vapor 
tags: sendgrid integration vapor swift email client web application
description: SendGrid is a customer communication platform for transactional and marketing email. How we can integrate Sendgrid with our Web application (Vapor) to send emails.  
---

[SendGrid](https://sendgrid.com/) is a customer communication platform for transactional and marketing email. SendGrid provides a cloud-based service that assists businesses with email delivery. How can we integrate Sendgrid with our Web application ([Vapor](https://docs.vapor.codes)) to send emails?
<!--more-->
### Vapor 4 Integration

#### Package.swift

Add the sendgrid package to the dependency and target.
```swift
dependencies: [
    .package(url: "https://github.com/vapor-community/sendgrid.git", from: "4.0.0"),
],
targets: [
    .target(name: "App", dependencies: [
        .product(name: "SendGrid", package: "sendgrid"),
    ]),
]
```
#### Controller/Model

Import the package and use the sendgrid email client to send the email. In the below example we are using a template created in the Sendgrid to send the email.
```swift
import SendGrid

func sendEmail(_ req: Request, email: String) throws -> EventLoopFuture<HTTPStatus> {
    let to = EmailAddress(email: email)
    let from = EmailAddress(email: "from@sender.com", name: "Sender")
    let personalization = Personalization(to: [to], dynamicTemplateData: ["key": "value"])
    var emailContent: [String: String] = [:]
    emailContent["type"] = "text/html"
    emailContent["value"] = "Dummy Email Content"
    var email = SendGridEmail(personalizations: [personalization], from: from, content: [emailContent])
    email.templateId = "d-test0template1id2"
    let sendGridClient = req.application.sendgrid.client
    do {
        return try sendGridClient.send(emails: [email], on: req.eventLoop).transform(to: HTTPStatus.ok)
    } catch {
        req.logger.error("\(error)")
        return req.eventLoop.makeFailedFuture(error)
    }
}
```

### Vapor 3 Integration

#### Package.swift

Add the sendgrid package to the dependency and target.
```swift
dependencies: [
    .package(url: "https://github.com/vapor-community/sendgrid-provider.git", from: "3.0.6"),
],
targets: [
    .target(name: "App", dependencies: ["SendGrid",]),
]
```

#### configure.swift

Register the config and the provider:
```swift
import SendGrid

let sendGridConfig = SendGridConfig(apiKey: Environment.get("SENDGRID_API_KEY")!)
services.register(sendGridConfig)
try services.register(SendGridProvider())
```

#### Controller/Model

Import the package and use the sendgrid email client to send the email. In the below example we are using a template created in the Sendgrid to send the email.

```swift
import SendGrid

func sendEmail(_ req: Request, email: String) throws -> Future<HTTPStatus> {
    let to = EmailAddress(email: email)
    let from = EmailAddress(email: "from@sender.com", name: "Sender")
    let personalization = Personalization(to: [to], dynamicTemplateData: ["key": "value"])
    var email = SendGridEmail(personalizations: [personalization], from: from)
    email.templateId = "d-test0template1id2"
    let sendGridClient = try req.make(SendGridClient.self)
    return try sendGridClient.send([email], on: req).transform(to: HTTPStatus.ok).catchMap({ (error) -> (HTTPStatus) in
        print(error)
        return HTTPStatus.internalServerError
    })
}
```
That's it. Have a nice day.