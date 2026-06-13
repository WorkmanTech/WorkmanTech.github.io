---
title: "building an oracle: a knowledge base of how my team actually reviews code"
date: 2026-06-12
description: I build knowledge-base engines for a living, so I pointed one at my own team. It mined a thousand PR discussions into the judgment behind our code reviews, and now it reviews our PRs and learns from whether we take its advice. Here's how the Table22 Oracle works, and how I built it in an afternoon.
tags: [ai, code-review, knowledge-base, claude-code, agentic-development, engineering-judgment]
---

We've been digging into how to use LLMs to understand our partners better at Table22. A couple of months ago I took over building our onboarding pipeline. I'll get into that pipeline properly in a future post, but the short version is that it builds a prospective partner's brand description and tone of voice by reading their existing website and social presence. The core of it is the **Entity Knowledge Base Engine**, a composable, transferable engine that defines categories and pulls values for them out of messy, unstructured blobs of text.

Building that engine got me thinking about the general shape of a knowledge base that grows and corrects itself over time. What if I pointed the same idea at us instead of our partners? What if I ingested _all_ of our PRs, our Slack tech threads, and our weekly dev-chat transcripts? Could I build a knowledge base that captured how the team actually builds software, the patterns we reach for, the preferences we argue about, and the standards we hold each other to in review? And if I could capture all of that, could I hand it back to the team as the ideal reviewer and consultant?

## meet the oracle

Let me skip to the punchline. The details are below if you want them, but first say hello to the 🔮 **Table22 Oracle**.

The Oracle is a knowledge base of about fifty **principles** and forty **candidates**, mined by pattern-matching close to a thousand PR discussions across three of our repos and five languages. A principle is a single distinctive rule about how we build, and every one of them is grounded in at least two verbatim quotes from real review comments, each attributed to whoever actually wrote it. A candidate is a pattern an agent thinks it sees but hasn't earned yet. Nothing gets promoted to a principle on an agent's say-so.

The bar that matters most is that a principle has to be _distinctive_. "Writes clean code" or "cares about tests" describes every competent engineer alive, so that kind of thing is banned outright. The only judgment worth capturing is what separates how _we_ work from generic best practice, and the rules are conditional, each one recording the context where it flips. Real judgment is never absolute.

There are two ways to use it, and both ship as skills an agent loads on demand. As a **reviewer**, you point it at a diff and it reviews the PR the way the team would, and when it raises something it names the principle and quotes the actual comment behind it, so every note traces back to a real decision a real person made. As a **consultant**, you bring it a design call you're still chewing on and it weighs in from that same captured judgment instead of a generic model prior. When it reaches past what's actually confirmed, it says so out loud rather than dressing a guess up as house style.

## how it was built

The build was the fun part. I spent a Saturday afternoon pointing Opus 4.8 workflows at our GitHub history and fanning out dozens of agents to read hundreds of PRs in parallel, each one pulling verbatim review comments and tagging them with who said them. A verify gate sat at the end of the run and threw out any quote that wasn't an exact substring of a real comment, because a knowledge base that paraphrases is a knowledge base that lies. Two hours of agents reading on a weekend, and I had the first real corpus.

Not every voice in that corpus counts the same. I weighted our principal engineers' patterns the heaviest, because their review comments are where the team's hard-won standards actually get taught, the same call explained across a hundred PRs until it finally sticks. They're the foundation the rest of the knowledge base gets reconciled against, and the genuinely sharp principles, the ones a generic reviewer would never think to raise, almost all trace back to them.

The test for whether something was a durable standard or just a quirk of one codebase was simple: did it recur somewhere structurally different. A principle first seen in our Rails app that then shows up, unprompted, in a Python and TypeScript service is portable judgment, not framework trivia. That cross-repo recurrence is what earned a pattern its place, and it's why the corpus spans more than one stack on purpose.

## the future

For the last week the Oracle has been running on a fifteen-minute schedule, reviewing our PRs on its own. It still lives entirely on my machine and the team knows it's there, so think of this as the trial run, not a product launch. But here's the part I find genuinely exciting: it's no longer just applying the foundational knowledge base, it's growing it from our current work, minting new candidates and promoting new principles off the PRs we're shipping right now.

It does that by learning from what happens after it comments. It posts a review, follows up on the replies, and watches the outcome. If the author takes the note and pushes a fix, that's an accept, and the pattern behind it gets stronger. If they push back with a real constraint, that's a signal to back down, and the rule gets walked back. If the PR just merges untouched, that gets noted too. Acceptance like this is weaker evidence than a principal engineer stating a standard outright, so anything the Oracle learns this way is marked as such, never auto-promoted to high confidence, and automatically demoted the moment the team stops agreeing with it. The knowledge base is built to walk its own opinions backward, which is the whole game. The fastest way to make a tool like this useless is to let it freeze a stale take and keep quoting it long after everyone has moved on.

It's early, and it's earning trust one review at a time before it becomes anything the team actually leans on. But the idea underneath it is the one I keep coming back to. Most of a team's hard-won judgment lives in two places, a handful of senior heads and a graveyard of old PR threads nobody reads twice, and getting it out of both, grounded in what people actually said and kept current as they change their minds, is a knowledge-base problem. It's the same one I work on every day at Table22. Turns out the most interesting entity to point that engine at was us
