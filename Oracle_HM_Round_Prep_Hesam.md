# Hiring Manager Round Prep - Hesam Fathi Moghadam (M4)

Wed Aug 12, 10:00-11:00 AM | Tags: Team Fit, Product Roadmap, Put Customers First

---

## What this round actually is

Per Oracle's own prep pack and cross-referenced Glassdoor/Blind data: this is 45-60 minutes of STAR-format behavioral questions, likely opening with "tell me about yourself / your background" and a "why OCI / why this team" discussion, then digging into 2-3 real situations per competency tag. Some HM rounds also include a light design or "how would you approach X" prompt (one Glassdoor reviewer got "design an e-commerce platform" cold from an HM) - be ready to think out loud on a system-level question even though this isn't Arshdeep's dedicated design round.

Oracle's STAR framework (from their own prep pack): Situation, Task, Action, Result. Always land on Result with a number or a concrete outcome, and be ready for "what would you do differently" - Oracle explicitly asks this.

---

## Opening: "Tell me about yourself"

**This is very likely the literal first question of the entire loop, not just this round - have it ready cold, 60-90 seconds spoken.** The structure: current role first, then chronological background, then the thread connecting it all, landing on why this specific role. Don't bury the "why OCI" part inside this - it's the natural close, not a separate thing to remember to add later.

**Full narrative:**

"I'm currently a Founding AI Software Engineer at LiliSolutions, where I've spent the last several months building production agentic AI systems end to end - a RAG-based compliance assistant, a HIPAA-aware voice bot, and a GCP-native alerting platform.

Before that, my path was enterprise engineering into applied AI research. I spent four years at Colgate as a backend engineer and later technical lead, on a platform serving 15 countries and 5 million daily API calls - that's where I got deep, hands-on experience with the less glamorous but critical work of production reliability: caching architecture, database performance, event-driven messaging, and later leading a major data migration.

From there I went to Oregon State for two research roles. One was BugSleuth, a fault-localization tool that got submitted to ICSE 2025. The other was EphFlow, a published systems paper extending an open-source serverless framework with a novel VM orchestration mechanism. That research period is really where I built the habit of being rigorous about claims and evaluation - not just shipping something that works, but being able to defend exactly why it works and what its actual limits are.

That habit is what's carried directly into my current work at Lili, where I've built real evaluation gates, security boundaries, and observability into every system I've shipped, in a regulated healthcare domain where getting it wrong has real consequences, not just a bad demo.

That combination - enterprise-scale engineering discipline, research-level rigor, and hands-on production AI systems work - is what draws me to this specific role. OCI's AI Innovation team is the first place I've seen agent execution, evaluation, and AgentOps treated as a first-class engineering discipline rather than an afterthought, and that's the work I want to be doing next."

**Q. "Why leave Lili so soon?"**

A: "Lili's been a great place to build - I've had real ownership over production systems from day one, and I'm proud of what's shipped. But it's a small team, and there's a ceiling on how much AgentOps maturity, platform-scale infrastructure, and evaluation tooling I get exposed to there, just given the size. I'm looking for the next step to be somewhere agent execution and evaluation are treated as their own engineering discipline, with the platform maturity to build things other teams then build on top of - that's a different kind of problem than what a small team building its own product day to day gives you, and it's the specific gap I'm looking to close."

**Q. "Why OCI AI Innovation specifically?" (as a standalone follow-up, not part of the full introduction)**

A: "OCI's AI Innovation team is the first place I've seen agent execution, evaluation, and AgentOps treated as a first-class engineering discipline rather than an afterthought - not just shipping a model integration and calling it done, but real infrastructure around observability, evaluation gates, and safe rollout for agentic systems. That's exactly the kind of platform-level problem I want to be working on next, and it's a natural extension of what I've already been building at smaller scale."

---

## Competency 1: Put Customers First

Oracle's own sample prompt: *"Tell me about a time you've had to prioritize your customers' needs. What was the situation? What would you do differently knowing what you know now?"*

### Primary story: The voice bot's eval-harness-gates-CI decision

**Situation:** Building the HIPAA-aware patient collection voice bot, the fast path to a demo would have been to ship the conversation flow and iterate based on real call outcomes.

