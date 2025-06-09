---
categories: [Programming]
title: Modelling value types in kotlin
date: 2020-11-04 00:00:00
tags: [kotlin, domain modeling]
---

In this article, you'll see how to model value objects, particularly in Kotlin.

Imagine we want to improve the following model:

```kotlin
class Customer(id: UUID, email: String, ...)

class Order(id: UUID, ...)
```

I had two reasons for modeling value types:

## Type Safety

Many times, I want to distinguish between, let's say, `customerId` and `orderId` so that accidentally passing the wrong ID can be avoided. The best way is to use the inline class [^2] concept, but be careful with its stability status.

```kotlin
// option #1
inline class CustomerId(val value: UUID)
// option #2
data class CustomerId(val value: UUID)
// option #3
data class CustomerId(private val value: UUID) {
    override fun toString(): String = value.toString()
    fun asString(): String = value.toString()
    fun asUUID(): UUID = value
}
```

Options 1 and 2 are more recommended, whereas option #3 is an example of over-modeling.

> I often have a tendency to make properties private by default, but when we're modeling immutable data types,
> you don't need to mark properties as private.

## Representing a Domain Concept with Constraints

Let's take the example of `email`, which needs to match some fancy regex.

This is a bit more complicated, especially with Kotlin because of language design.

But we have three options here:

1. Use the constructor to throw violations 😟

   ```kotlin
   data class Email(val value: String) {
       init {
           // if (!isValid(value))
           // then throw InvalidEmail()
       }
   }
   ```

   This option is not bad, but it has a few limitations, such as:

   - The constructor is not honest enough because it can blow up unexpectedly.
   - It can hijack your program’s execution.

2. Don't use Kotlin due to a well-known design flaw [^3] 😅
3. [Use the NoCopy plugin 🎯](https://github.com/AhmedMourad0/no-copy#nocopy-compiler-plugin----){:target="_blank"}
   For more details, see the article "Value-based Classes and Error Handling in Kotlin" on Medium: [Value-based Classes and Error Handling in Kotlin](https://medium.com/swlh/value-based-classes-and-error-handling-in-kotlin-3f14727c0565){:target="_blank"}

## Summary

Kotlin has a few limitations, but those can be worked around. Always go for a more transparent and honest representation.

[^1]: <https://martinfowler.com/bliki/ValueObject.html> "Value object"
[^2]: <https://kotlinlang.org/docs/reference/evolution/components-stability.html#current-stability-of-kotlin-components> "inline class is at Alpha stability level"
[^3]: <https://youtrack.jetbrains.com/issue/KT-11914> "Confusing data class copy with private constructor"
