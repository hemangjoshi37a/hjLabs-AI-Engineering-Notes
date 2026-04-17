---
layout: article
title: "The Enterprise AI Buyer's Checklist: 12 Questions to Ask Before Hiring an AI Consultancy"
subtitle: "Four groups of three questions: Delivery Proof, Technical Depth, Engineering Practices, Business Fit."
description: "The checklist I wish every CTO and VP of Engineering had before signing an AI consulting SOW. Each question with why it matters, green-flag answer, and red-flag answer."
cover: /assets/images/05-cover.png
reading_time: "10 min read"
tags: [enterprise-ai, ai-consulting, procurement]
order: 5
canonical: "https://www.linkedin.com/pulse/enterprise-ai-buyers-checklist-12-questions-ask-hiring-joshi/"
---

After auditing dozens of failed AI consulting engagements, I've noticed buyers keep asking the wrong questions. "Do you have AI expertise?" "Can you build an LLM app?" "What's your day rate?" These questions filter nothing. Every consultancy on Earth answers yes, yes, and a reasonable number. Six months later, the buyer has a stalled proof-of-concept, a burned budget, and an internal team that trusts "AI vendors" a little less than they did before.

The real problem isn't vendor dishonesty. It's that the questions on most RFPs were written for the era of web development and haven't been updated for a field where the tooling changes every quarter and "done" is genuinely hard to define. So here is the checklist I wish every procurement leader, CTO, and VP of Engineering had in front of them before they signed a statement of work. It's organized into four groups: Delivery Proof, Technical Depth, Engineering Practices, and Business Fit.

Use it even if you don't hire us. That's the point.

---

## Group 1 — Delivery Proof (Questions 1-3)

### 1. "Can you show me three production systems you've shipped in the last 18 months?"

**Why it matters:** AI moves fast enough that a portfolio from 2022 is effectively archaeology. Retrieval-augmented generation, agent frameworks, evaluation tooling — almost none of the stack a serious team uses today existed in production form three years ago. You need proof of recent shipping.

