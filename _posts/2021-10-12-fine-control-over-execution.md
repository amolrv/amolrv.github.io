---
categories: [Programming]
title: Fine control over execution in kotlin
date: 2021-10-12 10:00:00
tags: [kotlin, concurrency and parallelism]
image:
  path: /assets/blog/2021-10-12-fine-control-over-execution.jpg
  src: /assets/blog/2021-10-12-fine-control-over-execution.jpg
  w: 800
  h: 500
---
Programming languages that support concurrency and parallelism need to provide a cancellation mechanism as well.
A well-thought-out cancellation mechanism also gives space to clean up resources.

In this blog post, we'll see how to cancel coroutines and the impact of suspend functions on them.

## Basics

The [`Job`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/index.html){:target="blank"} interface has a method called [`cancel`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/cancel.html){:target="blank"}, which allows you to cancel the job.

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

Calling cancel has the following effects on the job:

- Ends execution of the job at the first suspension point
- If the job has children, they are also canceled at the first suspension point
- Once a job is canceled, it cannot be used as a parent for any new coroutines.

Let's verify our first assumption by replacing the `delay` call with `Thread.sleep`:

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

Once we remove the delay function, the job has no suspension point and hence continues to work until the end of computation. `Thread.sleep` is a thread-blocking function.
Let's extract the repeat code block into a suspended function.

```kotlin
fun main() = runBlocking {
    val job = launch {
        repeat(100) {
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
// 1:main => printing 0
// 1:main => printing .. 🧐
// 1:main => printing 99 🤔
// 1:main => job completed
// 1:main => cancelled successfully
```

The `job` is running on a single thread, and even though we have a suspended function `threadBlockingFn`, the `main` thread does not get unblocked to observe the cancellation request. So, how do we create a suspension point in this case?

In order to have more fine control, let's create a coroutine for every call of `threadBlockFn` and join it back with the parent job:

```kotlin
fun main() = runBlocking {
    val job = launch {
        repeat(1_00) {
          launch { threadBlockingFn(it) }.join()
        }
        // also produces the same output
        // repeat(1_00) { async { threadBlockingFn(it) }.await()   }
        println("job completed")
    }
    delay(1_150)
    job.cancelAndJoin()
    println("cancelled successfully")
}
// output:
// 1:main => printing 0
// 1:main => printing 1
// 1:main => printing 2
// 1:main => printing 3
// 1:main => printing 4
// 1:main => printing 5
// 1:main => cancelled successfully
```

Let's use the default dispatcher to delegate `threadBlockFn` execution:

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
// 14:DefaultDispatcher-worker-1 => printing 0
// 14:DefaultDispatcher-worker-1 => printing 1
// 14:DefaultDispatcher-worker-1 => printing 2
// 14:DefaultDispatcher-worker-1 => printing 3
// 14:DefaultDispatcher-worker-1 => printing 4
// 14:DefaultDispatcher-worker-1 => printing 5
// 1:main => cancelled successfully
```

### Summary

Coroutines, dispatchers, and suspended functions provide a very powerful mechanism to have full control over the execution of code blocks.
Default functions such as `delay` are cancel-aware functions.
To summarize, I would suggest the following thumb rules to achieve fine control over code execution:

1. Have more suspension points in your codebase
2. Avoid thread-blocking code areas
3. Coroutines are very lightweight; use them more frequently
4. Use `async/await`, `withContext` in appropriate places
5. Break down computation-heavy operations into smaller pieces with more suspension points
