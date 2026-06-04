---
title: "the repo is the tracker: solo dev project management in the time of ai"
date: 2026-06-03
description: When AI agents do the building, the slow part stops being code and becomes coherent intent. Here's the in-repo project management system I built to keep a fast-moving solo project from drifting, and the ideas behind it that transfer to any team.
tags: [ai, claude-code, agentic-development, project-management, solo-dev]
---

Working as a solo dev is always a challenge, but with the recent advancements in LLMs and AI tooling, it's more realistic than ever to launch a one-person product company. I can oversee multiple agents working on multiple streams of work and epics at once. This has had me thinking a lot about how I run dev teams at work and all of the experience I've gained over the years, including my time consulting at Carbon Five.

I have found success and great results running Kent Beck's Extreme Programming practices with teams. Now, I'm translating those processes over to a team full of AI agents and have been really impressed with the results.

## what worked for one night stopped working for one month

I have been revving on this idea of XP in the age of AI. The [Pokémon TCG app](https://workman.tech/side-projects/2026-05-24-pokemon-card-scanner-in-one-night.html) that I spun up in one night was my first attempt at running agents with Extreme Programming.

The setup was deliberately light: a `ROADMAP.md` of high-level phase road-signs, a short primer each agent read first, and numbered task files each scoped to a single PR, all planned just in time. For something I finished in one night, that was exactly enough structure and not an ounce more. But the thing I'm working on now is not a one-night job, it's months of work across several phases, a couple of them running in parallel, with multiple agent sessions a day. At that scale the lightweight version started to creak, and the cracks were always about memory: work I'd finished but never marked done, decisions an agent re-litigated because they lived in a chat log that no longer existed, and later phases quietly built on assumptions the earlier ones had already broken.

## the repo is the tracker

So the first real decision was to stop keeping the plan somewhere my agents couldn't read it.

Think about where project state usually lives. Jira, Linear, a GitHub project board, a doc in Notion. Every one of them built for a human to click around in a browser. But my team doesn't click anything. It reads files. Asking an agent to manage work out of a web UI was slow and a massive token-sink.

So I moved the plan into the repo. Markdown files, with the status of every piece of work encoded in YAML frontmatter the agent can grep in a second. Pull `main`, read the tree, and you know what's done, what's blocked, and what's next. No second system to keep in sync, and no context stranded outside the place the work actually happens.

The repo is the tracker

That one sounds small. It's the whole game. Once the plan and the code live in the same place, the agent writing the code is the same agent reading the plan, and the gap where intent used to leak out just closes.

## a team that forgets everything overnight

Working with agents means working with a team that has no long-term memory. Every session starts cold with a finite context window, so unless the project's decisions, state, and history live somewhere durable an agent can read on demand, that knowledge is gone the moment the session ends.

## the shape of it

### lighthouse

I think of this as a project kick-off. On a team, we'd spend a full day talking about the project, the problem we're solving, the high-level architecture, etc. All of these decisions would then guide us as we worked through the project over the coming weeks and months. This, obviously, changes as we encounter unknowns, pivot our approach, etc. The agents, just like a human dev team, need some high-level working memory of the project with known tech debt and the latest decisions. This is where the **lighthouse** comes in.

The lighthouse lives in a `docs/` directory, one per project. It holds the locked-in decisions: the architecture we agreed on, the contracts the rest of the work has to honor, and the reasoning behind the calls we don't want re-litigated three weeks later. Right alongside it sits a living decision log and a running registry of known tech debt, both append-only, so when an agent (or a human) asks _why_ something is built the way it is, the answer is still on record. It's the document I'd hand a new engineer on day one, and it's exactly what a fresh agent needs to read before it touches anything.

Each project also gets an `AGENT_INIT.md`: the primer a new agent reads first. It's the bootstrap prompt that points at the lighthouse and says here's the project, here's what to read and in what order, here's what's already been decided. On a human team this is the onboarding doc and the kickoff meeting rolled into one. I feed it to every fresh session, because with agents, every session starts cold.

### epic

The lighthouse answers _why_ we're tackling a given project and _how_ we want to solve the known problems. I think of the **epic** as the brain for the project manager. It's all about _what_ we need to do in order to deliver and complete the project.

The epic lives in a `plans/` directory that sits right next to `docs/` in the same repo. It's a body of work big enough to need real planning and real architectural decisions, and it carries the high-level scope, the risk, and the running status of everything beneath it. Where the lighthouse holds the _why_ and the _how_, the epic holds the _what_: the full slate of work we have to get through to call the project done. Everything below it, the phases and the tasks and the status of each, hangs off the epic.

### phase

In XP, there are no sprints. There is a prioritized, maintained, groomed backlog. When a developer is finished with a task, they simply take the next task off of the backlog. **Phases** are how we emulate this process with agents.

When I scope an epic, I lay its phases out up front as high-level road-signs: just enough to confirm the order of operations makes sense, not a detailed spec for work that's still months away. A phase is roughly the chunk you'd scope in a single backlog grooming, and that restraint matters even though an agent would happily plan two hundred tasks in one sitting. A spec that detailed is wrong by phase three anyway, because the early work always teaches you something that rewrites the rest.

So phases get fleshed out _just in time_, one at a time, right before I build them. Each phase ends with a wrap step whose whole job is to look at what actually shipped, fold the new decisions and debt back into the lighthouse, and lay out the next phase from current reality instead of kickoff optimism. It's the same loop XP runs on a real backlog: finish the work in front of you, re-groom, pull the next thing off the top.

### task

Finally, in XP, we build small, single-responsibility tickets categorized as either a `bug`, `chore` or `feature`. AI agents, similar to humans, work best when given a contained surface with explicit deliverables. **Tasks** are basically tickets written _by_ agents, _for_ agents.

A task is scoped to a single PR: one contained change, explicit deliverables, a clear definition of done. And like the phases, tasks are written _just in time_. Only the active phase has task files; the phases ahead of it stay empty until their turn. When a phase wraps, it scaffolds the next phase's tasks against what the codebase actually looks like right then, not a guess made at kickoff. That's the whole point, a task written the moment before it's worked is grounded in current reality, so it never gets the chance to go stale.

Each task carries more than a one-line goal. It names the docs the agent has to read first, cites the decisions it isn't allowed to re-open, and lists a few commands to run up front to prove its prerequisites actually exist instead of being assumed. On a human team a senior dev packs that context into a well-written ticket. Here the ticket is written by an agent, for an agent, and every section in it exists to keep the next one from drifting off course.

## done is a claim, not a checkbox

The last piece is the one I underestimated: what it means to mark something done.

On a human team, done is a status somebody types. Here it has to mean the code merged and the plan got realigned to match what actually shipped, in the same motion. Plans drift hardest right at the finish line, because the thing you built is never quite the thing you specced. So I wrapped that moment in a little tooling of its own. Landing a task doesn't just flip a flag, it re-reads the merged diff and grooms the downstream plan against what really happened, not the guess I made before I started. Done isn't a checkbox you tick. It's a claim the system reconciles against reality.

The same tooling runs the other direction too. I can ask it where a project stands and it reads the entire tree and hands back a briefing: where I am, the next actionable task, the decisions that govern it, and anything that's drifted out of sync. A project manager in a box, grounded entirely in files that are always current because they live right next to the code.

## the part i keep coming back to

Coding with AI agents is, at the end of the day, just another tool. It's a powerful one, and it changes the way we think about building software, but it doesn't change _why_ we landed on the best practices we already have. AI writes code really quickly. The job now is figuring out how to fold that speed into the practices that lead to secure, scalable, maintainable software.

## what's next

The catch with everything above is that it currently lives as a tangle of skills and scripts wired into one specific repo. So I'm pulling the whole system out into a small open-source library you can drop into any codebase and have running in a few minutes. When it ships I'll write the whole thing up properly, so keep an eye out
