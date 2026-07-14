---
title: "The $0 Game Master"
date: 2026-07-13
draft: false
tags: ["ai-agents", "side-projects", "cost", "infrastructure"]
summary: "A real-time multiplayer game your friends play from their phones, narrated by an AI running on a laptop — for essentially nothing per turn. How the 'zero' works, and why the model was never the expensive part."
---

A friend in another city opens a link on their phone. A map loads — a candlelit tavern, their character token standing by the door. They type what they want to do. A few seconds later a voice narrates the scene back to them: the smell of the hearth, the innkeeper looking up, a die already rolling behind the words.

The game master running that game — narrating, rolling, tracking the world, handling combat, keeping four players straight — is not a service I pay per message for. It's an AI daemon — [one of a few I keep running](/posts/fleet-fixed-its-own-tooling/) — on a MacBook on my desk. And each turn of that game costs, to a very good approximation, **zero dollars**.

This post is how that works, and why the "zero" is the interesting part.

## Why it's free

Most AI products are metered. You call a model API, you pay per token, and a chatty application becomes a running bill. A live game — where every player action is a fresh generation of narration, dice resolution, and world updates — would burn tokens continuously.

The trick here is that the brain isn't a metered API at all. It's a **persistent AI process running under a flat-rate plan I already pay for** — the same subscription I use for everyday work. From the game's perspective, generating a turn of narration costs the same as the process sitting idle: nothing marginal. The meter isn't running because there is no meter.

Everything *around* the brain is chosen to stay at zero too:

- The public URL players hit is fronted by a **free tunnel** — no domain-hosting bill, no load balancer.
- The always-on piece that holds game state and serves the web client is a **tiny cloud box** small enough to live in a free or near-free tier, and it sleeps when nobody's playing.
- The connection from that box back to the laptop is a **private encrypted tunnel** — also free.

So the whole cost structure is: a subscription I'd pay anyway, plus infrastructure that rounds to zero. The marginal cost of one more turn, one more session, one more player, is nothing.

## The path a turn takes

Here's the trip a single player action makes, start to finish:

1. **Phone → public URL.** The player's browser talks to a normal web address, fronted by the free tunnel.
2. **→ the cloud box.** A small always-on relay serves the game client and holds the authoritative game state — the map, the tokens, whose turn it is.
3. **→ private tunnel → the laptop.** When a turn needs a *decision* — narration, a skill check, a combat resolution — the relay hands it across an encrypted tunnel to the Mac.
4. **→ the daemon.** A persistent AI process reads the turn: the scene, the character, the action. It writes the narration, resolves the dice, updates the world.
5. **→ all the way back.** The result flows back out to the relay, out through the tunnel, and onto the player's screen — as text, and as spoken audio.

No player ever knows a laptop is involved. From their phone it's just a game that answers.

## The actually-hard part

The economics are a trick. The engineering is a real problem, and it's this: **a persistent, interactive AI process is not naturally a service.**

The daemon I'm using is built to be *talked to* — an open-ended assistant, not a request/response endpoint. Turning it into something a game can *call*, one turn at a time, reliably, meant building a bridge: a small message channel the game posts a turn into, and the daemon answers back through, in a structured shape the game can render. The brain proposes the narration and the dice; the machinery around it enforces the turn order, the map, the rules.

Getting a headless, always-on AI to behave like a dependable backend — to wake for a request, answer in a fixed format, and go quiet — was most of the work. The model was never the bottleneck. The *plumbing* to make a conversational thing act like a service was.

Once that bridge existed, the rest of a real game followed: a consistent narrator voice, real dice a player can roll from the screen, turn-based combat on a rendered map, multiple players sharing one game master, and text-to-speech so the story is heard, not just read.

## Why it matters

The lesson isn't "run your production app on a laptop." It's a hobby rig — one brain, it naps when idle, and there are rough edges I'm still smoothing. Don't put your business on it.

The lesson is about **where the cost of an AI product actually lives.** For most builders it's the model bill, and that bill is what makes ambitious ideas — a game that generates every scene live, an assistant that never sleeps — feel financially impossible for one person. But if the brain runs on a flat plan you're already holding, that number collapses. What's left is just plumbing, and plumbing you can learn.

Flip that one assumption and the space of things a single person can build gets a lot bigger. A friend playing a live, AI-narrated game from their phone, for free, is a small proof of a larger point:

Your laptop can be the brain behind a real-time app people actually use. The model was never the expensive part. It was just the part everyone else is still paying for.
