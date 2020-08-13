---
layout: post
title: How to add autoincrement field in Vapor
categories: Vapor 
tags: autoincrement field vapor fluent postgres database table column 
description: How to add autoincrement column to postgres using Vapor fluent
---

We have created a new model in Vapor with id as UUID and other fields. 
```swift
// Property Wrapper
@ID(key: .id)
var id: UUID?
```
Now we got a requirment to add an autoincrement field which can be used to identify the object like model_number. Since the default id field is UUID we cannot use the exisitng id for this purpose. If we are creating a new model with this scenario, we could have used the id as an autoincrement identifier:
```swift
// Migration
.field("id", .int, .identifier(auto: true))
```
<!--more-->
Since we need the id as UUID and we need an additional autoincrement field to use it as a model_number, we used a custom type `serial`. PostgreSQL has the data types smallserial, serial and bigserial, these are similar to AUTO_INCREMENT property supported by some other databases. The type name serial creates an integer columns. The type name bigserial creates a bigint column. The type name smallserial creates a smallint column.

We can use the serial type to create an autoincrment field in Vapor.
```swift
// Property Wrapper
@Field(key: "model_number")
var modelNumber: Int?

// Migration
.field("model_number", .custom("serial"))
```
No need to assign any values while create and update, the values will be autoincremented by the database.              