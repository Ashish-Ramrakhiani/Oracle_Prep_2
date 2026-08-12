# BarTender Round Prep - Ashley Edsall (IC4)

Thu Aug 13, 2:00-3:00 PM | Tags: BT Choice, Innovate Together

---

## What this round actually is (from Blind/Glassdoor research)

Confirmed across multiple independent sources this session:

- The bartender is a senior engineer or manager **from outside your team**, brought in specifically to hold the hiring bar consistent across the org - not someone who'll work with you day to day.
- **Mostly behavioral, STAR-format**, with a chance of a light design or coding tangent if time allows - but the center of gravity is "tell me about a time," not a technical gauntlet.
- **Oracle bartenders do not have unilateral veto power** the way Amazon's Bar Raisers do - one Blind thread confirmed this explicitly. They facilitate the hiring-committee debrief discussion rather than singlehandedly blocking an offer.
- That said, they **do steer the debrief conversation**, and interviewer social dynamics mean a bartender who didn't like you can sway a room even without formal veto - so tone and rapport matter as much as content here.
- "BT Choice" isn't a standard published Oracle Core Value (unlike Ashley's other tag, "Innovate Together," which is) - most likely shorthand for something like "good bar-tender judgment / hiring-bar choices," i.e., they're specifically listening for whether you'd raise or lower the bar for the team. Treat it as: does this candidate meet a consistent standard, independent of any one team's specific need.

**Bottom line: this round is won on genuine, specific stories told with self-awareness - not on technical depth.**

---

## Competency: Innovate Together

Oracle's actual Core Value language: *"We practice empathy and respect in our interactions. We challenge our internal biases... We seek and celebrate diverse perspectives... We foster an inclusive environment where collaboration drives innovation."*

Sample Oracle behavioral prompts that map here: *"Tell me about a time you worked with someone whose personality was different from yours." "While working on a team, how did you deal with a conflict?"*

### Primary story: Colgate cross-functional disagreement (same anchor as HM doc, different angle)

**For this round, emphasize the empathy and process angle over the decision-making angle** - Ashley's tag is about collaboration and diverse perspectives, not about your authority to make the call, which is the emphasis the HM version of this same story leads with.

**Situation:** Two engineers I directed had genuinely different working styles on the same feature - one wanted to move fast and iterate, one wanted to nail the design up front before writing code, and the disagreement was starting to slow the team down.

**Task:** Resolve the disagreement in a way that respected both working styles, rather than just picking whichever style happened to match my own instinct.

**Action:** Rather than picking a side, I asked both engineers to make their case with a concrete tradeoff instead of a stated preference - forcing the conversation to be about the actual work rather than about whose instinct was right. If I'd just sided with the person whose style matched mine, I'd have been optimizing for my own comfort, not for the best outcome, and I'd have taught the team that disagreement gets resolved by whoever the lead happens to agree with rather than by evidence.

**Result:** The team kept moving on an evidence basis rather than a seniority basis, and it became the default way disagreements got resolved going forward - both engineers stayed engaged because the process respected their perspective even when their specific preference didn't win.

**What I'd do differently:** I'd introduce the "bring a concrete tradeoff, not a preference" framework earlier and more explicitly as a team norm, rather than applying it reactively each time a disagreement came up.

### Backup story 1: EphFlow academic collaboration across disciplines

**Situation:** Working with ecological forecasting domain scientists at Virginia Tech on the FLARE case study, I was collaborating with people from a genuinely different discipline - different vocabulary, different risk tolerance since a failed forecast has real ecological-monitoring consequences, and a different definition of "done" than a systems engineer would default to.

**Task:** Get to a shared understanding of what the system needed to guarantee, in their terms, rather than assuming my own engineering definition of correctness was the one that mattered.

**Action:** I spent real time understanding what "done" meant to them specifically - not "the DAG validates and executes without error," but "the forecast actually runs on schedule and produces a result they can trust for their monitoring work." That meant translating what `RequiresVM: true` needed to mean in terms of their actual workflow, not just describing the flag's technical behavior.

**Result:** The collaboration produced a real, validated result that made it into the published paper, and it worked specifically because I didn't just impose my own frame of what mattered onto their problem.

**What I'd do differently:** I'd ask more upfront questions about their definition of success before proposing a technical approach, rather than proposing something and then checking whether it matched their actual needs.

### Backup story 2: Colgate - splitting ownership by vertical slice, not by layer

**Situation:** Directing 6 engineers at Colgate, I had a choice in how to structure ownership of the platform's features - the conventional approach would split the team horizontally, frontend specialists, backend specialists, database specialists, each staying in their own lane.

**Task:** Decide whether to optimize for individual efficiency, which a horizontal split generally gives you, or for something else.

