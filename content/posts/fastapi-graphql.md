+++
title = 'Should I use GraphQL with FastAPI?'
date = 2023-09-22T23:46:51-04:00
draft = false
tags = ['graphql', 'fastapi']
+++

# FastAPI and GraphQL

At work, we use [FastAPI](https://fastapi.tiangolo.com/).

FastAPI works well for us for many reasons, but the top 3 are:

1. Extremely robust request parameter validation with Pydantic
2. Dependency injection
3. Automatic Swagger documentation

FastAPI gives us a huge productivity and code quality boost. Coming from a frameworkless PHP  backend, pretty much *anything* would give us a productivity boost.

Recently, I evaluted if GraphQL would be a good fit for our platform. I wanted to know if the traditional way a REST API is built was slowing us down. My conclusion was overwhelmingly: no. GraphQL is not a good fit for us, and I'll explain why.


## GraphQL

What is GraphQL? Here's the exceprt from [graphql.org](https://graphql.org/):

> GraphQL is a query language for APIs and a runtime for fulfilling those queries with your existing data. GraphQL provides a complete and understandable description of the data in your API, gives clients the power to ask for exactly what they need and nothing more, makes it easier to evolve APIs over time, and enables powerful developer tools.

It might be easier to understand when compared to REST API.

In a traditional REST API, there are endpoints that provide access to a resource (i.e., `/api/users`), and those endpoints have methods associated for managing that resource (i.e. `POST /api/users` might create a user). Endpoints (typically) respond with well-known JSON response bodies.

In GraphQL, there's a single endpoint which accepts "queries" and "mutations". Queries are akin to a `GET` request, while a mutation is like a `POST`, `PUT`, `PATCH`, or `DELETE`. Instead of querying or mutating an endpoint, a client will query or mutate one or many resource(s) like a `User`.


Here's an example that might be helpful - loading a user by username from the API.

With a REST API, if I request `GET /api/user/james` from the server to get the logged-in user's identity, it will respond with something like:
```json
{
    "id": 1,
    "username": "james",
    "status": "ACTIVE",
    "admin": true
}
```

The equivalent request in GraphQL looks like this:
```graphql
{
    user(username: "james") {
        id
        username
        status
    }
}
```

And the response looks like this:

```json
{
    "user": {
        "id": 1,
        "username": "james",
        "status": "ACTIVE"
    }
}
```

Notice in the GraphQL example, the `admin` property is not there. That is because it wasn't included in the original query. Only `id`, `username`, and `status` were queried. This is quite useful - the client can request individual fields from the API!

Another advantage with GraphQL is that it can run multiple queries in a single request. For example, if I want to query my REST API for two users, I have to run the request twice: `GET /api/users/james` and `GET /api/users/jeff`. GraphQL allows the client to request multiple resources in a single query:

```graphql

{
    jeff: user(username: "jeff") {
        id
        username
        status
    }
    james: user(username: "james") {
        id
        username
        status
    }
}
```

And the response looks like:

```json
{
    "jeff": {
        "id": 2,
        "username": "jeff",
        "status": "ACTIVE"
    },
    "james": {
        "id": 1,
        "username": "james",
        "status": "ACTIVE"
    }
}
```

I could go into how this looks in code, but that is dependent on which GraphQL implementation you choose. This also isn't a post about what GraphQL can & cannot do. The above example is very trivial and barely scratches the surface of GraphQL's capabilities. Have a look at their [docs](https://graphql.org/learn/) for a more in-depth look. 

## The Problem

I don't have a problem with GraphQL at all. It is a very powerful tool, and I would like to work on a GraphQL application one day. If the opportunity comes, I might even build one.

However, if you're already on FastAPI, I don't see much advantage to adding GraphQL to your stack.


### Input Validation

FastAPI ships with a powerful input validation system using Pydantic. If we were to add GraphQL to our stack, we'd have to re-implement our Pydantic models as types in our GraphQL library of choice.

[strawberry](https://strawberry.rocks/docs/integrations/pydantic#pydantic-support) has experimental support for Pydantic, but still requires a wrapper types for each model before adding it to GraphQL's schema. You can't use Pydantic types directly. Sure, there are some other libraries out there that try to do this, but none seem to have much traction today.

### Dependency Injection

FastAPI's dependency injection system helps immensely with code deduplication, performance, and code readability.

strawberry's [context_getter](https://strawberry.rocks/docs/integrations/fastapi#context_getter) allows for using FastAPI's dependency injection, but it does not allow each resolver/field to specify which dependencies it needs. This means all operations share the same dependencies, or you just define your most common dependencies and let each operation allocate the dependencies it needs.

### SQLAlchemy

Over-selecting.

In the GraphQL example above we omitted the `admin` property from the user query. At face value, this seems very useful. The frontend is able to select the fields it needs from the user. However, this does not propagate down to the query language. Our ORM loaded all of user columns from the database and returned them to GraphQL, but GraphQL only responded with the fields that the user selected.

This is not a huge issue but I suspect it would lead to widespread over-selecting throughout a large codebase.

## Should you though? Does it matter?

On a new project, I think GraphQL is a fine choice, but I wouldn't run it on FastAPI.

Does it matter? Maybe. Because the database is the primary bottleneck, you don't get *that much* of a performance increase with GraphQL, especially when you can use browser APIs like [`Promise.allSettled()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/allSettled). GraphQL may even be slower than a well built asynchronous REST API.

## Final thoughts

For an established and well-oiled FastAPI project, GraphQL does not give us a productive advantage. In fact, it requires us to either abandon or kludge many of the features that makes FastAPI so attractive.

In fact, there's even a [callout](https://github.com/tiangolo/fastapi/blob/69a7c99b447c9ef103dc03e93d172cabd99ac832/docs/en/docs/how-to/graphql.md?plain=1#L7-L12) in FastAPI's page about GraphQL that cautions readers to evaluate their use case for GraphQL.

I'd love to see a tighter integration between FastAPI & strawberry that allows developers to use their existing Pydantic models with full validation support and better dependency injection support, but I don't have my fingers crossed.

If you want to use GraphQL, I don't think FastAPI is the way to go about it.

Did I miss anything? Did I get anything wrong? Are you using GraphQL with FastAPI in production? Feel free to send feeback to [sam@redeemed.dev](mailto:sam@redeemed.dev).