**Green flag:** They walk you through specific deployments, name the models and frameworks used, describe user load, and tell you what broke in the first 30 days. Bonus if they can share anonymised architecture diagrams or [case studies](https://hjlabs.in/AIML/case-studies.html).

**Red flag:** They show you demo videos, sandbox prototypes, or hackathon wins. "We can't share that, it's under NDA" across all three requests usually means there isn't a third.

### 2. "What's your average PoC-to-production conversion rate?"

**Why it matters:** The dirty secret of AI consulting is that most PoCs die in the handoff. A consultancy that averages 20% conversion is selling science experiments. One that averages 70%+ is selling systems. You want the latter.

**Green flag:** A specific number, with context — "About 65%, and the ones that don't convert usually fail at data readiness, which we now audit in week one." That's a team that has learned from its losses.

**Red flag:** "Every PoC we've done has gone to production." That's either untrue or the PoCs are so cautious they prove nothing.

### 3. "Walk me through a project that went wrong and how you handled it."

**Why it matters:** Every consultancy has wreckage. The ones worth hiring have processed theirs. This question separates teams that learn from teams that perform.

**Green flag:** A genuine, slightly uncomfortable story — with a clear description of the root cause (data drift, scope creep, a hallucination that reached production), the recovery plan, and what changed in their internal process afterwards.

**Red flag:** A sanitised story where the client was the villain, or worse, "Nothing's ever really gone wrong." Walk away.

---

## Group 2 — Technical Depth (Questions 4-6)

### 4. "Which agent framework — CrewAI, LangGraph, AutoGen, or something else — would you pick for my use case, and why?"

**Why it matters:** There is no universally correct answer. LangGraph gives you explicit state machines and observability. CrewAI optimises for role-based collaboration. AutoGen leans toward conversational multi-agent patterns. Custom orchestration beats all three when latency or determinism matters. A serious partner has opinions, and those opinions are tied to your workload — not to whichever framework they learned first.

**Green flag:** They ask about your latency budget, failure tolerance, observability needs, and team's ability to maintain the system before recommending anything. They can explain trade-offs of at least two options. See how we think about this in our [agentic AI practice](https://hjlabs.in/AIML/services/agentic-ai/).

**Red flag:** "We always use [framework X] — it's the best one." That's a hammer looking for a nail.

### 5. "How do you evaluate LLM output quality over time?"

**Why it matters:** LLM outputs drift. Base models get silently updated. Prompts that scored 92% on your eval set in January can score 78% in June without a single line of code changing on your side. If your consultancy doesn't have a continuous evaluation story, they're shipping a time bomb.

**Green flag:** Specific eval frameworks (Ragas, DeepEval, Promptfoo, custom LLM-as-judge pipelines), a test set versioning strategy, scheduled regression runs in CI, and dashboards showing quality trends per release.

**Red flag:** "We spot-check outputs." Spot-checking is not evaluation.

### 6. "Describe your approach to RAG — what chunking, embedding, and reranking strategy?"

**Why it matters:** Naive RAG — split documents, embed with the default model, cosine-similarity top-k — produces demos, not products. Production RAG requires hybrid retrieval, semantic chunking, query rewriting, and a reranker. If your vendor's RAG story starts and ends with a vector database, expect mediocre recall.

**Green flag:** They talk about chunk size experimentation, parent-document retrieval, BM25 + vector hybrid search, query expansion, and a reranker such as Cohere Rerank or a cross-encoder. They also mention how they evaluate retrieval separately from generation.

**Red flag:** "We just use [vector DB] with OpenAI embeddings and that works fine." It works fine for hello-world.

---

## Group 3 — Engineering Practices (Questions 7-9)

### 7. "How do you handle prompt versioning and rollback?"

**Why it matters:** Prompts are code. They need to be versioned, reviewed, A/B tested, and rolled back when they misbehave. Consultancies that store prompts as string literals in Python files are about to ship you a maintenance nightmare.

**Green flag:** Prompts live in a registry (PromptLayer, LangSmith, Langfuse, or a custom system), are tied to release versions, and every deployment can be rolled back in under five minutes.

**Red flag:** "We update the prompt directly in the code and redeploy." That's not versioning — that's hope.

### 8. "What's your approach to production observability for AI systems?"

**Why it matters:** Traditional APM tools don't cover the things that actually break AI systems: token cost spikes, hallucination rates, tool-call failures, context window exhaustion, provider outages. You need AI-native observability.

**Green flag:** Distributed tracing of agent steps (LangSmith, Langfuse, Arize, Helicone), latency/cost/quality dashboards per route, alerting on hallucination-rate regressions, and a clear incident response playbook for model provider outages.

**Red flag:** "We log everything to CloudWatch." That's plumbing, not observability.

### 9. "How do you implement guardrails and safety rails?"

**Why it matters:** A regulated enterprise cannot afford a single prompt-injection incident or PII leak. Safety cannot be an afterthought bolted on before launch — it has to be part of the architecture from day one.

**Green flag:** Layered defences — input validation, output validators (NeMo Guardrails, Guardrails AI), PII redaction pipelines, jailbreak detection, content filtering, rate limiting, and audit logs that would survive a compliance review.

**Red flag:** "The model won't say bad things — we tested it." Testing is not a guardrail.

---

## Group 4 — Business Fit (Questions 10-12)

### 10. "Who specifically will be on my engagement — and are they senior?"

**Why it matters:** The bait-and-switch is the oldest consulting scam: senior engineers sell the deal, juniors deliver it. For AI work, this is lethal — the gap between a senior and junior engineer on ambiguous ML problems is often 10x, not 2x.

**Green flag:** Named individuals with LinkedIn profiles, GitHub histories, and relevant recent projects. A written commitment that the lead engineer will remain on the engagement for its duration.

**Red flag:** "We'll assign the right people at kickoff." That means whoever is on the bench.

### 11. "What's your communication cadence and escalation process?"

**Why it matters:** AI projects have more unknowns than traditional software. You need more communication, not less — and you need a clear escalation path when a technical surprise threatens the timeline.

**Green flag:** Weekly written updates, a shared Slack or Teams channel, a named escalation contact, and a 24-hour response SLA for blocking issues. Demos every two weeks, not quarterly reveals.

**Red flag:** "We'll send a monthly status report." In AI timelines, a month is a season.

### 12. "If we wanted to take the system fully in-house in six months, how would you enable that?"

**Why it matters:** A good consultancy makes itself replaceable on purpose. A bad one architects lock-in. This question instantly reveals which one you're sitting across from.

**Green flag:** A concrete knowledge-transfer plan — documentation standards, pair programming with your engineers, runbooks, architecture decision records, training sessions, and a formal handover milestone in the SOW.

**Red flag:** "Why would you want to do that?" or vague reassurances about "great documentation." You're being sold a subscription, not a system.

---

## Using the Checklist

You don't need all 12 answers to be perfect. You need all 12 to be **specific, grounded, and intellectually honest**. A consultancy that responds to every question with concrete examples, named tools, and genuine trade-offs is a partner. One that responds with generalities, buzzwords, or "we customise our approach to every client" is a future line item in your post-mortem.

The premium you pay a serious partner is real. So is the premium you pay an unserious one — it just gets billed later, in missed timelines, rebuilt systems, and internal credibility you can't buy back.

Print this checklist. Take it into your next vendor meeting. Watch the room.

And if you'd like to see how we'd answer all 12 — with specifics, not slides — book 30 minutes at [cal.com/hemangjoshi37a](https://cal.com/hemangjoshi37a). Bring the hardest question on your list.

---

*Hemang Joshi leads hjLabs.in, a B2B AI/ML consultancy building production agentic systems for enterprises in the US, UK, and the Gulf. He has spent the last decade shipping ML systems that have to survive Monday morning, not just demo day.*

---

**Hashtags:** #EnterpriseAI #AIConsulting #CTO #AgenticAI #LLMOps #RAG #AIStrategy #MachineLearning #AIProcurement #DigitalTransformation #TechLeadership #AIGovernance #AIEthics #ProductionAI #B2BTech