**Task:** As the engineer responsible for both the orchestrator and the compliance gates, I had to decide how much evaluation rigor to build before any real patient interaction, knowing every week of extra harness-building delayed the pilot.

**Action:** I built a 108-scenario eval suite covering identity verification failures, opt-out mid-call, ambiguous patient responses, and payment negotiation - and gated it in CI so a regression on any critical scenario blocks merge. That's slower than shipping and monitoring in production, but the customer here is a patient in a regulated healthcare interaction, and "we'll catch it in production" isn't an acceptable failure mode when the surface is PHI and TCPA compliance, not a UI bug.

**Result:** The pilot practice got a system where a bad prompt change literally cannot reach a real patient call without failing a critical scenario first. That's slower time-to-ship but it's the trade a regulated domain demands - doing right by the patient outweighed doing what was fastest for the roadmap.

**What I'd do differently:** I'd build the critical/non-critical scenario split from day one rather than discovering which regressions actually mattered after the harness was already large - I initially over-indexed on coverage breadth before I'd thought hard about severity tiering.

### Backup story 1: Colgate API contract - additive-only backward compatibility

**Situation:** As the sole owner of the REST API contract serving six enterprise consuming services, I was under pressure from one team to ship a breaking change quickly to unblock their roadmap, rather than going through a slower, formal versioning process.

**Task:** Decide whether to prioritize that one team's speed, or protect the other five teams from a change they hadn't asked for and weren't prepared to absorb.

**Action:** I held to additive-only versioning with a formal deprecation window, rather than the faster path of a breaking change with a heads-up email. That meant proposing a parallel versioned endpoint instead - the requesting team got their new shape without the other five services being touched at all, and I compressed the deprecation timeline specifically for their use case rather than making them wait for my standard window.

**Result:** The requesting team hit their deadline on the new version, and none of the other five services saw any disruption - a customer-facing outage during a version cutover was the risk I was protecting against, not something I was willing to accept as a reasonable cost of moving fast for one team.

**What I'd do differently:** I'd propose the parallel-versioned-endpoint compromise earlier in the conversation rather than leading with why my policy existed - leading with the constraint before the solution is what made it feel like resistance at first, rather than a real conversation about tradeoffs.

### Backup story 2: Tome Raider's honest-fallback design (Put Customers First applied to a personal project)

**Situation:** Building Tome Raider's retrieval loop, the easy path after a failed rewrite-and-retry cycle would be to answer anyway with whatever context was retrieved, since a confident-sounding wrong answer often reads better to a user in the moment than an admission of not knowing.

**Task:** Decide what the system does when it genuinely can't answer well - optimize for looking capable, or optimize for being trustworthy.

**Action:** I capped the rewrite loop at 2 retries and built an explicit honest-fallback path that tells the user the available documents don't contain enough information, rather than generating a plausible-sounding answer from weak context. That's a deliberate trade against the metric that's easiest to optimize for in a demo - "the system always has an answer" - in favor of the one that matters for a real user relying on the system for accurate information.

**Result:** A user of the system can trust that an answer it gives is actually grounded, because the alternative to a grounded answer is an explicit "I don't know," never a confident guess. That's the same design principle a compliance or regulatory RAG deployment would need even more strictly.

**What I'd do differently:** I'd instrument how often the fallback actually fires in practice - right now I know the mechanism exists and works, but I don't have real usage data on how often users hit it, which is the next thing I'd want to know before calling this "good enough."

---

## Competency 2: Team Fit

Oracle's own sample prompt: *"Give an example illustrating there's more than one way to solve a problem."* / *"Describe a work situation where you faced a big obstacle."*

### Primary story: Colgate team disagreement resolution (real anchor)

**Situation:** As Technical Lead directing 6 engineers, two of them disagreed on the approach for one of the three major platform features I was accountable for.

**Task:** Resolve the disagreement without either defaulting to seniority (mine or theirs) or letting it drag and stall the feature.

**Action:** I asked both engineers to bring a concrete tradeoff - latency, complexity, or failure mode - rather than a stated preference. Most of the time the data made the choice obvious once it was on the table instead of staying abstract. Where it genuinely didn't resolve cleanly, I made the call and owned the outcome, because letting ambiguity drag cost the team more than a wrong-but-decided direction would have.

