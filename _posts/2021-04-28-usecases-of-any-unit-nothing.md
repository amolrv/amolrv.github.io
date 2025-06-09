---
categories: [Programming]
title: Any, Unit and Nothing
date: 2021-04-28
tags: [kotlin, domain modeling]
---

This article explains the basic differences between Any, Unit, and Nothing in the Kotlin language.

## Any

`Any` is by default the superclass of all classes and has three functions: `equals`, `hashCode`, and `toString`. This is equivalent to the `Object` class in Java.
You can create an object of the `Any` class directly or override these functions in any other class.

```kotlin
public open class Any {
     public open operator fun equals(other: Any?): Boolean
     public open fun hashCode(): Int
     public open fun toString(): String
}
```

## Unit

The Unit class is a singleton and also an object. You can't extend it or create another object of it.
The Unit class is equivalent to the void type in Java.
The superclass of Unit is Any, and it has an overridden toString method.

```kotlin
public object Unit {
    override fun toString() = "kotlin.Unit"
}
```

## Nothing

Nothing is a non-open (final) class that can't be extended, and its constructor is also private, which means you can't create an object of it.
This is usually used to represent the return type of a function that will always throw an exception.
The superclass of Nothing is Any.

```kotlin
public class Nothing private constructor()
```
