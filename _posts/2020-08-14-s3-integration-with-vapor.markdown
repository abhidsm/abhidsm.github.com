---
layout: post
title: S3 integration with Vapor
categories: Vapor 
tags: aws s3 cloud storage integration with Vapor swift web framework application
description: S3 integration with Vapor. We use S3 to store files in the cloud for easy and secure access fromthe web application. How to integrate S3 access with Vapor web applications. Vapor is a Swift web framework. 
---

We use S3 to store files in the cloud for easy and secure access. We use some Vapor packages to integrate Vapor application with S3. The implementation is different for the Vapor versions.

### Vapor 4 Integration

#### Package.swift

Add the S3 package to the dependencies and target.
```swift
dependencies: [
    .package(name: "AWSSDKSwift", url: "https://github.com/swift-aws/aws-sdk-swift.git", from: "4.7.0"),
],
targets: [
    .target(name: "App", dependencies: [
        .product(name: "S3", package: "AWSSDKSwift"),
    ]),
]
```
<!--more-->
#### Model/Controller

Put Object:
```swift
import S3

struct FileRequest: Content {
    var file: File?
}

let fileRequest = try req.content.decode(FileRequest.self)
let file = fileRequest.file
let data =  Data(file.data.readableBytesView)
let putObjectRequest = S3.PutObjectRequest(acl: .publicRead, body: data, bucket: "bucket", contentLength: Int64(data.count), key: "path")
let s3 = S3(accessKeyId: Environment.get("S3_KEY")!, secretAccessKey: Environment.get("S3_SECRET")!, region: .region)
return s3.putObject(putObjectRequest).flatMap { _ in
    // Store the path to db for future reference
}
```
Get Object:
```swift
let s3 = S3(accessKeyId: Environment.get("S3_KEY")!, secretAccessKey: Environment.get("S3_SECRET")!, region: .region)
return s3.getObject( S3.GetObjectRequest(bucket: "bucket", key: "path")).flatMapThrowing({ response in
    return String(data: response.body!, encoding: .utf8)!
})
```

### Vapor 3 Integration

#### Package.swift

Add the S3 package to the dependencies and target.
```swift
dependencies: [
    .package(url: "https://github.com/LiveUI/S3.git", from: "3.0.0"),
],
targets: [
    .target(name: "App", dependencies: ["S3",]),
]
```
#### configure.swift

Register the service:
```swift
import S3

try services.register(s3: S3Signer.Config(accessKey: Environment.get("S3_KEY")!, secretKey: Environment.get("S3_SECRET")!, region: Region(name: .region)), defaultBucket: "bucket")
```
#### Model/Controller

Put Object
```swift 
import S3

struct FileRequest: Content {
    var file: File?
}

return try req.content.decode(FileRequest.self).flatMap { fileRequest in
    let file = File.Upload(data: Data(fileRequest.file.data.withByteBuffer { $0 }), bucket: "bucket", destination: "path", access: .publicRead)
    let s3 = try req.makeS3Client()
    return try s3.put(file: file, headers: [:], on: req).flatMap { _ in
        // Store the path to db for future reference
    }
}
```
Get Object:
```swift
let s3: S3Client = try S3(defaultBucket: Environment.get("DEFAULT_PUBLIC_BUCKET")!, signer: req.makeS3Signer())
return try s3.get(file: "path", on: req).map({ response in
    return String(data: response.data, encoding: .utf8)!
})
```
