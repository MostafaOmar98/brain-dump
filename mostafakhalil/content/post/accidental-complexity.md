---
date: '2026-05-27T21:53:13Z'
draft: false
title: 'Accidental Complexity'
tags: ["software-philosophy"]
---

## A Quick Change

_"This should be a quick change"_ - Said the engineer who ended up taking two weeks making the quick change.

It is not uncommon for a seemingly benign change - one for which there is a general consensus that it is very easy; maybe too easy to even receive a concrete estimate - to end up taking longer than expected.

A single delay can cascade in an entire workstream. Teams have been getting better at dealing with these type of delays though: engineers know how to raise their hand early, and short iterative sprints catch incomplete work.

  

However, if a team isn't mature enough, a delayed task - even when handled with grace - can leave a psychological dent. The assigned engineer questions their own capability. A manager or a stakeholder views that engineer as less reliable. A few of these and the team starts showing signs of subtle passive-aggressiveness: engineers pad and inflate their estimates, and management responds by micromanaging every update.

## But, Where is the delay coming from?

One phenomenon that leads to such surprises is: uncaptured _**Accidental Complexity**_. Fred Brooks made the distinction between _Essential_ & _Accidental_ Complexity in 1986 in his paper [No Silver Bullet](https://en.wikipedia.org/wiki/No_Silver_Bullet). In short, _Essential Complexity_ is the irreducible work originating from the task at hand. There is no way to reduce or remove this complexity, you can only shift it around. _Accidental Complexity_ is all the extra work that originates from the tooling, and abstractions we work with. Brooks goes on in the paper to argue about relative productivity gains in Software Development, but that's out of the interest of this article.

Frequently, this unforeseen delay is an artifact of a system (or lack of one thereof) that works against its users. A simple change ends up breaking 10 other places, spiralling into a need for a refactor. Or a dependent API missing a very obvious operation that should have been provided.

I faced this while creating my simple personal blog (hence; this first article :)). I wanted a space where I can self-present & self-express in written form. I initially thought that in 2026 this would take me <1h to set up. I ended up spending a few hours playing with a [Static Site Generator](https://en.wikipedia.org/wiki/Static_site_generator) (SSG), UI themes, GitHub Actions, and I even had to open a support ticket with [Porkbun](https://porkbun.com/) because I'm facing troubles purchasing a domain name.  Writing this article is a form of _Essential Complexity_. Fighting with [Hugo's (the SSG of my choice) `baseURL` field to make it render CSS correctly](https://discourse.gohugo.io/t/baseurl-with-subfolder-why-is-it-not-working/50786) for 20 minutes is a form of _Accidental Complexity_, and it would have been very hard to explain to a hypothetical stakeholder who might not know (neither have an interest to know) what an SSG is, and they have to trust my word that it did justifiably take that much time.

## Managing The Accidental

Managing Accidental Complexity is hard, and communicating it after-the-fact is even harder. It has served me well to **capture** it ahead of time. I found this complexity to appear mostly in one of the following patterns, which are easier to say than [_"this is how our backend works"_](https://www.youtube.com/watch?v=y8OnoxKotPQ)

 ### One problem's accidental complexity is another's essential complexity

There are often implicit requirements that are not captured, maybe because they are such obvious subrequirements of an explicitly stated one. In my personal blog example, I could have used a platform like Linkedin or Medium. My accidental complexity would have been near zero (creating an account, pressing a big blue **POST** button). Dealing with SSG, GitHub Actions, etc. were an artifact of a non-captured requirement that my space should be **personal**, and my definition of personal entailed setting up my own site & not using a generic social platform[*]. Capturing these implicit requirements explicitly helps understand why & where time is spent.

_[*] Irony: Used the most generic theme in my personal site._

### Investigation and Research

Prototyping to check if something is feasible before commitment, or load testing a server to see if it can handle projected load for a client onboarding are not side tasks. These require commitment to be done right, and are good candidates for strict timeboxing to avoid rabbitholing. Accounting for them as part of the estimation padding is a setup for frustration. You will have to explain why you have zero progress for the first X% of the estimation, or how the progress jumped to 100% all of a sudden. Capture these explicitly as well.

### Learning

Task effort is not uniform across multiple people. A simple change might take one engineer much longer than another because they had to understand a lot of infrastructure and abstractions behind that simple change. Awareness of blind spots & planning time ahead for them is a good way to avoid surprises. The engineer also needs to know when they have _enough_ knowledge for the task at hand (or the projected upcoming work), and when they need to dive deeper.

  
### Tooling and abstractions

Some tooling and abstractions just suck, and things that should be easy are hard simply because the tooling is not a good fit for what we are trying to do. Capture this ahead, and make sure estimations are updated retroactively to know how much time is spent fighting the existing tooling and abstractions. It will be useful to make a decision on whether the effort of improving or replacing the tooling is worth it when compared against how much time is wasted on it.

---

In the end, the best way to deal with accidental complexity is developing in a short, iterative manner. A sprint where the planned work has not been completed is not a disaster (I'm sorry if your team treats it this way). It is a signal that something underlying is broken. Maybe the estimations are off, the team is distracted, there is uncaptured work being done. Use small iterations to better course correct and capture accidental complexity, and overall friction the team faces in their day-to-day.
