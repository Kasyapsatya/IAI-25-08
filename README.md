Workshop notebooks for the IAI session "AI for Actuaries — From Foundations to AI Agents" (25 August 2026, Delhi).


IP agent:

Hi everyone. Let's talk about Income Protection coverage. In this notebook I'll walk you through an agent that prices IP coverage. As we know, this coverage kicks in when the insured isn't in a capacity to work due to sickness or injury, and makes income replacement payments.

First things first — the objective of this notebook is not to build a sophisticated pricing algorithm and show it off, but to build a rather simple algorithm that actually works, and see how we wrap it into an agent and unfold the magic. The data generation part is something we can walk through later when time permits.

Before we get to the tools, let's talk about the assumptions we make upfront, so the objective of each tool is clear.

We assume:

Incidence rates — how often someone falls sick in the first place — vary by occupation, age, and prior sickness history.
The flat exit rates governing how quickly someone typically recovers are pooled and treated as the same for everyone — a known simplification. Claim severity, though, is not the same for everyone: it depends on the deferred period chosen and on prior sickness history, which is exactly what Tools 2 and 3 exist to price.
Premium is expected annually, at the start of each coverage year.
The rate implicitly assumes premium is paid only while the policyholder is not claiming — since Tool 1's exposure base counts only Healthy and Sick(deferred) weeks, this is a waiver-of-premium design: no premium is charged while a claim is being paid.

Tool 1 calculates incidence rates for each age × occupation cell, by dividing the number of new sickness episodes by the total healthy (plus deferred) exposure years in that cell.

Tool 2 looks at the chosen deferred period and returns the appropriate severity components: what fraction of sickness spells actually go on to become a paid claim, and the average claim cost once they do. A shorter deferred period lets more — typically shorter, cheaper — spells cross into paid claims; a longer one filters out all but the more severe spells, so fewer claims occur but each one is more expensive on average. This assumes the underlying duration data isn't truncated by deferred period; if it were, we'd apply the same logic, just limited to the deferred-period choices we actually have data for.

Tool 3 looks at historical claims and compares the average severity of claimants with 0, 1, or 2+ prior sickness episodes against the total pooled average, blending that comparison with a credibility weight — more trust given to bands backed by more data — to return a loading factor for each band.

We then have four guardrails that check whether the state, occupation, deferred-period option, or episode band being asked about actually exists in one of our three tools' tables. If it doesn't, the agent refuses to answer rather than inventing a number.

The final premium formula takes Tool 1's incidence rate for the policyholder's age and occupation, multiplies it by Tool 2's probability-of-claiming and average severity for their chosen deferred period, scales that by their income relative to the portfolio average, and applies Tool 3's loading factor for their prior sickness history — giving the annual premium this individual has to pay.