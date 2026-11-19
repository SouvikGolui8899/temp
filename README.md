# CloudServe AI-Powered Ticket Support System

Discovery, requirements, architecture, implementation, evaluation and governance report

**Prepared by Souvik Golui**  
Capstone period: 26 October–15 November 2026 \| 30 recorded hours  
Version 1.0 \| 16 November 2026

| **Evidence basis** This report synthesizes the completed discovery workbook, PRD, prompt library, sprint plan, revision log and effort log. Where the artefacts do not provide measured results, the report says so rather than inventing them. |
|----|

# Contents and report map

| **Section** | **Purpose** | **Target length** |
|----|----|----|
| 1\. Executive summary | Problem, system, claimed achievement and caveat | 1 page |
| 2\. The problem | Request, evidence and gap | 2–3 pages |
| 3\. Discovery findings | Evidence-led findings | 3–4 pages |
| 4\. Requirements | Trace from findings to identifiers | 2–3 pages |
| 5\. Architecture and design | System, alternatives and rationale | 3–4 pages |
| 6\. Implementation | Build, difficulty and restructuring | 2–3 pages |
| 7\. Evaluation | Method, results, interpretation and limits | 3–4 pages |
| 8\. Governance and risk | Risk, fairness, logging and incidents | 2–3 pages |
| 9\. Requirements revision | Change record and learning | 1–2 pages |
| 10\. Conclusions | Next steps and uncertainty | 1–2 pages |
| Appendices | Evidence register and selected tables | As required |

| **Reading note** The report distinguishes baseline evidence, design targets, completed build claims and unverified outcomes. This is essential because the source artefacts record that the validation gate passed, but do not include the full metric table needed to independently confirm FCR, accuracy, latency, fairness or zero unsupported answers. |
|----|

# 1. Executive summary

> CloudServe did not primarily lack answers. It lacked a reliable, channel-independent way to find, ground and route those answers at support speed.

CloudServe was handling approximately 500 tickets per week with six agents, an 8–12 hour resolution cycle, and a manual workflow that sorted tickets mainly by age. Discovery showed that 71.4% of the sampled tickets were answerable from existing documentation, yet 56% were escalated, first-contact resolution was only 43.8%, and 68% were repeat contacts. The operational gap was therefore retrieval, prioritization, confidence management and escalation context rather than document absence.

The project built an agentic ticket-processing application using Google ADK-style orchestration, Chroma as the vector store, a Random Forest classifier for intent/category/urgency, rule-based routing, an LLM generator, response guardrails, Prometheus metrics and Grafana dashboards. The design supports email, chat, documentation comments and forum tickets rather than treating the problem as a chat-only assistant. Incoming tickets are normalized, classified, semantically matched to knowledge-base content, routed to auto-response or escalation, generated with citations, and validated before release.

| **Outcome area** | **Baseline / need** | **Designed target or completion claim** |
|----|----|----|
| First-contact resolution | 43.8% in sampled data; PRD rounded baseline to 42% | ≥65% |
| Time to first reply / resolution | 8–12 hours | \<2 hours queue wait; \<5 minutes processing |
| Escalation | 56% in discovery; PRD later records 58% | 35% target |
| Customer satisfaction | 3.0 in the data; PRD later records 3.2 | 4.2 / 5 |
| Grounding | Keyword title search and informal agent FAQs | Cited RAG answers; escalate if unsupported |
| Build gate | No autonomous system at baseline | Effort log states full unattended validation run passed |

| **Most important caveat** The source documents do not contain the full evaluation result tables or code-level test outputs. “Gate passed” is a completion claim, not sufficient evidence that the 65% FCR, 0% unsupported-answer, latency, fairness and CSAT targets were achieved. The project should therefore be treated as a validated prototype pending reproducible metric export and production pilot evidence. |
|----|

# 2. The problem

## 2.1 What was asked for

The stated request was a chatbot that could resolve at least 65% of support tickets without escalation. That framing suggested a conversational interface and an answer-generation problem. The discovery evidence showed a broader operations problem: tickets arrived through four channels, severity was not driving queue priority, knowledge search relied on titles and exact words, and escalations reached Tier Two without a summary of what had already been attempted.

## 2.2 What discovery revealed

- Volume and staffing created structural delay: roughly 500 tickets per week were handled by six support agents.