**Action:** I deliberately split ownership by vertical slice instead - each person or pair owned a feature end-to-end, API, service logic, and tests together, rather than owning one layer across many features. That was a direct bet on collaboration and shared context over pure per-person efficiency, since horizontal splitting is often faster individually but means no one has full context on what actually shipped, which slows review and creates narrow-perspective silos.

**Result:** The vertical-slice model cost some short-term velocity compared to a horizontal split, but every engineer could speak to the whole feature they owned, not just their layer of it - which made reviews faster and made the team far less dependent on any one specialist being available.

**What I'd do differently:** For a genuinely large, complex feature, I'd revisit this - vertical ownership works well at the scale I was directing, but at a much bigger team size, some horizontal specialization probably becomes necessary again, and I'd want to recognize that inflection point rather than assuming vertical slices scale indefinitely.

### If asked directly "tell me about a conflict with a coworker" (not just differing styles)

**Constructed from your real Colgate API-contract ownership - verify against what actually happened and adjust specifics before using live.**

**Situation:** As the sole owner of the REST API contract serving six enterprise consuming services, I held a strict additive-only backward-compatibility policy - no breaking changes within a version, formal deprecation window before sunsetting anything. One of the six consuming teams had a business deadline that a breaking change would have hit faster than my policy allowed, and the engineer driving that team's integration pushed back directly and repeatedly - this wasn't a peer on my own team I could resolve through internal process, it was a stakeholder outside my authority with a real deadline of their own.

**Task:** Resolve this without either compromising the other five services' stability (a breaking change doesn't stay contained to one consumer) or being the person who blocked another team from hitting their deadline for what looked, from their side, like a process technicality.

**Action:** I didn't just hold the line and say no. I proposed a parallel versioned endpoint - they got their new shape immediately on a new version, the other five services kept running unaffected on the existing version, and I compressed the deprecation window for the old path specifically for their use case rather than applying my standard timeline uniformly. That meant walking them through, concretely, what would break for the other five teams if I skipped that step, so the pushback turned into a real technical conversation instead of a standoff.

**Result:** They hit their deadline on the new version, none of the other five services saw any disruption, and the compressed-deprecation approach for isolated cases became something I'd reuse rather than treating my policy as rigid in every circumstance.

**What I'd do differently:** I'd have proposed the parallel-versioned-endpoint compromise earlier in the conversation instead of leading with "here's why the policy exists" - leading with the constraint before the solution is what made it feel like a wall at first rather than a real conversation.

---

### If asked directly "tell me about a time you were wrong / caught your own mistake"

**Constructed from your real Colgate DB-tuning work - verify against what actually happened and adjust specifics before using live.**

**Situation:** While redesigning composite indexes to cut Postgres query latency, I made a change that optimized the dominant query pattern I was targeting but I hadn't fully checked its effect on a second, less-frequent query pattern that used a different column order on the same table.

**Task:** Once deployed, I needed to catch whether that second pattern had regressed before it became customer-visible, and if it had, own that I'd missed it rather than let monitoring quietly absorb the blame.

**Action:** I was already watching APM traces and query plans closely from the original latency work, and I caught the regression myself within the query plan analysis I was already running as a follow-up check - the second pattern had gone from using an index scan to a sequential scan because my new composite index's column order no longer matched its filter clause. I rolled back that specific index change immediately, flagged it to the team openly rather than quietly patching it, and added the second query pattern explicitly to the set of patterns I checked before any future index change, since the real gap was in my own review process, not just that one index.

**Result:** The regression was live for under a day and caught before it hit the 80K-daily-query production load in any customer-visible way, and the review-process gap it exposed - checking only the pattern I was optimizing for, not adjacent patterns on the same table - became a standing checklist item for the rest of the DB tuning work.

**What I'd do differently:** This is already the "what I'd do differently" story in a sense - the fix was changing my own process, not just the code. If I'm pushed further on it, the honest answer is I should have run the full query-pattern audit before the change shipped, not after.

---

## Competency: "BT Choice" (hiring-bar judgment)

Since this isn't a published Core Value, the safest interpretation is: they want evidence you hold yourself and others to a real standard, not that you cut corners under pressure or let mediocre work pass because it was expedient.

### Primary story: The voice bot eval-harness decision, reframed for bar-setting

**Same underlying story as in the HM doc, reframed here around the standard itself rather than the customer-impact framing** - "BT Choice" is probing for whether you hold a bar even when nobody's forcing you to.

**Situation:** Building the HIPAA-aware patient collection voice bot, the fast path to a demo would have been to ship the conversation flow and iterate based on real call outcomes, without the overhead of a formal evaluation harness gating every change.

**Task:** Decide what standard to hold the system to before any real patient interaction, knowing a stricter standard meant slower shipping.

