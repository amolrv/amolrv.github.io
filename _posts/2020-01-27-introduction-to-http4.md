---
title: Introduction to Http4k
layout: post
date: 2020-01-27 09:52:21
tags: [http4k, kotlin]
---

Couple of weeks ago, I was looking for very simple way to create web server.
I found [http4k] which is a lightweight but fully-featured HTTP toolkit
written in pure Kotlin that enables the serving and consuming of HTTP services
in a functional and consistent way.

#### Some of Features of http4k

- small, written in functional kotlin with zero dependencies.
- very simple, no magic, no reflections.
- immutable http model, which make it very easy to test/debug.
- serverless / server independent

## WebServer as Function

http4k is designed as application as function.
Web server can be looked as one big function which handles requests.
so it's literally represented as typealias

```kotlin
typealias HttpHandler = (HttpRequest) -> HttpResponse
```

An endpoint which echos request body can be created just by 2 simple lines.

```kotlin
val app: HttpHandler = {
    request: Request -> Response(OK).body(request.body)
}
val server = app.asServer(SunHttp(8000)).start()
```

Request and Response are immutable objects.
SunHttp is container which only meant for development.
For production use Netty, Jetty, ApacheServer etc.
Because `app` is just kotlin function, it can be composed very easily.

## Filters

http4k provides a `Filter` function which is basically
to intercept the Request/Response pipeline.

```kotlin
typealias Filter = (HttpHandler) -> HttpHandler
```

Below code snippet shows typical composition of HttpHandlers.

```kotlin
val setContentType = Filter { nextHandler ->
        { request -> nextHandler(request)
            .header("Content-Type", "text/plain")
        }
    }

val composedApp = setContentType.then(app)
```

## Routing and composition

http4k provides some high level functions which helps to compose application
which handles each request differently depending upon url, method etc.
These functions accepts multiple httpHandlers and return composed handler.

```kotlin
val app : HttpHandler = routes(
    "/api" bind GET to apiApp
    "/other" bind routes(
        "/get" bind GET to {_:Request -> Response(OK)}
        "/post" bind POST to otherPostHandlerFn
    )
)
```

## Conclusion

http4k really stands with the promises they made.
API from http4 are written very thoughtfully and are not opinionated.
They are easy to reason and work with.
Only downside can be no support for coroutine.

To know more about http4k rationale visit [http4k/rationale](https://www.http4k.org/rationale/)

[http4k]: https://www.http4k.org/
