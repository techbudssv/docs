---
title: "CRUD"
metaTitle: "Hypi Platform CRUD Documentation"
metaDescription: "Hypi platform documentation for the core app's CRUD operations generated for each app"
---

## Overview

Every app can have one or more release. Each release has its own schema.
A schema is a set of GraphQL type definitions which define the data model you would like to have in your app.

When you define a schema, Hypi automatically generates a number of APIs for you.
One of those APIs is the CRUD API i.e. **C**reate, **R**ead, **U**pdate, **D**elete.
The CRUD API in Hypi allows you to get data in and out of your apps.
The platform will generate some GraphQL types for you.

On this page we will use the following the GraphQL schema as an example.
```graphql
type Message {
    content: String!
    from: Account
    to: Account
}
```

## Generated types
Hypi will generate a number of GraphQL elements from this `Message` type.
Firstly, in GraphQL this is called an "output type". When working with GraphQL there is a distinction between the types you query and the types you use to insert data.
In Hypi, the input type and output type for the CRUD API has the same structure (same fields).

If you wish, you can use a different input structure for your custom APIs. The types currently generated by Hypi are:

> This is true for any type in your schema
>
> We use `Message` in this example, however the same thing happes for **all** types defined in your schema.

### MessageInput
An input type which matches the output type you created, including required fields
```graphql
input MessageInput {
    hypi: HypiInput
    content: String!
    from: AccountInput
    to: AccountInput
}
```
### MessageInputOpt
An input type which matches the output type you created but **does not** keep the required properties you defined.
```graphql
input MessageInputOpt {
  hypi: HypiInputOpt
  content: String
  from: AccountInputOpt
  to: AccountInputOpt
}
```
### MessageAggs
An output type that is returned if you run an aggregation query over the `Message` data

```graphql
type MessageAggs {
  content: AggOtherScalar
}
```
Aggregations can be performed on scalar fields hence `content` is of type `AggOtherScalar`.
If your type has a numeric field such as `Int` or `Float` then instead of `AggOtherScalar` the generated field will have type `AggInt` or `AggFloat` respectively.
These are described in more details in the [aggregations API](/products/axiom/aggregations-api) documentation.

### MessageFields
An enum of the fields you defined in the `Message` type.
```graphql
enum MessageFields {
    hypi
    content
    from
    to
}
```

## Insert and update data

The first thing you may want to do once you create an instance is add some data.
In Hypi there is one function that is used for both inserting new data and updating existing ones called `upsert`.

```graphql
upsert(values: HypiUpsertInputUnion!): [Hypi!]!
```

Notice the argument `values` is plural because it allows you to create or update multiple values in a single request.
Its type, `HypiUpsertInputUnion` is automatically generated from the types in your app.
In this case it will look similar to this
```graphql
type HypiUpsertInputUnion {
  Message: [MessageInputOpt!]
  #...fields for other types here
  Account: [AccountInputOpt!]
}
```

In other words, `HypiUpsertInputUnion` has a field for every type in your app whether defined by you or inherited from an [app dependency](/products/axiom/app-dependencies).

### Insert Example
To add, you can use a query similar to the following

<div className={"code-container"}>

<div className={"code-column"}>

```
#GraphQL query
mutation Upsert($values: HypiUpsertInputUnion!) {
  upsert(values: $values) {
    id
  }
}

#GraphQL varibles
{
  "values": {
    "Message": [
      {
        "content": "This is example message 1"
      },
      {
        "content": "This is example message 2"
      }
    ]
  }
}
```
</div>
<div className={"code-column"}>

```json
{
  "data": {
    "upsert": [
      {
        "id": "01ED7DZ7JHDRGTWHS4GXKE17BT"
      },
      {
        "id": "01ED7DZ7JR2QF7M2KEPF99Z93Y"
      }
    ]
  }
}
```
</div>

</div>

### Update Example
To update data, you can use a query similar to the following. Notice that is is the same GraphQL query.
The difference is that the `hypi.id` field is provided.

