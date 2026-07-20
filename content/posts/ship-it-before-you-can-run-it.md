---
title: "Ship It Before You Can Run It"
date: 2026-07-19
draft: false
tags: ["aws", "iam", "least-privilege", "deployment", "reliability"]
summary: "I shipped a feature I didn't have permission to run — on purpose. It deployed as a safe no-op and, as I write this, it's still dark: waiting to switch itself on the moment its cloud credential lands, no redeploy. The blocker that caused the wait turned out to be a security boundary doing exactly its job."
---

I shipped a feature I did not have permission to run. That wasn't a mistake, and it isn't a workaround I'm sheepish about — it's a pattern I now reach for on purpose, and it turns an annoying class of blocker into a non-event.

The setup: I was building a small service, and one part of it calls a hosted image-generation API. That call needs a cloud credential the service didn't have yet, and getting that credential provisioned was blocked on a permission I didn't have yet — more on that shortly. The old instinct is to sit on the whole feature until the credential shows up. Instead I shipped it dark.

## Gate the feature, don't block on it

The move is a runtime capability check. Instead of assuming the credential is present, the service asks — at run time, each time it needs the credential — *can I actually do this right now?* If yes, it calls the real API. If no, it falls back to a safe, generic placeholder and carries on. No exception, no half-finished state, no blocked flow. It deployed as a safe no-op.

The property that makes this worth it is that it self-heals. Once the credential is provisioned, the next time the feature runs its check passes and it lights up on its own — no redeploy, no cutover, no coordinated release. The code is already in production, dormant, waiting for its dependency to arrive.

That decoupling — ship the code and provision the dependency in either order — is the whole point. Most features are quietly held hostage to their most annoying dependency: the one credential, the one permission, the one thing you're blocked on that lives outside your own code. A capability check plus a *real* fallback breaks the hostage situation, and it does it without a big-bang switch-on that you have to babysit.

## The permission that couldn't grant permission

Here's the specific wall I hit, because it's a good one. The credential I needed was an IAM role. When I went to create it, AWS said `AccessDenied` — even though the identity I was using had **PowerUserAccess**, which sounds like it can do anything.

It can't. PowerUserAccess is AWS's "full run of the services, none of the keys to the kingdom" policy: it lets you spin up compute, storage, queues, databases — nearly every service — but it withholds the IAM *management* that hands out authority, along with Organizations and Account control. The permission set literally named *PowerUser* is the one that can't grant power. It can operate the services all day, and it can *assume* a role that already exists — but it can't do the IAM that matters: no creating an arbitrary role, no attaching a policy, not even *passing* a role to the compute it just launched. About all the IAM it gets is reads and the narrow service-linked-role case. It can use power; it can't hand it out.

That felt like a bug for about thirty seconds, and then it landed as exactly the boundary you'd want. Creating IAM roles is the ability to grant privilege — including to yourself. If a PowerUser identity (or something that had stolen its credentials) could mint roles, "power user" would quietly mean "administrator," and least privilege would be fiction. The wall isn't in your way by accident; it's the line between *operating* the infrastructure and *changing who's allowed to operate* it, and those are meant to be different hands. The day your automation genuinely needs `iam:CreateRole` is the day it has outgrown PowerUser — and should be asking for a narrow, explicit, reviewed grant, not a bigger blanket.

## Graceful degradation is a security posture, not just a reliability one

Step back and the two halves of this are the same idea in different hats.

Two properties are riding together here, and the security-minded reader will want them pulled apart. The privileged *capability* fails **closed**: with no credential, the image call simply never happens — the service doesn't limp into a half-privileged state or retry its way into one. The *feature*, meanwhile, fails **soft**: rather than stop, it serves a harmless placeholder and carries on. One event, two lenses — the part that matters for security (don't attempt the privileged action you aren't authorized for) shuts, while the part that matters for the user (keep working) degrades quietly. A surprising amount of production pain is code that gets that pairing backwards: the capability limps on while the feature falls over.

The capability check is the habit that makes that possible: don't assume you hold a privilege — probe for it, and have a defined behavior for "no." It's the same discipline behind every well-behaved client that checks a token's scope before it acts, every job that confirms it can write before it deletes. Ask what you're actually allowed to do, not what you're hoping you are.

And the PowerUser boundary is the platform enforcing that same principle from the other side: it refuses to let a broadly capable identity quietly expand its own authority. Your code degrading gracefully and AWS withholding IAM from PowerUser are the same stance — least privilege and failing closed — meeting in the middle.

## The takeaway

Two things I'm keeping:

- **Decouple shipping the code from provisioning its dependency.** Gate the feature on a runtime capability check, give it a genuine safe fallback, and let it self-activate when the dependency lands. One dependency stops being able to hold a whole feature hostage.
- **When a permission surprises you, read it as a boundary before you read it as a bug.** PowerUser can't mint an arbitrary role because minting roles is how you hand out power, and that's supposed to take a different, smaller, reviewed key. The annoyance *is* the control working.

I shipped a feature I can't run. As I write this, the role still isn't provisioned — minting it takes exactly the kind of elevated, reviewed grant this whole post is about, and I haven't made that call yet. That isn't a loose end; it's the proof. The code has been sitting in production the whole time, dark and safe, serving its placeholder — and the day that deliberate step happens, the very next run flips it on with no redeploy from me. A feature that's *still* dark and *still* safe makes the case better than a tidy "and then it lit up" ever could.
