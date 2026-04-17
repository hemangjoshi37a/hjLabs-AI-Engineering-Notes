# Deploying Autonomous AI Agents in Production: A 2-Week Playbook

Most teams quote 3-6 months to deploy production AI agents. We've done it in 2 weeks. Repeatedly. Not demos. Not hackathon toys. Agents that handle real tickets, call real tools, pass real evals, and don't set the observability dashboard on fire at 2 AM.

The difference isn't some secret framework. It's ruthless scoping, opinionated tooling choices, and a hardening phase that most teams skip because they're still arguing about whether to use LangGraph or AutoGen on day 40.

Here is the exact playbook we run for enterprise AI teams. Fourteen days, start to production. Steal it.

---

## Days 1-2: Scoping and Success Metrics

If you skip this step, the rest of the playbook collapses. Ninety percent of "failed" agent projects I've inherited failed here, not in the code.

The first question is not "which framework" or "which model." It is: **what does a working agent actually output, measured how, at what cost, at what latency?**

On day one we lock down four numbers:

1. **Task success rate target.** For a tier-1 support triage agent, we aim for 85% deflection on classified intents with <2% escalation-to-human-too-late. For a sales research agent, 90% correct CRM field population on a 200-row golden set.
2. **P95 latency budget.** Streaming-first agents usually land at 4-8s time-to-first-token and 20-40s time-to-final. Non-streaming batch agents get 60-120s. Anything over that and users will abandon or your queue depth explodes.
3. **Unit economics ceiling.** Cost per successful task, not cost per token. A $0.40/task agent that replaces a $12 human action is a business. A $0.08/task agent that hallucinates 15% of the time is a liability.
4. **Blast radius.** What's the worst thing this agent can do to a production system if it misfires? If the answer is "wire money" or "drop a database," we architect a human-in-the-loop gate from hour one, not hour 200.

We run a one-hour scoping workshop with the product owner and two senior engineers from the customer side. The output is a one-page spec with those four numbers, three to five representative tasks, and a list of tools the agent must call. That's it. If the customer can't agree on the four numbers, we don't start coding. We've walked from projects at this stage. Nothing saves more time.

## Days 3-5: Framework and Tool Selection

By now half the reader is shouting "just tell me which framework." Fine.

**CrewAI** wins when the workflow is naturally role-based and relatively linear: a researcher agent, a writer agent, an editor agent, pass the baton. It's fast to stand up, the abstractions match how non-engineers think about work, and it plays well with both OpenAI and Anthropic tool-calling. Weakness: anything with complex conditional branching or long-running state becomes awkward.

**LangGraph** wins the moment the agent needs real state, cycles, interrupts, or human-in-the-loop checkpoints. It is our default for anything touching financial workflows, medical triage, or multi-step enterprise processes with approval gates. The graph model maps cleanly to real business processes. The learning curve is real, though; budget a day for your team to internalize the state schema discipline.

**AutoGen** wins for research-flavored problems where multiple agents genuinely debate, critique, and iterate. It is overkill for anything deterministic, and we usually don't ship it to production untouched. We port the design to LangGraph once the conversation topology stabilizes.

**Raw tool-calling loops with no framework** win more often than anyone admits. If your agent is a single role with 4-8 tools and a 20-step max horizon, a 200-line custom loop with structured outputs will beat any framework on latency, debuggability, and on-call sanity. Frameworks are not free; they add call overhead, hidden prompt templates, and upgrade risk.

Tool definitions matter more than framework choice. We standardize on JSON Schema with strict mode enabled (OpenAI) or XML-tagged tool-use blocks (Anthropic). Every tool gets a deterministic name, a one-sentence description optimized for the LLM not the human, typed parameters with enums wherever possible, and a structured error return so the agent can self-correct instead of hallucinating a retry.

More on our approach to agent architectures here: https://hjlabs.in/AIML/services/agentic-ai/

## Days 6-8: Build the PoC Agent

Three days. Not three weeks. If the PoC takes longer, the scope is wrong.

The system prompt is written last, not first. We start with tool definitions and a golden task, let the model try to solve it with zero instructions, and watch where it fails. The system prompt is written to patch those specific failures, not to pre-empt imagined ones. Short, behavioral, and in present tense. We keep it under 800 tokens in the final version. Long system prompts are a smell.

Memory: working memory is the conversation plus a scratchpad. Episodic memory goes in Postgres with a simple schema. Semantic memory, if needed at all, goes in the RAG layer (next section), not in a "memory framework." The cottage industry of agent memory frameworks has produced more LinkedIn posts than shipped products.

Error handling: every tool call wraps in a retry with exponential backoff on transient errors, a structured error return on permanent ones, and a hard circuit breaker on the total step budget. An agent that has taken 30 tool calls to solve a task is almost certainly in a loop. We kill it, log the transcript, and fall back to human handoff. Never let an agent run unbounded in production. This is the single most common production outage pattern we see when auditing other teams' deployments.

