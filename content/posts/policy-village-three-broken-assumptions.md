---
title: "Three Talks, Three Broken Assumptions"
date: 2026-09-11
draft: false
author: "Christine C."
tags: ["defcon", "policy", "ransomware", "incident-response", "vulnerability-management"]
summary: "Policy Village at DEF CON 34 ran three separate sessions on three unrelated problems. All three turned out to be the same problem: rules calibrated to an assumption that stopped being true."
---

Policy Village at DEF CON 34 put on three sessions I sat through that had nothing to do with
one another. One on ransomware economics. One on infrastructure incident policy. One on
vulnerability management in the age of cheap discovery.

Different speakers, different problems, no coordination between them. And all three landed on
the same structural complaint: **the rule is calibrated to an assumption that is no longer
true, and nobody has gone back to check the assumption.**

That is worth writing down, because it is a diagnosis you can carry into a room where nobody
is talking about ransomware or CIRCIA at all.

## One: the assumption that prohibition changes behavior

*"The best of three bad ideas? Ransomware taxes in lieu of bans or doing nothing" — Joe
Uchill, RAND, formerly Axios, The Hill, and Motherboard.*

The title ends in a question mark and that turns out to be the honest punctuation. Uchill held
up three options for dealing with ransomware, conceded all three are bad, and spent his time
on which one fails most gracefully.

A **ban on ransom payments** has the appeal of a clean mechanism: no payments, no business
model. The costs land on people already having their worst week. Prolonged outages in sectors
where "stays down longer" is not an abstraction — a hospital, a water utility. Revictimization,
where the organization that just got attacked is now also the organization breaking the law.

And the third cost, which deserves more attention than it gets. Under a ban, some victims pay
anyway. The moment they do, they have handed the attacker a second thing to extort them over.
Not the data — the payment. **A ban converts a quiet transaction into leverage and hands that
leverage to the person who caused the problem.** The policy meant to end the extortion invents
a fresh round of it.

A **ransomware tax** works on price instead of prohibition. Attackers already set ransoms near
the victim's walk-away point. Add a tax and the total crosses that line, so the victim walks.
If attackers adapt by cutting the ransom so ransom-plus-tax lands back at the walk-away point,
the payment goes through but the attacker takes home less. Fewer payments, or smaller ones. No
version where the attacker's economics improve.

That reaches past any single transaction, because ransomware is a supply chain — access
brokers, developers, affiliates, all paid from the same pot. Shrink the pot and you shrink the
incentive to specialize into it.

The unanswered question is who actually pays. If it lands on the victim, the deterrent bites
and so does the second injury. If insurers absorb it, as they already absorb much of the ransom
itself, the deterrent is swallowed by the same mechanism that quietly made payment the path of
least resistance. **A tax that insurance pays is a tax the victim never feels.**

The broken assumption: that making something illegal changes whether it happens, rather than
changing who has leverage when it does.

## Two: the assumption that incidents are events

*"Assume Breach, But For Real: Rewriting Infrastructure Policy for the Adversaries Who Never
Left" — Travis Berent (Microsoft, formerly NSC and FBI) and Adam Hickey (Mayer Brown, formerly
a decade running DOJ's national security cyber program).*

An energy provider learns it has been breached when a control-room operator notices a setpoint
that changed for no reason. Not an alert. A person looking at a number that should not have
moved.

Initial access happened ninety days earlier through a contractor credential that outlived the
contract it was issued for. By then the disclosure clocks are already running, the scope is
unknown, and the systems cannot go offline because they are the systems keeping the lights on.

Cyber policy rests on three premises about incidents: that they are **detectable** in
reasonable timeframes, **attributable** with confidence, and **containable** once identified.
Incident response says otherwise on all three.

Actors like Volt Typhoon do not deploy malware that trips signatures. They use valid
credentials and the tools already on the box, doing what those tools are supposed to do. There
is no payload to catch — there is an authorized account behaving plausibly. Detection lags by
weeks or months and usually arrives through operational impact rather than telemetry. **That is
not a missed alert. It is the absence of an alert to miss.**

Which breaks disclosure deadlines specifically, because the clocks are aligned to the calendar
rather than to the investigation. CIRCIA's seventy-two hours runs from when an entity
*reasonably believes* a covered incident occurred, not from when it confirms one. The SEC's
four business days run from a materiality determination made on partial information. All of
them start while visibility is incomplete and scope is a guess.

The result is not better information. It is three kinds of bad information: **undercompliance**,
where something genuinely reportable is missed in the fog; **overcompliance**, where everything
is reported defensively and the signal drowns; and **hedging notification**, where a filing
satisfies the clock while communicating almost nothing. A rule demanding an answer before an
answer exists does not produce an earlier answer. It produces a worse one.

