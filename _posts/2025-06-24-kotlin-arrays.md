---
categories: [Programming, Data Structures and Algorithms]
title: Kotlin Arrays - Fundamentals and Operations
date: 2025-06-24
tags: [kotlin, data-structures, algorithms, arrays]
---

Arrays are among the most fundamental data structures in programming. In this article, we'll explore Kotlin's array implementations, operations, efficiency considerations, and practical use cases.

## Creating and Using Arrays in Kotlin

Kotlin offers several ways to create arrays, each suited for different scenarios:

```kotlin
// Creating arrays of primitive types
val intArray = IntArray(5) // [0, 0, 0, 0, 0]
val intArrayWithValues = intArrayOf(1, 2, 3, 4, 5)

// Creating arrays of objects
val stringArray = Array(3) { "" } // ["", "", ""]
val names = arrayOf("Alice", "Bob", "Charlie")

// Creating arrays with a lambda initializer
val squaresArray = IntArray(10) { i -> i * i }
println(squaresArray.joinToString()) // 0, 1, 4, 9, 16, 25, 36, 49, 64, 81

// Creating arrays of nullable types
val nullableArray = arrayOfNulls<String>(3) // [null, null, null]
```

### Static Allocation and Memory Layout

Unlike many other collections in Kotlin, arrays have a fixed size that must be defined at creation time. This static allocation offers several advantages:

1. **Predictable Memory Usage**: The memory needed is allocated upfront
2. **Contiguous Memory**: Elements are stored in adjacent memory locations
3. **Direct Indexing**: O(1) access to any element by index

```kotlin
// Memory representation of an IntArray (conceptual)
// [32 bits][32 bits][32 bits][32 bits][32 bits]
//    ^       ^        ^        ^        ^
//  arr[0]   arr[1]   arr[2]   arr[3]   arr[4]
```

## Array Access and Manipulation

The primary strength of arrays is their constant-time access to elements:

```kotlin
val numbers = intArrayOf(10, 20, 30, 40, 50)

// Accessing elements - O(1)
val thirdElement = numbers[2] // 30

// Modifying elements - O(1)
numbers[1] = 25
println(numbers.joinToString()) // 10, 25, 30, 40, 50

// Iterating - O(n)
for (number in numbers) {
    println(number)
}

// Using indexing in iteration
for (i in numbers.indices) {
    println("Index $i: ${numbers[i]}")
}

// Using destructuring with withIndex()
for ((index, value) in numbers.withIndex()) {
    println("Index $index: $value")
}
```

### Array Transformation Operations

Kotlin provides rich functionality for working with arrays:

```kotlin
val numbers = intArrayOf(1, 2, 3, 4, 5)

// Mapping - O(n)
val doubled = numbers.map { it * 2 }
println(doubled) // [2, 4, 6, 8, 10]

// Filtering - O(n)
val evenNumbers = numbers.filter { it % 2 == 0 }
println(evenNumbers) // [2, 4]

// Reducing - O(n)
val sum = numbers.reduce { acc, i -> acc + i }
println(sum) // 15

// Finding elements - O(n)
val firstEven = numbers.first { it % 2 == 0 }
println(firstEven) // 2

// Checking conditions - O(n)
val allPositive = numbers.all { it > 0 }
println(allPositive) // true

// Sorting - O(n log n)
val unsortedArray = intArrayOf(5, 2, 9, 1, 7)
unsortedArray.sort()
println(unsortedArray.joinToString()) // 1, 2, 5, 7, 9

// Reverse sorting
val descendingArray = intArrayOf(5, 2, 9, 1, 7)
descendingArray.sortDescending()
println(descendingArray.joinToString()) // 9, 7, 5, 2, 1
```

## Multi-dimensional Arrays

Kotlin supports multi-dimensional arrays, which are essentially arrays of arrays:

```kotlin
// Creating a 3x3 matrix
val matrix = Array(3) { IntArray(3) { 0 } }

// Initializing with values
for (i in matrix.indices) {
    for (j in matrix[i].indices) {
        matrix[i][j] = i * 3 + j + 1
    }
}

// Printing the matrix
for (row in matrix) {
    println(row.joinToString())
}
// Output:
// 1, 2, 3
// 4, 5, 6
// 7, 8, 9

// Accessing elements in 2D array - O(1)
val element = matrix[1][2] // 6

// Creating arrays with different row lengths (jagged arrays)
val jaggedArray = Array(3) { i ->
    IntArray(i + 1) { it * it }
}

// Results in:
// [0]
// [0, 1]
// [0, 1, 4]
for (row in jaggedArray) {
    println(row.joinToString())
}
```

## Binary Search and Sorted Arrays

When arrays are sorted, we can perform binary search for efficient lookups:

```kotlin
val sortedArray = intArrayOf(1, 3, 5, 7, 9, 11, 13, 15)

// Binary search - O(log n)
val index = sortedArray.binarySearch(7)
println("Found 7 at index: $index") // 3

// Element not in array
val notFoundIndex = sortedArray.binarySearch(8)
println("8 would be inserted at: ${-notFoundIndex - 1}") // 4

// Binary search with custom comparator
data class Person(val name: String, val age: Int)

val people = arrayOf(
    Person("Alice", 25),
    Person("Bob", 30),
    Person("Charlie", 35),
    Person("David", 40)
)

// Search by age
val ageComparator = compareBy<Person> { it.age }
val personIndex = people.binarySearch(Person("", 35), ageComparator)
println("Found person with age 35 at index: $personIndex") // 2
```

## Array Efficiency and Limitations

While arrays provide efficient access, they have important limitations:

### 1. Fixed Size

Once created, an array's size cannot change:

```kotlin
// To add elements, you must create a new array
fun addElement(array: IntArray, element: Int): IntArray {
    val newArray = IntArray(array.size + 1)
    array.copyInto(newArray)
    newArray[array.size] = element
    return newArray
}

val original = intArrayOf(1, 2, 3)
val expanded = addElement(original, 4)
println(expanded.joinToString()) // 1, 2, 3, 4
```

### 2. Inefficient Insertions and Deletions

Inserting or deleting elements in the middle requires shifting elements:

```kotlin
// Insertion into an array (without built-in functions)
fun insertIntoArray(array: IntArray, position: Int, value: Int): IntArray {
    // Create a new array with one extra space
    val result = IntArray(array.size + 1)

    // Copy elements before insertion point
    for (i in 0 until position) {
        result[i] = array[i]
    }

    // Insert the new value
    result[position] = value

    // Copy elements after insertion point
    for (i in position until array.size) {
        result[i + 1] = array[i]
    }

    return result
}

val original = intArrayOf(1, 2, 4, 5)
val expanded = insertIntoArray(original, 2, 3)
println(expanded.joinToString()) // 1, 2, 3, 4, 5
```

This operation creates an entirely new array and has O(n) time complexity.

### 3. Memory Usage

Arrays allocate all their space upfront, which can be inefficient if the exact size needed is unknown:

```kotlin
// If we only use a fraction of the allocated space
val potentiallyWasteful = IntArray(1000) // Allocates space for 1000 elements
var actualUsed = 0

// Only fill a small portion
for (i in 1..10) {
    potentiallyWasteful[actualUsed++] = i
}

// 990 slots remain unused but still consume memory
```

## Specialized Array Types in Kotlin

Kotlin provides specialized array implementations for primitive types to improve performance:

