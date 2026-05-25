---
title: i built a pokémon card scanner in one night with claude code
date: 2026-05-24
description: A garage sale gave me a reason to finally figure out what my Pokémon cards are worth, so I built a photo-to-catalog scanner in one night with Claude vision, Next.js, and an agent-led XP workflow.
tags: [ai, claude-code, agentic-development, nextjs, side-projects]
---

I was selling off a chunk of my Pokémon card collection at a neighborhood
garage sale. A few buyers messaged ahead to say they were coming specifically
for the cards, which is when it hit me that I had no idea what most of them
were actually worth.

So I needed something quick and functional. Point my phone at a page of cards,
snap a photo, and get a current price on the rarer ones. No standing there
flipping through TCGPlayer one card at a time while a line of buyers waits on
me.

They say necessity is the mother of invention... thus my [Pokémon TCG Catalog](https://pokemontcg.workman.tech) was born.

## a side project, run like a real project

My team at work has leaned in hard on agent-driven engineering recently, and
I've been driving a good part of that shift. This project was a chance to take
the same practices I use professionally and point them at a problem of my own.

The architecture was obvious before I wrote a line. A JS framework for the app,
a small SQL database, Claude vision to read the cards, and a free Pokémon API
for the canonical data and prices. It was the perfect tightly-scoped app to be
built using a few agents without babysitting every keystroke.

So I spent about 30 minutes with Claude Opus 4.7 hashing out the architecture,
a handful of user stories and how I envisioned all of the pieces coming
together.
Out of that came six phases of work, each cut into four or five
single-responsibility tasks.

This is just how I run projects at work now. I lean on Kent Beck's Extreme
Programming fundamentals and bend them to fit agent-led execution. The planning
is just-in-time. A top-level `ROADMAP.md` lays the phases out as high-level
road-signs, just enough to confirm the order makes sense, and I only flesh a
phase out in full right before I execute it. A phase is roughly the chunk of
work you'd scope in a single backlog grooming, no more. You wouldn't point and
prioritize months of stories in one sitting, and that restraint still matters
even though Claude could happily plan hundreds of tasks up front. Just because it
can doesn't mean it should. A spec that detailed would be wrong by phase three
anyway.

Each phase gets its own directory, and at the top sits an `AGENT_INIT.md`.
That's the primer an agent reads first: the goal of the phase, the constraints,
the files to read and in what order, the decisions already made, and what's
explicitly out of scope. Under it are numbered task files, each one basically a
ticket scoped tight enough to land as a single PR. Small, focused, independently
verifiable. The `AGENT_INIT.md` is the source of truth for the phase. It's where
the agents go to figure out intent, and where pivots get written down when
reality disagrees with the plan. Reality usually disagrees with the plan. The
nice part is I can move in small steps and change direction halfway through
without losing the bigger thread.

Alright, plan in hand and the first phase sketched, I handed the tasks to
Claude Code and started working through them.

## what got built

Here's the shape of what came out the other side:

- **Next.js 16 (App Router) on React 19, in TypeScript.** Server-first by
  default. The database, the auth, and the external API calls all live on the
  server, and the client is just a rendering layer.
- **Neon (serverless Postgres) via Drizzle ORM.** One small schema. Cards, the
  copies I own, and their condition, foil status, and price.
- **Claude Sonnet 4.6 for the actual identification.** This is the heart of it.
  One photo of up to nine cards goes out in a single API call, and Claude hands
  back each card's name, set, number, foil status, a rough condition read, and
  a normalized bounding box. The app uses those boxes to crop one tight image
  per card right on the device, so every row in my inventory ends up with its
  own clean photo.
- **pokemontcg.io for canonical data and prices.** It's free, it indexes every
  set back to Base Set, and it bundles daily TCGPlayer and Cardmarket prices
  right into the card response. No second integration just for pricing, which
  was a nice surprise.
- **Vercel for hosting and Blob storage**, with HeroUI v3 on Tailwind v4 for a
  clean, mobile-first UI that stays out of the way of the card art.

A couple choices I was glad I made.

**Forced structured output instead of parsing JSON.** Rather than ask Claude
for JSON and hope, I gave it a single tool, `report_identified_cards`, with a
strict JSON Schema, and forced the model to call it. Anthropic validates the
shape server-side before any of it reaches my code. No `JSON.parse`, no
stripping markdown fences, no schema drift to defend against. The client just
reads the validated payload. Boring, in the best possible way.

**Caching over cron.** The identification system prompt never changes, so it
goes out with prompt caching, and after the first call the bulk of it is about
90% cheaper. That keeps each multi-card photo down to roughly a couple cents.
And instead of a scheduled job to keep prices fresh, the app caches each card
for 24 hours and lazily refreshes it on view. That matches how often the
upstream prices actually move, and it was one less moving part to build at
11pm.

## the result

I worked through all six phases and had the thing live on Vercel in about four
hours. Then I did the one thing it was built for. I scanned in my highest-value
cards, watched them resolve to the right printings with current prices, and
walked into the garage sale with a catalog on my phone instead of a search box
and a prayer.

It worked exactly as designed

## the honest version

Let me be clear about what this is and what it isn't. It's a personal tool that
solves a personal problem, and it has the rough edges to prove it. If I wanted
to run this for anyone other than me, I'd have real work to do on performance
and load balancing, and I'd have to swap the single shared-password admin for
an auth setup that's actually resilient and scalable. And the scanning, the
whole centerpiece of the thing, only nails the identification cleanly about
half the time. So I built a manual search fallback for the cards Claude whiffs
on.

None of that changes the takeaway though. The cards were just the excuse. That
30 minutes of planning with Opus up front, figuring out what to build and in
what order, is the entire reason four hours of agent-driven building actually
landed a working app. I had a real problem the night before a garage sale, and
I went to bed with it solved. Hard to be mad at that.

## the part i'm actually excited about

Here's the thing I keep coming back to. A few years ago, solving a problem this
specific meant one of two things. Either I find a SaaS product that handles 80%
of it, pay every month, and bend my actual problem to fit the 20% it was never
designed for. Or I block out a whole weekend to wire up a framework, a database,
auth, and hosting by hand before I can even start on the fun part. Both of those
are a lot of friction for "I want to price my Pokémon cards at a garage sale."

This time I described what I wanted, planned it out, and had a working tool by
bedtime. A couple cents in API calls and a free Vercel plan. That's genuinely
wild, and I don't think we should pretend it isn't.

I came into tech as a career changer, and the best part of this whole ride has
been watching the barriers keep falling. The stuff that used to gatekeep
building software, the setup, the boilerplate, the sheer pile of things you had
to know before you could make anything at all, keeps getting lower. Tools like
Claude Code just knocked another big chunk of it down. That's not a threat. That
is the dream. More people building more specific things for themselves is
exactly the point.

So if you've been sitting on some dumb little idea, some problem only you have,
consider this your sign. The floor has never been lower than it is right now. Go
build the thing. I can't wait to see what you make