What the panel said actually works was not more process. Two capabilities carried the weight:
**segmentation**, which decides whether an intrusion stays local or becomes systemic, and
**monitoring**, which decides whether you learn from telemetry or from a confused operator.

The broken assumption: that a breach is an event with a start time you can report against. The
contractor credential was not an event. Nothing was exploited — no vulnerability, no malware,
no zero-day. An account that should have been deactivated was used exactly as intended, for
ninety days, while every control registered authorized activity by a valid user. **That is not
an event. It is a condition, and it was true the entire time.**

## Three: the assumption that finding bugs is expensive

*"From Disclosure to Defense: Rebuilding Vulnerability Management for the AI Era" — a panel
with Lindsey Cerkovnik (CISA), Elizabeth Eigner, John Banghart, and Ben Flatgard.*

Every part of how we handle software vulnerabilities — disclosure etiquette, triage queues,
patch windows, the policy scaffolding around all of it — was designed for a world where
finding a bug was the expensive part. Careful, methodical process made sense when the input
arrived in a trickle.

The panel's argument was not that AI created a crisis. It is that AI **exposed** one: these
institutions were quietly built on an assumption of scarcity that no longer holds.

The number that makes it concrete. When the panel met on August 8, the reference point was
July 2026 — the largest Patch Tuesday Microsoft had ever shipped. Roughly 570 by Tenable's
count, 621 by ZDI's, 622 by Microsoft's own, and the fact that three careful organizations
cannot converge on one figure is itself part of the story. June had set the previous record at
206. July roughly tripled it.

There is no version of thoughtful, methodical triage that survives contact with 600
vulnerabilities in thirty days. Not because anyone got lazy. Because the arithmetic stopped
working.

What the panel could not know is that July's record would last about eight weeks. August came
in lower — 398 to 457, depending on the ledger. Then September landed around 966 to 974: the
largest month-over-month jump in Patch Tuesday history.

The August dip is the part that makes the case. The trend is not a clean line upward; it is
lumpy, and any month can come in under the last. But the **floor** moved. August was a quiet
month by 2026 standards and still roughly double June's all-time record from eight weeks
earlier.

And the qualifier that keeps this a process story rather than a doom story: **the cost of
finding vulnerabilities collapsed, but exploitation has not risen to match.** Those curves came
apart. Discovery scaled immediately. Exploitation still requires targeting, infrastructure, and
follow-through — work that has not gotten proportionally cheaper. The pressure on defenders is
a volume problem, not a danger problem. The queue got enormous. The queue did not get uniformly
more lethal.

Which is why the institutional answer is about sorting rather than panicking. CISA's **BOD
26-04** abandons flat deadlines and scores a vulnerability against four questions: is the
affected thing internet-reachable, is it already being exploited, can the attack be automated,
and what does the attacker actually get. Hit all four and the clock is three days. It binds
federal civilian agencies only, and the four questions travel well beyond government.

The panel also argued something less comfortable: **the disclose-immediately reflex has stopped
paying for itself.** That reflex made sense when a published vulnerability still took real work
to weaponize — early disclosure bought defenders lead time. If finding and validating bugs got
cheap for defenders, it got equally cheap for attackers, and a published unmitigated
vulnerability is a much shorter path to a working exploit than it used to be. Rushing
disclosure before you can act on it does not accelerate the fix. It publishes the roadmap
first.

Not an argument for secrecy. A narrower and harder one: disclosure timing has to become a
judgment about whether you have enough information to be useful, rather than a race.

The broken assumption: that severity ranks priority. CVSS was built to sort a manageable number
of things. When the number stops being manageable, a severity score is not a priority — it is
one input among several, which is exactly what BOD 26-04's four questions encode.

## The pattern

Three sessions, three domains, one shape.

| The rule assumes | What is actually true |
|---|---|
| Prohibition changes whether payment happens | It changes who holds leverage when it does |
| An incident is an event with a start time | It is often a condition that was true all along |
| Finding vulnerabilities is the expensive part | Deciding which ones matter is the expensive part |

None of these are failures of intent. Each rule was calibrated correctly to a world that
existed when it was written. The failure is that nobody went back to check whether the
calibration still held, and the assumptions are buried deep enough that the rules look
reasonable right up until you try to follow one during an actual incident.

Which suggests a question worth asking about any control you are subject to, or any you
impose: **what did this assume about the world, and is that still true?** Most of the time
nobody has checked. That is not cynicism about policy. It is the most useful thing I took out
of a week in Las Vegas.
