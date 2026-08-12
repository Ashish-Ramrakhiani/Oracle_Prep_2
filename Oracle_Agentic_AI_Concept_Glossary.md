# Agentic AI Concept Q&A Bank - Oracle OCI AI Innovation, Req 339619

Every concept in the JD, reframed as the actual question an interviewer might ask, answered in depth. Status tags carried over from the glossary version:

- **STRONG** - real project experience, defensible in depth
- **PARTIAL** - you've touched the underlying idea, terminology or depth needed polishing (now filled in below)
- **GAP** - genuinely new ground - answered from first principles, clearly flagged as not tied to your own build

After you read through a section, ask me anything unclear - and once you're solid, I'll quiz you cold the same way we've done with the projects. New questions you get asked (by me or in the real interview) get added here as we go.

---

## 1. Core agentic capabilities

**Q. What's the difference between reasoning and planning in an agentic system - and which does your EphFlow agent layer actually do?** - PARTIAL

A: Reasoning is the model working through a problem to decide a single next action given the current state and goal - it can be implicit, just prompting a standard model to "think step by step," or explicit, where the model architecture itself is trained to allocate more computation to harder problems before answering (see reasoning models below). Planning is different in scope: deciding a whole sequence of actions upfront, before executing any of them, versus deciding one step at a time and re-evaluating after each.

There are two dominant patterns for how an agent structures this. Plan-and-Execute commits to a full plan before doing anything, and only revisits the plan if something fails partway through. ReAct-style agents interleave reasoning and acting continuously - decide one step, observe the result, decide the next step, repeat - trading a higher inference cost per step for the ability to react immediately to new information.

EphFlow's agent layer actually demonstrates both patterns, at two different points. Task decomposition is Plan-and-Execute - the agent commits to a full DAG upfront, deciding all action boundaries and RequiresVM flags before any execution starts, based on the task description and historical S3 log data. The error-recovery layer is closer to reactive re-planning - it only engages when a deterministic failure escalates to it, reads a structured digest of what actually happened, and makes a single bounded decision (reassign, flip a flag, or escalate) rather than continuously interleaving reasoning with execution the way a true ReAct loop would. Being able to name which pattern each part of your own system actually follows - rather than calling the whole thing "the agent reasons and plans" vaguely - is exactly the kind of precision a senior-level interviewer is listening for.

---

**Q. Explain tool calling mechanically - what does the model actually output, and where have you built something like it?** - PARTIAL

A: Tool calling is the mechanism that lets an LLM invoke external functions rather than only generating free text. The calling application gives the model a set of tool schemas in its context - each one a name, a natural-language description of what it does, and a JSON-schema-typed parameter list. When the model decides a tool is needed, instead of emitting prose it emits a structured tool call: a function name plus arguments that conform to that schema. The application code parses that call, actually executes the underlying function, and feeds the result back into the model's context as a new turn - the model then continues, now with that result available to reason over. This is what OpenAI's "tools" parameter and Anthropic's tool-use API formalize as a first-class part of the message format, rather than something you'd have to hand-parse out of free text yourself.

The closest thing in your own work is EphFlow's agent-layer decision primitives - reassign_platform, flip_requires_vm, escalate. Structurally, that's identical to tool calling with exactly three available functions: the agent's output is a small structured decision, not free text, and a separate piece of code executes the actual effect. The honest caveat worth stating if asked directly "have you used the formal tool-calling API" - the EphFlow build predates or doesn't necessarily route through OpenAI's or Anthropic's formal tools parameter; it's the same underlying pattern implemented directly, not via that specific API surface. Given your resume lists both MCP and the OpenAI Agents SDK as skills, this question is likely to get a direct follow-up on actual hands-on depth with the formal APIs specifically - worth having a clear, honest answer ready on exactly how much you've used the real tool-calling parameter versus building the pattern by hand.

---

**Q. What is workflow execution and multi-step orchestration, and what's the strongest example from your own work?** - STRONG

A: Workflow execution is the runtime that carries a task through a defined sequence of steps - nodes, in most frameworks - handling the transitions between them, whatever state needs to persist across steps, and whatever conditional logic decides which step comes next. Multi-step orchestration is the broader discipline of designing that sequence itself: what the steps are, how they depend on each other, and how failures in one step propagate or get contained.

You have three genuinely different implementations of this same underlying idea, worth being able to contrast directly. LangGraph, powering Tome Raider, is itself a workflow-execution engine at its core - nodes, edges, and a checkpointed state object that persists across the graph's execution, with conditional edges that can loop the graph back on itself, which is exactly what the grade-and-rewrite cycle uses. EphFlow's DAG substrate is a different flavor - a directed acyclic graph (no loops by design), with dependency resolution, cross-platform dispatch, and automatic augmentation of the graph itself (VM lifecycle injection) based on declared action properties. The voice bot's 44-node conversation flow is a third flavor again - a state machine where transitions are decided by LLM classification rather than deterministic code, but the topology itself is still fixed and hand-authored, same as the other two. All three share the same underlying principle - a graph of steps, explicit state, and controlled transitions - applied to three very different problems: retrieval, distributed compute, and conversation. That range is a strong thing to have ready if asked to demonstrate breadth on this specific JD line.

---

**Q. What does human-in-the-loop escalation actually mean, and where does it show up in your work?** - STRONG

