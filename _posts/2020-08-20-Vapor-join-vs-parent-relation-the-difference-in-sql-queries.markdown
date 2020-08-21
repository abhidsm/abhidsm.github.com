---
layout: post
title: The difference in sql queries for join vs parent relation in Vapor
categories: Vapor 
tags: Vapor Fluent join vs parent relation the difference in sql queries
description: Vapor Fluent join vs parent relation the difference in sql queries
---
### Parent Relation
> The @Parent relation stores a reference to another model's @ID property.
```swift
final class Star: Model {
    static let schema = "stars"

    @ID(key: .id)
    var id: UUID?
} 

final class Planet: Model {
    // Example of a parent relation.
    @Parent(key: "star_id")
    var star: Star
} 
```
We can eager load the star of planets with the following method:
```swift
Planet.query(on: database).with(\.$star).all().map { planets in
    for planet in planets {
        // `star` is accessible synchronously here 
        // since it has been eager loaded.
        print(planet.star.name)
    }
}
```
### Join
Another method to eager load the star is to join the Star to Planets in the same query:
<!--more-->
```swift
Planet.query(on: database).join(Star.self, on: \Planet.$star_id == \Star.$id).all().map { planets in
    for planet in planets {
        // `star` is accessible synchronously here 
        // since it has been eager loaded.
        print(try planet.joined(Star.self).name)
    }
}
```

### The difference in SQL queries

**Relation**

Relation makes 2 SQL queries first one to fetch all the Planets and second query to fetch all the stars with ids in an array.
```sql
SELECT * from planets;

SELECT * from stars where id in (1,2,3);
```

**Join**

Join makes only a single SQL query with inner join.
```sql
SELECT * from planets INNER JOIN stars ON "planets"."star_id" = "stars"."id";
```

There is a difference in the usecase as well, if we want to do a `filter` or `order` on the parent table field then better go with join. We won't be able to do the `filter` or `order` on the relation table using the relation approach, but we can do it with the join:

```swift
Planet.query(on: database).join(Star.self, on: \Planet.$star_id == \Star.$id).filter(Star.self, \.$name, .custom("ILIKE"), "%Sun%").all().map { planets in
    for planet in planets {
        print(try planet.joined(Star.self).name)
    }
}
```

Thanks for Reading :sunglasses: