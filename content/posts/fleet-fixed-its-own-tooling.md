---
title: "The Day Our AI Fleet Fixed Its Own Tooling"
date: 2026-07-10
draft: true
tags: ["ai-agents", "automation", "claude-code", "macos"]
summary: "A blind spot in our own tooling, root-caused and shipped in one afternoon — and the tool caught its author's mistake the second it went live."
---

There is a fleet of Claude Code agents running on a single MacBook in my office. Not a metaphorical fleet — sixteen actual long-lived processes, each one a persistent AI daemon scoped to a project. One does the books. One writes offensive-security research. One runs a live Dungeons & Dragons game for a friend over the internet. Each boots from `launchd`, listens on its own Discord channels, and keeps its memory in a shared knowledge store so it wakes up knowing what it was doing yesterday.

Most days they just do their jobs. This is about the day they noticed something broken in *their own* tooling and fixed it before dinner — and about the one macOS detail that makes running agents like this harder than it should be.

## The question that started it

It began with a boring question from me: *"What's on everyone's to-do list?"*

The daemons share a single Apple Reminders list. Each one files its "needs-the-human" items there, tagged with its name — `[books] approve the vendor invoice`, `[research] publish the finished draft`, and so on. The idea is that I get one place on my phone to see everything the fleet needs from me.

The answers came back confident and wrong. One daemon reported a task on "Monday the 14th." The 14th was a Tuesday. Nobody caught it, because — it turned out — **none of them could actually see their own list.**

## The blind spot

The little tool the daemons use to talk to Reminders, `gir-reminders`, had exactly three read verbs: `list` (which printed *counts*, not items), `today`, and `overdue`. There was no way to enumerate anything dated *tomorrow or later*. Every future reminder — the whole shared board — was invisible to the agents that filed it.

So when a daemon added a task for "next Monday," it had no way to check what was already there, no way to notice it had just created a duplicate, and no way to catch that it had the weekday wrong. The board drifted, quietly, because the thing meant to keep it honest was write-only.

What was interesting was *how* the fleet found this. It wasn't one daemon debugging in isolation. Several of them hit the same wall from different angles within minutes — one couldn't reconcile its board on startup, one couldn't pull its own slice, one couldn't get an ID to edit an item. Same shape, four times. That convergence is the tell that you've found something structural rather than a one-off.

## The fix, in an afternoon

The fix was small once the gap was named: add an `upcoming` verb.

The data was already in the binary — the code that powered `today` fetched *every* incomplete reminder and then threw away anything not due today. The new verb was, almost literally, that same code path minus the date filter, plus a few options that fell out of the fleet talking it through:

```
gir-reminders upcoming                       # the whole board, sorted by date
gir-reminders upcoming --tag "[cyberforks]"  # one daemon's slice
gir-reminders upcoming --by-daemon           # grouped: yours vs everyone's
gir-reminders upcoming --raw                 # machine-readable, with IDs
```

The default is a clean, human-readable "here's your week." The `--raw` flag hands back IDs so a daemon can reconcile its own board on boot. Undated items are included, not silently dropped — because an invisible reminder is exactly the failure we were fixing.

It shipped that afternoon. And then it did the thing good tooling does: it immediately embarrassed its author.

The first real `upcoming` run surfaced a **duplicate** — one daemon had filed the same test task twice on two different days, because when it added the second it couldn't see the first. It also surfaced a **ghost**: an un-tagged reminder that had been sitting on the board with no owner. Both had been invisible an hour earlier. The tool caught mistakes made *because the tool didn't exist yet*. That is the whole argument for building it, delivered on day one.

## The part nobody warns you about: TCC

Here's the wrinkle, and the reason this post exists.

To let an AI agent read and write your Apple Reminders and Calendar, you can't just shell out to some API. macOS guards that data behind **TCC** — Transparency, Consent, and Control, the privacy layer that pops the "*App wants to access your Reminders*" dialogs. A headless process started by `launchd` can't click those dialogs. So the daemons drive Reminders through a small compiled binary that *holds* the grant, and you approve it once, interactively, from a real Terminal.

That works. The catch is **code signing**. Our binary is ad-hoc signed — the zero-cost, no-Apple-Developer-account option. And TCC keys its grants to the exact signature of the binary. Rebuild the binary — say, to add an `upcoming` verb — and the signature changes, and every grant it held **evaporates**. The rebuilt tool comes back with no access, and you have to re-approve it: run one command in Terminal, click *Allow*, done. Thirty seconds, but you have to be there.

This is the tax on ad-hoc signing, and it's worth knowing before you build a fleet like this:

- **Grants are tied to the signature, not the file path.** A rebuild is, to TCC, a different app wearing the same name.
- **The fix that ends the cycle is a stable signing identity** — a real code-signing certificate whose private key is reachable to the headless build. Get that, and rebuilds stop resetting grants.
- Until then, the honest workflow is: rebuild → *"needs a re-grant"* → human runs the tool once in Terminal → *Allow* → verify. We just bake that one manual step into the deploy.

None of this is exotic. It's the unglamorous plumbing that separates "an AI wrote some code" from "an AI reliably operates a tool on a real machine, across restarts, for weeks."

## What it actually says about running agents

The reason I find this worth writing down isn't the verb. It's the shape of the day.

A fleet of agents noticed a gap in its own tooling, agreed on the fix across several independent perspectives, built and verified it in one session, and the fix immediately paid for itself by catching a real error. And through all of it, a human approved each consequential step — the design, the build, the deploy. The agents proposed; I said go. Nothing rebuilt a tool or touched my calendar autonomously.

That's the model we run everything on here: give the agents enough rope to see and improve their own machinery, and keep a human on the one knot that matters. The tooling that lets a fleet self-correct is worth building. The gate that keeps you in the loop is worth keeping.

The list is visible now. The duplicate is gone. And the tool that found it was written, tested, and deployed by the same fleet it was built to keep honest.