A: Human-in-the-loop escalation is the design decision that an autonomous system should stop and hand control to a person once it hits a situation past its own authority or confidence to resolve - rather than either guessing or looping indefinitely on an unsolvable problem. The hard part isn't the mechanism itself, it's deciding the trigger condition: what counts as "past this system's authority" has to be defined precisely, or the escalation either never fires (defeating the purpose) or fires too often (making the automation useless).

You have three real, structurally different examples of this. EphFlow's tier-three escalation fires when a deterministic failure survives both of the agent's mutation primitives - reassigning platform and flipping the VM flag - which is a clean, mechanical trigger condition: two specific remedies exhausted. Tome Raider's proposed multi-agent design escalates when routing confidence is low and the topic hits high-risk keywords simultaneously - a compound trigger, deliberately requiring both conditions so a merely uncertain but low-stakes question doesn't unnecessarily escalate. The voice bot's transfer-agent is a genuinely different flavor again - it's escalating a conversation, not a discrete decision, triggered by the node-classification step recognizing the patient needs something the flow wasn't authored to handle, like a dispute. Across all three, the design principle is the same: escalation should always come with a structured summary of what was tried and why it failed, not just a generic "human needed" flag - which is true in all three of your implementations and worth stating explicitly, since a bare escalation without context just moves the problem to the human without actually helping them solve it faster.

---

## 2. AI platform capabilities

**Q. What is "agent execution" as a platform capability, and what have you built that qualifies?** - PARTIAL

A: Agent execution is the runtime layer specifically responsible for running an agent's loop end to end - invoking the model, executing whatever tool calls it emits, feeding results back into context, checking termination conditions like a max-step count or a timeout, and handling the bookkeeping of state across that loop. It's a narrower concept than "workflow orchestration" - orchestration is about the shape of the graph, agent execution is about actually running one node of that graph safely, with the right guardrails against infinite loops or runaway cost.

Two things in your own work qualify, both because they're explicitly bounded. Tome Raider's rewrite-retry loop is a small agent-execution loop with a hard termination condition - capped at 2 retries, after which it falls back rather than continuing indefinitely. EphFlow's tier-based decision handling is a lighter version of the same idea - the agent's loop terminates in exactly one of three ways every time (retry succeeds, mutation succeeds, or escalation fires), so there's no path where it can spin without terminating. The connecting thread, worth stating explicitly: neither of your agent-execution loops trusts the model to decide when to stop - the termination condition is enforced by code outside the model's control, which is the actual safety property that matters at production scale, not just "the loop usually stops."

---

**Q. What is model serving and inference infrastructure - walk me through the core concepts.** - GAP

A: This is the layer that actually runs a trained model to produce completions at scale, and it's genuinely different territory from anything in your own projects, since everything you've built calls hosted APIs rather than operating a serving stack directly - worth saying that plainly rather than stretching an API-consumer story into an infrastructure-operator one.

First, what happens per request, the actual computation: text gets tokenized into integers, each token becomes a vector via an embedding lookup, those vectors pass through the model's stacked layers where attention lets each token weigh relevance against every earlier token, the final layer produces a probability across the whole vocabulary, one token gets picked, and the loop repeats - generate one token, append it, repeat - until a stop token or length limit is hit. That's identical whether you're the only user or one of a million.

Serving infrastructure is the layer wrapped around that loop for when many people are doing it simultaneously on the same expensive GPU. Concretely, with real request-level examples:

Batching - three requests arrive within the same few milliseconds: "what's the capital of France," "summarize this email," "translate hello to French." Instead of the GPU fully finishing the first before even looking at the second, the server groups all three into one batch and processes them together in a single pass, and all three get answers back at roughly the same time rather than the third request waiting for the first two to fully finish.

KV-cache - a user asks for a three-sentence explanation. By the time the model is generating the 20th word, it needs everything it already wrote to stay coherent. Instead of re-reading all 19 previous words from scratch before producing word 20, it reuses what it already computed the first time, and only does new work for the newest word - which is why a long response doesn't get proportionally slower to generate as it gets longer.

Quantization - a company wants to serve a model to millions of people cheaply, so instead of running the full-precision version, they run a compressed version of the same model. A simple query like "what's 2+2" gets essentially identical answer quality from the compressed version, but because it takes up less memory, more copies fit on the same GPUs, serving more users for the same hardware cost.

Continuous batching - three requests start together, two want short answers and one wants a long essay. The two short ones finish quickly while the essay is still generating. A new request arriving moments later slides into one of the now-empty spots immediately, running alongside the still-in-progress essay, rather than waiting for the whole batch to fully complete before any new request can join.

Paged attention - the same three users have very different response lengths, but a naive server has to guess upfront how much memory to reserve per user, reserving worst-case space "just in case" even for the user who only needed ten words. Paged attention hands out memory in small chunks as each response actually grows, so a short response only ever claims a small amount, freeing the rest for other concurrent users instead of sitting reserved and unused.

vLLM, TensorRT-LLM, and Triton are real production frameworks that implement all of the above automatically, so a company doesn't have to build this batching-and-memory-management machinery by hand.

