# How CibrAI Automated 80% of Their Security Analyst Workflow With Agentic AI

*A case study in building pragmatic agent systems for cybersecurity operations — written for CISOs and security leaders who are tired of throwing more headcount at alert fatigue.*

---

When CibrAI's security team was drowning in alerts, they didn't need more analysts — they needed AI that could triage like one.

That single reframing became the starting point for one of the most rewarding engineering engagements we've had in the last eighteen months. This is the story of how a focused, two-phase agentic AI build freed the CibrAI security team from tier-1 triage drudgery, sharpened their mean time to resolution on real incidents, and gave their senior analysts their weekends back.

It's also a story about restraint. We didn't build a chatbot. We didn't replace anyone. We built an agent that behaves like a junior analyst who never sleeps and never panics — and we put the senior humans firmly in charge of anything that matters.

---

## The Problem: A Wall of Alerts, a Finite Team

CibrAI is a cybersecurity firm with a lean, high-signal detection team. Like most modern security organisations, their stack produces a large and relentless stream of telemetry: EDR alerts, identity anomalies, cloud posture findings, network IDS hits, phishing reports, and vulnerability notifications. On a typical weekday their analysts were processing thousands of incoming events.

The problem wasn't detection quality. Their tooling was good. The problem was human attention.

Before the engagement, their workflow looked roughly like this:

- Every alert landed in a shared queue, regardless of severity or context.
- Analysts opened each ticket cold, without historical context pre-attached.
- Roughly 70-80% of alerts turned out to be benign once enriched — known-good processes, expected admin behaviour, stale IOCs, or low-confidence correlations.
- Senior analysts were routinely pulled into low-severity triage instead of doing the threat hunting and response work they were hired for.

The cost wasn't just time. It was judgement quality. When a team spends seven hours out of eight clicking through benign alerts, the one real incident in that shift gets the tired brain.

The CISO, Andy Curtis, put it plainly in our first call: the team didn't need a bigger headcount. It needed the first pass to stop being a human problem.

---

## The Approach: An Agent, Not an Automation

There's a temptation in security automation to reach for rigid playbooks — SOAR-style flowcharts that execute the same sequence every time. They work until they don't. Real alert contexts are messy and the interesting decisions are almost always the ones the playbook author didn't anticipate.

We took a different path: an [agentic AI system](https://hjlabs.in/AIML/services/agentic-ai/) that reasons about each alert, decides what information it needs, fetches that information using tools, and escalates when it reaches the edge of its confidence.

The design principles we agreed on up front:

1. **The agent never takes destructive action.** It can read, query, correlate, and recommend — but anything that touches production (isolating a host, disabling an account, blocking a hash) stays with a human.
2. **Every decision must be auditable.** Each triage decision produces a structured rationale with the tool calls, evidence, and confidence score that led to it.
3. **The agent must know when it doesn't know.** Escalation to a human analyst is a first-class outcome, not a failure mode.
4. **The senior analysts define truth.** The ground-truth labels that trained the retrieval layer and shaped the prompts came from their historical triage decisions, not from a generic threat taxonomy.

---

## Technical Architecture

The system is built around a central agent loop with tool use, grounded by a retrieval-augmented generation (RAG) layer over the customer's own security knowledge.

**The agent loop.** A reasoning LLM sits at the centre, receiving each new alert as a structured input. It plans — what do I need to know to classify this? — and then executes tool calls to gather evidence. After each tool call it re-evaluates, either asking for more information or committing to a classification and recommended action.

**Tool surface.** We gave the agent a carefully scoped set of tools:

- SIEM query tools for pulling related events in a time window around the alert
- Asset context lookups (is this host a domain controller? a developer laptop? a crown-jewel server?)
- Identity context lookups (is this user a privileged admin? on PTO? recently onboarded?)
- Threat intelligence enrichment for IPs, domains, and file hashes
- Historical case lookup — "have we seen a similar alert pattern before, and how did we triage it?"
- A human-escalation tool that opens a high-priority ticket with the agent's full reasoning trace attached

**The RAG layer.** This is the piece that turned a generic capable model into a CibrAI-specific analyst. We indexed their runbooks, historical closed tickets, internal threat intel notes, and their written detection logic. The agent retrieves from this corpus before committing to a classification, so its judgement reflects how *this team* thinks about alerts, not how the public internet does.

**Observability.** Every run produces a structured trace: inputs, retrieved documents, tool calls, intermediate reasoning, final verdict, confidence. These traces feed a weekly review where senior analysts flag disagreements — and those flags become the next round of RAG corpus improvements. The feedback loop is the product.

---

## Results

We rolled out in two phases. Phase one ran in shadow mode for six weeks — the agent triaged every alert alongside the humans, but its decisions didn't route tickets. We compared its verdicts against analyst decisions, tuned the retrieval corpus, tightened the prompts, and added tools where the agent was consistently asking for information it couldn't reach.

Phase two put the agent in the live path for tier-1 triage, with mandatory human escalation above a configurable confidence threshold.

The headline numbers after the first full quarter in production:

- **~80% of tier-1 triage is now handled by the agent end-to-end.** These are the clear benign-by-context alerts and the clear low-severity confirmations that previously ate senior analyst hours.
- **Mean time to resolution on genuine incidents dropped materially.** Because the agent pre-enriches every escalated ticket with its full reasoning trace, analysts start investigations with context already attached instead of building it from scratch.
- **Senior analysts shifted time into proactive threat hunting.** The hours freed didn't disappear into the backlog — they went into higher-leverage work that the team was previously postponing indefinitely.
- **Analyst-reported alert fatigue dropped noticeably.** Subjective, but important. The team is doing more of the work they were hired to do.

As Andy Curtis, CISO at CibrAI, put it in his reference for the engagement:

> "Fantastic AI engineer with pragmatic business and technical skills."

That word — pragmatic — is the one we're proudest of. The engagement didn't sell a vision. It shipped a working system and then kept tuning it.

---

## Four Lessons for Other Security Teams

If you're considering a similar build, these are the lessons that would have saved us weeks:

**1. Start in shadow mode, and budget real time for it.** Six weeks of shadow evaluation sounds long until you see the categories of edge case that only surface in production traffic. Don't cut this phase short.

**2. Your runbooks are your moat.** The difference between a generic agent and one that behaves like a member of your team is almost entirely in the retrieval corpus. Invest heavily in curating historical tickets, runbooks, and team conventions before you tune a single prompt.

**3. Confidence thresholds are a product decision, not a model decision.** Where you set the escalation cutoff determines the split between analyst time saved and analyst trust earned. Start conservative, measure, and loosen deliberately.

**4. The agent's reasoning trace is as valuable as its verdict.** Analysts adopted the system faster once they realised the escalated tickets arrived with a structured explanation they could audit in seconds. Make the trace a first-class output, not a debug log.

---

## Closing

If your security team is stuck in manual triage, an agentic AI approach can transform the workflow without replacing people. Done well, it doesn't introduce a black box into your SOC — it introduces a tireless junior analyst whose reasoning you can inspect, correct, and improve every week.

You can read more engagements like this one on our [case studies page](https://hjlabs.in/AIML/case-studies.html), or book a 30-minute consultation at [cal.com/hemangjoshi37a](https://cal.com/hemangjoshi37a) to talk through what an agentic triage layer could look like on top of your existing stack.

No pitch deck. Just a conversation about whether the approach fits your team.

---

#AgenticAI #Cybersecurity #SOCAutomation #CISO #ThreatDetection #AIinSecurity #SecurityOperations #LLMAgents #RAG #IncidentResponse #AIEngineering #CaseStudy
