---
categories: [Programming]
title: Clear and honest code
date: 2023-08-12
tags: [👍rules]
---

When working with a huge code base, it's common to forget how a specific code block (function, method, or class) is implemented. However, the IDE suggests using a signature. Sometimes calling a code block can come with unpleasant shocks. By creating honest and clear code blocks, this can be prevented.

For example:

```kotlin
// (account: Account, amount : BigDecimal) -> Account
fun credit(account: Account, amount : BigDecimal): Account {
  if(amount < ZERO) throw NegativeAmountError(amount)
  return account.copy(balance = account.balance + amount)
}
```

By looking at the signature, caller simply does't know that it's going to blow up. This kind of code blocks affect engineer in 2 ways
1. Increase cognitive overload of reading and understanding each branch of the code block before calling it
2. hampers trust of code block.

Honest signature could have been something like
```kotlin
(account: Account, amount : NonNegativeCurrency) -> Account
// or
(account: Account, amount : BigDecimal) -> Either<NegativeAmountError, Account>
```

These 2 signatures appear quite straightforward, yet they use 2 different techniques.

- The first signature prevents entry into an incorrect state by forbidding negative input.
- While allowing erroneous input, the second signature communicates output more precisely. It informs the caller that it may provide an error or credit to their account.

Other helpful technics
1. Treat Type as set Read [here](https://blog.ploeh.dk/2021/11/15/types-as-sets/) and [here](https://guide.elm-lang.org/appendix/types_as_sets.html)