If asked directly whether you've operated infrastructure like this: no, honestly - your work has been as a consumer of hosted inference APIs (OpenAI, GPT-4o-mini, Retell's managed platform), not as an operator of a serving stack. You understand the per-request mechanics and the serving-layer concepts and the tradeoffs they represent, but haven't run vLLM or a comparable system in production. That's a precise, honest answer, and precision here is worth more than vague hand-waving toward experience you don't have.

---

**Q. How would you actually evaluate an AI system, and what's LLM-as-judge?** - STRONG (practice) / GAP (formal vocabulary)

A: You have two concrete, real evaluation stories, and it's worth connecting them to the formal vocabulary this question is actually probing for. The voice bot's 108-scenario suite, gated in CI with a critical/non-critical severity split, is a golden-dataset-style offline evaluation - a fixed, curated set of scenarios with expected behavior, run before any change can merge, which is precisely what "eval suite" means in the AgentOps sense. BugSleuth's evaluation against Defects4J - measuring top-5 localization accuracy against 704 real, labeled defects - is the same offline-eval pattern applied to a research problem instead of a production one.

LLM-as-judge is a specific evaluation technique worth knowing by name: using a model to grade another model's output against a rubric, rather than requiring a human to review every output or relying only on exact-match scoring, which often can't capture whether a free-text answer is actually good. Here's a connection worth making explicitly if asked to define this term - Tome Raider's grading step, architecturally, already is a lightweight LLM-as-judge: given a question and retrieved context, a structured-output LLM call judges whether the context is sufficient, which is the same pattern as LLM-as-judge applied to a narrower, binary question rather than an open rubric. Being able to say "I've actually already built a narrow version of LLM-as-judge, here's how" is a much stronger answer than defining the term in the abstract.

The other piece of vocabulary worth having ready: offline evaluation happens against a fixed test set before deployment - both of your examples are this - while online evaluation measures real production behavior after deployment, often via A/B testing two variants against live traffic. You don't have a concrete online-eval story from your own projects, and that's fine to say directly rather than reaching for one.

---

**Q. What does observability mean specifically for AI and agent systems, as opposed to traditional systems?** - PARTIAL

A: Traditional observability rests on three pillars - logs, metrics, and traces - and that's necessary but not sufficient for an AI or agent system. The extra thing you need to capture is the actual reasoning surface: the exact prompt and context sent to the model at each step, the model's raw output before any parsing or post-processing touches it, which tools were called with what specific arguments, and token counts and cost per step. The reason this matters more here than in a traditional service: debugging "why did the agent do X" for a non-deterministic system requires seeing the exact context the model reasoned over at that moment - a generic error log telling you a request failed doesn't tell you why the model made the decision it made, the way a stack trace tells you why traditional code threw an exception.

LangSmith, part of Tome Raider's stack, is a real example of an observability platform built specifically for this - it traces each LangGraph node's input and output, tracks token usage and latency per step, and lets you inspect a full run's decision path after the fact. Being able to describe what it actually captures, not just that you used it, is the difference between name-dropping a tool and demonstrating you understand what agent observability requires structurally. If your proposed multi-agent extension came up, the same principle would apply at a larger scale - a supervisor's routing decision plus which specialist handled a request plus that specialist's internal trace, all visible as one continuous history, is exactly the kind of observability requirement that matters more as the system gets more complex, not less.

---

## 3. Production agent services

**Q. What's the difference between short-term and long-term agent memory - and which does your system actually have?** - PARTIAL

A: Short-term or working memory is whatever's actively available in the model's context right now - the current conversation, the current task's intermediate state. Long-term memory is persisted across sessions entirely - facts learned, user preferences, past interactions that can be retrieved later even after the original context is gone, typically implemented via a vector store or a structured database keyed to a user or entity rather than a single conversation thread.

Tome Raider's Postgres checkpointer is short-term memory done well - it's genuinely durable, surviving across turns within one conversation thread, backed by real persistence rather than living only in a server's RAM. But it is explicitly not long-term, cross-session memory - there's no mechanism in the system for it to recall something from a user's conversation last week when they start a brand new thread today. Being precise about this distinction matters a lot if asked directly "does your RAG system have memory" - the honest, precise answer is yes, within a thread, durably; no, not across threads or sessions. A system with true long-term memory would need a separate mechanism entirely - for example, periodically summarizing completed threads and writing those summaries into the same vector store used for document retrieval, so a future conversation could retrieve "what we discussed last time" the same way it retrieves an uploaded document. That's a real, buildable extension, but it's not what's shipped.

---

**Q. Walk me through context management strategies - what are the options, and which does your system actually use?** - PARTIAL

A: There are four broad strategies for fitting relevant information into a model's bounded context window. Truncation simply cuts off older content once a limit is hit - simple, but loses information with no attempt to preserve what matters. Summarization or compaction periodically compresses older turns into a shorter summary, trading some fidelity for keeping more effective history within budget. Sliding window keeps a fixed number of the most recent turns and drops everything older, which is simpler than summarization but has a hard cliff - anything just past the window is gone entirely, not even summarized. Retrieval-based context doesn't try to carry history at all in the traditional sense - instead, it pulls in only what's relevant to the current query from a larger store, which is literally what RAG does for document content.

