# Product Management Portfolio | Uditanshu Singh

Passionate Product Manager specializing in **Agentic AI systems, Technical PRDs, and Data-Driven Product Strategy**.

---

## 👨‍💻 About Me & Product Philosophy

With an academic foundation in Finance (BBA, Netaji Subhas University of Technology) and hands-on experience across operations and growth, I approach product management through two core principles:

1. **System-First Architecture Over Prompt Wrappers:** GenAI products succeed or fail on state management, context budgeting, and structured data contracts. I build agentic prototypes that solve hallucination, latency, and context amnesia at the system level.
2. **Ruthless Unit Economics & Funnel Rigor:** Whether reducing drop-off friction in checkout funnels or optimizing LLM Time-to-First-Token (TTFT), every feature must tie back to measurable business viability and user retention.

---

## 📂 Repository Index

### 🤖 1. Agentic AI Projects
End-to-end technical specifications, state architectures, and functional prototypes.

* **[RecallTutor](./agentic-ai-projects/recall-tutor/PRD.md)**  
  *Deterministic Agentic State & Memory Architecture for High-Stakes Exam Prep.*  
  * **Core Problem:** Standard LLM tutoring sessions suffer from context amnesia and question-type hallucination over multi-day spaced repetition intervals.  
  * **Solution:** Replaced stateless chat with an asynchronous extraction worker (`learnFromExchange`) linked via Supabase, enforcing typed state schemas and micro-prompt injection (<150 tokens).  
 * **Deliverables:** [Full Technical PRD](./agentic-ai-projects/recall-tutor/PRD.md) • [Live Prototype ↗](https://recall-buddy-00.lovable.app)

* **[Conversational Voice AI Agent](./agentic-ai-projects/conversational-voice-ai/SYSTEM_SPEC.md)**  
  *Low-Latency Voice Architecture & Real-Time Orchestration.*  
  * **Core Problem:** Turn-taking lag and conversational latency in spoken AI interfaces break human conversational flow.  
  * **Solution:** Real-time audio pipeline evaluating streaming transcription, interruption detection, and sub-800ms TTFT responses.  
  * **Deliverables:** [System Specification & Latency Evals](./agentic-ai-projects/conversational-voice-ai/SYSTEM_SPEC.md)

---

### 🔍 2. Product Teardowns & Turnaround Strategies
Deep-dives into conversion bottlenecks, trust & safety friction, and strategic product pivots.

* **BlaBlaCar — Fixing the Last-Mile Coordination Gap**  
  *Product Teardown, Trust & Safety UX, and Anonymized VoIP Architecture*  
  * **About the Platform:** A community-based carpooling platform connecting drivers with empty seats to passengers traveling long distances to share travel costs[cite: 15].  
  * **The Problem:** In-app messaging lag at highway speeds backs users into a corner—forcing drivers and passengers to sacrifice their personal phone numbers off-platform just to coordinate crowded roadside pickups[cite: 15].  
  * **The Fix:** Scoped an anonymized in-app VoIP calling MVP and pinpoint location sharing flow, backed by MoSCoW backlog prioritization, persona journey mapping, and verified App Store user complaints[cite: 15].  
 * **Deliverables:** [View Slide Deck (PDF) ↗](./teardowns-and-turnarounds/BlaBlaCar_Product_Teardown_Last_Mile_Coordination.pdf) • [LinkedIn Post ↗](https://lnkd.in/p/gtzZGCg5)
* **[CRED — Deal of the Day Checkout Optimization](./teardowns-and-turnarounds/cred-checkout-optimization/CASE_STUDY.md)**  
  *Funnel Teardown & Multi-Item Cart UX*  
  * **The Problem:** CRED's single-item checkout loop forces shoppers to repeatedly unlock and pay for items one-by-one. Survey data revealed 50% of users abandoned purchasing additional daily deals due to repetitive checkout fatigue.  
  * **The Fix:** Designed a unified "Add to Deal Cart" flow with single-transaction checkout, eliminating repetitive navigation loops and recovering lost basket size.  
  * **Deliverables:** [Read Funnel Teardown](./teardowns-and-turnarounds/cred-checkout-optimization/CASE_STUDY.md)

* **[Google Glass — Audio-First Accessibility Pivot](./teardowns-and-turnarounds/google-glass-turnaround/STRATEGY_MEMO.md)**  
  *Failure Autopsy & Strategic Repositioning Memo*  
  * **The Problem:** Google Glass collapsed under high hardware costs ($1,500), social stigma, and severe privacy backlash over public cameras—lacking a defensible consumer use case.  
  * **The Fix:** Formulated a turnaround strategy stripping out the expensive visual prism to create a discreet, audio-first assistive wearable for visually impaired students, backed by primary user interviews and non-profit/government distribution channels.  
  * **Deliverables:** [Read Strategy Memo](./teardowns-and-turnarounds/google-glass-turnaround/STRATEGY_MEMO.md)put Validation, Schema Enforcement, Context Budgeting, Model Provider Benchmarking (TTFT vs. Cost), Spoken Audio Prompt Guardrails.
* **Technical Product Specs:** Relational Data Modeling (PostgreSQL / Supabase), REST & Real-Time Event Flows, BDD Acceptance Criteria (`Given/When/Then`), MoSCoW Prioritization.
* **Product Strategy & Growth:** Quantitative User Surveys, Funnel Teardowns, User Journey Mapping, Metric Trees ($L_0$ North Star to $L_2$ System Metrics), Market Viability Modeling.