**Result:** The team kept moving, both engineers stayed engaged because the process was evidence-based rather than "because I said so," and it became the default way disagreements got resolved on the team going forward.

**What I'd do differently:** Earlier in my time as lead I let one or two of these disagreements run longer than they needed to before stepping in - I'd trust my instinct to intervene sooner now.

### Backup story 1: Cross-discipline collaboration on EphFlow (research + domain science)

**Situation:** EphFlow's FLARE case study meant working directly with ecological forecasting researchers at Virginia Tech, who had zero interest in FaaS internals - they needed their 1.5-hour lake simulation to just run reliably, and had no context on serverless execution constraints at all.

**Task:** Get to a shared understanding of what the system could and couldn't do, across a genuine domain-expertise gap, without either overwhelming them with infrastructure detail they didn't need or oversimplifying to the point where they couldn't make informed decisions about their own workflow.

**Action:** Rather than explaining the system in my own vocabulary - RequiresVM flags, DAG augmentation, execution substrates - I translated every technical constraint into terms tied directly to their actual problem: not "this action needs RequiresVM true" but "your simulation won't time out anymore." I spent time understanding what they actually needed to know to trust the system, rather than what I found interesting to explain about how I'd built it.

**Result:** The collaboration produced a real published result - the FLARE case study became the paper's core validated example - and the researchers were able to actually use and trust a system whose internals they never needed to understand in my terms.

**What I'd do differently:** Earlier in the collaboration I defaulted to my own technical vocabulary before catching myself and translating - I'd start from their frame of reference from the very first conversation now, rather than translating after the fact.

### Backup story 2: "Describe a work situation where you faced a big obstacle" - EphFlow's Lambda timeout wall (Oracle's own sample prompt)

**Situation:** The FLARE ecological forecasting workflow needed to run a 1.5-hour lake-dynamics simulation, but our initial deployment target, AWS Lambda, has a hard 15-minute execution ceiling and a 10GB memory limit - the simulation blew past both.

**Task:** Solve this without abandoning the serverless model entirely (which the whole project was built around) or compromising forecast accuracy by algorithmically simplifying the science to fit the platform's constraints.

**Action:** Rather than accepting one of the conventional workarounds - persistent EC2 management, which reintroduces the server-management burden serverless was supposed to remove, or time-segment checkpointing, which complicates the science code - I designed the ephemeral VM orchestration mechanism: actions can be flagged as needing VM resources, and the system automatically injects VM lifecycle actions into the DAG at registration time, so the resource-intensive part runs on a real VM while everything else stays serverless.

**Result:** The forecast actions ran for their full 1.5 hours on an EC2 instance with no timeout or memory constraint, while data-prep and visualization steps stayed on Lambda at 30-90 seconds each - the workflow now runs successfully on a daily schedule, and this became the paper's core novel contribution, not just a one-off fix.

**What I'd do differently:** I'd have explored the VM-orchestration approach earlier - we spent real time on the conventional workarounds before concluding they each compromised something important enough to justify building the more ambitious solution instead.

---

## Competency 3: Product Roadmap

**No sample Oracle prompt for this one specifically - construct from likely phrasing: "how do you decide what to build," "tell me about a time you said no to a feature," "how do you handle competing priorities."**

### Primary story: Scoping the eval harness severity tiers (prioritization under time pressure)

**Situation:** Building the voice bot's eval harness, I could have kept expanding scenario coverage indefinitely - there's no natural stopping point for "what could go wrong on a patient call."

**Task:** Decide what actually needed to ship for the staging pilot versus what was gold-plating that would delay the roadmap without proportionate risk reduction.

