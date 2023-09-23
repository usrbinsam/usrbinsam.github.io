+++
title = 'FastAPI & GraphQL'
date = 2023-09-22T23:46:51-04:00
draft = true
+++
At work, we use [FastAPI](https://fastapi.tiangolo.com/).

One of the features we lean on most heavily is FastAPI's ability to auto-validate request (and response) parameters. This helps our small team rapidly build out projects without manually validating user input parameters.

I recently took the time to evaluate if GraphQL would be a useful addon that could boost productivity. FastAPI's touts support for GraphQL and recommends Strawberry.

I built a simple Query for loading user-account information through the GraphQL endpoint. By the time I had written the first query, I realized that GraphQL support in FastAPI just seems like a "look what we can do!" feature. Here's why ...

FastAPI's tight integration with Pydantic gives it a huge advantage in building a REST API without making the developer fuss over input validation, and allows them to get straight to writing business logic. When you add in GraphQL, you lose this feature, somewhat defeating the purpose of being on FastAPI in the first place.

A neat feature of GraphQL is the ability to select individual fields of a type. In my test setup the account query returns the user's ID, username, and several other important pieces of information the frontend will need. If a query on the frontend determines it *only* needs a subset of those fields, it can tell GraphQL only to respond with those fields. This de-couples the frontend & backend too much and creates the ability to overquery.

Here's what I mean ...

In my standard FastAPI user account info, we return a JSON object that looks something like this:

```json
{
    "id": 1821,
    "username": "sam",
    "avatar": "https://example.com/avatar.png",
    "roles": ["admin", "billing", "forum"],
    "status": "ACTIVE",
    "firstName": "Samuel",
    "lastName": "Hoffman"
}
```

Our SQLAlchemy query selects from the `users` table for most of this information and joins to the `roles` table.