Tome Raider actually runs two of these strategies simultaneously, which is worth naming explicitly if asked to explain context management broadly rather than picking just one. Conversation history uses a sliding window - the last 6 turns, feeding the contextualize step. Document content uses retrieval-based context - hybrid search pulling in only the chunks relevant to the current question, rather than ever attempting to fit an entire uploaded corpus into context. The honest limitation worth stating alongside this: the sliding window has a hard cliff exactly like the general pattern - a follow-up on turn 20 referencing something from turn 1 would fail, since there's no summarization layer catching what falls outside the 6-turn boundary. That's a real, acknowledged gap, not a hidden one.

---

**Q. What problem does the Model Context Protocol solve, and how is it actually structured?** - GAP, HIGH PRIORITY

A: MCP is an open protocol, originated by Anthropic, that standardizes how AI applications connect to external tools and data sources. The problem it solves is worth framing precisely: without a standard, every AI application that wants to use, say, a GitHub integration and a Slack integration and a database integration each needs its own bespoke code to talk to each of those tools - if you have M applications and N tools, you potentially need M times N separate integrations, since each application-tool pair might be built differently. MCP standardizes the interface tools expose, so any MCP-compatible application can talk to any MCP-compatible tool without custom code for that specific pairing - reducing the problem from M times N integrations down to roughly M plus N, since each tool only needs to implement the protocol once, and each application only needs to implement the client side once.

The architecture has three roles. An MCP host is the AI application itself - a chat interface, an IDE, an agent framework. The host contains an MCP client, which is the component that actually speaks the protocol. That client connects to one or more MCP servers, each of which exposes a specific tool or data source - a GitHub server, a Slack server, a database server - over a standardized transport, either stdio for a local process or HTTP with server-sent events for a remote service. Each server can expose three kinds of primitives: tools, which are functions the model can call to take an action; resources, which are data the model can read, like the contents of a file or the result of an API call; and prompts, which are reusable prompt templates the server provides for common tasks against that tool.

Given your resume already lists MCP as a skill, the honest self-assessment worth having ready before the interview: can you describe a specific MCP server you've connected to, what tools or resources it exposed, and what actually broke or surprised you when you first wired it up? A protocol-level definition like the one above is necessary but not sufficient if pressed for hands-on specifics - that's the follow-up most likely to expose whether the resume line reflects real depth or surface familiarity.

---

**Q. Explain vector retrieval and how your hybrid approach actually works.** - STRONG

A: Vector retrieval means finding relevant content by comparing numerical embeddings rather than exact keyword matches - text gets converted into a high-dimensional vector such that semantically similar text ends up close together in that vector space, and a query's embedding gets compared against a corpus of document embeddings using a similarity measure like cosine similarity, returning whichever documents are closest.

Tome Raider doesn't rely on vector retrieval alone - it's a hybrid system, running dense vector search over pgvector embeddings in parallel with sparse full-text search via Postgres's `ts_rank`, then fusing the two ranked lists with Reciprocal Rank Fusion rather than picking one or blending raw scores. The reason hybrid beats either approach alone: dense vector search is excellent at capturing semantic similarity - a query about "canceling a subscription" can find a document about "terminating a service agreement" even with no shared words - but it can miss exact-match cases like a specific error code, contract clause number, or proper noun that a keyword search would catch trivially. Sparse full-text search is the reverse - excellent at exact terms, weak at paraphrase and semantic similarity. Fusing both, by rank rather than by raw score, avoids the problem of two fundamentally different scoring scales - cosine similarity and a text-rank score aren't comparable numbers - which is exactly why RRF operates on rank position instead of trying to normalize and average two incompatible scales.

---

**Q. What is multi-agent coordination, and how would it actually work in a system you've designed?** - STRONG

A: Multi-agent coordination is the discipline of getting multiple specialized agents to work together toward a shared goal, without either duplicating each other's work or stepping on each other's decisions - the two main coordination patterns are a supervisor architecture, where a central node routes work to specialists and receives their results back, versus a peer-to-peer mesh, where agents hand off to each other directly without central routing.

The multi-agent extension we designed for Tome Raider is a real, defensible example of a supervisor architecture, worth walking through concretely if this comes up. A supervisor classifies each incoming query and routes it to one of several specialists - a QA specialist reusing the existing RAG loop unchanged, a compliance-judgment specialist with its own stricter grading threshold and dual-citation requirement, a drafting specialist using a draft-critique-revise loop instead of retrieval-rewrite, and an escalation path for cases needing human review. The actual coordination mechanism, worth being precise about since "coordination" can sound vaguer than what it really is: it's not agents passing messages to each other - it's one shared, checkpointed state object every specialist reads from and writes to, scoped to the conversation thread, with routing implemented via a Command object that bundles a state update together with a "go to this node next" directive. The reason for choosing supervisor over peer-to-peer specifically for this use case: auditability - a compliance domain needs a traceable answer to "which specialist handled this and why," which a supervisor-mediated architecture gives you for free, while a mesh of peer handoffs would need to be reconstructed after the fact from a scattered trail of inter-agent messages.

---

**Q. What is policy enforcement in an agent system, and where have you actually built it?** - PARTIAL

A: Policy enforcement means constraining what an agent is allowed to do independent of what the model itself decides - a layer that can intercept an action the model wants to take and allow it, deny it, or modify it, based on rules that exist outside the model's own reasoning entirely. The key property that makes this real policy enforcement rather than just a suggestion in a prompt: the constraint has to be enforced by code the model can't talk its way around, not by asking the model nicely to follow a rule.

