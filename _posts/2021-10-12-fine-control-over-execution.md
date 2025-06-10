---
categories: [Programming]
title: Fine control over execution in Kotlin
date: 2021-10-12 10:00:00
tags: [kotlin, concurrency and parallelism]
image:
  path: /assets/blog/2021-10-12-fine-control-over-execution.jpg
  src: /assets/blog/2021-10-12-fine-control-over-execution.jpg
  w: 800
  h: 500
---

One of Kotlin's most powerful features is its approach to concurrent programming through coroutines. At the heart of this system lies a concept called "cooperative cancellation" - a mechanism that allows coroutines to work together efficiently while maintaining the ability to be cancelled gracefully. In this post, we'll explore how suspension points enable this cooperation and how they affect your code's cancellation behavior.

## Understanding Suspension Points

A suspension point in Kotlin is a point in your code where a coroutine can:

1. **Check for cancellation**: Verify if it should stop execution
2. **Release the thread**: Allow other coroutines to run
3. **Resume later**: Continue execution when resources are available

Think of suspension points as "cooperation checkpoints" where your coroutine says, "I'm at a good stopping point. Does anyone need me to pause or stop?"

## The Basics: Jobs and Cancellation

The [`Job`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/index.html){:target="blank"} interface is central to Kotlin's cancellation system. It provides a [`cancel`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/cancel.html){:target="blank"} method that initiates the cancellation process. Here's a real-world example:

```kotlin
class UserRepository(
    private val api: UserApi,
    private val database: UserDatabase,
    private val scope: CoroutineScope
) {
    private var syncJob: Job? = null

    fun startSync() {
        // Cancel any existing sync
        syncJob?.cancel()
        
        syncJob = scope.launch {
            try {
                while (true) {
                    println("Starting sync cycle...")
                    
                    val users = withContext(Dispatchers.IO) {
                        api.fetchUsers() // Suspension point - network call
                    }
                    
                    users.forEach { user ->
                        // Yield periodically to cooperate with cancellation
                        yield()
                        database.updateUser(user)
                    }
                    
                    println("Sync completed")
                    delay(60_000) // Wait for 1 minute before next sync
                }
            } catch (e: CancellationException) {
                println("Sync was cancelled")
                throw e
            } finally {
                println("Cleaning up sync resources")
            }
        }
    }

    fun stopSync() {
        syncJob?.cancel()
    }
}

// Usage
fun main() = runBlocking {
    val repository = UserRepository(mockApi(), mockDatabase(), this)
    
    repository.startSync()
    delay(3000) // Let it sync for 3 seconds
    
    repository.stopSync()
    println("Sync stopped")
}
```

When we call cancel on a job, three key things happen:

1. The job is marked for cancellation
2. At the next suspension point, the job checks this cancellation flag and stops execution
3. Any child coroutines follow the same process at their next suspension points
4. The cancelled job becomes unusable as a parent for new coroutines

Let's look at a common mistake when dealing with cancellation - blocking operations:

```kotlin
class ImageProcessor(private val scope: CoroutineScope) {
    private var processingJob: Job? = null

    fun processImages(images: List<Image>) {
        processingJob?.cancel()
        
        processingJob = scope.launch {
            try {
                images.forEach { image ->
                    println("Processing image: ${image.name}")
                    
                    // BAD: This blocks the thread and ignores cancellation
                    Thread.sleep(100)
                    image.applyFilter()
                    
                    // GOOD: This cooperates with cancellation
                    // delay(100)
                    // ensureActive() // Explicit cancellation check
                    // image.applyFilter()
                    
                    println("Image processed: ${image.name}")
                }
            } catch (e: CancellationException) {
                println("Image processing cancelled")
                throw e
            }
        }
    }
}

// Usage showing the difference
fun main() = runBlocking {
    val processor = ImageProcessor(this)
    val images = List(100) { Image("IMG_$it.jpg") }
    
    processor.processImages(images)
    delay(250) // Let it process a few images
    
    processor.processingJob?.cancel()
    println("Cancellation requested")
}

// With Thread.sleep: Continues processing all images
// With delay: Stops after ~2-3 images
```

## Handling Long-Running Operations

Here's a real-world example of how to handle long-running operations with proper resource management:

```kotlin
class FileUploader(
    private val api: FileApi,
    private val scope: CoroutineScope
) {
    private val activeUploads = mutableMapOf<String, Job>()

    fun startUpload(fileId: String, file: File) {
        // Cancel existing upload if any
        activeUploads[fileId]?.cancel()
        
        activeUploads[fileId] = scope.launch {
            try {
                val fileStream = withContext(Dispatchers.IO) {
                    FileInputStream(file)
                }
                
                fileStream.use { stream ->
                    val buffer = ByteArray(8192)
                    var bytesRead: Int
                    var totalBytes = 0L
                    
                    while (stream.read(buffer).also { bytesRead = it } != -1) {
                        ensureActive() // Check for cancellation
                        
                        withContext(Dispatchers.IO) {
                            api.uploadChunk(fileId, buffer, bytesRead)
                        }
                        
                        totalBytes += bytesRead
                        println("Uploaded $totalBytes bytes")
                        
                        yield() // Cooperate with other coroutines
                    }
                }
                
                println("Upload completed for $fileId")
            } catch (e: CancellationException) {
                println("Upload cancelled for $fileId")
                throw e
            } finally {
                activeUploads.remove(fileId)
                println("Cleaned up upload resources for $fileId")
            }
        }
    }

    fun cancelUpload(fileId: String) {
        activeUploads[fileId]?.cancel()
    }
}
```

### Summary

Through these real-world examples, we've learned that effective coroutine cancellation relies on proper suspension points. Here are the key principles for writing cancellation-aware coroutines:

1. Use suspend functions that create real suspension points (`delay`, `yield`, `withContext`, etc.)
2. Avoid thread-blocking operations that prevent cooperation
3. Properly handle resources with try-finally blocks
4. Check cancellation status regularly with `ensureActive()`
5. Use structured concurrency with parent-child job relationships
6. Consider using supervisorScope for independent child failures
7. Always clean up resources in finally blocks

By following these guidelines, you'll create more robust and maintainable concurrent applications that can handle cancellation gracefully.
