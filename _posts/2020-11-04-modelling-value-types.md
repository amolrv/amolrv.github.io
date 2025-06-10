---
categories: [Programming]
title: Modelling value types in kotlin
date: 2020-11-04 00:00:00
tags: [kotlin, domain modeling]
---

In this article, I'll show you how to effectively model value objects in Kotlin, addressing two key challenges I've encountered in domain modeling.

Let's start with a common scenario. Consider this basic model that we want to improve:

```kotlin
class Customer(id: UUID, email: String, ...)

class Order(id: UUID, ...)
```

I had two main motivations for implementing value types in my codebase:

## Type Safety

One of the most common issues I've faced is the need to distinguish between similar IDs, such as `customerId` and `orderId`. Without proper typing, it's easy to accidentally pass the wrong ID to a function. The most effective solution is to use Kotlin's value class feature (formerly known as inline classes) [^2]. As of Kotlin 1.5+, the `@JvmInline value class` is the stable way to define these types.

```kotlin
// option #1 (recommended)
@JvmInline
value class CustomerId(val value: UUID)

// option #2
data class CustomerId(val value: UUID)

// option #3
data class CustomerId(private val value: UUID) {
  override fun toString(): String = value.toString()
  fun asString(): String = value.toString()
  fun asUUID(): UUID = value
}
```

I recommend going with option #1, as it provides the necessary type safety without runtime overhead, since value classes are optimized by the Kotlin compiler. Option #2 is also acceptable for simpler cases, while option #3 demonstrates what I consider over-modeling.

> While I typically default to making properties private, I've learned that when dealing with immutable data types,
> this level of encapsulation isn't necessary.

## Representing a Domain Concept with Constraints

Let's consider email validation as an example. We want to ensure that an email matches specific validation rules (like a regex pattern). This presents some interesting challenges in Kotlin due to its language design.

Here are two approaches I've explored, with the second being my recommended solution:

1. Constructor-based validation 😟

```kotlin
data class Email(val value: String) {
  init {
    // if (!isValid(value))
    // then throw InvalidEmail()
  }
}
```

While this approach works, it has notable drawbacks:

- The constructor isn't transparent about its behavior since it can throw exceptions
- It can disrupt the normal flow of program execution unexpectedly

1. Functional domain modeling approach 🎯

   ```kotlin
   @JvmInline
   value class Email private constructor(val value: String) {
     companion object {
       private val emailRegex = Regex("[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}")
       
       fun create(email: String): Result<Email> =
         when {
           email.isBlank() -> Result.failure(
             IllegalArgumentException("Email cannot be blank")
           )
           !emailRegex.matches(email) -> Result.failure(
             IllegalArgumentException("Invalid email format")
           )
           else -> Result.success(Email(email))
         }
     }
   }
   ```

   This approach has several advantages:
   - Uses Kotlin's `Result` type for explicit error handling
   - Private constructor prevents invalid instances
   - Validation rules are centralized in the companion object
   - No runtime exceptions during normal flow
   - Can be used in a functional pipeline

   Usage example:

   ```kotlin
   fun createUser(emailStr: String) {
     Email.create(emailStr)
       .map { email -> User(email) }
       .fold(
         onSuccess = { user -> saveUser(user) },
         onFailure = { error -> handleError(error) }
       )
   }
   ```

## Summary

While Kotlin has made significant improvements in modeling value types with the stable `value class` feature, some challenges remain when dealing with validation and constraints. From my experience, the key is to prioritize transparency and honesty in your domain model representation. The functional approach to domain modeling provides a clean, type-safe way to handle these challenges while maintaining clear boundaries and explicit error handling.

[^2]: <https://kotlinlang.org/docs/inline-classes.html> "Kotlin value classes documentation"
