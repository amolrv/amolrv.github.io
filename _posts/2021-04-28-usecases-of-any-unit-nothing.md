---
categories: [blog]
permalink: usecases-of-any-unit-nothing
title: Any, Unit and Nothing
layout: post
date: 2021-04-28 00:00:00
tags: [kotlin]
---

## Any

`Any` is by default the superclass of all the classes and has 3 functions: `equals`, `hashCode` and `toString`. This is equal to `Object` class in Java.
We can create an object of `Any` class directly or even override these functions in any other class.

```kotlin
public open class Any {
     public open operator fun equals(other: Any?): Boolean
     public open fun hashCode(): Int
     public open fun toString(): String
}
```

## Unit

Unit class is a singleton class and also is an object, we can't extend or even create an another object of it.
Unit class in equal to void type in Java.
The superclass of Unit is Any and it has overridden toString method.

```kotlin
public object Unit {
    override fun toString() = "kotlin.Unit"
}
```

## Nothing

Nothing is non-open (final class) which can't be extended and its constructor is also private that means we can't create the object also.
This is usually used to represent the return type of function which will always throw an exception.
The superclass of Nothing is Any.

```kotlin
public class Nothing private constructor()
```