- The sampled dataset contained 22 intent categories, far more than the four broad categories recalled by the Head of Support.

- Email was the largest channel (212 of 500), followed by chat (155), documentation comments (78) and forum (55). A chat-only interface would therefore miss 69% of sampled tickets.

- The principal manual time sink was documentation search, estimated at 3–30 minutes per ticket, followed by answer writing or escalation at 1–10 minutes.

- High-urgency tickets represented 146 of 500 tickets, but the manual queue was described as age-sorted rather than severity-led.

- The system needed to fail safely on security, billing and account scenarios, where a fluent but wrong answer could create financial, access or compliance harm.

## 2.3 Gap between request and need

| **Dimension** | **Initial framing** | **Evidence-led need** |
|----|----|----|
| Interaction model | Chatbot | Channel-independent ticket processor |
| Core capability | Generate an answer | Prioritize, retrieve, route, generate, validate and explain |
| Knowledge access | Keyword/title matching | Semantic retrieval over the knowledge base |
| Failure mode | Generic fallback | Explicit escalation with reason, confidence and retrieved context |
| Operational visibility | Not specified | Decision logs, Prometheus metrics and Grafana dashboards |
| Safety | Implicit | Grounding, citation, PII checks, prompt-injection separation and human hand-off |

| **Problem statement** CloudServe needs an autonomous but bounded ticket-processing service that reduces time to a grounded first response across all support channels, prioritizes urgent work, resolves documented issues without escalation, and hands unsupported or risky cases to human agents with usable context. |
|----|

# 3. Discovery findings

> The findings below interpret the evidence. Detailed inventories are kept in the appendices rather than interrupting the argument with raw tables.

## Finding 1: Existing knowledge is underused

71.4% of tickets were marked answerable from existing documentation, while only 43.8% were resolved at first contact. The 27.6 percentage-point gap indicates that access and confidence, not only content coverage, constrained performance.

## Finding 2: Escalation is both high and information-poor

Discovery calculated a 56% escalation rate. Tier Two reported receiving tickets without a summary of prior attempts, forcing work to restart. Reducing escalation volume without improving escalation quality would move the bottleneck rather than remove it.

## Finding 3: The work is multi-channel

Only 31% of the sample came from chat. Email accounted for 42.4%, documentation comments 15.6%, and forum 11%. The system therefore needs a normalized ticket contract independent of channel.

## Finding 4: Stakeholder memory understated category diversity

The data contained 22 categories. Data export and data residency were the largest at 29 tickets each, while the interview shorthand emphasized authentication, deployment, billing and API. Architecture and evaluation must use the observed taxonomy, not only stakeholder recall.

## Finding 5: Repeat contact is a major symptom

68% of tickets were repeat contacts. This is consistent with slow or incomplete first responses, although the artefacts do not establish causality. Repeat-contact rate should be monitored alongside FCR to prevent superficial closure.

## Finding 6: Language fluency did not explain overall CSAT in the sample

24% of tickets were marked non-fluent, but their average CSAT was reported as 3, the same as the overall average. This does not prove fairness. It simply means this sample did not show a mean-score penalty at the aggregate level.

## Finding 7: Search dominates controllable handling time

Documentation search was estimated at 3–30 minutes, much larger than ticket reading at 1–2 minutes. Semantic retrieval therefore has a stronger operational case than using an LLM merely to rewrite an answer.

## Finding 8: Baselines are inconsistent across artefacts

Discovery reports FCR 43.8%, CSAT 3.0 and escalation 56%, while the PRD success table uses 42%, 3.2 and 58%. These differences should be resolved from one governed metric extract before production claims are made.

## 3.1 Implications for the product

- Build one normalization layer for all four channels.

- Use semantic retrieval and source metadata, not title keyword matching.

- Separate retrieval confidence, classifier confidence and safety checks.

- Treat escalation as an enriched product output, not as failure.

- Measure subgroup outcomes and abstention rates, not only average CSAT.

- Establish a single metric dictionary before comparing results with targets.

# 4. Requirements

The PRD translated discovery into seven functional requirements and seven non-functional requirements. The strongest trace is from documentation coverage and escalation risk to grounded answers, citations and abstention. The weakest trace is the treatment of documentation freshness as “will not”; freshness is outside the application’s direct control, but detecting stale or unversioned content should still be an operational requirement.

