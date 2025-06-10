---
categories: [Programming]
title: Introduction to Http4k
date: 2020-01-27 09:52:21
tags: [toolkit, kotlin, functional programming]
---

While searching for a minimalist approach to creating web servers, I discovered [http4k]{:target="_blank"}, a remarkable HTTP toolkit that caught my attention. Written in pure Kotlin, http4k stands out by offering a lightweight yet comprehensive solution for both serving and consuming HTTP services. What makes it truly special is its functional approach and consistent API design.

## Key Features of http4k

- **Pure Kotlin Implementation**: Built entirely in functional Kotlin with zero external dependencies
- **Simplicity at its Core**: No magic, no reflection - just pure functional programming
- **Immutable HTTP Model**: Makes debugging and testing straightforward and predictable
- **Cloud-Ready**: Works seamlessly in both serverless and traditional server environments

## The "Server as a Function" Concept

At its heart, http4k embodies a beautifully simple concept: a web server is essentially just a function that processes requests. This idea is elegantly captured in a single typealias:

```kotlin
typealias HttpHandler = (HttpRequest) -> HttpResponse
```

Here's how straightforward it is to create an endpoint that echoes back the request body:

```kotlin
val app: HttpHandler = {
  request: Request -> Response(OK).body(request.body)
}
val server = app.asServer(SunHttp(8000)).start()
```

Request and Response objects in http4k are immutable, which enhances reliability and predictability. While SunHttp serves well for development purposes, for production environments, you can easily switch to more robust options like Netty, Jetty, or ApacheServer. Since `app` is fundamentally just a Kotlin function, it's highly composable - a key feature we'll explore next.

## Filters: Enhancing Your HTTP Pipeline

http4k provides a powerful concept called `Filter`, which allows you to intercept and modify the Request/Response pipeline. A Filter is defined as:

```kotlin
typealias Filter = (HttpHandler) -> HttpHandler
```

Here's a practical example of how to compose HttpHandlers using filters:

```kotlin
val setContentType = Filter { nextHandler ->
    { request -> nextHandler(request)
      .header("Content-Type", "text/plain")
    }
  }

val composedApp = setContentType.then(app)
```

## Routing and Composition

http4k excels at composing applications through its high-level routing functions. These functions allow you to create sophisticated routing hierarchies while maintaining code clarity:

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

My experience with http4k has been overwhelmingly positive. The framework delivers on its promises with thoughtfully designed, unopinionated APIs that are both intuitive and pragmatic. The functional approach makes the code easy to reason about and maintain. While the framework currently doesn't support coroutines, its excellent design principles and simplicity make it a compelling choice for Kotlin HTTP applications.

For a deeper dive into http4k's design philosophy and rationale, visit [http4k/rationale](https://www.http4k.org/rationale/){:target="_blank"}

[http4k]: https://www.http4k.org/