The voice bot's four compliance gates are a genuine example of this, even though the system isn't built on a formal tool-calling LLM loop. The gates - HMAC auth, calling-window enforcement, DNC opt-out, TCPA consent - sit between "the system has decided to make this call" and "the call actually happens," and any one of them can veto the dial entirely, in pure code, with zero input from the model. That's architecturally identical to a policy layer intercepting a tool call before execution in a more formally agentic system - the pattern is the same even though the surface-level implementation looks like a compliance checklist rather than an "agent policy" in name. Worth stating explicitly if asked to define this term abstractly: you already have a production example of the underlying pattern, just not badged with that specific vocabulary in the original build.

---

## 4. Distributed systems for AI platforms

**Q. What's the latency-throughput tradeoff specific to GPU inference, and how does it relate to distributed-systems tradeoffs you already understand?** - PARTIAL (general) / GAP (GPU-specific)

A: In GPU inference serving, the core tension is that batching multiple requests together dramatically improves throughput - GPUs are expensive, parallel hardware, and keeping them busy processing a full batch is far more cost-efficient than processing one request at a time - but forming a batch requires waiting a small amount of time to gather enough concurrent requests, which adds latency to any individual request that arrives just after a batch was dispatched. Bigger batches mean better GPU utilization and lower cost per request, but worse worst-case latency for an unlucky request; smaller batches or no batching means better latency but wasted GPU capacity and higher cost per request served.

This is a genuine cousin of tradeoffs you've already worked with directly, even though the underlying resource is different. At Colgate, the caching-and-invalidation work was fundamentally a similar tradeoff in a different domain - serving from cache is fast but risks staleness, hitting the database is always fresh but slow, and the actual engineering decision was about where to draw that line and how to bound the worst case, which resulted in a TTL backstop specifically to bound staleness the same way a serving system bounds worst-case batching latency with a maximum wait time before dispatching a partial batch anyway. The pattern - trading a resource-efficiency dimension against a latency dimension, and choosing a bounded worst case rather than optimizing one dimension purely - is the same instinct, applied to GPUs and inference requests instead of a database and cache keys. Worth drawing that connection explicitly if this comes up, since it demonstrates the underlying judgment transfers even without direct GPU-serving experience.

---

**Q. How do you handle multi-tenant isolation in an AI system - what's actually enforcing it?** - STRONG

A: Multi-tenant isolation means ensuring one tenant's - one user's or one customer's - data and behavior can never leak into another tenant's experience, and the important engineering question is always where that isolation is actually enforced, not just declared. The weakest version of multi-tenancy is a filter in application code that's supposed to scope every query to the current tenant - weak because a single missed filter in a single code path is a full data leak. The strongest version pushes enforcement down to a layer that can't be bypassed by a coding mistake in application logic.

You have two real examples, at different points on that spectrum, worth contrasting directly. Tome Raider uses Postgres Row-Level Security - isolation enforced at the database layer itself, meaning even if application code had a bug that forgot to filter by user, the database would still refuse to return another tenant's rows, because the policy is attached to the table, not to any particular query path. The voice bot uses per-tenant Firestore isolation - a different mechanism, since Firestore's model doesn't have RLS in the same sense, so isolation there is more likely enforced through separate collections or documents scoped per tenant at the application layer, which is a weaker guarantee structurally than RLS even if correctly implemented, since it depends on every code path correctly scoping its reads and writes. Being able to name this distinction - RLS as database-enforced versus per-tenant collections as application-enforced - and say plainly that one is a stronger guarantee than the other, is a stronger answer than describing both as equivalently "isolated" without acknowledging the difference in where the guarantee actually lives.

---

**Q. How do you think about service boundaries, APIs, and state management when designing a system - give a real example.** - STRONG

A: A service boundary defines what one component is responsible for and, just as importantly, what it deliberately isn't responsible for - the API is the contract at that boundary, and state management is the question of what data has to persist across requests versus what can be recomputed or discarded.

Three real examples worth having ready, each demonstrating a different aspect of this. Colgate's API contract, owned end to end across 6 consuming services, is the clearest boundary-design story - additive-only versioning with a formal deprecation window, specifically chosen because a breaking change at that boundary doesn't stay contained to one consumer, it cascades to all six, so the boundary had to be designed defensively against that blast radius. EphFlow's schema-first DAG design is a different flavor - the FaaSr.schema.json contract is the actual service boundary between "what a workflow author submits" and "what the deterministic execution substrate will accept," and every validation rule at that boundary - cycle detection, the conditional-successor mixing rule - exists specifically to reject malformed input before it can cause a harder-to-debug failure deeper in the system. Tome Raider's checkpointer state model is the clearest state-management story - deciding that conversation state needed to be durable across turns, backed by Postgres rather than in-memory, was a direct consequence of deciding what data absolutely couldn't be lost if a server restarted mid-conversation, versus what could safely be recomputed, like the retrieved context for a given turn, which doesn't need to persist since it can be regenerated from the same query.

---

**Q. Walk me through a consistency tradeoff or failure mode you've actually had to design around.** - STRONG

