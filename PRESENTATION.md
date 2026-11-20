# CloudServe | AI-Powered Ticket Support

**From a chatbot request to a governed, multi-channel support system**  
Souvik Golui | Forward Deployed AI Engineering Capstone

---

# The Problem

>The request was narrower than the need

**CloudServe asked for:** A chatbot that resolves at least **65% of tickets without escalation**.

**Discovery found:** A broader support operations bottleneck.

- Approximately **500 tickets/week**, handled by **six agents**; reported resolution cycle **8–12 hours**.
- Tickets arrived through **email, chat, documentation comments and forums**, not chat alone.
- Agents searched documentation by title/keywords, sorted tickets mainly by age and escalated without useful prior-attempt context.

**Why it matters:** A better chatbot would not fix the majority of non-chat tickets, urgent-ticket prioritization or repeated investigation. The solution needed to process tickets across channels, retrieve trusted evidence, route safely and preserve context.

---

# Discovery

### 1. The answers largely existed, but were hard to reach
- **71.4%** of 500 sampled tickets were marked answerable from existing documentation.
- Only **43.8%** were resolved on first contact: a **27.6 percentage-point gap**.
- **Design decision:** Semantic knowledge retrieval with grounded, cited responses rather than relying on title search or informal FAQs.

*Evidence: [S1] Sec. 4 Row 1*

### 2. Escalation needed to be smarter, not merely less frequent
- **56%** of tickets were escalated; Tier Two lacked summaries of what had already been tried.
- **146/500** tickets were high urgency, while the manual queue was primarily age-based.
- **Design decision:** Classify intent and urgency; route by explicit rules and confidence; attach supporting context and reasons to human hand-offs.

*Evidence: [S1] Sec. 4 Row 5*

### 3. A chat-only solution would miss most demand
- Of 500 tickets: **212 email**, **155 chat**, **78 documentation comments**, **55 forum**.
- **69%** originated outside chat; the data spanned **22 categories**.
- **Design decision:** Normalize all channels into one ticket-processing workflow and evaluate across the observed category mix.

*Evidence: [S1] Sec. 6 Row 3*

---

# The system

**1. Receive and standardize**  
Convert email, chat, forum and documentation-comment requests into a common ticket format.

**2. Understand and find evidence**  
A Random Forest classifier predicts intent and urgency. Semantic search retrieves relevant passages from the knowledge base stored in ChromaDB.

**3. Decide whether to answer**  
A rule-based router considers classification confidence and document relevance. Supported cases proceed; uncertain or unmatched cases escalate to human support.

**4. Generate and validate**  
An LLM drafts a concise response from retrieved passages with document citations. Guardrails check the output before release; failed checks should lead to abstention or escalation.

**5. Leave an audit trail**  
Decision logging records routing and supporting evidence. Prometheus and Grafana expose processing and operational metrics.

**Implementation:** Sequential agents orchestrated with Google ADK; ChromaDB retrieval with a configured **0.5 similarity cutoff**; OpenRouter LLM integration; Dockerized services. The report also describes a generic-answer fallback if the LLM fails, which requires additional safety validation before production use.

---

# Demo

**Live Demo**

---

# What the numbers say

### Business outcomes | Offline validation, 80 tickets

- **57/80 auto-answered (71.25%)**, above the **65%** project target by **6.25 percentage points**.
- **23/80 escalated (28.75%)**, below the **35%** target.
- Discovery baseline first-contact resolution: **43.8%**; offline auto-answer rate is **27.45 percentage points higher**, but these are **not equivalent proof of live customer resolution**.
- **Customer satisfaction target: 4.2/5.** Not measured; as it can only be measured in live environment from customer feedback.

### Technical performance

- **80/80** validation tickets processed in **98.6 seconds** wall-clock time.
- **Intent accuracy 99%** and **macro F1 0.95**; **retrieval hit rate 86.79%**.
- **Routing accuracy 71.25%**: classification performance does not establish routing correctness.

### Governance evidence

- **80/80** tickets had reconciled decision logs; **zero recorded guardrail blocks** and **zero private-data detections**.
- Zero observed blocks is **not** evidence that guardrails stop adversarial cases; dedicated attack tests were not supplied.

**Uncertainty:** The evaluation has been performed on a small offline validation sample. Also, no production CSAT scoring is performed or, any customer-confirmed resolution. The system has been built out of the development_tickets.json, the training data and the documentation.json, the knowledge base and if a drift in either the data or the knowledge base becomes stale then the system might fail to produce value to the CloudServe.

---

# Governance and risk

>Bounded autonomy, human accountability

| What could go wrong | Design control | Remaining verification |
|---|---|---|
| Confident but unsupported answer | Grounded generation, citations, confidence-based abstention and escalation | Test whether every cited passage actually supports each claim |
| Sensitive information in a response | Output validation and PII checks; avoid private data in monitoring labels | Adversarial leakage tests and privacy review |
| Malicious instructions hidden in tickets | Prompt explicitly treats ticket text as data, not instructions | Prompt-injection red-team tests |
| Stale or contradictory knowledge | Source references and human escalation when evidence is inadequate | Document ownership, versioning, freshness alerts and contradiction tests |
| Wrong priority or unfair routing | Explicit routing logic, decision logs and subgroup monitoring | Audit urgency recall and outcomes by language, tier and channel |
| Model/service failure | Logged failure handling and human escalation path | Verify generic fallback cannot bypass grounding or guardrails |

**Operating principle:** Automate only when evidence and policy allow it; otherwise make the human hand-off informative and traceable.

**Release recommendation:** Start with a **human-approved shadow pilot**. Reconcile metric definitions, harden guardrails and document freshness controls, then measure real FCR, repeat contacts, customer satisfaction and agent effort before expanding autonomous delivery.

---

# Source notes | Attached evidence

- **[S1]** `Stage_1_Discovery_Workbook.docx` — stakeholder interviews, 500-ticket analysis, workflows and discovery findings.
