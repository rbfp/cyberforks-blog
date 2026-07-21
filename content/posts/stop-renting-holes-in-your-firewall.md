---
title: "Stop Renting Holes in Your Firewall"
date: 2026-07-20
draft: false
tags: ["networking", "zero-trust", "ssh", "security", "homelab"]
summary: "I spent an afternoon chasing a firewall rule that kept breaking, and the fix that finally stuck was deleting the port behind it off the public internet entirely. Source-IP allowlists are a maintenance treadmill that quietly rots into someone else's access — here's the inversion that ends it."
---

I spent an afternoon fixing a firewall rule, and the fix that finally stuck was deleting the rule — and the port behind it — off the public internet entirely. The best allowlist entry, it turned out, was no open port at all.

The box is a personal game server; it hosts dice for friends, nothing dramatic. But the way it broke, and the way I stopped it from breaking again, is a small clean lesson about what "locked down" actually means.

## The rule that aged into a lie

Routine SSH to the box, one afternoon, just hung — a plain TCP timeout. The server was up, DNS resolved, the game was serving fine over HTTPS. The thing in the way was my own firewall rule: SSH was restricted to a single source IP — my home connection — and that public IP had changed out from under me. Lease churn, a modem cycle, a provider reshuffle; it doesn't matter which. The rule was still there, still enforced, even helpfully *labeled* with my machine's name. The label had outlived the truth. It pointed at an address that was no longer mine.

I swapped the rule to my current IP and moved on. Correct fix. That part worked.

## The red herring

Then I retried, and it timed out *again* — and I made the classic mistake: I assumed the thing I'd just been debugging was still the problem. More firewall, surely. It wasn't. The server had an idle timeout to save money, and it had quietly powered itself off in the gap between my fix and my retry. I was debugging *reachability* to a machine that wasn't *running*.

That's worth stating plainly, because it's cost me hours more than once: **before you debug why you can't reach something, confirm the something exists.** Is it on? A stopped host and a blocked port both present as a timeout, and the wrong assumption sends you rewriting network rules to reach a box that's simply asleep. Check state before you touch the network.

## The treadmill

With the box awake and the new rule in place, SSH worked. But the whole episode left a bad taste, because the *fix* was the disease. Pinning SSH to a source IP is security by mailing address — and residential IPs move. Every pinned rule is a future outage waiting on the next ISP lease renewal, plus a stale artifact nobody remembers to clean up.

And that stale artifact is the part that should actually worry you. That IP went back into my provider's pool, destined for whoever's modem comes next — and until I noticed, my rule was granting **network reachability to my SSH port** to an address I no longer controlled. Whoever inherits that IP skips the one filter I'd built; SSH's key becomes the only thing left in their way, where a moment before my network rule had stood too. I have no reason to think anyone knocked — but "defense-in-depth that quietly drops to one layer for whoever inherits my old address" is not a sentence you want true about your infrastructure. A source-IP allowlist doesn't just rot into downtime — it rots into *reachability for a stranger.*

## The fix I'd already half-shipped and forgotten

The real fix wasn't another rule. It was getting rid of the public door.

The obvious next question — why is this box reachable on the public internet at all? — sent me scoping a move onto a private mesh network: the kind where machines authenticate to each other by cryptographic identity instead of by where they're connecting from (I used Tailscale; any WireGuard-based mesh does the same). And in the middle of scoping it, I found a note in my own runbook, two days old, in my own handwriting: the box had *already joined the mesh.* The migration I was carefully planning was ninety percent done, and I'd forgotten I'd done it.

I'll leave that in because it's the truest part of the story. The best security move was sitting there, already built, waiting for me to notice — which is its own small lesson about how often "do the hard thing" is really "finish the thing you started."

## Deleting the door

So I finished it, that same afternoon. Repointed my SSH config at the machine's mesh address — a stable identity that travels with the box no matter what my connection does — then **deleted port 22 from the public internet** and checked all three: mesh path up, public port dark, the game still serving over HTTPS. Not restricted. Deleted. The public attack surface for remote login went from "one IP I have to keep chasing" to zero.

That's the inversion that matters. Source-IP allowlisting trusts *where a connection comes from* — a fact you don't control and can't keep current. Identity-based mesh access trusts *what the machine can prove it is* — a credential that moves with it. The first is a treadmill: curate the list, chase the churn, clean up the corpses. The second deletes the list. There's nothing on the public internet to allowlist because there's nothing on the public internet to reach.

## What I kept

Three things, from one afternoon and one dice server:

- **A rule labeled with a fact ages into a lie.** "Allow my home IP" was true the day I wrote it and false by the time it bit me — and the label made it look trustworthy the whole way down. An access rule that encodes a mutable fact needs an expiry, or it needs to not encode the fact at all.
- **Check existence before reachability.** A stopped host and a closed port look identical from outside. Confirm the thing is running before you rewrite the network to reach it.
- **The best hole in your firewall is the one that isn't there.** You can spend forever curating who's allowed through port 22, or you can take port 22 off the public internet and let identity — not IP address — be what gets in. One of those is maintenance forever; the other is done.