A: Two real examples, both concrete enough to walk through mechanically rather than abstractly. Colgate's idempotent RabbitMQ consumers deal with the at-least-once delivery guarantee that most message brokers provide by default - a consumer can see the same message more than once, either from a genuine redelivery after a crash or from a retry after an ambiguous acknowledgment failure. The fix wasn't trying to force exactly-once delivery, which is notoriously difficult to guarantee end to end - it was making the consumer's handling of a duplicate message safe regardless, via a dedup key derived from the business event itself, checked against a processed-events store before the message is applied, so a duplicate delivery is a no-op rather than a double-application.

EphFlow's retry-and-backoff across 5 cloud backends is a different consistency problem - coordinating execution state across genuinely different platforms (Lambda, Cloud Run, OpenWhisk, GitHub Actions, SLURM), each with its own failure modes and timing characteristics, using S3 as a shared coordination point rather than trying to maintain a live connection or shared memory across platforms that have no native way to talk to each other. The failure mode being designed around there is a platform going down or timing out mid-execution without cleanly signaling failure - the coordination model has to tolerate that ambiguity (did it fail, or is it just slow) rather than assuming failures are always cleanly reported, which is why retry-with-backoff exists as the default response to an action not completing in the expected window, rather than assuming silence always means failure.

---

**Q. What are SLIs and SLOs, and why do AI services need something beyond the traditional definition?** - PARTIAL

A: An SLI, Service Level Indicator, is a metric you actually measure - p99 latency, error rate, availability. An SLO, Service Level Objective, is a target for that indicator - "p99 latency under 500 milliseconds, 99.9 percent of the time" is a complete SLO. Traditional services can define reliability almost entirely in these terms, because correctness is usually binary - a request either returned the right data or it didn't, and you can measure that with a simple pass/fail check.

AI and agent services break that assumption, which is the real point of this question. An agent can return a response that's successful by every traditional infrastructure metric - 200 status code, low latency, no exception thrown - while still being wrong, unhelpful, or unsafe, because "correctness" for a non-deterministic system isn't a binary the infrastructure layer can check on its own. That means AI-specific SLOs typically need a quality dimension layered on top of the traditional availability and latency dimensions - some measure of output quality, checked either by an eval suite or a sampling-based quality metric, treated with the same seriousness as an uptime target rather than as a separate, softer concern.

The voice bot's CI-gated critical-scenario check is a real example of exactly this pattern, worth connecting directly: it functions as an SLO gate on quality specifically, not on uptime - a regression on a critical scenario blocks the merge the same way a traditional SLO breach would block a deploy, even though nothing about "did this scenario pass" is a conventional infrastructure metric like latency or error rate. That's the instinct this question is actually testing for - do you understand that AI systems need reliability engineering to expand what "reliability" even means, not just apply the old definition to a new kind of service.

---

**Q. What's a rollout strategy, and how is it different from an eval harness - aren't they solving the same problem?** - PARTIAL

A: They're complementary, not substitutes, and the distinction is about timing. An eval harness is a pre-deploy gate - it runs against a fixed set of test scenarios before a change is allowed to merge, and its job is preventing bad changes from ever reaching production in the first place. A rollout strategy is a post-deploy mechanism - canary releases, which expose a change to a small percentage of real traffic before going to everyone; blue-green deployment, which maintains two full parallel environments and switches traffic atomically; feature flags, which let you toggle a change on or off without a redeploy. Its job is limiting the blast radius of a change that already passed the eval gate but turns out to have a problem the eval suite didn't catch.

For AI systems specifically, this extends into shadow-testing - running a new prompt or model version against real production traffic without actually serving its output to users, purely to observe how it would have behaved - and A/B testing two model or prompt variants against live traffic to compare real-world performance rather than only performance against a fixed test set. You don't have a rollout-strategy story from your own projects specifically - the voice bot's eval harness is entirely a pre-deploy mechanism, and nothing in what's documented adds a post-deploy gradual-exposure layer on top of it. That's an honest, precise gap worth naming directly if asked, rather than trying to stretch the eval harness into covering both concerns - the fact that you can articulate why they're different, and that the eval harness alone doesn't cover the rollout-strategy half, is itself a stronger answer than conflating the two.

---

## 5. Enterprise integration

**Q. How do you approach integrating an agent securely with enterprise systems - APIs, identity, secrets?** - STRONG/PARTIAL mix

A: The general pattern across your work is the same regardless of which enterprise system is involved: never let the component with the most exposure - the conversational or generative surface - hold direct credentials or direct access to the sensitive system; put a narrower, auditable boundary component in between.

Three concrete examples demonstrate this. Colgate's API contract, integrating 6 enterprise services, used standard JWT/OAuth-style patterns for service-to-service auth, with the actual reliability engineering focus on the contract's versioning discipline rather than novel auth mechanics. The voice bot's HMAC-signed request pattern is a more agent-relevant example - the Integration Layer, not the conversational surface, holds the only credentials capable of touching the EHR, and the orchestrator authenticates to it via a shared secret verified through HMAC, never a direct database or API credential. Tome Raider's stateless ES256/JWKS authentication is a third pattern - asymmetric key verification, meaning the service verifying a token doesn't need to hold the same secret used to issue it, which matters for scaling verification across multiple service instances without needing to distribute a shared secret to all of them.