| **ID** | **Requirement summary** | **Evidence link** | **Acceptance focus** |
|----|----|----|----|
| FR-01 | Resolve queries supported by existing documents | 71.4% answerable; 43.8% FCR | ≥65% FCR on representative evaluation data |
| FR-02 | Escalate low-confidence tickets | Wrong answers are unacceptable | All below-threshold responses escalated |
| FR-03 | Include relevant citations | Need for explainability and compliance | At least one valid supporting citation |
| FR-04 | Append matching documents to escalations | Tier Two starts from scratch | Relevant snippets, scores and original ticket retained |
| FR-05 | Escalate when no document matches | Coverage gap of 28.6% | No-match route always triggers escalation |
| FR-06 | Do not send unsupported answers | Primary risk register item | 0% confidently unsupported customer replies |
| FR-07 | Documentation freshness is external | Technical writer owns source content | Out of scope in v1; dependency must be visible |

## 4.1 Non-functional controls

| **ID** | **Category** | **Requirement / verification** |
|----|----|----|
| NFR-01 | Latency | Process a ticket in under five minutes; log elapsed time. |
| NFR-02 | Availability | Do not leave an arrived ticket waiting more than two hours; log queue aging. |
| NFR-03 | Accuracy | Achieve at least 65% FCR; calculate monthly. |
| NFR-04 | Privacy | Do not expose customer PII; validate and log safely. |
| NFR-05 | Auditability | Retain relevant decisions without retaining unnecessary PII. |
| NFR-06 | Fairness | Compare outcomes and feedback across language fluency and customer tier. |
| NFR-07 | Cost | Monitor LLM token use and surface limit breaches. |

## 4.2 Requirements quality observations

- FR-01 should say “resolve supported queries” rather than “all queries,” because the system explicitly abstains.

- FR-02 needs a calibrated threshold procedure, not a single unexplained confidence value.

- NFR-06 conflates fairness with hallucination prevention. Fairness needs subgroup outcome measures.

- CSAT is a lagging metric affected by human support and product quality; it should not be attributed solely to the RAG service.

- The requirement set should add document freshness metadata, retrieval hit-rate by category, prompt-injection resistance and incident response time.

# 5. Architecture and design

## 5.1 High-level service architecture

The system uses a small service architecture in which the ADK Web App calls the application service. The application exposes ticket-processing endpoints, writes and queries the vector store, and emits metrics through a Prometheus client. Prometheus Server polls those metrics and Grafana queries Prometheus to display operational dashboards.

| **ADK Web App** | **Application Service** | **Chroma Vector DB** |
|-----------------|-------------------------|----------------------|

*request → normalize / decide / respond → semantic knowledge access*

| **Prometheus Client** | **Prometheus Server** | **Grafana** |
|-----------------------|-----------------------|-------------|

*application metrics → scrape and retain → dashboard and alerting*

## 5.2 Application pipeline

| **Input Parser** | **Ingestion Agent** | **Classifier Agent** | **Retriever Agent** |
|----|----|----|----|

↓

| **Router Agent** | **Generator Agent** | **Validator Agent** | **Response / Escalation** |
|----|----|----|----|

| **Component** | **Primary responsibility** | **Key output** |
|----|----|----|
| Input parser / ingester | Normalize channel-specific tickets into one schema | Ticket ID, channel, subject, body, metadata |
| Random Forest classifier | Predict intent/category and urgency | Labels plus confidence |
| Chroma retriever | Semantic search over document chunks | Document IDs, passages and similarity scores |
| Rule-based router | Choose auto-response or escalation | Decision, threshold reason and evidence |
| LLM generator | Draft short grounded response | Answer with citations |
| Validator / guardrails | Reject unsafe, unsupported or private output | Validated answer or escalation |
| Observability | Expose decision and quality metrics | Prometheus series and Grafana panels |

## 5.3 Alternatives considered

| **Decision** | **Chosen approach** | **Alternative** | **Why this fit the capstone** |
|----|----|----|----|
| Orchestration | Google ADK agent pipeline | Single monolithic function; LangGraph | Agent boundaries mirror traceable stages and simplify per-stage logging. |
| Retrieval | Chroma vector store | Title keyword search; managed vector service | Improves semantic matching with low setup cost for a local prototype. |
| Classification | Random Forest | LLM-only classification; rules | Deterministic, inexpensive and testable on labelled ticket features. |
| Routing | Rules and thresholds | LLM autonomous routing | Safer for controlled abstention and easier to audit. |
| Monitoring | Prometheus + Grafana | Application logs only | Separates metrics collection from visualization and supports operational dashboards. |
| Architecture | Micro-service-oriented components | Large distributed deployment | Logical separation without requiring a high-operations production platform for a 30-hour build. |

