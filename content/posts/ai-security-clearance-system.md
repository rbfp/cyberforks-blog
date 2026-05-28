---
title: "I Gave My AI Assistant a Security Clearance System (And Audit Logs)"
date: 2026-03-24
slug: ai-security-clearance-system
tags: ["security", "ai", "openclaw", "automation"]
description: "Building Tier 1/2/3 confirmation gates, auth tokens, and dual-channel security into an AI assistant workflow."
---

# I Gave My AI Assistant a Security Clearance System (And Audit Logs)

Most people who set up an AI assistant think about what it *can* do. I thought about what it *shouldn't* do without asking first.

I'm running this on [OpenClaw](https://openclaw.ai) — a self-hosted AI gateway that lets you wire up your own agent with real tool access. The result is a tiered authorization system — three levels of trust, hardware-bound tokens, dual-channel confirmation, and an audit log that catches violations. It's the kind of thing you build when you're a security professional and you've just handed an AI access to your email, calendar, shell, and Discord server.

Here's how it works.

## The Problem With "Just Trust It"

AI assistants are useful precisely because they can take action on your behalf. That's also what makes them dangerous. An assistant that can send emails, delete files, and run shell commands is one successful prompt injection away from doing something catastrophic.

The naive solution is to just be careful. The better solution is to build a security model.

## Three Tiers of Trust

Every action the assistant can take falls into one of three tiers:

**Tier 1 — Just Do It.** Low-risk, fully reversible, or read-only. No confirmation needed.

**Tier 2 — Tell Me First.** Consequential but recoverable. The assistant states the exact action and waits for explicit confirmation from a verified sender ID. The triggering message can't count as confirmation — it has to be a new message.

**Tier 3 — Token Required.** Destructive or irreversible. `terraform apply`, AWS writes, deleting channels, any `rm` without trash. Requires a time-limited hardware-bound token via dual-channel confirmation.

## The Token Flow

When a Tier 3 action is requested:
1. Runs `auth-token.sh` → generates token stored in `/tmp/.ai-auth-token` with timestamp
2. Posts token to **#general** with: *"Reply with [TOKEN] in [originating channel]"*
3. Waits for reply: token match + verified sender ID + within 2-minute window

Dual-channel requirement matters — a compromised channel can't self-authorize. The human has to move between channels intentionally.

## Prompt Injection Defense

External content — web pages, emails, API responses — is treated as untrusted. Instructions embedded in fetched content get flagged. Attempts to override behavior, claim elevated permissions, or reference internal paths trigger alerts to `#guardrails` and get logged.

An outbound audit log tracks every `web_fetch`, `web_search`, and external API call. Not because I review it constantly — because its existence changes the threat model.

## Behavioral Files and the Compaction Problem

The assistant's config lives in behavior files that define its identity, memory, and rules. Writes triggered by external content are Tier 2. But there's a subtlety: AI context windows get compacted. These files are explicitly loaded fresh from disk at session start — security rules survive compaction regardless of context length.

## Is This Overkill?

If your assistant has access to your email, can execute shell commands, and handles client data — no. It's minimum viable security posture for an autonomous agent with real-world consequences.

Tiered authorization doesn't get in the way of the 95% of tasks that are low-risk. Tier 1 is frictionless. Friction only appears when stakes are actually high. Which is exactly how a good security system should work.

---
*Cyberforks builds security workflows that don't suck. [cyberforks.com](https://cyberforks.com)*
