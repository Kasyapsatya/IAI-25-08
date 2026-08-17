# Guide Notes — IP Semi-Markov Pricing & Experience-Rating Agent

Hi everyone. Let's talk about Income Protection coverage. In this notebook I'll walk you through an agent that prices IP coverage. As we know, this coverage kicks in when the insured isn't in a capacity to work due to sickness or injury, and makes income replacement payments.

First things first — the objective of this notebook is not to build a sophisticated pricing algorithm and show it off, but to build a rather simple algorithm that actually works, and see how we wrap it into an agent and unfold the magic. The data generation part is something we can walk through later when time permits.

Before we get to the tools, let's talk about the assumptions we make upfront, so the objective of each tool is clear.

## Assumptions

- **Incidence rates** — how often someone falls sick in the first place — vary by occupation, age, and prior sickness history.
- **Recovery timing** — how long someone stays sick, and whether that time is spent in the deferred period versus paid claiming — follows one shared duration model. We don't report this as a separate rate table, because pricing never draws on it directly; it shows up wherever it actually matters, which is severity: Tool 2's average claim duration and cost, adjusted further for prior sickness history by Tool 3.
- Premium is expected annually, at the start of each coverage year.
- The rate implicitly assumes premium is paid only while the policyholder is *not* claiming — since Tool 1's exposure base counts only Healthy and Sick(deferred) weeks, this is a waiver-of-premium design: no premium is charged while a claim is being paid.

Throughout, we'll keep the story split into two practical pieces — **frequency** (how often something happens) and **severity** (how long it runs, and how much it costs) — since that split is exactly how the three tools divide the work.

## Tool 1 — Frequency

Tool 1 calculates incidence rates for each age × occupation cell, by dividing the number of new sickness episodes by the total healthy (plus deferred) exposure years in that cell. This answers one question only: *how often does a healthy person in this age/occupation group fall sick?* It says nothing yet about how bad a claim turns out to be — that's Tool 2's job.

## Tool 2 — Severity, by deferred period

Tool 2 looks at the chosen deferred period and returns two things: **frequency** — what fraction of sickness spells actually go on to become a paid claim — and **severity** — how long those claims run on average, and how much they cost. A shorter deferred period lets more, typically shorter and cheaper, spells cross into paid claims; a longer one filters out all but the more severe spells, so fewer claims occur but each one that does is more expensive on average.

This assumes the underlying duration data isn't truncated by the deferred period historically in force. If it were, we'd apply the same restating logic, just limited to whichever deferred-period choices the data actually lets us observe — widening a deferred period from the historical one is generally safe to infer, narrowing it usually isn't, since we don't record exact recovery timing for spells that never became a claim.

## Tool 3 — Severity, by prior sickness history

Tool 3 looks at historical claims and compares the average severity of claimants with 0, 1, or 2+ prior sickness episodes against the total pooled average, blending that comparison with a credibility weight — more trust given to bands backed by more data — to return a loading factor for each band. This loading is itself a blend of frequency and severity: people with more prior episodes are both more likely to have a sickness actually turn into a paid claim, and more likely to have that claim run longer once it does. We don't try to separate those two effects — the loading captures both at once.

## Guardrails

We then have four guardrails that check whether the state, occupation, deferred-period option, or episode band being asked about actually exists in one of our three tools' tables. If it doesn't, the agent refuses to answer rather than inventing a number.

## Final premium formula

The final premium formula takes Tool 1's incidence rate for the policyholder's age and occupation, multiplies it by Tool 2's crossing probability and average severity for their chosen deferred period, scales that by their income relative to the portfolio average, and applies Tool 3's loading factor for their prior sickness history — giving the annual premium this individual has to pay.