<div className={"code-container"}>

<div className={"code-column"}>

```
#GraphQL query
mutation Upsert($values: HypiUpsertInputUnion!) {
  upsert(values: $values) {
    id
  }
}

#GraphQL varibles
{
  "values": {
    "Message": [
      {
        "hypi": {
          "id": "01ED7DZ7JHDRGTWHS4GXKE17BT"
        },
        "content": "This is example message 1 updated"
      },
      {
        "hypi": {
          "id": "01ED7DZ7JR2QF7M2KEPF99Z93Y"
        },
        "content": "This is example message 2 updated"
      }
    ]
  }
}
```
</div>
<div className={"code-column"}>

```json
{
  "data": {
    "upsert": [
      {
        "id": "01ED7DZ7JHDRGTWHS4GXKE17BT"
      },
      {
        "id": "01ED7DZ7JR2QF7M2KEPF99Z93Y"
      }
    ]
  }
}
```
</div>

</div>

> Keep in mind
>
> You are free to mix insert and update operations. The system will accept and process them correctly.
>
> The order of execution is **undefined**. Hypi can choose to process multiple requests sent in one query in parallel.
>
> There is a limit of 25 items per request
>
> There is a request timeout of 2 seconds within the entire operation must complete

## Querying data

Once data is in your app, the next step is getting it back out.
There are two GraphQL functions for getting data out of the platform.

### Get an object by ID
The first is the get method. It allows you to get a single object using its ID.

<div className={"code-container"}>

<div className={"code-column"}>

```
#GraphQL query
{
  get(type: Message, id: "01ED7DZ7JHDRGTWHS4GXKE17BT"){
    ... on Message {
      hypi{
        id
        created
        updated
      }
      content
    }
  }
}
```
</div>
<div className={"code-column"}>

```json
{
  "data": {
    "get": {
      "hypi": {
        "id": "01ED7DZ7JHDRGTWHS4GXKE17BT",
        "created": "2020-07-14T07:49:04Z",
        "updated": "2020-07-14T07:53:11Z"
      },
      "content": "This is example message 1 updated"
    }
  }
}
```
</div>

</div>


### Find objects that match a query

The second approach to getting data out of your app is by using the `find` function.
Unlike the `get` function, this returns a list of objects matching the filter provided.

```graphql
find(
    type: HypiMutationType!
    arcql: String!
    first: Int
    after: String
    last: Int
    before: String
    includeTrashed: Boolean
): HypiFilterConnection!
```

This function has a number of parameters that enable you to filter and page through data.

| Parameter | Description | Example |
|-|-|-|
| type | The type that you want to find data for | `Message`, `Account` |
| arcql | The ArcQL that will be used to filter the data | `content \* 'example.*updated'` |
| first | Limit the number of results returned when used with the **after** parameter |  |
| after | Return data after this token. This is the ID of an object returned previously that you'd like to get results following it |  |
| last | Limit the number of results returned when used with the **before** parameter |  |
| before | Return data before this token. This is the ID of an object returned previously that you'd like to get results before it |  |
| includeTrashed | If true, data that was marked as *trashed* will be included in the results as well, `false` by default. |  |


<div className={"code-container"}>

<div className={"code-column"}>

```
#GraphQL query
{
  find(type: Message, arcql: "*") {
    edges {
      cursor
      node {
        ... on Message {
          hypi {
            id
            created
            updated
          }
          content
        }
      }
    }
    pageInfo {
      hasPreviousPage
      hasNextPage
      startCursor
      endCursor
      pageLimit
      previousOffsets
      nextOffsets
    }
  }
}
```
</div>
<div className={"code-column"}>

