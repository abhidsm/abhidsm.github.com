---
layout: post
title: JSON encoding/decoding for Vapor Models 
categories: Vapor 
tags: JSON encoding decoding to / from HTTP messages for Vapor Fluent Models
description: How to achieve JSON encoding decoding for Vapor Fluent Models to / from HTTP messages. There are different approaches to achieve this, using DTO and using KeyStrategies.
---
>Vapor's content API allows you to easily encode / decode Codable structs to / from HTTP messages. JSON encoding is used by default with out-of-the-box support for URL-Encoded Form and Multipart. 

In Vapor 3 we were able to use `struct` to create Models and using `CodingKeys` we were able to map the model property name to the DB field name, also we were able to use this to encode/decode the models.  
```swift
struct User: Model, Content {
    var id: Int?
    var email: String
    var passwordHash: String

    enum CodingKeys: String, CodingKey {
        case id
        case email
        case passwordHash = "password_hash"
    }
}
```
For this model the JSON data will be:
```json
{
  "id": 1,
  "email": "test@test.com",
  "password_hash": "testPassword"
}
``` 
<!--more-->
In Vapor 4, the models are created using `class` and we are using property wrappers to map the property to the DB field. 
```swift 
final class User: Model {
    static let schema = "users"
    
    @ID(custom: "id")
    var id: Int?
    @Field(key: "email")
    var email: String
    @Field(key: "password_hash")
    var passwordHash: String
}
```
JSON format is:
```json
{
  "id": 1,
  "email": "test@test.com",
  "passwordHash": "testPassword"
}
```
As you noticed the JSON keys are the Model variable names, which are in camelCase and not in snake_case format.

Since `CodingKeys` won't work with Models in Vapor 4, we have 2 approaches to JSON encode / decode a Model to JSON with snake_case format:

### 1. Use Data Transfer Objects (DTO)

We should use separate `struct` to represent the request and response of a Model. For the User model we should use the structs UserRequest and UserResponse as DTOs:
```swift
struct UserRequest: Content {
    var id: Int?
    var email: String
    var passwordHash: String

    enum CodingKeys: String, CodingKey {
        case id
        case email
        case passwordHash = "password_hash"
    }
}

struct UserResonse: Content {
    var id: Int?
    var email: String

    enum CodingKeys: String, CodingKey {
        case id
        case email
    }
}
``` 
This way we can decode the HTTP request using UserRequest and assign the values to the User model and save. Similarly, for GET requests, we can fetch the data using Model and create UserResponse objects by passing the values and return the UserResponse objects. These DTOs will take care of the encoding and decoding of JSON data in snake_case format.

From Vapor Docs:
>When serializing to / from Codable, model properties will use their variable names instead of keys. Model's default Codable conformance can make simple usage and prototyping easier. However, it is not suitable for every use case. For certain situations you will need to use a data transfer object (DTO).
>
>Even if the DTO's structure is identical to model's Codable conformance, having it as a separate type can help keep large projects tidy. If you ever need to make a change to your models properties, you don't have to worry about breaking your app's public API. You may also consider putting your DTOs in a separate package that can be shared with consumers of your API.
>
>For these reasons, we highly recommend using DTOs wherever possible, especially for large projects.

### 2. keyEncodingStrategy with convertToSnakeCase and keyDecodingStrategy with convertFromSnakeCase
>ContentConfiguration.global lets you change the encoders and decoders Vapor uses by default. This is useful for changing how your entire application parses and serializes data.

Using `ContentConfiguration.global` we can set the behavior of encode / decode from any data. These changes will affect the entire application, so we can use this to set the conversion between snake_case and camelCase where it required.
```swift
// configure.swift

let encoder = JSONEncoder()
encoder.keyEncodingStrategy = .convertToSnakeCase
encoder.dateEncodingStrategy = .iso8601
ContentConfiguration.global.use(encoder: encoder, for: .json)
    
let decoder = JSONDecoder()
decoder.keyDecodingStrategy = .convertFromSnakeCase
decoder.dateDecodingStrategy = .iso8601
ContentConfiguration.global.use(decoder: decoder, for: .json)
``` 
The above code will take care of converting camelCase keys to snake_case while encoding and snake_case to camelCase while decoding. This will also take care of parsing the String date format to iso date format and vice versa.

In this approach we should be careful about the Model property names, those variable names will be automatically converted to snake_format. So there is a direct relationship with the model variable name to the JSON key name. Also in this approach, we don't have to create a DTO if not required, for example, if both request and response are the same as the model attributes, then we don't have to create the DTOs.

Please let me know if there is any other approach that is easier to implement. Thanks.