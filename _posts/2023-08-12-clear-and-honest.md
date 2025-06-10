---
categories: [Programming]
title: Clear and honest code
date: 2023-08-12
tags: [👍rules]
---

Every software developer has encountered this scenario: you're working in a large codebase, and you need to use a function. The IDE shows you its signature, it looks straightforward, but when you use it, surprises await. This is where the importance of honest and clear code becomes evident.

Consider this example:

```kotlin
// (account: Account, amount: BigDecimal) -> Account
fun credit(account: Account, amount: BigDecimal): Account {
  if (amount < ZERO) throw NegativeAmountError(amount)
  return account.copy(balance = account.balance + amount)
}
```

The function signature appears simple, but it hides a critical detail - it can throw an exception. This lack of transparency creates two significant problems:

1. **Cognitive Overhead**: Developers must dive into the implementation to understand the complete behavior
2. **Trust Issues**: When signatures don't tell the whole story, developers become wary of using any code without thorough investigation

Here are two alternative approaches that make the function's behavior explicit:

```kotlin
(account: Account, amount: NonNegativeCurrency) -> Account
// or
(account: Account, amount: BigDecimal) -> Either<NegativeAmountError, Account>
```

Both signatures are more honest, but they use different strategies:

- The first version uses type constraints to prevent invalid states entirely
- The second version explicitly communicates possible outcomes through the return type

By making our code's behavior explicit in its signature, we create more maintainable and trustworthy systems.

References

1. Treat Type as set ([Types as Sets by Mark Seemann](https://blog.ploeh.dk/2021/11/15/types-as-sets/){:target="_blank"} and [Elm Guide: Types as Sets](https://guide.elm-lang.org/appendix/types_as_sets.html){:target="_blank"})
2. [Algebraic data type](https://en.wikipedia.org/wiki/Algebraic_data_type){:target="_blank"}
