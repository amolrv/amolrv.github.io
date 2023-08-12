---
categories: [Programming]
title: Clear and honest code
date: 2023-08-12
tags: [👍rules]
---

Often when you're working on large code base, we don't remember how particular code block (function or method or class) is implemented. However signature is suggested by IDE. When you call a code block, sometimes it can throws nasty surprises at you. This can be avoided by writing clear and honest code blocks.

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

Other helpful technics
1. Treat Type as set Read [here](https://blog.ploeh.dk/2021/11/15/types-as-sets/) and [here](https://guide.elm-lang.org/appendix/types_as_sets.html)