## 5.4 Design trade-offs and failure modes

- Similarity score is not answer correctness. A top retrieved document can still be irrelevant or stale.

- Classifier confidence and retrieval confidence are different signals and should not be collapsed without calibration.

- Rule-based routing is transparent but brittle if category distributions change.

- Chroma is appropriate for a prototype, but backup, tenancy, access controls and high availability must be designed for production.

- A generator can cite a source that does not support the specific claim. Citation presence must be checked for entailment, not only formatting.

- Prometheus metric labels must avoid ticket text, customer IDs and other high-cardinality or private values.

# 6. Implementation

The effort log records ten hours for build and revision across 2–6 November 2026. The sequence followed the sprint dependency chain: architecture and Docker setup, then ingestion/classification/retrieval/routing, then generation and guardrails, followed by monitoring, the evaluation harness, unattended validation and fairness audit.

| **Date** | **Delivered** | **Interpretation** |
|----|----|----|
| 2 Nov | Architecture document | Defined service boundaries before coding. |
| 3 Nov | Environment and Docker configuration | Established reproducible runtime. |
| 4 Nov | Ingester, classifier, retriever and router | Completed the decision pipeline core. |
| 5 Nov | LLM answer generator and guardrails | Added customer-facing generation and validation. |
| 6 Nov | Decision monitoring, evaluation harness, Grafana dashboard, unattended validation and fairness audit | Completed the prototype gate; detailed results were not embedded in the artefacts. |

## 6.1 What was difficult

Guardrails and validation took longer than expected. This is consistent with the architecture: generation is a single model call, while validation must handle grounding, privacy, unsupported claims, prohibited commitments, malformed citations and unsafe routing. The effort log also records time spent fine-tuning the ChromaDB setup that ultimately did not matter, indicating that early retrieval tuning was not sufficiently tied to a stable evaluation measure.

## 6.2 What should be restructured

- Replace agent-to-agent free-form payloads with versioned Pydantic event contracts.

- Separate offline evaluation code from runtime request handling, but share the same pipeline functions.

- Create a policy service for routing thresholds, prohibited categories and exception rules.

- Version prompts, embeddings, chunking configuration and knowledge snapshots together.

- Persist a decision record with ticket hash, model/version, retrieval IDs/scores, route reason, validator checks and latency.

- Introduce an explicit “abstain” state distinct from technical failure and human escalation.

- Add unit tests for normalization and rules, retrieval tests for known queries, and end-to-end golden cases for grounded answer or escalation.

## 6.3 Reproducibility baseline

| **Minimum reproducible run** A production-quality submission should pin Python and package versions, build with Docker, snapshot the knowledge base, seed model training, record the train/validation split, export environment variables without secrets, and produce a machine-readable evaluation report on every run. |
|----|

# 7. Evaluation

## 7.1 Intended method

The evaluation harness was planned to run the full pipeline over development and validation datasets, with an unattended hidden-set run acting as a gate. A sound evaluation should score each stage as well as the final business outcome to show where errors enter the pipeline.

| **Layer** | **Metric** | **Recommended calculation / threshold** |
|----|----|----|
| Classifier | Macro F1 and per-class recall | Report across 22 categories; highlight rare-class recall. |
| Urgency | High-urgency recall | Prioritize missed high cases; target should exceed overall accuracy. |
| Retriever | Recall@k, MRR and document coverage | Known supporting article appears in top k; inspect by category. |
| Router | Unsafe auto-response rate and escalation precision | Primary safety metric; unsupported auto-response target = 0 in test set. |
| Generator | Groundedness, citation correctness and usefulness | Claim-level support plus human rubric. |
| System | FCR, escalation, latency and repeat contact | Compare to governed baselines and confidence intervals. |
| Fairness | Outcome gaps by fluency, tier and channel | FCR, escalation, unsupported-answer and latency gaps. |

## 7.2 Results available in the artefacts

