---
title: "The Morning Briefing: How My AI Starts My Day"
date: 2026-03-31
slug: morning-briefing
tags: ["automation", "ai", "openclaw", "productivity"]
description: "A cron job, a weather API, and an email scanner walk into a Discord channel. Here's how my AI delivers a daily briefing that actually helps."
---

# The Morning Briefing: How My AI Starts My Day

Every weekday at 6 AM, a message appears in my Discord. It's a briefing — weather, email summary, calendar, and anything the AI thinks I should know before I start working. I didn't ask for it that morning. I set it up once and forgot about it.

That's the point. The best automation is the kind you stop noticing.

## What's in the Briefing

The format has evolved over a few months, but the current version covers:

**Weather** — A 7-day forecast pulled from Open-Meteo. No API key needed. Each day gets an emoji, a condition, high/low temps, and precipitation chance. Formatted for mobile — no tables, just clean lines I can scan while making coffee.

```
☀️ Mon 3/31 — Clear — 78° / 54°F — 0% rain
🌤️ Tue 4/1 — Partly Cloudy — 75° / 52°F — 5% rain
🌧️ Wed 4/2 — Rain — 62° / 48°F — 80% rain
```

**Email Summary** — The AI scans multiple email accounts and surfaces anything that needs attention. Not every email — just the ones that matter. New client communications, invoices, anything flagged as urgent. Each gets a one-line summary with a number so I can say "file 3" or "reply to 5" later.

**Calendar** — Today's events and anything in the next 48 hours that I should be aware of. Meeting times, locations, prep notes if I've added any.

**Action Items** — Anything the AI picked up from yesterday that's still open. A follow-up it noticed I didn't send. A task I mentioned but didn't complete. This section is the most useful and the most annoying — which means it's working.

## How It Works

The briefing runs on [OpenClaw](https://openclaw.ai)'s cron system. Two jobs:

- **Weekdays** — fires at 6 AM, 10 AM, 2 PM, and 6 PM. The morning run is the full briefing. The afternoon runs are lighter check-ins — new emails, calendar changes, anything urgent.
- **Weekends** — one briefing at 6 AM. Shorter. No action items. I'm off the clock.

Each cron job triggers an isolated agent session that gathers the data, formats the briefing, and posts it to a dedicated `#daily-briefing` channel. The channel is output-only — I don't chat there. It's a feed.

## Why a Dedicated Channel

Mixing briefings into the main conversation channel was a mistake I made early on. The briefing gets buried under other messages. You can't scroll back to find this morning's weather without passing through a dozen unrelated conversations.

A dedicated channel means:
- The briefing is always the latest message
- I can glance at it on my phone without context-switching
- It serves as a log — I can scroll back to see last Tuesday's briefing

## The "Battle Rhythm" Add-On

After the morning briefing, a second cron job fires 15 minutes later with a "battle rhythm" check. This one's more operational:

- Any GitHub PRs or issues that need attention
- Pending tasks from project channels
- Reminders I've set
- Anything from the heartbeat system that flagged overnight

It posts to `#general` instead of `#daily-briefing` because it's conversational — I might want to act on something immediately.

## What I Tried That Didn't Work

**Full inbox dumps.** The first version listed every unread email. That was 30+ messages across accounts. Useless. The AI needed to filter, not just enumerate.

**Markdown tables.** They look great on desktop. They're unreadable on mobile Discord. Switched to bullet lists and haven't looked back.

**Too many check-ins.** I originally had hourly updates. By noon I was ignoring them. Four times a day on weekdays is the sweet spot — frequent enough to catch time-sensitive things, infrequent enough that each one gets read.

## The Cost

Each briefing is one agent turn — a single LLM call that gathers data and formats the output. At current API pricing, the daily briefings cost roughly $0.15-0.30 per day depending on email volume. About $6/month for a system that saves me 15-20 minutes every morning.

The real cost was the two hours of setup — configuring cron jobs, tuning the format, deciding what to include and what to filter. That investment paid for itself in the first week.

## Why This Matters

The briefing isn't impressive technology. It's a cron job, some API calls, and a language model that formats the output. What makes it valuable is that it runs without me thinking about it, it surfaces what matters, and it's there when I wake up.

The best productivity tool isn't the one with the most features. It's the one that does its job before you remember to ask.

---

*Cyberforks builds security workflows that don't suck. [cyberforks.com](https://cyberforks.com)*
