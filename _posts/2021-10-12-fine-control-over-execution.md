---
categories: [blog]
title: Fine control over execution in kotlin
date: 2021-10-12 10:00:00
tags: [kotlin, coroutine]
image:
  path: /assets/blog/2021-10-12-fine-control-over-execution.jpg
  width: 800
  height: 500
---
Programing language that supports concurrency and parallelism needs to provide cancellation mechanism as well. Well thought cancellation mechanism also gives space to clean up resources.

In this blog post we'll take see how to cancel coroutine and impact of suspend functions on it.

## Basics

The [`Job`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/index.html){:target="blank"} interface has a method called [`cancel`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/cancel.html){:target="blank"}, which allows to cancel the job.

```kotlin
fun main() = runBlocking {
  val job = launch {
    repeat(1_00) { i ->
      delay(200)
      println("printing $i")
    }
  }

  delay(1_150)
  job.cancelAndJoin()
  println("cancelled successfully")
}
// printing 0
// printing 1
// printing 2
// printing 3
// printing 4
// cancelled successfully
```

Calling cancel has following effects on job.

- ends execution job at first suspension point
- if job has some children, they are also canceled at first suspension point
- Once job is canceled, it can not used as a parent for any new coroutines.

Lets verify our first assumption by replacing `delay` call by `Thread.sleep`

```kotlin
fun main() = runBlocking {
  val job = launch {
    repeat(1_00) { i ->
      // delay(200)
      Thread.sleep(200)
      println("printing $i")
    }
  }

  delay(1_150)
  job.cancelAndJoin()
  println("cancelled successfully")
}
// what will be the outcome?

// outcome:
// printing 0
// printing ..
// printing 99
// cancelled successfully
```

## Thread blocking code

Once we removed delay function job has no suspension point and hence it continues to work until the end of computation. `Thread.sleep` is thread blocking function
let's extract repeat code block into suspended function.

```kotlin
fun main() = runBlocking {
    val job = launch {
        repeat(1_00) {
          threadBlockingFn(it)
        }
        println("job completed")
    }
    delay(1_150)
    job.cancelAndJoin()
    println("cancelled successfully")
}

suspend fun threadBlockingFn(i: Int) = coroutineScope {
    Thread.sleep(200)
    println("printing $i")
}

fun println(msg: Any) = with(Thread.currentThread()) {
    kotlin.io.println("$id:$name\t=> $msg")
} 
// what will be the outcome?

// outcome:
// 1:main	=> printing 0
// 1:main	=> printing .. 🧐
// 1:main	=> printing 99 🤔
// 1:main	=> job completed
// 1:main	=> cancelled successfully
```

`job` is running on single thread and even we have suspended function `threadBlockingFn` `main` thread does not get unblocked to observe cancellation request. So *how to create suspension point in this case?*

In order to have more fine control, lets create coroutine for every call of `threadBlockFn` and join it back with parent job

```kotlin
fun main() = runBlocking {
    val job = launch {
        repeat(1_00) {
          launch { threadBlockingFn(it) }.join()
        }
        // also produces same output
        // repeat(1_00) { async { threadBlockingFn(it) }.await()   }
        println("job completed")
    }
    delay(1_150)
    job.cancelAndJoin()
    println("cancelled successfully")
}
// output: 
// 1:main	=> printing 0
// 1:main	=> printing 1
// 1:main	=> printing 2
// 1:main	=> printing 3
// 1:main	=> printing 4
// 1:main	=> printing 5
// 1:main	=> cancelled successfully
```

Lets use default dispatcher to delegate `threadBlockFn` execution

```kotlin
fun main() = runBlocking {
    val job = launch {
        repeat(1_00) { 
          withContext(Dispatchers.Default) { 
            threadBlockingFn(it) 
          } 
        }
        println("job completed")
    }
    delay(1_150)
    job.cancelAndJoin()
    println("cancelled successfully")
}
// output: 
// 14:DefaultDispatcher-worker-1	=> printing 0
// 14:DefaultDispatcher-worker-1	=> printing 1
// 14:DefaultDispatcher-worker-1	=> printing 2
// 14:DefaultDispatcher-worker-1	=> printing 3
// 14:DefaultDispatcher-worker-1	=> printing 4
// 14:DefaultDispatcher-worker-1	=> printing 5
// 1:main	=> cancelled successfully
```

### Summary

Coroutine, dispatcher along with suspended functions provides very powerful mechanism to have full control over execution of code blocks. Default function such as `delay` are cancel aware functions.
To summarize, I would say use following thumb rules to achieve fine controlled code

1. Have more suspension points in code base
2. Avoid thread blocking code areas
3. coroutines are very lightweight, use them more frequently
4. use `async/await`, `withContext` around appropriate spaces
5. Break down computation heavy operation into smaller pieces with more suspension points