We enable prompt caching from day one. On Anthropic, cache the system prompt and tool definitions; on OpenAI, rely on automatic prefix caching and order your prompt carefully to maximize hits. A well-cached agent costs 40-70% less at scale than the same code without caching, and the change is literally two lines.

## Days 9-10: RAG Knowledge Base

Most agents that "hallucinate" are actually starving for context. A good RAG layer fixes 80% of perceived agent quality issues.

Our defaults:

- **Chunking.** Semantic chunking on paragraph and section boundaries, not fixed-size. 400-800 tokens per chunk with 15% overlap. Tables and code get their own chunk type with preserved structure.
- **Embedding model.** `text-embedding-3-large` for English-dominant corpora, `voyage-3` for technical documentation, `cohere-embed-multilingual-v3` for multilingual. We benchmark on the customer's own content before committing; generic leaderboards lie about domain performance.
- **Vector store.** Qdrant for self-hosted production (binary quantization gets you 32x memory compression with <1% recall loss), Weaviate when hybrid search and BM25 matter out of the box, Pinecone when the team wants zero ops and will pay for it. We avoid pgvector at scale; it's fine under 500k vectors, painful above.
- **Reranker.** Non-negotiable. Cohere Rerank 3 or a fine-tuned BGE reranker on top-50 candidates, returning top-5 to the LLM. Rerankers typically add 15-25 points of answer quality on our evals for about 80ms of latency. This is the highest-ROI component in the stack.
- **Query rewriting.** A small model (Haiku, GPT-4o-mini) rewrites the user query into 2-3 retrieval queries before search. Cheap, dramatic recall improvement on conversational inputs.

Full writeup of our RAG architecture: https://hjlabs.in/AIML/services/rag-systems/

## Days 11-12: Evaluation Harness

No evals, no production. Full stop.

The golden dataset is built by hand, by a domain expert, in a spreadsheet. 80-150 tasks, each with the input, the expected tool-call trajectory (not verbatim, but the set and rough order), and the expected final output shape. Yes, hand-built. Synthetic eval sets have their place, but the foundation has to be real.

We run three layers of evals:

1. **Deterministic checks.** Did the agent call the required tools? Did the output parse as valid JSON against the schema? Did it stay under the step budget? These run in <5 seconds per task and catch 60% of regressions.
2. **LLM-as-judge.** A stronger model (Claude Opus or GPT-4o) grades task success against a rubric. We version the rubric and the judge model together. Pairwise comparison beats absolute scoring for subjective tasks.
3. **Human spot checks.** 10-20 tasks per release, reviewed by the domain expert. This catches the rubric drift that pure LLM-as-judge will always miss eventually.

The eval harness runs in CI on every prompt or tool change. A regression in task success rate blocks the merge. This single practice separates teams that ship from teams that demo forever.

Langfuse or Arize as the trace and eval backend. We've used both in production; Langfuse is faster to self-host, Arize has the edge on enterprise features and SOC2 paperwork.

## Days 13-14: Production Hardening

The last two days are where most teams run out of budget and ship a prototype to prod. Do not be most teams.

Observability: every agent run emits a structured trace with inputs, every tool call, every LLM call with token counts, and the final output. Langfuse or Arize, tagged by customer, environment, and agent version. On-call engineers must be able to pull a full trace for any user-reported issue in under 60 seconds. If they can't, you have no observability, you have logs.

Rate limiting: per-user, per-tenant, per-tool. A runaway agent that hammers an expensive tool 200 times is a six-figure incident, and we've seen it happen.

Fallbacks: every agent has a tiered model strategy. Primary model with streaming, secondary model on timeout or 5xx, static fallback response on total failure. Users see degradation, not outages.

Cost monitoring: dashboards per agent per customer, with alert thresholds at 1.5x and 3x daily budget. Token costs are the most unpredictable line item in any AI product. Treat them like an unmetered cloud service, because that is what they are.

Guardrails: input and output. Input guardrails block prompt injection and prevent out-of-scope queries from reaching the agent. Output guardrails validate structure, redact PII, and check for policy violations before response is emitted. We use Guardrails AI or NeMo Guardrails for heavier needs, a 50-line custom validator for simpler ones. Do not skip this layer because the LLM "seems to behave"; it will stop behaving on a Friday evening.

---

## The Real Lesson

Two weeks is not a magic number. It is the result of refusing to let any single phase expand beyond its budget. Scoping eats forever if you let it. Framework debates eat forever if you let them. RAG tuning eats forever if you let it. The playbook works because each phase has a hard stop and a crisp deliverable.

Teams that struggle are almost never struggling with the model. They are struggling with scope discipline, eval discipline, and the unsexy production hardening work that doesn't make for good demos.

Want this playbook executed for your team by engineers who have shipped it to enterprise production more than a dozen times? We run this exact process for enterprise AI teams end to end. Book a 30-min scoping call: https://cal.com/hemangjoshi37a

---

#AIAgents #LLM #MachineLearning #AIEngineering #RAG #LangGraph #CrewAI #ProductionAI #MLOps #GenerativeAI #EnterpriseAI #AgenticAI #AIInfrastructure #LLMOps #SoftwareEngineering
