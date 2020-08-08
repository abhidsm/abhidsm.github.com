---
layout: post
title: Vapor 4 migration issues and fixes
categories: Vapor 4 migration issues and fixes Swift Vapor4  
description: The issues we have faced and fixes made while migrating Vapor 3 application to Vapor 4. This includes the 
---

### Introduction

I was working on a software product with backend server implemented using Swift and Vapor. We were using Vapor 3 and wanted to migrate it to Vapor 4, since Vapor 4 released on Apr 2020. We followed the [upgrading link](https://docs.vapor.codes/4.0/upgrading/) from Vapor docs to start the upgrading process. In this blog post I will give some additional information about the issues we had faced while upgrading to Vapor 4.

### Package.swift

We have to put the latest version of packages which has Vapor 4 support in the dependency section. Some of the packages we were using didn't have Vapor 4 support in their latest version, so we had to look for any pull requests or forked versions with Vapor 4 support.

For S3 we have changed the library from `LiveUI/S3.git` to `swift-aws/aws-sdk-swift.git`. Also for SwiftDate, we have used the master branch instead of the version.

Also there were some issues with the naming of the product and package. In Vapor 4 while adding the dependencies to targets we have to mention the product name and package name, but there was an error: `product name not matching`. To fix this issue we have to add name to the package.

```yaml
dependencies: [
  ...,
	.package(name: "AWSSDKSwift", url: "https://github.com/swift-aws/aws-sdk-swift.git", from: "4.7.0"),
  .package(url: "https://github.com/Maxim-Inv/SwiftDate.git", .branch("master")),
],
targets: [
    .target(name: "App", dependencies: [
        .product(name: "S3", package: "AWSSDKSwift"),
        .product(name: "SwiftDate", package: "SwiftDate"),
```

We have removed the deprecated packages like `auth` and `vapor-ext` from the dependencies.

### Configure.swift 

We have removed some unnecessary code which were getting build errors, for example `ContentConfig, MiddlewareConfig, MigrationConfig, NIOServerConfig, SendGridConfig, DatabasesConfig, defaultDatabase, etc.`

To convert between camelCase and snake_case in the json request and response we have to use `keyEncodingStrategy` and `keyDecodingStrategy`, and to set the maxBodySize we have to use `app.routes.defaultMaxBodySize = "5MB"` in Configure file. 

There are some changes with the integration of SendGrid, we need only the `import SendGrid` required in this file. We can directly use the Send mail code where it is required.

### Routes.swift

There are some changes with the syntax of using middlewares in Routes, `User.guardAuthMiddleware()` is changed to `User.guardMiddleware()`, `User.basicAuthMiddleware(using: BCryptDigest())` is changed to `User.authenticator()` and `User.authSessionsMiddleware()` is changed to a combination of `app.sessions.middleware` and `User.sessionAuthenticator()`.

```swift
let guardMiddleware = User.guardMiddleware()
let userAuth = User.sessionAuthenticator()
let sessionMiddleware = app.sessions.middleware
let sessionRouter = app.grouped(sessionMiddleware, userAuth, guardMiddleware)
sessionRouter.post("v1", "test_report", use: reportsController.fetchTestReport)
```

We have made some changes to the `User` model to support the Authentication. Removed the `TokenAuthenticatable`, `SessionAuthenticatable` and  `PasswordAuthenticatable` extensions and added the following extensions:
```swift
  extension User: ModelSessionAuthenticatable { }
  extension User: ModelAuthenticatable {
    static let usernameKey = \User.$email
    static let passwordHashKey = \User.$password
    func verify(password: String) throws -> Bool {
      try Bcrypt.verify(password, created: self.password)
    }
  }
```
Added the token based authentication changes to SessionToken model. 
```swift
  extension SessionToken: ModelTokenAuthenticatable {
     static let valueKey = \SessionToken.$value
     static let userKey = \SessionToken.$user
     var isValid: Bool {
         true
     }
  }
```

### Services 

There are some changes in the way we use ClientRequest/ClientResponse to send requests and get response. Before we were using `try req.client().get(url)` to send a request and get the reponse, but in Vapor 4 we have to use `req.client.get(URI(string: url))`. Similarly creating the Response object also changed from `Response(http: HTTPResponse(status: r.http.status, version: r.http.version, headers: [:], body: r.http.body), using: req)` to `Response(status: r.status, version: req.version, headers: [:], body: Response.Body.init(buffer: r.body ?? ByteBuffer(string: "")))`.

The .env file will be loaded automatically, so no need to mention `Environment.dotenv()` in the Configure.swift.

### NIO

When we throw an error inside a flatMap closure, we have to catch the code block. Either we can use do/catch or tuple-chaining as mentioed in the Vapor docs. On an addional note, if we are throwing an error like `throw Abort(.badRequest)` inside a flatMap block and only this line of code causing Throwing flatMap then we can rewrite this line to `req.eventLoop.makeFailedFuture(Abort(.badRequest))` to avoid catching the throw.

Many places we had to use the ByteBuffer to fix the error. For example while creating a Response object we did some changes as follows:
```swift
// Old Code
Response(http: HTTPResponse(status: r.http.status, version: r.http.version, headers: [:], body: r.http.body), using: req)

// New Code
Response(status: r.status, version: req.version, headers: [:], body: Response.Body.init(buffer: r.body ?? ByteBuffer(string: "")))
```
Also while decoding to a type from the JSON object we have to use ByteBuffer `decoder.decode(Type.self, from: ByteBuffer(string: data))`. To get the request body data we were using `request.body.string`, but it was returning an empty string, so we had to use `String(buffer: request.body.data ?? ByteBuffer(string: ""))` to get the request body data.

### Fluent

There are lots of changes in the Fluent like `use req.db instead of req`, `use class instead of struct`, `property wrappers`, `removed route path components`, `migrations`, `joins`, etc. 

We were using `.ilike` operator in the fluent filters, but it is not supported in Fluent 4. So we have to change the filter to `query.filter(\Model.$field, .custom("ILIKE") , "%"+value+"%")`. Also the `join` query got changed as below:
```swift
// Old Code 
Foo.query(on: req).join(\Baz.id, to: \Foo.bazId).alsoDecode(Baz.self)
FooResponse(foo: $0.0, baz: $0.1)

// New Code
Foo.query(on: req.db).join(Baz.self, on: \Foo.$bazId == \Baz.$id)
try FooResponse(foo: $0, baz: $0.joined(Baz.self))
```
There is one challenge we have faced with the field name while upgrading to Vapor 4, which is the `id` attribute is mandatory. In some of the models we were using uuid field as the primary key and its name was uuid, but after upgrading to Vapor 4 we started getting errors on this. We fixed it by adding the following property wrappers:
```swift
@ID(custom: "uuid", generatedBy: .user)
var id: String?
``` 
In Vapor 3 we were using struct for models and the initializer was available without writing any explicit intializer inside the struct. But with Vapor 4, since the models are classes, we have to create the initializers with the arguments required. Another challenge we faced in some parts of the code is related to updating an object, when we update an object we were getting an `existing object` error. We were using a decoded object, changes its values and save, but while saving fluent assumes its new object and trying to create. Since the id already exists we get this error, the solution is to add the following:
```swift
object._$id.exists = true
object.save(on: req.db)
```

### XCode

After Vapor 4 migration the Xcode console stopped showing the debug information like sql queries etc. To fix this we have to make some changes in the Edit Scheme -> Run -> Arguments, add name `LOG_LEVEL` and value `debug` to the Environment Variables section. This will fix the debug information not showing issue. 

### Dockerfile

We are using docker images to deploy the application, so we had to make some changes in the Dockerfile as well. 
```ruby
FROM vapor/swift:5.2
WORKDIR /app
ADD . ./
RUN apt-get update && apt-get install -y wget apt-transport-https software-properties-common libcurl4-openssl-dev git uuid-dev
RUN swift package clean
RUN swift build -c release --enable-test-discovery
RUN mkdir /app/bin
RUN mv `swift build -c release --show-bin-path` /app/bin
EXPOSE 8080
ENTRYPOINT ./bin/release/Run serve -e production -b 0.0.0.0
```