**Action:** I built a 108-scenario eval suite gated in CI, where a regression on any critical scenario blocks the merge outright. That's a deliberately higher bar than "good enough to demo" - the actual standard is "good enough that a regression physically cannot reach a real patient without failing a critical test first."

**Result:** The team ships changes knowing a dangerous regression is structurally blocked, not just hopefully caught in review - that's the bar I'd want a team to hold generally, not just on this one project, and it's a bar I set for myself before anyone asked for it.

**What I'd do differently:** I'd document the severity-tiering rationale explicitly as a reusable standard from the start, so holding this bar becomes something the whole team owns, not something that depends on me personally remembering to apply it to the next project.

### Backup story 1: Colgate composite-index regression, self-caught

**Situation:** While redesigning composite indexes to cut Postgres query latency, I made a change that optimized the dominant query pattern I was targeting but hadn't fully checked its effect on a second, less-frequent query pattern using a different column order on the same table.

**Task:** Once deployed, catch whether that second pattern had regressed before it became customer-visible, and own it directly if it had, rather than letting monitoring quietly absorb the blame.

**Action:** I was already watching APM traces and query plans closely from the original latency work, and caught the regression myself within a follow-up check I was already running - the second pattern had gone from an index scan to a sequential scan because my new composite index's column order no longer matched its filter clause. I rolled back that specific index change immediately, flagged it to the team openly rather than quietly patching it, and added the second pattern explicitly to the checklist I ran before any future index change.

**Result:** The regression was live for under a day and caught before it hit the full production query load in any customer-visible way, and the review-process gap it exposed became a standing checklist item going forward.

**What I'd do differently:** I should have run the full query-pattern audit before the change shipped, not after - the fix was ultimately about changing my own review process, not just the one index.

### Backup story 2: BugSleuth's "no labeled training data" discipline as a research-integrity bar

**Situation:** Building BugSleuth, using some labeled bug-location data during development would have been an easy way to sanity-check or even lightly tune the genetic algorithm's behavior faster.

**Task:** Decide whether to allow any labeled-data leakage into the development process, even informally, given the entire value proposition of the approach was that it needed none.

**Action:** I held a strict line during development that the ground-truth bug locations in Defects4J were never used to influence the genetic algorithm's search or fitness function - only used afterward, to score the final result. That meant debugging and validating the approach was genuinely harder, since I couldn't use "does this match the known answer" as a fast feedback signal during development the way I could have if I'd allowed myself that shortcut.

**Result:** The eventual result - beating supervised SOTA and LLM baselines without labeled data - is only a credible claim because that discipline was real throughout development, not just true of the final evaluation run. A result that quietly leaked labels during development wouldn't have survived peer review.

**What I'd do differently:** Nothing on the discipline itself - if anything, I'd document the "no leakage" boundary more explicitly and earlier in the project so it was a stated constraint from day one rather than something I was personally careful about.

---

## Tone guidance specific to this round

- **Ask him what he actually does day to day, early** - since he's outside your prospective team, showing genuine curiosity about a different part of Oracle signals the "diverse perspectives" value directly, and it's a natural rapport-builder given he doesn't have the built-in "we'll be teammates" connection Hesam or Haoran might.
- **Don't perform confidence you don't have** - self-aware "here's what I'd do differently" answers are explicitly what Oracle's own STAR guidance asks for, and a bartender specifically screening for hire-bar judgment is more likely to read false confidence as a red flag than genuine reflection.
- **If a light coding or design question comes up**, treat it exactly like the DSA framework or system-design prep you already have - don't let the "this is the behavioral round" framing throw you if it pivots technical.

---

## Quick-reference: which story for which likely question

| If asked... | Lead with |
|---|---|
| "Time you worked with someone very different from you" | Colgate cross-functional disagreement (empathy framing), or EphFlow cross-discipline collaboration |
| "Conflict with a coworker" | Colgate API-contract stakeholder conflict story |
| "Time you held a standard under pressure to cut corners" | Voice bot eval-harness-as-bar story, or BugSleuth no-labeled-data discipline story |
| "Time you were wrong and admitted it" | Colgate composite-index regression, self-caught |
| "How you split work / delegate on a team" | Colgate vertical-slice ownership story |
| Generic "tell me about yourself" | Shorter version of the HM-round opening narrative - this round doesn't need the full "why OCI" pitch, just orient him quickly |

---

## One honest flag before Aug 13

The direct-conflict story (Colgate API contract) and the caught-my-own-mistake story (composite-index regression) are both constructed to be maximally consistent with your real, documented work - not memories. Every other story in this doc has the same status. Before the 13th, it's worth 15-20 minutes checking whether either of the two hardest-probed stories (conflict, self-caught mistake) resembles something that actually happened closely enough to hold up under a real follow-up - if not, swap in your own version of the same shape.