| **Evidence** | **What can be stated** | **What cannot be concluded** |
|----|----|----|
| Discovery dataset | 500 tickets; 43.8% FCR; 71.4% documentation-answerable; 56% escalated; average CSAT 3; 68% repeat contacts. | These are baselines, not system results. |
| Effort log | The full unattended validation run and fairness audit were completed; Gate 1 is recorded as passed. | No confusion matrix, FCR result, groundedness rate, latency distribution or subgroup output is available. |
| Sprint check-ins | B-11 and B-14 are marked finished. | Completion does not establish metric quality. |
| PRD | Targets were defined for FCR, reply time, CSAT and escalation. | The report cannot verify target attainment from the supplied documents. |

## 7.3 Interpretation

The available evidence supports a conclusion that the full prototype pipeline was implemented and exercised end-to-end. It does not support a quantitative claim that the prototype achieved 65% FCR, 0% unsupported answers, under-five-minute per-ticket latency, a fair subgroup outcome, or a CSAT increase. CSAT in particular cannot be established without a live or simulated user study, because offline ticket datasets do not contain post-deployment customer reactions.

## 7.4 Limitations

- The validation-set size and class distribution are not included.

- No confidence intervals or statistical significance tests are reported.

- The “0% wrong answers” target is absolute and can only be bounded on a finite test set, not guaranteed in production.

- The fairness audit method and subgroup sample sizes are absent.

- The retrieval corpus contains 29 articles, but freshness, versioning and article coverage by category are not quantified.

- The baseline values conflict slightly between discovery and PRD tables.

- There is no adversarial test set for prompt injection, contradictory sources, malicious ticket text or PII leakage.

## 7.5 Evaluation package required before pilot

- Export one CSV/JSON row per ticket with truth labels, predictions, retrieved document IDs, scores, route, generated answer, validator result and latency.

- Publish aggregate metrics and per-category/per-subgroup slices.

- Manually review all auto-responses plus a stratified sample of escalations.

- Run prompt-injection, no-answer, contradictory-document, stale-document and PII red-team cases.

- Calibrate the route threshold on development data and lock it before evaluation.

- Define a production rollback trigger, for example any confirmed unsupported customer response or a statistically material high-urgency miss increase.

# 8. Governance and risk

## 8.1 Risk register

| **Risk** | **Likelihood / impact** | **Control** | **Owner / evidence** |
|----|----|----|----|
| Unsupported confident answer | Material / high | Grounded generation, entailment validation, abstain and escalate | Support AI owner; decision log and reviewed samples |
| PII leakage | Unknown / high | Input/output PII detection, least-privilege logging and redaction | Security and privacy owner; leakage tests |
| Stale documentation | Ongoing / high | Document version, owner, last-reviewed date and freshness alert | Technical writer; ingestion manifest |
| Prompt injection in ticket text | Material / high | Strict instruction/data delimiters, tool allow-list and adversarial tests | Application owner; security test suite |
| Unequal service by language/tier/channel | Unknown / medium-high | Subgroup metrics, minimum sample sizes and human review | Product owner; fairness report |
| Wrong priority classification | Material / high | High-urgency recall monitoring and conservative escalation | Support operations; confusion matrix |
| Metric or dashboard data exposes customer content | Low-medium / high | No free text in Prometheus labels; pseudonymous IDs | Platform owner; label audit |
| Model or dependency change causes regression | Material / high | Version pinning, evaluation gate and rollback | Engineering owner; release record |

## 8.2 Fairness audit

The discovery workbook identified language fluency and customer tier as relevant segments. A defensible fairness audit should compare FCR, escalation, latency, citation validity and unsupported-answer rate across fluent/non-fluent groups, Standard/Business/Enterprise tiers and all four channels. Equal average CSAT is not enough: differences can be hidden by small groups, high variance or selective escalation.

| **Decision rule** Do not auto-deploy a threshold if any subgroup shows a materially higher unsupported-answer rate, high-urgency miss rate or unexplained latency gap. Investigate sample size, category mix and retrieval coverage before changing the model. |
|----|

## 8.3 Decision logging

| **Field group** | **Minimum fields** |
|----|----|
| Identity | Decision ID, ticket hash, timestamp, channel |
| Versions | Application, classifier, prompt, LLM, embedding model, knowledge snapshot |
| Classification | Predicted category, urgency, probabilities |
| Retrieval | Query, document IDs, chunk IDs, scores, top-k configuration |
| Routing | Auto-response/escalate/abstain, thresholds, rule fired |
| Generation | Response hash, citations, token use |
| Validation | Grounding, PII, prohibited-content and policy check results |
| Operations | Stage latency, error code, human override and final outcome |