**Action:** I split scenarios into critical (blocks merge on regression) and non-critical (visible, doesn't block) rather than treating all 108 as equally load-bearing, which meant making an explicit judgment call about which failure modes were actually dangerous - a patient not getting a payment link is an inconvenience; a patient's PHI leaking or an unauthorized call outside consent windows is not comparable at all.

**Result:** The team could keep shipping conversation-flow improvements at a normal pace instead of the harness itself becoming the bottleneck, while the genuinely dangerous failure modes stayed hard-gated.

**What I'd do differently:** I'd document the severity-tiering rationale more explicitly up front so it's a repeatable framework for the next feature, not a judgment call I made once and now have to re-explain.

### Backup story 1: Colgate SAP migration prioritization

**Situation:** Leading the SAP-to-Snowflake data migration, I had a choice between a faster append-only migration path that would ship a few days sooner, and a slower, idempotent upsert-by-natural-key design that took longer to build.

**Task:** Decide whether shipping speed or backfill safety mattered more for a migration moving financial and operational data at scale, knowing the roadmap had pressure to move quickly.

**Action:** I chose the idempotent upsert design, deliberately giving up a few days of speed, because an append-only path meant any backfill or replay after a failure would risk duplicate or inconsistent records - a risk I judged worse than the delay, given what the data was actually used for downstream.

**Result:** That decision paid off directly - a backfill did need to be replayed later, and it cost zero extra engineering time, because the idempotent design already handled it safely. An append-only design would have turned that replay into an incident.

**What I'd do differently:** I'd have quantified the tradeoff more explicitly upfront - "a few days slower, but a backfill-safe design" - rather than defending the decision after the fact when the backfill need actually arose; having the tradeoff already justified in writing would have made that later conversation faster.

### Backup story 2: "What do you do when priorities change quickly" (Oracle's own sample prompt) - the Colgate latency crisis reprioritization

**Situation:** As Technical Lead, my team had a planned quarter of feature roadmap work already underway when a serious latency problem - a 12-second response time on a consumer- and retailer-facing endpoint - became visible as a real risk to the platform, not just a known rough edge.

**Task:** Decide whether to keep the team on the planned feature roadmap and treat the latency issue as a lower-priority cleanup item, or stop feature work and redirect the team's effort immediately, given this was a platform running across 15 countries at 5 million daily API calls where that latency was a genuine liability.

**Action:** I reprioritized the team's work immediately - paused the planned feature roadmap and redirected effort into root-causing the latency problem, which meant JVM flame graph analysis and APM tracing to find the actual bottleneck, then designing and shipping the fix: event-driven Redis caching with Kafka-based invalidation. I communicated clearly to stakeholders why the roadmap was shifting and what the revised timeline looked like, rather than quietly deprioritizing the original commitments without explanation.

**Result:** Latency dropped 92%, from 12 seconds to under 1 second, removing a real platform-wide risk before it became a bigger incident, and the team returned to the original feature roadmap afterward - now building on a meaningfully more reliable foundation than before.

**What I'd do differently:** I'd want earlier detection built in specifically so this kind of reprioritization decision doesn't have to happen reactively - a latency problem at this scale should ideally surface as a monitored trend well before it becomes urgent enough to halt the whole roadmap, which is exactly the instinct that shows up later in how I approached the alerting platform.

---

## Quick-reference: which story for which likely question

| If asked... | Lead with |
|---|---|
| "Tell me about yourself" | Full opening narrative, all of it - this is likely question one |
| "Why OCI AI Innovation specifically" (as its own follow-up) | Just the closing two sentences of the opening narrative |
| "Time you prioritized customer needs" | Voice bot eval-gates-CI story, or Tome Raider honest-fallback story |
| "Obstacle you overcame" | EphFlow Lambda-timeout-wall story (strongest single-obstacle narrative), or Colgate disagreement resolution |
| "More than one way to solve a problem" | Colgate disagreement resolution (data-driven tradeoff framing), or EphFlow Lambda-timeout-wall (VM orchestration vs. conventional workarounds) |
| "How priorities change quickly" | Colgate latency-crisis reprioritization (direct match to Oracle's exact prompt wording), or eval harness severity tiering |
| "Why this team" | Opening narrative, second half |
| Cold system-design prompt | Think out loud, ask clarifying questions first (per Oracle's own guidance: "ask questions and qualify everything before you start") |

---

## Final notes

- Oracle explicitly evaluates for team fit alongside technical skill in this round - don't out-technical this conversation; let it be a real conversation, ask him questions back.
- Have one real question ready for him about the team's current agentic-AI roadmap or how they think about evaluation at scale - shows genuine engagement, and ties back to your own eval-harness work naturally.
- If he goes deep into any project mentioned above, the corresponding Q&A in `Oracle_Project_Deep_Dive_Prep_v2.md` has the technical depth.
