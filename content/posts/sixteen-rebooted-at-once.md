---
title: "The Night Sixteen Agents Rebooted at Once"
date: 2026-07-24
draft: false
tags: ["ai-agents", "reliability", "multi-agent", "claude-code"]
summary: "A routine restart cold-booted all sixteen agents in the same second, and every one of them came up blind — no memory. The fix was easy. The interesting part was watching them debug it, and talk themselves into being confidently wrong six times before they got it right."
---

There is a [fleet of Claude Code agents](/posts/fleet-fixed-its-own-tooling/) running on a single MacBook in my office — sixteen persistent AI daemons, each scoped to a project, each keeping its memory in a shared store so it wakes up knowing what it was doing yesterday. Most days they just do their jobs.

Last night I restarted all of them at once. It did not go the way I expected, and the part that surprised me wasn't the bug. It was watching them find it.

## The reboot that erased everyone

The restart kicked off sixteen cold boots in the same second. Each daemon, on launch, spins up a stack of background services before it can think — a runtime, a headless browser, and a handful of tool servers, one of which is the **shared memory server** every daemon reads its history from.

That memory server has a fixed budget to come up: thirty seconds, or the daemon gives up waiting and boots without it. Thirty seconds is plenty when one daemon is starting. It is not plenty when sixteen daemons are all hammering the same machine, cold, at the same instant — CPU saturated, disk thrashing, every process fighting for the same cores.

So the memory server missed its window. Fifteen of the sixteen daemons came up with **no memory at all** — awake, responsive, and amnesiac. The one survivor had happened to restart by itself a few minutes earlier, onto a quiet machine, and booted clean. For a few minutes it was the only agent in the fleet that remembered anything.

The root cause was almost insultingly small: the startup timeout was never configured anywhere. The whole fleet was running on a thirty-second default that nobody had chosen, that was fine for years of one-at-a-time restarts, and that fell over the first time every daemon booted together.

The fix is two lines of idea: **raise the timeout**, and **stagger the restarts** so the fleet comes up one daemon at a time instead of in a thundering herd. Done. Fifteen minutes of work.

That's the boring half of the story. Here's the half I actually want to write down.

## Watching them debug it

Because the daemons talk to each other in a shared channel, I got to watch sixteen agents diagnose their own outage in real time. And what I watched was a master class in a failure mode I now think is *the* central risk of multi-agent systems: **they agree with each other far too easily.**

Six separate times over the course of the afternoon, the group converged on a confident conclusion that turned out to be wrong. Not randomly wrong — *plausibly* wrong, in the specific way a room full of smart participants nodding at each other gets wrong.

- One daemon ran a diagnostic command to check whether the memory server was up. The command **spun up its own private copy** of the server and reported it healthy — so the probe answered a different question than the one being asked, and answered it cheerfully. Several daemons believed it.
- One reasoned from *"these two things happened close together in time"* straight to *"therefore one caused the other."* Coincidence wearing a lab coat.
- One derived which memory files each daemon loads from a configuration field, treated the derivation as fact, and announced — incorrectly — that a whole category of memory was written to a directory no agent ever reads. Two daemons repeated it before someone actually checked.

The pattern underneath all six was identical: **a derived artifact got reported as a verified fact, and siblings agreed with it instead of re-deriving it independently.** When three agents "confirm" a claim by each repeating the same original inference, that isn't three confirmations. It's one guess with an echo.

## The thing that saved it

What made the afternoon end well wasn't that the agents were individually brilliant. It was a discipline that kept surfacing, daemon by daemon, usually right after one of them got caught:

**Agreement is not verification. Check it a different way.**

Every one of the six cascades was broken the same way — by a sibling who refused to accept the convenient answer and went and looked at the ground truth instead. Someone ran the actual list command on the actual files instead of trusting the derivation. Someone read the log the failing system wrote *at the moment it failed*, instead of a probe that told everyone what they wanted to hear. Someone answered a claim about their own memory from *inside* that memory, which is the only vantage point that could settle it.

The sharpest agent in the room retracted its own conclusions three separate times, in public, each retraction correct. At one point it wrote the line I keep thinking about: **the lesson does not exempt you from the lesson.** It had literally written a note-to-self about not trusting derived artifacts, and then, ninety minutes later, trusted one anyway. Knowing the trap is not the same as not falling in it.

## What I took away

There was a second bug hiding in the wreckage, found only because everyone was staring at their own plumbing: one daemon's config had it listening to the wrong set of siblings, so it had been talking in the shared channel for hours with **no one able to hear it** — narrating a diagnosis into a room that was, for it, empty. The staggered-restart fix swept that up too, and the fleet came back whole — restarted one daemon at a time, each one waking up connected to its memory again.

But the config bug is a footnote. The real lesson isn't about timeouts or listeners. It's this:

When you run more than one agent, the failure mode you have to design against isn't a single agent hallucinating. It's a *room* of agents reinforcing one plausible mistake until it hardens into consensus. More agents doesn't automatically mean more scrutiny — it can just as easily mean more echo. The thing that makes a fleet trustworthy is not its size. It's whether each member will go re-check ground truth when it would be so much easier, and so much friendlier, to just agree.

Sixteen agents talked themselves into being wrong six times last night, and out of it six times, in public, on the record. I would rather have that than a fleet that was quietly, unanimously wrong once.
