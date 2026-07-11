---
title: "The Grant That Keeps Evaporating: TCC, Code Signing, and AI Agents on macOS"
date: 2026-07-11
draft: false
tags: ["ai-agents", "macos", "tcc", "code-signing", "automation"]
summary: "How to let a headless AI agent read and write your Reminders and Calendar on macOS — the bundle-identity trick that makes TCC grant access, and the code-signing tax nobody warns you about."
---

The re-grant dialog popped up for the third time that week. Same tool, same *Allow* button, same thirty seconds of walking over to a Terminal to click it. I'd rebuilt a small binary — added one verb — and macOS had, once again, quietly forgotten it was ever allowed to touch my data.

We run a fleet of AI agents on a single Mac here — persistent daemons that keep the books, write to the calendar, file reminders. In [the last post](/posts/fleet-fixed-its-own-tooling/) I mentioned there was one macOS detail that makes running agents like this harder than it should be, and promised to come back to it. This is that detail. If you want an agent to actually *operate* on a Mac — not just generate code, but read your Reminders, write to your Calendar, send through Mail — you run headfirst into it. It's the least glamorous part of the whole setup, and the part that most cleanly separates a demo from something that runs for weeks. This is the plumbing.

## The wall: TCC

macOS guards your personal data behind **TCC** — Transparency, Consent, and Control. It's the subsystem that pops the *"App wants to access your Reminders"* dialogs. Reminders, Calendar, Contacts, Photos, the whole lot sit behind it. You can't just call an API; something has to be *granted* access, and the grant comes from you clicking a dialog.

That's fine for a normal app. It's a problem for an agent, because the daemons are headless — started by `launchd`, no window, nobody at the keyboard. A headless process can't click a consent dialog. And if it can't *be granted*, it can't read a single reminder.

So the naive approach — have the agent shell out to a script that talks to EventKit — fails immediately with *access denied*, and there's no dialog to rescue it.

## The trick: borrow a bundle's identity

The fix is indirection. Instead of the agent touching Reminders directly, it drives a **small compiled binary that holds the grant** — and the grant is given once, interactively, by a human in a real Terminal.

But there's a subtlety that cost me an afternoon: **TCC keys its grants to code identity, and a bare binary doesn't have a stable one.** Compile a plain Mach-O executable, grant it access, and the grant lands on the parent process — or nowhere — depending on how it was launched. Direct-exec the binary from a shell and TCC attributes the request to the *shell*, not the tool. The grant never sticks to the thing you want.

What TCC *can* pin a grant to is an **app bundle**. So the pattern that works:

1. Wrap the binary in a real `.app` bundle — a directory with `Contents/MacOS/<binary>` and an `Info.plist` that declares a **bundle identifier** (`com.example.mytool`) and the **usage-description keys** TCC requires (`NSRemindersUsageDescription`, and so on).
2. Launch it through Launch Services — `open -W -n -a MyTool.app --args ...` — *not* by exec'ing the inner binary.

That second step is the one people miss. Going through `open` makes Launch Services start the app under its **own bundle identity**, so TCC attributes the grant to `com.example.mytool` and it survives every future launch. Exec the inner binary directly and you're back to the shell-identity problem. The wrapper is three lines of shell, and it's load-bearing.

Now a human runs the tool once from Terminal, the dialog appears *(because `open` surfaces it)*, they click *Allow*, and the bundle holds the grant. The agent drives the wrapper from then on and never sees a dialog.

## The tax: ad-hoc signing

Here's where that recurring re-grant dialog comes from.

TCC doesn't key its grant to the bundle *identifier* alone — it keys to the bundle's **code signature**. And my binary is **ad-hoc signed** (`codesign -s -`), the zero-cost option that doesn't need a paid Apple Developer account. Ad-hoc signing produces a signature derived from the binary's own contents.

You can see the problem coming. Rebuild the binary — add a verb, fix a bug — and the contents change, so the ad-hoc signature changes, so *to TCC it is a different app wearing the same name*. Every grant it held evaporates. The rebuilt tool comes back with no access, and you have to re-approve it: run it once in Terminal, click *Allow*, done. Thirty seconds — but you have to be *there*, which means that step can't be fully automated.

That's the tax on ad-hoc signing, and it's worth knowing before you build a fleet of these:

- **Grants are tied to the signature — not the file path, not the bundle ID.** A rebuild is, to TCC, a brand-new app.
- **Every rebuild resets the clock.** Ship a feature, re-grant.
- The honest deploy step is: rebuild → *"needs a re-grant"* → human runs it once → *Allow* → verify. Bake that click into the process instead of pretending it isn't there.

## The fix that should end the cycle — and why it didn't

There's a real fix: a **stable signing identity**. A proper code-signing certificate produces a signature tied to the *certificate*, not the binary's contents. Sign every rebuild with the same cert and the signature is stable — so TCC's grant survives. Rebuild all you want; the grant holds.

So I made a self-signed code-signing certificate, dropped it in the keychain, pointed the build at it — and it didn't work, for a reason worth flagging, because it's the kind of thing that eats an evening.

The build runs **headless**, under `launchd`. From that context, `security find-identity -v -p codesigning` returned **zero valid identities** — even though the certificate was sitting right there in the keychain. A code-signing *certificate* is not the same as a usable code-signing *identity*: the identity needs the certificate **and** its private key, paired and reachable from the process doing the signing. In a headless, non-interactive `launchd` context, the private key wasn't reachable — keychain access from that context is its own maze of partition lists and access prompts that, ironically, need a human to click.

So the stable-cert fix is real, but it isn't free: it's blocked behind *another* interactive keychain step — the exact thing I was trying to eliminate. For now, we stayed on ad-hoc-plus-re-grant, and filed the stable-cert work as a known, deferred fix.

## What it actually takes

None of this is exotic. There's no clever exploit, no private API. It's an app bundle, a shell wrapper, a signature, and a permission dialog. But it is *exactly* the layer that separates "an AI wrote some code that calls EventKit" from "an AI reliably operates a tool on a real machine, across restarts, for weeks."

The demo is easy. The durability is plumbing. If you're going to hand an agent real access to real data on a real Mac, budget time for the unglamorous parts:

- Wrap tools in `.app` bundles and launch via `open`, so TCC has a stable identity to grant.
- Expect ad-hoc signing to cost you a re-grant on every rebuild — and put that one manual click *in* your deploy, not in your blind spot.
- Know the stable-cert fix exists, and that reaching the private key from a headless build is its own small project.

The agent can write the code. Someone still has to click *Allow*. That's not a flaw in the model — it's the operating system doing its job, and the whole point is to work *with* that boundary, not tunnel under it.