```kotlin
// Primitive type arrays
val byteArray = ByteArray(5)
val shortArray = ShortArray(5)
val intArray = IntArray(5)
val longArray = LongArray(5)
val floatArray = FloatArray(5)
val doubleArray = DoubleArray(5)
val booleanArray = BooleanArray(5)
val charArray = CharArray(5)

// Performance comparison with boxing/unboxing
fun comparePerformance() {
    val size = 10_000_000

    // Primitive int array (no boxing)
    val intArray = IntArray(size)
    val intArrayStart = System.nanoTime()
    for (i in 0 until size) {
        intArray[i] = i * 2
    }
    val intArrayTime = System.nanoTime() - intArrayStart

    // Object Integer array (requires boxing)
    val boxedArray = Array(size) { 0 }
    val boxedArrayStart = System.nanoTime()
    for (i in 0 until size) {
        boxedArray[i] = i * 2
    }
    val boxedArrayTime = System.nanoTime() - boxedArrayStart

    println("IntArray time: ${intArrayTime / 1_000_000} ms")
    println("Boxed Array time: ${boxedArrayTime / 1_000_000} ms")
    println("Overhead: ${boxedArrayTime.toDouble() / intArrayTime}x")
}
```

## Practical Applications of Arrays

### 1. Sliding Window Algorithm

A common technique for array processing is the sliding window approach:

```kotlin
// Find maximum sum subarray of size k
fun maxSubarraySum(array: IntArray, k: Int): Int {
    if (array.size < k) throw IllegalArgumentException("Array too small")

    // Compute sum of first window
    var windowSum = 0
    for (i in 0 until k) {
        windowSum += array[i]
    }

    var maxSum = windowSum

    // Slide the window
    for (i in k until array.size) {
        // Add next element and remove first element of previous window
        windowSum = windowSum + array[i] - array[i - k]
        maxSum = maxOf(maxSum, windowSum)
    }

    return maxSum
}

val array = intArrayOf(1, 4, 2, 10, 2, 3, 1, 0, 20)
val k = 4
println("Maximum sum of $k consecutive elements: ${maxSubarraySum(array, k)}") // 24
```

### 2. Two-Pointer Technique

The two-pointer technique is useful for problems involving arrays:

```kotlin
// Find if an array has a pair with a given sum
fun hasPairWithSum(array: IntArray, targetSum: Int): Boolean {
    val sorted = array.sorted()
    var left = 0
    var right = sorted.lastIndex

    while (left < right) {
        val currentSum = sorted[left] + sorted[right]

        when {
            currentSum == targetSum -> return true
            currentSum < targetSum -> left++
            else -> right--
        }
    }

    return false
}

val numbers = intArrayOf(8, 4, 2, 1, 15, 10, 5)
val target = 12
println("Array has pair with sum $target: ${hasPairWithSum(numbers, target)}") // true (2+10)
```

### 3. Prefix Sums

Prefix sums enable efficient range queries:

```kotlin
// Calculate prefix sums
fun calculatePrefixSums(array: IntArray): IntArray {
    val prefixSums = IntArray(array.size)
    prefixSums[0] = array[0]

    for (i in 1 until array.size) {
        prefixSums[i] = prefixSums[i - 1] + array[i]
    }

    return prefixSums
}

// Query sum in range [start, end] inclusive
fun rangeSum(prefixSums: IntArray, start: Int, end: Int): Int {
    return if (start == 0) {
        prefixSums[end]
    } else {
        prefixSums[end] - prefixSums[start - 1]
    }
}

val array = intArrayOf(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
val prefixSums = calculatePrefixSums(array)

println("Sum of range [2, 5]: ${rangeSum(prefixSums, 2, 5)}") // 3+4+5+6 = 18
println("Sum of range [0, 3]: ${rangeSum(prefixSums, 0, 3)}") // 1+2+3+4 = 10
```

## Conclusion

Arrays in Kotlin provide efficient random access and straightforward memory layout, making them ideal for scenarios where:

1. The collection size is known in advance
2. Direct access by index is frequent
3. Performance and memory efficiency are critical
4. Working with primitive types (using specialized array implementations)

However, their fixed-size nature makes them less suitable for collections that need to grow or shrink dynamically. In those cases, dynamic data structures like ArrayList (which we'll cover in the next article) may be more appropriate.

Remember that choosing the right data structure involves understanding the specific requirements of your application and the trade-offs between different options.
