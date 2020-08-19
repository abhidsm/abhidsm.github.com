---
layout: post
title: Self referencing table and join query with ModelAlias in Vapor
categories: Vapor 
tags: Self referencing table and join query with ModelAlias in Vapor Fluent
description: Self referencing table and join query with ModelAlias in Vapor Fluent.
---
### Self Referencing Table
> A self referencing table is a table where the primary key on the table is also defined as a foreign key. Self-referencing table is a table that is a parent and a dependent in the same referential constraint. I. e. in such tables a foreign key constraint can reference columns within the same table.

For example we have a table for `categories` where we are storing all the categories and sub-categories:

| id   | name   | parent_id |
| ---- | ------ | --------- |
| 1    | OS     |           |
| 2    | iOS    | 1         |

In the above example, the sub-category `iOS` has a `parent_id` which is an `id` of a record in the same table. We can implement the same in Vapor as below:
```swift
// Category.swift
@ID(custom: "id")
var id: Int?
@Field(key: "name")
var name: String
@Field(key: "parent_id")
var parentId: Int?
```
<!--more-->
We want to return the categories with Parent object as below:
```swift
//CategoryResponse.swift
struct CategoryResponse: Content {
  var id: Int?
  var name: String
  var parentId: Int?
  var parent: Category?

  static func fromCategory(req: Request, category: Category) -> EventLoopFuture<CategoryResponse> {
    return Category.query(req: req.db).filter(\.$id == category.parent_id).first().map { parentCategory in 
      return CategoryResponse(id: category.id, name: category.name, parentId: category.parentId, parent: parentCategory)
    }
  }
}

// CategoriesController.swift
return Category.query(req: req.db).all().map { categories in 
  return categories.map { CategoryResponse.fromCategory(req: req, category: $0) }
}
```
This code will work and return the parent object for all the categories. But this will cause a performance issue if you have lots of records in this table, because we are fetching the parent category inside a loop. The number of queries required to fetch the parent categories depends on the number of categories in the table, so more number of records cause more time to complete this task. 

### Join

To avoid the N+1 queries, we use `joins`. For example we have 2 tables `users` and `profiles`:
```swift
// User.swift
@ID(custom: "id")
var id: Int?
@Field(key: "name")
var name: String

// Profile.swift
@ID(custom: "id")
var id: Int?
@Field(key: "image")
var image: String
@Field(key: "user_id")
var userId: Int?
``` 
We can join the tables to avoid performance issue:
```swift
// UserResponse.swift
struct UserResponse: Content {
  var id: Int?
  var name: String
  var profile: Profile?
}

// CategoriesController.swift
return User.query(req: req.db).join(Profile.self, on: \User.$id == \Profile.$userId).all().flatMapThrowing { users in 
  return try users.map { try CategoryResponse(id: $0.id, name: $0.name, profile: $0.joined(Profile.self)) }
}
```
This will generate only one query to fetch all the users and its profiles. But we cannot use join the same way in the self referencing table, because join on the same model won't work in Fluent.

### ModelAlias

We have to use `ModelAlias`, Model aliases allow you to join the same model to a query multiple times. To declare a model alias, create one or more types conforming to ModelAlias.

The categories example can be implemented as below:
```swift
// Category.swift
@ID(custom: "id")
var id: Int?
@Field(key: "name")
var name: String
@Field(key: "parent_id")
var parentId: Int?

@OptionalParent(key: "parent_id")
var parentCategory: Category?

//ParentCategory.swift
final class ParentCategory: ModelAlias {
    static let name = "parent_categories"
    let model = Category()
}

//CategoryResponse.swift
struct CategoryResponse: Content {
  var id: Int?
  var name: String
  var parentId: Int?
  var parent: ParentCategory?
}

// CategoriesController.swift
return Category.query(req: req.db).join(ParentCategory.self, on: \Category.$parentCategory.$id == \ParentCategory.$id).all().flatMapThrowing { categories in 
  return try categories.map { try CategoryResponse(id: $0.id, name: $0.name, parentId: $0.parentId, parent: $0.joined(ParentCategory.self)) }
}
``` 
