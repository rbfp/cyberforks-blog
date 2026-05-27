---
title: "Watching the watchers: how we observe a fleet of Claude Code daemons"
date: 2026-05-27
draft: false
tags: ["ai", "agents", "claude-code", "observability", "python", "open-source"]
---

*The tool we wrote to make `tail -F` work again, and what it taught us about running AI agents as infrastructure.*

Cyberforks runs a fleet of long-lived AI agents. They book-keep, draft posts, manage social media, scope client engagements, and keep me honest about whether I actually went to the gym this week. Each agent runs as a macOS LaunchAgent — a background daemon, persistent across reboots, listening to its own slice of Discord, reading and writing files, and using tools.

We call them GIR daemons. There are about a dozen at the moment. They're built on [Claude Code](https://docs.claude.com/en/docs/claude-code/overview), which is Anthropic's CLI agent — originally for software engineering, but capable of anything you can express as tool use.

For a small firm, this turns out to be a real operational layer. One agent does the books. Another runs marketing. A third handles security research. A fourth runs my D&D campaign because I have hobbies and it's my business what they are. They share a host, they share a model provider, but they don't share workspaces — each has its own personality, its own memory, its own Discord channels, its own mandates.

Running this is mostly easy. Watching it, it turns out, is the hard part.

## The problem: TUI logs are not log files

Claude Code is interactive. It draws a TUI — a terminal interface with a prompt, syntax highlighting, status bars, spinning indicators, the works. When you run it interactively, that's great: you see what's going on, the model thinks, tool calls scroll past, you read along.

When you run it as a daemon, the TUI has nowhere to go. So we wrap each daemon process in `script(1)`, the standard Unix command that records terminal sessions to a file:

```
/usr/bin/script -q ~/Library/Logs/claude-assistant-NAME.log -- claude
```

That file is, on its surface, the daemon's "log." Want to know what the agent is doing right now? Open the log. Want to find out what it said three hours ago? Scroll up in the log. Easy, except for one detail: **the file is not text.**

It's a recording of every byte the program sent to the terminal. Every cursor move, every clear-screen, every color change, every redraw. If you run `tail -F` on it, what you see is something like:

```
[H[2J[1;1H[7m...[0m
[?25l[1;1H[31mError:[0m something
[2;1H[K[2;1H[32mok[0m
[H[2J[1;1H[7m...
```

That's not a corrupted log. That's the actual data. Those `[H` and `[2J` and `[7m` sequences are cursor moves and screen clears — they only make sense to a terminal that's tracking position and state. A bare `tail -F` does no tracking, just dumps bytes in order. The result is mostly noise, occasionally with a recognizable word swimming in it. We tried.

We tried stripping ANSI codes with `sed`, which gave us readable text but lost the layout entirely — the agent would print a status bar across the top of the screen, overwrite a previous line three rows down, and the strip-and-concat result was an incoherent stream of fragments. We tried full terminal recorders like `asciinema`, which are great if you record interactively from the start, but don't help when you want to *look at the current state* of an agent that's been running for two weeks. We considered piping to a real terminal emulator and screenshotting it. We are not that kind of operation.

What we wanted was the world's simplest thing: **`tail -F`, but the screen instead of the bytes.**

## What we built: girlog

