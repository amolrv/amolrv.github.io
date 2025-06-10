---
categories: [Lessons]
title: Understanding Change to Improve Release Planning
date: 2022-02-15 10:00:00
tags: [product, change]
image:
  path: /assets/blog/aziz-acharki-PUvPZckRnOg-unsplash.jpg
  src: /assets/blog/aziz-acharki-PUvPZckRnOg-unsplash.jpg
  w: 800
  h: 500
---

In this article, I share the lessons and insights I’ve gained about change. Developing a clear understanding of the **nature, impact, and value of change** has given me greater confidence and enabled me to create more effective release plans.

---

## Situation 🧺

Like many of my colleagues, I’ve participated in several _Taskforces_. Recently, however, the taskforce I was part of struggled to develop a solid release plan. We found it challenging to design a plan that could be delivered both _incrementally_ and _iteratively_.

---

## Background 📜

### What Is a Taskforce? 🤔

At [layer](https://golayer.io/about/){:target="_blank"}, we assemble a triplet of _Tech, Product, and Business_ representatives—typically one person from each area—to refine features. Sometimes, the Product Owner steps in to provide business expertise or acts as a proxy for the business. For each epic, team members rotate through these roles, which fosters learning, growth, and helps minimize bias within the company. We call this trio a _Taskforce_.

The taskforce is responsible for:

- Refining features (also known as epics)
- Acting as the main point of contact for all QA during critical periods
- Preparing the release plan

---

## The Picture Remained Unclear 🧠

Despite several brainstorming sessions, we still lacked a clear direction. To break the deadlock, I decided to analyze the epic more critically and explore different approaches.

---

## Our Approach 🤞

I went back to basics and examined the feature from the user’s perspective. For each concept or change we wanted to introduce, I asked myself:

1. What problem does this solve for the user?
2. What new problem could this create for the user?
3. What is the nature of this change?

I categorized each change as one of the following:

- **Breaking change** – Significantly alters or removes an existing concept or feature, affecting user flow
- **New change** – Introduces something completely new to the system; may or may not impact user flow
- **Enhancement** – Improves an existing concept without major disruption; may not affect user flow

After this analysis, I discovered that over 70% of the changes were either enhancements or new features, while about 30% were breaking changes. This exercise alone helped unblock a large portion of features and brought much-needed clarity.

For breaking changes, I took an additional step: I asked the same three questions about the existing feature. This allowed me to clearly see what we were removing and what we were offering in return. It became much easier to spot and address any gaps.

---

## Conclusion

In summary, understanding the nature, impact, and value of upcoming changes is invaluable when shaping an effective release plan. This clarity not only streamlines the planning process but also ensures that the team delivers meaningful improvements to users.
