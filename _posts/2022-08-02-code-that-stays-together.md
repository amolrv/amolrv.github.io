---
categories: [Programming]
title: Code that changes together, stays together
date: 2022-08-02
tags: [Architecture, 👍rules]
---

In traditional clean/layered/onion architecture code is organized in layers and so as the abstractions such as controllers, services, repositories.

![Clean Architecture](/assets/blog/clean-architecture.jpeg)

What I have seen most of people do is tried to organize code per layer by structuring code around those layer.
For example

```yaml
- src
 - controllers
 - services
 - use case
 - entities
 - repositories
```

In some projects, where I see team wanted to build modular monolith, they started to group based on some heuristic, that these pieces goes together probably we should modularize it.

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

I consider this way of modularizing code base as premature as it suffers from forcing you to make decision when you the least knowledge about the system. Sometimes this kind of modularizing felt so artificial.

So I was wondering 🤔

- how we could defer decision of making modularizing `code structure` as much as I can?
- How can I let `code structure` evolve itself organically?

There was another problem, If I want to make any changes, I have to make changes in upon 4-5 different packages such as controller, services, use case. In lot of cases I felt that it's lot of ceremony and dancing around so many files. There was urge of keeping things together.

Suddenly something clicked

> *"Neurons that fire together wire together" or "Family thats eats together, stays together"*
{: .prompt-info }

which leads to

> ***Code** thats **changes** together, stays together*
{: .prompt-tip }

![vertical slice](/assets/blog/vertical-slice.png)

![vertical slice](/assets/blog/vertical-slice-detail.png)

As I move code which was changing together closer, 2 things happened

1. Code thats was coming closer, coupling between them increased(on vertical access).
2. Coupling between different slices started to drop

After spending more time with this approach, I learned that code thats shared across slices bubbled up and moved into core of the feature or is a domain concept. or it's just cross cutting concern. which then could moved up and finds it own place not just in code base but also conceptually.

In summary this approach provide me following flexibilities

- [x] Code structure can evolve as organically as with our understanding of the domain concepts.
- [x] Provide a way be more pragmatic per slice.
- [x] Provide way of grouping which can scale with growing feature sets.

---

Final code structure was similar to 👇🏼

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