[girlog](https://github.com/rbfp/girlog) is a tiny terminal emulator wrapped in a CLI. It reads the raw byte stream, replays it into a virtual screen, and shows you the reconstructed image. Live, with colors, with cursor position, with the same shape the agent's actual TUI is drawing right now.

```
$ girlog
DAEMON       STATE        SIZE   LAST WRITE   LOG
assistant    up 3421     4.1MB    8m ago      ~/Library/Logs/claude-assistant.log
cyberforks   up 3422     1.7MB    1m ago      ~/Library/Logs/claude-assistant-cyberforks.log
dnd          stopped       890KB   2h ago     ~/Library/Logs/claude-assistant-dnd.log

Watch one live:  girlog <daemon>      Snapshot:  girlog <daemon> -s
```

```
$ girlog cyberforks --fill
```

That second command takes over your terminal and gives you the agent's screen, refreshed live as new bytes arrive. Ctrl-C to leave. If you want a still — for a screenshot, for a debug paste — `girlog cyberforks -s` renders the current screen once and exits. If you want plain text for grep, `-p` strips the ANSI on the way out.

Three things make this work:

1. **A small terminal emulator.** Around 300 lines of Python that tracks a fixed cell grid, a cursor, and an SGR (color/style) state. It handles the subset of escape sequences that real TUIs produce in practice: cursor moves, erase, insert, scroll regions, save/restore, colors. Not every quirk of every terminal type — just enough that the picture is right.

2. **A resync trick.** Replaying a multi-megabyte log from the top every time you want a snapshot would be slow. Instead, girlog seeks to the last 512 KB of the file and looks for the most recent clear-screen anchor (`ESC [ 2 J`, `ESC [ H`, etc.) — every time a TUI redraws from scratch, it leaves one of these. Replay from there, and you converge on the current screen in milliseconds. Use `--full` if you want to replay the whole history.

3. **Zero dependencies.** It's a single Python file. No pip install, no venv, no Docker. `curl`, `chmod`, drop it somewhere on `PATH`. That's the whole install:

```
curl -O https://raw.githubusercontent.com/rbfp/girlog/main/girlog
chmod +x girlog
mv girlog /usr/local/bin/
```

It works on any raw PTY recording, not just Claude Code. Any `script(1)` output, any `asciinema rec --raw`, any program that vomits ANSI into a file you wish you could read.

## What we actually use it for

Once you can *see* a fleet, the use cases pile up:

- **"Is the agent stuck?"** Glance at the screen. Spinning indicator means it's thinking, prompt visible means it's waiting on input, error modal means we have a problem. A live picture answers in two seconds what a log scrape would take minutes.

- **"What's the marketing agent up to right now?"** `girlog cyberforks-cmo --fill`. Watch in real time. This is how you discover that your agent is patiently waiting for a `permission_request` DM that you missed.

- **"Send me a screenshot."** `girlog DAEMON -s | pbcopy` and the snapshot lands in your clipboard, ready to drop into Slack. Way more legible than "trust me, the agent is broken."

- **"Grep across the fleet."** `for d in DAEMON1 DAEMON2 DAEMON3; do girlog $d -p | grep -i error; done`. Plain-text snapshot, regular grep, every daemon at once.

- **"What did the agent do an hour ago?"** Open the raw log in a real terminal emulator (`cat raw.log` works fine inside `iTerm2` or `Terminal.app`, because real emulators *do* track state), or use `girlog --full` to render the entire history into one virtual screen and snapshot it.

Most of the time we're not actively watching anything. The daemons just run. But when something goes sideways — and at 12 daemons running 24/7, something always eventually goes sideways — we have a way to *look at the patient* without invasive surgery.

## Why we open-sourced it

There are exactly two reasons.

First: this is a problem someone else clearly has, because we have it, and we are not special. Anyone running long-lived terminal-UI processes — agents, REPLs, monitoring dashboards, ssh sessions wrapped in `script(1)` — runs into the same wall. The fix is small enough that publishing it is cheaper than not.

Second: agentic AI operations is a discipline that doesn't have shared tooling yet. Every team running a fleet of daemons is currently reinventing the same primitives in private. We're going to keep building these tools as we need them, and we're going to keep publishing them. The footprint is small, the gravity is real, and the alternative is a thousand isolated re-implementations of the same shell script.

If you want it, it's at [github.com/rbfp/girlog](https://github.com/rbfp/girlog). MIT licensed. No telemetry, no analytics, no hosted backend. It's a Python file.

## Why this might be relevant to you

If you're running AI agents or thinking about it — for your firm, your back office, your client work — observability is the thing that turns "I built a cool thing" into "I run it in production." Anyone can spin up a daemon. Watching it, hardening it, recovering it when it fails, knowing what it actually did, deciding when it should ask permission — that's the real work.

Cyberforks does AI implementation with security baked in. We help offices in the LA area adopt AI tools — Microsoft Copilot, ChatGPT Enterprise, Gemini for Workspace — without the data-leakage, governance, and audit-logging gaps that come standard with "just turn it on." We write the policy, we configure the guardrails, we set up the observability, we make sure the AI doesn't quietly become your firm's largest unmanaged risk.

If that's a conversation you want to have, [we can have it](mailto:robert@cyberforks.com). The agents will be watching.

— Robert Cao, Cyberforks LLC

💀🐝