## 8.4 Incident procedure

| **Step** | **Action** | **Target** |
|----|----|----|
| 1\. Detect | Alert on validator bypass, confirmed wrong answer, PII exposure or abnormal routing. | Immediate |
| 2\. Contain | Disable auto-response for affected category/model; route to humans. | Within 15 minutes |
| 3\. Preserve | Freeze logs, prompt/version, knowledge snapshot and affected ticket hashes. | Within 30 minutes |
| 4\. Assess | Determine scope, customer impact, root cause and whether notification is required. | Same business day |
| 5\. Correct | Patch rule/prompt/model/data; add regression tests and rerun the gate. | Before re-enable |
| 6\. Learn | Record decision, owner, timeline and preventive action. | Within 2 business days |

# 9. The requirements revision

The revision log labels the PRD as version 2.0 on 9 November 2026, but records zero requirements changed, added or removed and leaves the change, assumption and reflection tables blank. This is a substantive governance gap because the capstone explicitly required at least one evidence-triggered revision.

## 9.1 Revision that the build evidence supports

| **ID** | **Version 1** | **Recommended Version 2** | **Trigger** |
|----|----|----|----|
| FR-02 | Escalate low-confidence tickets. | Escalate when retrieval support, route confidence or validator outcome fails calibrated policy; record the failed condition. | Guardrails and validation took longer than expected; one generic confidence value is insufficient. |
| NFR-06 | Do not answer wrongly; verify with ratings. | Report FCR, escalation, unsupported-answer and latency by fluency, tier and channel; investigate material gaps. | Original text describes safety, not fairness. |
| NEW NFR-08 | Not present. | Every ingested document must have source ID, version, owner and last-reviewed date; stale documents trigger an alert. | Documentation freshness is a critical external dependency. |
| FR-03 | At least one relevant citation. | Every externally verifiable claim must be supported by a valid citation; unsupported claims cause abstention. | Citation presence alone does not prove grounding. |

## 9.2 What this teaches

The build did not invalidate the overall architecture, but it exposed that safety, fairness and evidence traceability need more precise requirements than the original PRD provided. A useful revision is not a cosmetic version increment. It should connect a specific implementation observation to a changed requirement and then to a changed test.

| **Governance action** Update the formal revision log with the revisions above, identify a reviewer other than the author, and attach the test cases that prove each revised requirement. |
|----|

# 10. Conclusions

The project correctly reframed CloudServe’s need from a chatbot into a bounded, observable ticket-processing system. The chosen architecture is coherent for a capstone prototype: deterministic classification and routing constrain an LLM, Chroma improves access to a small knowledge base, and Prometheus/Grafana provide an operational path beyond demo-only behaviour. The design also addresses the most important discovery insight: most tickets appear answerable from existing documents, but the current process cannot reliably reach and use those documents.

## 10.1 Recommended next steps

| **Priority** | **Next step** | **Exit criterion** |
|----|----|----|
| 1 | Export and review the full evaluation result table. | Reproducible metrics for classifier, retrieval, routing, generation, latency and subgroup slices. |
| 2 | Reconcile baseline definitions and values. | One metric dictionary and approved baseline extract. |
| 3 | Revise PRD and revision log based on build evidence. | At least one evidence-triggered requirement change with reviewer approval. |
| 4 | Harden document governance and prompt-injection defenses. | Versioned corpus, freshness alerts and passing adversarial tests. |
| 5 | Run a shadow pilot with human approval. | Two to four weeks; no direct customer auto-send; reviewed error taxonomy. |
| 6 | Introduce bounded auto-response by category. | Only categories meeting safety, support and volume thresholds are enabled. |
| 7 | Measure production impact. | FCR, repeat contact, latency, escalation quality, CSAT and agent effort monitored together. |

## 10.2 What remains uncertain

- Whether the validation gate actually met the numeric PRD targets.

- How the system behaves on new categories and data drift.

- The reliability of citations at claim level.

- The effect on repeat contacts, agent workload and CSAT in live operations.

- Whether non-fluent users or specific tiers/channels experience higher abstention or worse retrieval.

- The operational cost and latency of the chosen LLM at CloudServe’s real weekly volume.