The honest gap worth naming directly: none of these examples involve an agent needing scoped, time-limited credentials to act on a user's behalf specifically - which is a meaningfully harder problem than service-to-service auth, since it requires the credential itself to carry the authority boundary (this agent can only do X, only for user Y, only for the next Z minutes) rather than just proving which service is calling. That's a real, current problem in agentic system design - sometimes discussed under the term "delegated authority" or "on-behalf-of" tokens - and it's fair to say directly that your production experience is with service-level auth, not yet with scoped per-agent delegated credentials, if asked specifically about that distinction.

---

## 6. AgentOps / LLMOps

**Q. What's the difference between infrastructure monitoring and AI-quality monitoring - which have you actually built?** - STRONG (infra) / honest distinction needed

A: Infrastructure monitoring answers "is the service up and performing within normal bounds" - error rates, latency, resource usage, cost anomalies - and it's largely agnostic to what the service actually does semantically. AI-quality monitoring answers a different question entirely - "is the model still producing good outputs, has behavior drifted from what was validated" - and it can't be answered by infrastructure metrics alone, since a model can be perfectly healthy from an infra perspective while quietly degrading in output quality.

The cloud-ops alerting platform is a real, strong example of infrastructure monitoring specifically - it fans SLO burn, drift, error rate, and cost anomaly signals from Cloud Monitoring into a single alerting pipeline, and every one of those signal types is an infra-level concern, not a semantic one. It's worth being direct about this if asked to describe it as an "AI monitoring" system: it monitors the infrastructure that AI systems (among others) run on, but it doesn't inspect model outputs for quality or drift - that's a different, harder problem that would need something like periodic sampling of production outputs scored against a rubric, or tracking a proxy metric like user correction rate, neither of which is part of what's built. Naming that gap precisely, rather than letting "monitoring platform" imply more AI-specific sophistication than it has, is exactly the kind of honest scoping that's been the theme throughout this whole prep process.

---

**Q. What's experimentation in an AI/LLMOps context, and have you run anything like it?** - GAP

A: Experimentation here means structured A/B testing of model or prompt variants against real usage, with enough rigor to draw a statistically valid conclusion about which variant actually performs better - not just eyeballing outputs from two versions and picking the one that feels better. It requires a holdout or split mechanism to route real traffic to different variants, a defined success metric measured consistently across both, and enough traffic volume for the comparison to reach statistical significance rather than being noise.

This is a genuine gap - nothing in your projects runs a formal experimentation framework like this. The honest answer if asked directly: your evaluation work has been entirely offline, against fixed test sets before deployment, not online A/B comparison of live variants. That's a real and fair thing to say plainly, and it's worth pairing with a concrete sense of how you'd approach it if asked to design one - for a system like Tome Raider, a natural experimentation setup would compare two grading thresholds or two rewrite-retry caps against a metric like "percentage of queries reaching the honest fallback," split across user cohorts, which is a specific, defensible answer even without direct hands-on experience running it.

---

**Q. What are safety guardrails, and where have you actually built them?** - PARTIAL

A: Guardrails are checks that constrain what a generative system can input or output, independent of the model's own judgment - screening for content that shouldn't be processed or produced, regardless of whether the model itself would have flagged it as a problem. The key property, same as policy enforcement above: a guardrail has to be enforced by code the model can't reason its way around, not a soft instruction in a system prompt hoping the model complies.

Tome Raider's prompt-injection screening and PII masking, on both input and output, is a real guardrail system - worth naming it explicitly that way rather than only describing the mechanism, since "guardrails" is the term this question is fishing for. Injection screening exists because a document a user uploads, or even a user's own message, could contain text specifically crafted to hijack the model's instructions - a guardrail here inspects content before it reaches the model's context, independent of whether the model itself would recognize the attempt. PII masking on the output side exists for the reverse reason - even a well-intentioned model can inadvertently surface sensitive information present in retrieved context, so masking happens as a final check on what actually leaves the system, not just a hope that the model won't repeat something sensitive.

---

**Q. What's prompt or tool versioning, and why does it matter in production?** - GAP

A: Prompt and tool versioning means treating prompts, and the schemas that define available tools, as versioned artifacts with the same rigor as application code - tracked in version control, deployed deliberately, with the ability to roll back a specific prompt version if it turns out to cause a regression, rather than prompts living as untracked strings scattered through application code with no history of what changed or when. It matters in production because a prompt change can silently alter model behavior in ways that are much harder to catch in code review than a typical code diff - a small wording change can shift output quality in ways that only show up under real usage, so being able to pin, compare, and roll back specific prompt versions the same way you'd roll back a bad deploy is a real operational need, not a nice-to-have.

Nothing in your projects implements this formally, but it connects naturally to a versioning instinct you've already demonstrated elsewhere - EphFlow's FaaSr.schema.json is itself a versioned contract that workflow submissions are validated against, and the same discipline (a schema or prompt as a tracked, versioned artifact rather than an implicit assumption baked into code) is the same underlying principle applied to a different kind of artifact. Worth drawing that parallel directly if asked, since it shows the instinct transfers even without a literal prompt-versioning system in your own build.

---

## 7. Emerging technology

**Q. What's speculative decoding, and how does it speed up generation?** - GAP

