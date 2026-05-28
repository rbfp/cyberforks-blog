---
title: "What Happens When You Make an AI the Dungeon Master"
date: 2026-03-24
slug: ai-dungeon-master
tags: ["ai", "dnd", "automation", "openclaw"]
description: "Running Curse of Strahd with an AI DM — how it works, where it breaks, and why it's surprisingly good at dramatic tension."
---

# What Happens When You Make an AI the Dungeon Master

I run [OpenClaw](https://openclaw.ai) — a self-hosted AI agent that has access to my email, my calendar, my shell, and apparently now my Dungeons & Dragons campaign.

Yes. I made an AI the Dungeon Master for Curse of Strahd. Here's what happened.

## Why

Curse of Strahd is one of the best-written D&D modules ever made. It's a gothic horror sandbox set in a fog-drenched land ruled by a vampire count who is simultaneously the most compelling villain and the most insufferable ex in tabletop history. It deserves a DM who can hold the tone.

My group kept losing DMs to real life — jobs, moves, kids, the general entropy of adulthood. We kept getting three sessions in and stalling. I decided to stop waiting for a human who had the time and the commitment, and started building one that didn't need either.

## The Setup

The AI DM runs through OpenClaw with a dedicated Discord bot and its own set of channels: `#motd`, `#gameplay`, `#characters`, `#above-table`. The agent has the full Curse of Strahd source material in context, maintains persistent memory of what the party has done, and responds asynchronously. The fog of Barovia doesn't care about your work schedule.

## Where It Actually Works

**Atmosphere.** Gothic horror lives and dies by the prose, and the AI writes it well. The descriptions of Castle Ravenloft, the Amber Temple, the dead village of Barovia — they land. There's a particular kind of melancholy the model reaches for naturally that fits Strahd like a coffin lid.

**NPC consistency.** Strahd is hard to play because he's a tragic figure who believes himself the protagonist of his own story. The AI maintains that characterization across sessions with a consistency most human DMs struggle with.

**Rules adjudication.** The model knows 5e well enough to handle most combat and skill check situations without breaking the flow.

## Where It Breaks

**Player agency.** When players get genuinely creative — or genuinely chaotic — the DM sometimes retreats to safe responses. A human would lean into the chaos.

**Long-term consequence tracking.** Sessions accumulate. The AI's memory is good but not perfect, and Strahd is dense.

**Comedy.** The AI plays dramatic beats well but sometimes misses the timing on funny ones. A skilled human DM knows when to let the moment land.

## What I Actually Learned

Making an AI the DM is less about replacing a human and more about building a system that can hold a narrative open for people with limited time. For that specific problem, it works remarkably well.

The same properties that make AI assistants useful for security work — consistent rule application, persistent memory, 24/7 availability — make them surprisingly good at running tabletop games. Same capability, different problem.

Barovia will still be there when you get back. Strahd is patient. He's been waiting 400 years. Your schedule doesn't impress him.

---

Built with [OpenClaw](https://openclaw.ai). Repo: [rbfp/openclaw-ai-dungeon-master](https://github.com/rbfp/openclaw-ai-dungeon-master)

*Cyberforks builds things that probably shouldn't exist but definitely should. [cyberforks.com](https://cyberforks.com)*