| **Final judgement** The prototype is a credible foundation for a controlled pilot, not yet evidence for unattended production deployment. Its highest-value next deliverable is not another feature. It is a complete, reproducible evaluation and governance package. |
|----|

# Appendices

## Appendix A. Evidence register

| **Artefact** | **Role in report** | **Material completeness issue** |
|----|----|----|
| Stage_1_Discovery_Workbook.docx | Stakeholder evidence, ticket statistics, workflow, success criteria and initial risks | Some risk probabilities are expressed as data coverage and need correction. |
| Stage_2_PRD_Template.docx | Functional/non-functional requirements, targets and assumptions | Baseline values differ from discovery; open questions are blank. |
| Stage_3_Prompt_Library.docx | Specifications and two runtime prompts | Traceability table covers only FR-06; prompt register is incomplete. |
| Stage_4_Sprint_Plan.docx | Capacity, backlog, dependencies and check-ins | Definitions of done are sometimes too weak to verify quality. |
| Stage_5_PRD_Revision_Log.docx | Mandatory requirement-change record | Version increment recorded but no revisions or reflections entered. |
| Effort_Log.docx | Hours, implementation chronology and completion claims | No detailed evaluation metric export is attached. |

## Appendix B. Selected discovery statistics

| **Measure** | **Value** |
|----|----|
| Sample size | 500 |
| Channel split | Email 212; chat 155; docs comment 78; forum 55 |
| Urgency split | Medium 226; high 146; low 128 |
| First-contact resolution | 43.8% |
| Answerable from documentation | 71.4% |
| Escalation | 56% in discovery |
| Average CSAT | 3.0 |
| Non-fluent tickets | 24% |
| Repeat contacts | 68% |
| Knowledge-base articles | 29 |

## Appendix C. Category counts

| **Intent category**     | **Tickets** |
|-------------------------|-------------|
| data_export             | 29          |
| data_residency          | 29          |
| rollback_request        | 28          |
| deployment_failure      | 27          |
| compliance_request      | 26          |
| sso_configuration       | 26          |
| security_incident       | 26          |
| database_issue          | 26          |
| api_usage_question      | 24          |
| billing_query           | 24          |
| performance_degradation | 23          |
| quota_or_overage        | 23          |
| account_access          | 22          |
| onboarding              | 22          |
| integration_help        | 21          |
| webhook_issue           | 21          |
| feature_request         | 20          |
| authentication_failure  | 20          |
| api_key_issue           | 18          |
| configuration_help      | 17          |
| unclear_request         | 15          |
| rate_limit              | 13          |

## Appendix D. Requirement-to-component traceability

| **Requirement** | **Primary component** | **Verification evidence required** |
|----|----|----|
| FR-01 | Retriever + Generator + Router | Per-ticket answer/escalation and FCR calculation |
| FR-02 | Router + Validator | Threshold calibration and unsafe auto-response count |
| FR-03 | Generator + Validator | Citation validity and claim-support review |
| FR-04 | Retriever + Escalation payload | Escalated ticket inspection |
| FR-05 | Retriever + Router | No-match golden cases |
| FR-06 | Generator + Validator | Unsupported-answer adversarial suite |
| NFR-01/02 | Application + metrics | p50/p95 processing and queue latency |
| NFR-04/05 | Validator + decision log | PII tests and audit record review |
| NFR-06 | Evaluation harness | Subgroup metric report |
| NFR-07 | LLM client + Prometheus | Token/cost dashboard |

## Appendix E. Prompt register extract

PR-01 instructs the model to answer only from provided CloudServe passages, avoid guessing, avoid promises about refunds or fix dates, and treat instructions embedded in ticket text as data. PR-02 supplies channel, ticket description and retrieved context and requests a short cited reply. These prompts support grounding and injection resistance, but the register should explicitly map both prompts to FR-01, FR-02, FR-03 and FR-06 and define structured output for automated validation.

## Appendix F. Source and visual note

Internal source documents: Stage_1_Discovery_Workbook.docx; Stage_2_PRD_Template.docx; Stage_3_Prompt_Library.docx; Stage_4_Sprint_Plan.docx; Stage_5_PRD_Revision_Log.docx; Effort_Log.docx. The cover visual in this file was generated with Microsoft Copilot. Architecture diagrams in the report are native document tables derived from the supplied reference architecture.
