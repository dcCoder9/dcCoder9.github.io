---
title: "Reflections on the software blog posts that influenced me the most"
published: true
---

Over the years there have been a handful of ideas I've found in blogs posts that have become foundational to how I see certain parts of software development. These are the kind of posts that I frequently link to in my comments on code reviews. Here I'll share these articles in one place, and explain why they're so important to me.

## [The rule of three](https://blog.codinghorror.com/rule-of-three/)
"API development" is currently very trendy in software development. I've been on teams that were tasked with building APIs for internal usage, and it's incredibly easy to get caught in a trap of over-engineering code in order to meet every possible future use-case. It's hard to blame a developer for getting caught in this place - after all, the business people (and other development teams) are asking for "APIs", which are supposed to serve a broad purpose.

I think the language around this is problematic - what exactly is an API, and how is it different from any other piece of code that's exposed (e.g. an endpoint)? On my previous team, I realized that there were miscommunications around the expectations of APIs, so I introduced a new word to help set expectations when talking with other teams: single-use components (SUCs). The idea was that a SUC was built precisely for the use-case that the team had, and if two more similar future use-cases arose, we could consider building a formal API (to which we would commit to supporting publicly).

The clarification of language achieved two things: first, it really helped set expectations from other teams. Second, it made us less afraid to build a component because we no longer felt like it had to be the perfect solution to every future use-case.

## [Write tests. Not too many. Mostly integration.](https://kentcdodds.com/blog/write-tests)
I've spend a lot of time thinking about the best way to write automated tests. At the end of the day, there's no one-size-fits-all approach. Different domains require different forms and intensities of testing, and it's up to the engineers with business context to make those trade-offs.

What I like about this article is that it pushes back against an existing idea of blindly adding unit tests to everything by asking the question, "Given your new test is passing, how confident are you that your software works as intended?" This really forces me to think critically about my test - if I'm mocking most of the method calls my component makes, do I really trust my test? If I don't add any integration tests, will the code really succeed from the user's perspective?

## [Things You Should Never Do, Part I](https://www.joelonsoftware.com/2000/04/06/things-you-should-never-do-part-i/)
As a developer, I enjoy designing and architecting new solutions. Reading through old solutions is simply less exciting. However, after reading this, I'm _very_ wary of suggestions to "rewrite" a technology just because it's old or unfamiliar. If re-using functions is considered good practice, shouldn't re-using entire projects be even better?

## [How to Do Code Reviews Like a Human](https://mtlynch.io/human-code-reviews-1/)
So much technical information is communicated through pull request comments, but I've never seen much advice on how to make those comments as effective and as _kind_ as possible. This post has great advice that looks through the internet and towards the person on the other end of your code review.

## [How to Write Unit Tests for Generic Classes](http://codinghelmet.com/articles/how-to-write-unit-tests-for-generic-classes)
This post itself wasn't too influential on me, but I've read a lot of Zoran Horvat's posts and overall he's expanded the way I think about object-oriented design, functional design, and testing. There's a lot of interesting ideas in his blog posts, and I fully recommend reading them.

## [Think you understand the Single Responsibility Principle?](https://hackernoon.com/you-dont-understand-the-single-responsibility-principle-abfdd005b137)
It can be really tricky to define what a 'single' thing is - everything can be broken down into further pieces. I appreciate this definition of the SRP as a more concrete way to see and measure whether things are a single unit, or a composition of units.

From Martin: "Gather together the things that change for the same reasons. Separate those things that change for different reasons."