```json
{
  "data": {
    "find": {
      "edges": [
        {
          "cursor": "01ED7DZ7JHDRGTWHS4GXKE17BT",
          "node": {
            "hypi": {
              "id": "01ED7DZ7JHDRGTWHS4GXKE17BT",
              "created": "2020-07-14T07:49:04Z",
              "updated": "2020-07-14T07:53:11Z"
            },
            "content": "This is example message 1 updated"
          }
        },
        {
          "cursor": "01ED7DZ7JR2QF7M2KEPF99Z93Y",
          "node": {
            "hypi": {
              "id": "01ED7DZ7JR2QF7M2KEPF99Z93Y",
              "created": "2020-07-14T07:49:04Z",
              "updated": "2020-07-14T07:53:11Z"
            },
            "content": "This is example message 2 updated"
          }
        }
      ],
      "pageInfo": {
        "hasPreviousPage": false,
        "hasNextPage": false,
        "startCursor": "FIRST",
        "endCursor": "LAST",
        "pageLimit": 25,
        "previousOffsets": [],
        "nextOffsets": []
      }
    }
  }
}
```
</div>

</div>

## Deleting data

There are two types of deletions supported in the platform.
The first is known as a *soft delete* where data is not really deleted but instead "marked" as deleted.
Use the `trash` function to perform a soft delete.

The second type of delete is irreversible. The data is permanently deleted from the system and cannot be undone.
Use the `delete` or `deleteScalar` functions to permanently delete data. The difference between these two is explained below.

### trash

In some cases you want your data to appear as if it is deleted but have the ability to restore it.
For example, if you were storing email data. There is often a "recycle" or "trash" folder.
By using the `trash` function, you can achieve the same thing.

```graphql
trash(type: HypiMutationType!, arcql: String!): Int!
```

The function returns the number of records that were marked as trash.

### Example

<div className={"code-container"}>

<div className={"code-column"}>

```
#GraphQL query
mutation {
  trash(type: Message, arcql: "hypi.id = '01ED7DZ7JHDRGTWHS4GXKE17BT'")
}
```
</div>
<div className={"code-column"}>

```json
{
  "data": {
    "trash": 1
  }
}
```
</div>

</div>

If you use the `get` or `find` queries now, by default they will not return this object.
You must set the `includeTrashed` parameter to `true` to have them return the trashed object.

### untrash

The opposite of the `trash` function is `untrash`. Use this function to remove the trash marker from data that were previously marked as trash.

```graphql
untrash(type: HypiMutationType!, arcql: String!): Int!
```

The function returns the number of records that were marked untrashed.

### Example

<div className={"code-container"}>

<div className={"code-column"}>

```
#GraphQL query
mutation {
  untrash(type: Message, arcql: "hypi.id = '01ED7DZ7JHDRGTWHS4GXKE17BT'")
}
```
</div>
<div className={"code-column"}>

```json
{
  "data": {
    "untrash": 1
  }
}
```
</div>

</div>

### delete

If you want to delete data permanently, use the `delete` function.

```graphql
delete(type: HypiMutationType!, arcql: String!): Int!
```

The function returns the number of records that were marked deleted.

### Example

<div className={"code-container"}>

<div className={"code-column"}>

```
#GraphQL query
mutation {
  delete(type: Message, arcql: "hypi.id = '01ED7DZ7JHDRGTWHS4GXKE17BT'")
}
```
</div>
<div className={"code-column"}>

```json
{
  "data": {
    "delete": 1
  }
}
```
</div>

</div>

### deleteScalar

List fields in Hypi are not stored directly with the other scalar fields.
This allows you to be able to add unlimited number of items to a list field.
As a result of this, f you want to delete data permanently from a scalar list, use the `deleteScalar` function.

```graphql
deletedScalars(
from: HypiMutationType!
field: String!
values: [String!]!
id: String!
): Int!
```

| Parameter | Description | Example |
|-|-|-|
| from | The type which has the scalar list field you want to delete values from | `Message`, `Account` |
| field | The name of the field on the type |  |
| values | The list of scalar values to delete from the list |  |
| id | The ID of the object from which to delete the scalar values |  |

The function returns the number of records that were marked deleted.