---
categories: [tech, design]
title: Modelling value types in kotlin
date: 2020-11-04 00:00:00
tags: [kotlin,design]
---

In this article, you'll see how to model value object particularly in kotlin.

Imagine we want to improve below model:

```kotlin
class Customer(id: UUID, email: String,..)

class Order(id: UUID,..)
```

I had 2 use reason for modelling value type

## Type safety

lots of time I want to distinguish between let's say `customerId` and `orderId`
so that accidental passing of wrong Id can be avoided.
the best way is use inline class [^2] concept but be careful
with it's stability status.

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

Option 1 and 2 are more recommended where as option #3 is example of over modelling.
> I often have tendency to make props as private by default. but when we're modelling immutable data type.
You don't need to mark props as private.

## Represent a domain concept with constraint

Let's take an example of `email` which need to match some fancy regex.

This is bit complicated one especially with kotlin because of language design.

But we have 3 options here:

1. Use constructor to throw violations 😟

    ```kotlin
    data class Email(val value: String) {
        init {
            // if (!isValid(value))
            // then throw InvalidEmail()
        }
    }
    ```

    This option is not bad but it has few limitations and such as
    - constructor is not honest enough because it can blew on your face
    - it can hijack your program’s execution
2. don't use kotlin since well-known design flaw [^3] 😅
3. [Use NoCopy plugin 🎯](https://github.com/AhmedMourad0/no-copy#nocopy-compiler-plugin----)
    There is really nice article about NoCopy [here](https://medium.com/swlh/value-based-classes-and-error-handling-in-kotlin-3f14727c0565)

## Summary

Kotlin has few limitations but those can be fixed. Always go for more transparent and honest representation.

[^1]: <https://martinfowler.com/bliki/ValueObject.html> "Value object"
[^2]: <https://kotlinlang.org/docs/reference/evolution/components-stability.html#current-stability-of-kotlin-components> "inline class is at Alpha stability level"
[^3]: <https://youtrack.jetbrains.com/issue/KT-11914> "Confusing data class copy with private constructor"
