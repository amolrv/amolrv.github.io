---
categories: [Programming]
title: Code that changes together, stays together
date: 2022-08-02
tags: [Architecture, 👍rules]
---

In traditional clean/layered/onion architecture, code is organized in layers, and so are the abstractions such as controllers, services, and repositories.

![Clean Architecture](/assets/blog/clean-architecture.jpeg)

What I have seen most people do is try to organize code per layer by structuring code around those layers.
For example:

```yaml
- src
  - controllers
  - services
  - use case
  - entities
  - repositories
```

In some projects, where I see teams wanting to build a modular monolith, they start to group based on some heuristic, thinking these pieces go together and probably should be modularized.

```yaml
- src
  - module#1
  - controllers
  - services
  - use case
  - entities
  - repositories
  - module#2
  - controllers
  - services
  - use case
  - entities
  - repositories
```

I consider this way of modularizing the codebase as premature, as it forces you to make decisions when you have the least knowledge about the system. Sometimes, this kind of modularizing feels so artificial.

So I was wondering 🤔

- How could we defer the decision of modularizing `code structure` as much as possible?
- How can I let `code structure` evolve organically?

There was another problem: If I wanted to make any changes, I had to make changes in 4-5 different packages such as controller, services, use case. In a lot of cases, I felt that it's a lot of ceremony and dancing around so many files. There was an urge to keep things together.

Suddenly something clicked:

> _"Neurons that fire together wire together" or "Family that eats together, stays together"_
{: .prompt-info }

which leads to

> **Code that changes together, stays together**
{: .prompt-tip }

![vertical slice](/assets/blog/vertical-slice.png)

As I moved code that was changing together closer, two things happened:

1. Code that came closer increased coupling between them (on the vertical axis).
2. Coupling between different slices started to drop.

After spending more time with this approach, I learned that code that's shared across slices bubbled up and moved into the core of the feature or is a domain concept—or it's just a cross-cutting concern, which then could move up and find its own place not just in the codebase but also conceptually.

In summary, this approach provided me with the following flexibilities:

- [x] Code structure can evolve as organically as our understanding of the domain concepts.
- [x] Provides a way to be more pragmatic per slice.
- [x] Provides a way of grouping that can scale with growing feature sets.

---

The final code structure was similar to 👇🏼

```yaml
- src
  - todo
  - api
  - dto
  - controller
  - find-todos
  - complete-todo
  - new-todo
  - todo
  - notes
  - api
  - dto
  - controller
  - new-notes
  - recent-notes
  - note
```
