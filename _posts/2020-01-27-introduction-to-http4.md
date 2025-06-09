---
categories: [Programming]
title: Introduction to Http4k
date: 2020-01-27 09:52:21
tags: [toolkit, kotlin, functional programming]
---

A couple of weeks ago, I was looking for a very simple way to create a web server. I found [http4k]{:target="_blank"}, which is a lightweight but fully-featured HTTP toolkit written in pure Kotlin that enables the serving and consuming of HTTP services in a functional and consistent way.

#### Some Features of http4k

- Small, written in functional Kotlin with zero dependencies
- Very simple, no magic, no reflection
- Immutable HTTP model, which makes it very easy to test/debug
- Serverless / server independent

## WebServer as a Function

http4k is designed as an application as a function.
A web server can be viewed as one big function that handles requests.
It's literally represented as a typealias:

```kotlin
typealias HttpHandler = (HttpRequest) -> HttpResponse
```

An endpoint that echoes the request body can be created with just two simple lines:

```kotlin
val app: HttpHandler = {
    request: Request -> Response(OK).body(request.body)
}
val server = app.asServer(SunHttp(8000)).start()
```

Request and Response are immutable objects.
SunHttp is a container which is only meant for development.
For production, use Netty, Jetty, ApacheServer, etc.
Because `app` is just a Kotlin function, it can be composed very easily.

## Filters

http4k provides a `Filter` function, which is basically
a way to intercept the Request/Response pipeline.

```kotlin
typealias Filter = (HttpHandler) -> HttpHandler
```

The code snippet below shows typical composition of HttpHandlers:

```kotlin
val setContentType = Filter { nextHandler ->
        { request -> nextHandler(request)
            .header("Content-Type", "text/plain")
        }
    }

val composedApp = setContentType.then(app)
```

## Routing and Composition

http4k provides some high-level functions which help to compose applications
that handle each request differently depending upon URL, method, etc.
These functions accept multiple HttpHandlers and return a composed handler.

```kotlin
val app : HttpHandler = routes(
    "/api" bind GET to apiApp,
    "/other" bind routes(
        "/get" bind GET to {_:Request -> Response(OK)},
        "/post" bind POST to otherPostHandlerFn
    )
)
```

## Conclusion

http4k really stands by the promises they made.
APIs from http4k are written very thoughtfully and are not opinionated.
They are easy to reason with and work with.
The only downside can be the lack of coroutine support.

To know more about http4k rationale, visit [http4k/rationale](https://www.http4k.org/rationale/){:target="_blank"}

[http4k]: https://www.http4k.org/
