---
categories: [Programming]
title: Clear and honest code
date: 2023-08-12
tags: [👍rules]
---

When working with a huge code base, it's common to forget how a specific code block (function, method, or class) is implemented. However, the IDE suggests using a signature. Sometimes, calling a code block can come with unpleasant surprises. By creating honest and clear code blocks, this can be prevented.

For example:

```kotlin
// (account: Account, amount: BigDecimal) -> Account
fun credit(account: Account, amount: BigDecimal): Account {
  if (amount < ZERO) throw NegativeAmountError(amount)
  return account.copy(balance = account.balance + amount)
}
```

By looking at the signature, the caller simply doesn't know that it's going to throw an error. This kind of code block affects engineers in two ways:

1. Increases cognitive overload, as the caller has to read and understand the code block before using it.
2. Hampers trust.

An honest signature could have been something like:

```kotlin
(account: Account, amount: NonNegativeCurrency) -> Account
// or
(account: Account, amount: BigDecimal) -> Either<NegativeAmountError, Account>
```

These two signatures appear quite straightforward, yet they use two different techniques.

- The first signature prevents entry into an incorrect state by forbidding negative input.
- While allowing erroneous input, the second signature communicates output more precisely. It informs the caller that it may provide an error or credit to their account.

References

1. Treat Type as set ([Types as Sets by Mark Seemann](https://blog.ploeh.dk/2021/11/15/types-as-sets/){:target="_blank"} and [Elm Guide: Types as Sets](https://guide.elm-lang.org/appendix/types_as_sets.html){:target="_blank"})
2. [Algebraic data type](https://en.wikipedia.org/wiki/Algebraic_data_type){:target="_blank"}
