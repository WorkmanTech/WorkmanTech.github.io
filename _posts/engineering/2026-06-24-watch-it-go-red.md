---
title: "ai is backfilling your tests, and it's a regression"
date: 2026-06-24
description: TDD was never really about the tests. It was how we designed a feature while we built it, one failing test at a time. AI spins up whole features in minutes and backfills tests written to pass, which quietly turns the practice inside out. Here's why that's a regression, and how we taught our agents to write the test red first.
tags: [testing, code-review, ai, engineering-judgment, tdd, agentic-development]
---

Every engineer learns to trust a green test suite. We hang the whole feedback loop of modern software on it, the CI gate, the deploy, the late-night "is it safe to merge," all of it resting on a row of passing checks. I've spent the better part of a decade writing those tests and reviewing other people's, and these days with all of the AI infrastructure work at Table22, I now review a *lot* of tests that no human typed at all. This has led me to reconsider tests. What's the point if *no one* has ever seen them fail?

It's one of the dozens of principles in the [Oracle](/engineering/2026-06-12-building-an-oracle.html), the knowledge base of how my team actually reviews code that I built earlier this month, and it's one of the rules it reaches for most when it reviews our PRs. But the longer I've sat with it, the more I think it's pointing at something bigger than test hygiene. It's a way back to a discipline AI has been quietly eroding.

## green is the default, not a verdict

The entire job of a test is to fail when the thing it guards breaks. That's the contract, the whole reason the test exists. So a passing test only tells you something if it was *capable* of failing for the reason you care about, and a surprising number of tests are not.

A test can pass because the behavior is correct. It can also pass because the assertion was written with the knowledge of *how* the feature works. A green check mark is a green check mark when CI assesses it. The weight of a well-crafted assertion is exactly the same as that of an assertion that simply checks that the language is working as intended. They look identical right up until the day someone merges in a regression that takes down production.

With AI, I have found that a green test suite has become the default state. The only thing that turns it into a verdict is watching the test fail on the broken code.

## watch it go red

One of the first lessons drilled into me at Turing back in 2016 was test-driven development, a controversial practice that has been written about ad nauseam. TDD, however, is starting to become a real sticking point with me when it comes to agentic engineering. By default, AI agents don't care about TDD, or testing for that matter. They are programmed to satisfy the acceptance criteria given to them. If you tell them that tests need to be added to catch regressions, the models will satisfy that request by writing tests based off the code they wrote. This flow breaks the red-green-refactor cycle.

## tdd was never really about the tests

Here's the part we've half-forgotten. Test-driven development, red-green-refactor, the loop Kent Beck pinned down decades ago, was never really about ending up with tests. The tests were a byproduct. The real value was that writing the test first forced you to design the feature before you built it. You had to decide what the thing was called, what it took in, what it handed back, and what "correct" even meant, all before a line of implementation existed. I remember Jeff Casimir, Turing's founder, telling our class to think of this as "dream-driven development." When writing your first test, simply write what you dreamt the code would do and then build the code based off of that dream to make the test pass.

That idea of dreaming the architecture into existence has stuck with me, and it's the discipline a lot of us quietly let slide once the test suites became larger than life and the deadlines transitioned from ambitious to unrealistic. Writing the implementation first and the test second still gets you to green, so it feels like the same thing. It isn't. You've kept the byproduct and thrown away the design step that made the practice worth doing.

## ai turned it upside down

AI takes that quiet erosion and makes it the default. I can ask an agent for an end-to-end feature and have working code in minutes, and the tests show up *after*, backfilled against an implementation that already exists. Sit with what that does. The test is no longer a design tool, it's a formality written to ratify whatever the model already produced, and a test written against code that's already sitting there will bend itself to agree with whatever that code happens to do, bugs and all. We didn't design the feature, we generated it and then wrote tests as a way to pat ourselves on the back.

That isn't TDD sped up. Simply put, it's just not TDD and it's a bad practice that has been proven to lead to less resilient, less mindful architecture. We made the typing faster and dropped the thinking that gave the loop its value. The tests are being written to pass, which is exactly the failure mode the whole "watch it go red" idea exists to catch, now running at the speed and scale of an agent.

## an agentic flavor of tdd

Here's the interesting part (and I know that I buried the lede with this one): AI may be driving teams away from TDD, but it was AI that realized the pattern and pushed my team to correct the regression.

After reviewing hundreds of PRs and discussions, the Oracle identified TDD as one of our core principles and has implemented and enforced rules that guide other agents working on features. Similar to how a more senior engineer would guide a junior, the Oracle has been instructing the agents to write the failing tests first. Folding "watch it go red" into our agentic flow means the agent has to state the test, run it, and prove it fails for the right reason *before* it's allowed to write the feature that makes it pass. The red comes first, on purpose. It's the same loop Beck described, enforced on an agent instead of a person.

The effect went further than I expected. I went in hoping for better safety nets and fewer vacuous tests, and we got those. What I didn't see coming was that it changed the *architecture*. An agent made to write the test first has to commit to an interface before it commits to an implementation, exactly like a person does, and the code that comes out the other side is shaped by that constraint instead of reverse-engineered to satisfy a test bolted on at the end. The quality went up because the design went first. The thing that made TDD valuable for humans makes it valuable for agents for the exact same reason.

## fold it in, don't throw it out

I'm spending real time now on how far this goes, on how to extend TDD properly into agentic engineering the same way I've been adapting the rest of Extreme Programming into how my agents actually work, the phase planning, the small single-PR units, the just-in-time grooming. I don't have the whole shape of it yet, but the journey has been incredible and enlightening.

But the principle underneath it is the one I keep landing on. AI made exactly one thing dramatically faster, and that thing is writing the code. It did not make a single one of the practices we spent decades hardening obsolete: not TDD, not continuous integration, not small reversible steps, not any of the XP lineage that people a lot smarter than me refined through years of trial-and-error. If anything the speed makes them matter more, because the faster you can generate code, the easier it is to generate a mess at scale. The teams that rise in this era won't be the ones that throw out forty years of hard-won discipline because a model can finally type faster than they can. They'll be the ones who figure out how to fold AI into that discipline instead of around it. Speed was never the bottleneck. Judgment was, and it still is.

The smallest place to start is the one we opened with. Before you trust the next green check, human or agent, make it red once.