A: Speculative decoding uses a small, fast "draft" model to propose several tokens ahead speculatively, which the larger, more capable target model then verifies in a single parallel pass rather than generating each token one at a time itself. If the draft model's guesses match what the larger model would have generated anyway - which happens often for easy, predictable continuations - you get several tokens' worth of output for roughly the cost of one large-model forward pass. Where the draft diverges from what the large model would have chosen, the large model's own output is used instead from that point forward, so correctness isn't compromised - the technique is purely a speed optimization on top of the large model's actual output distribution, not an approximation of it. This is a genuine infrastructure-level technique with no direct connection to your own project work - worth knowing the mechanism cleanly if asked, without pretending to hands-on experience implementing it.

---

**Q. Why does RAG still matter as context windows get much longer - isn't retrieval becoming obsolete?** - GAP, but connects directly to Tome Raider

A: This is a legitimate tension worth being able to discuss rather than dismiss. As context windows grow into the hundreds of thousands or millions of tokens, the naive alternative to retrieval becomes "just include the entire corpus in context every time" - and for a genuinely small, static corpus, that can work. But it breaks down along several real dimensions even with a very large window. Cost scales with context length regardless of window size - processing a million tokens of mostly-irrelevant context on every single query is far more expensive than retrieving the handful of chunks that actually matter, every time, at scale. Latency scales similarly - a longer context takes longer to process even before generation starts. And there's a real quality concern often called "lost in the middle" - even models with very long context windows show measurably worse recall for information buried in the middle of a very long context compared to information near the beginning or end, meaning a bigger window doesn't guarantee the model actually uses distant information well.

Tome Raider's own design is a live answer to this question - it retrieves a small number of relevant chunks (`retrieval_k=4`) rather than ever attempting to load a full document corpus into context, precisely because cost, latency, and lost-in-the-middle risk all get worse, not better, as you try to substitute raw context length for actual relevance filtering. The honest nuance worth adding: for genuinely small, static, frequently-reused corpora, long context can be a real substitute for retrieval - it's not that retrieval is always superior, it's that the tradeoff shifts based on corpus size, update frequency, and query volume, and Tome Raider's use case - large, per-user, frequently updated document sets - sits squarely on the side where retrieval remains the better architecture even in a long-context world.

---

**Q. What's architecturally different about a reasoning model like o1 versus prompting a standard model to think step by step?** - GAP

A: Prompting a standard model to "think step by step" is a technique applied at inference time to a model that wasn't specifically trained for it - it often helps, but the model is essentially just continuing a pattern it learned from step-by-step reasoning examples in its training data, with no architectural mechanism dedicated to reasoning specifically. Reasoning models like the o1/o3 family are trained differently and use what's called test-time compute deliberately - the model generates a substantial number of internal reasoning tokens, exploring and often backtracking through a problem, before committing to a final answer, and this behavior is shaped specifically through reinforcement learning that rewards arriving at correct answers via extended reasoning traces, not just imitating step-by-step examples from training data. The practical tradeoff is real and worth naming: reasoning models are slower and more expensive per query, because that extended internal reasoning process consumes tokens and time before any user-facing answer appears - so the decision to use one is a real cost-latency tradeoff against the value of higher accuracy on genuinely hard multi-step problems, not a strictly-better replacement for a standard model on every task.

---

**Q. What's your actual take on AI-assisted development tools, and how do you use them in your own workflow?** - GAP in JD-naming, but should have a genuine current answer

A: The JD names Codex, Claude Code, Cursor, and Copilot specifically as tools the role expects familiarity with, alongside "agentic-first development" as a broader philosophy - designing software with the expectation that AI agents, not only humans, will be primary operators or contributors to a codebase, which is a notably self-referential detail given Claude Code is itself Anthropic's own example of exactly that. This is a low-technical-depth but high-authenticity question - it's less about reciting a definition and more about having a genuine, current, specific answer about your own workflow: which tools you actually use, for what kinds of tasks you trust an AI-assisted tool with versus where you still want to write it yourself, and where you've seen these tools fail or mislead you. A vague "I use AI tools to be more productive" answer is weak; a specific example - a real task you delegated, what worked, what you had to correct - is strong, and this is worth having ready with an honest, current, personal answer rather than a generic industry-trend summary.

---

## New questions - added as we go

This section grows as you ask follow-ups during study. Same format as above: question, status tag, in-depth answer with an example tied back to your own work wherever a real connection exists.

**Q. What do EphFlow's decision primitives (reassign_platform, flip_requires_vm) actually do mechanically, and how does the agent choose between them?** - directly deepens the tool-calling answer in section 1

A: `flip_requires_vm` flips the target action's `RequiresVM` field, triggering the paper's DAG-augmentation logic - VM Start/Poll/Stop injected around that action, moving it off a resource-ceilinged serverless platform onto a VM. `reassign_platform` changes which of the five platforms runs that action without touching `RequiresVM` - the fix for a failure that isn't resource-related, like a missing dependency or a platform-specific rate limit, constrained by whether the target platform actually supports the action's language. The choice between them comes from the failure signature: a resource-ceiling breach (runtime past MaxRuntime, an OOM matching the platform's memory limit) points to `flip_requires_vm`; anything else - a dependency error, a platform-specific issue - points to `reassign_platform`, since spinning up a VM unnecessarily is slower and more expensive than trying a different serverless platform. Full worked example (a dependency-error case, contrasted with the resource-ceiling case already in the deep-dive doc) is in `Oracle_Project_Deep_Dive_Prep_v2.md` under EphFlow.

*(Nothing else added yet.)*
