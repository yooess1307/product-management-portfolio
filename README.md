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

* **VocabCoach — Voice-First Vocabulary Tutor**  
  *Spoken English Practice & Conversational Prompt Design*  
  * **The Problem:** Most AI voice tutors sound like someone reading a Wikipedia article out loud. They drone on with long definitions and generate formatting (bullet points, asterisks) that trips up speech engines—leaving the user listening passively instead of practicing.  
  * **The Fix:** Built an interactive voice coach on ElevenLabs designed for natural spoken dialogue rather than text chat. It teaches one word at a time, caps explanations at two sentences using a relatable analogy, strips out markdown so the speech engine never stumbles, and immediately asks the user to use the word in an original sentence out loud before giving instant, constructive feedback.  
  * **Live Demo:** [Talk to VocabCoach ↗](https://elevenlabs.io/app/talk-to?agent_id=agent_9001m32hq75gf9nr6pd2vzgkvwv6&branch_id=agtbrch_3601m32hq8btftgr173zyq1rarfn)

  <details>
  <summary><b>View System Prompt & Voice Rules</b></summary>

  ```text
  You are VocabCoach, an encouraging, interactive, voice-first vocabulary tutor. Your goal is to help users learn and actively speak new high-impact words.
  Guidelines for conversation:
  1. One Word at a Time: Teach one word per round (e.g., Pragmatic, Ephemeral, Resilient, Ubiquitous, Diligent).
  2. Voice-Optimized: Keep all spoken explanations concise (maximum 2 short sentences). Never recite long dictionary definitions.
  3. Active Recall Loop:
     - Introduce the word and pronounce it clearly.
     - Explain its meaning using a simple, relatable real-world analogy.
     - Prompt the user to use the word in a sentence out loud.
  4. Evaluation: When the user speaks their sentence:
     - If used correctly, praise them specifically on why it worked, then ask if they want the next word or a harder challenge.
     - If used incorrectly or unnaturally, gently explain the nuance, provide a corrected variation, and encourage them to try again.
  5. Tone: Warm, energetic, concise, and conversational. Avoid bullet points, symbols, or asterisks since this is spoken audio.
  ```
* **[HireMate — AI-Powered Mock Interview Platform Launch Strategy](./HireMate_AI.pdf.pdf)**  
  *0-to-1 Go-to-Market Sequencing, Freemium Monetization, and Organic Growth Loops*  
  * **What It Is:** An AI interview copilot that provides role-specific mock interviews (tailored for roles like Product Managers and Software Engineers), structured multi-metric feedback reports (evaluating pace, filler words, and content quality), and industry benchmark scores against peer candidates.  
  * **The Problem It Solves:** Job seekers deal with intense preparation anxiety and lack realistic, structured ways to practice. Human coaching is expensive and difficult to schedule, peer-to-peer mocks require coordinated availability, and generic online guides fail to give candidates objective, data-backed feedback on where their performance actually stands.  
  * **The GTM Strategy:** A lean, 3-stage distribution roadmap designed to scale from 0 to 10,000 users in 6 months on a near-zero initial CAC.
  * **Campaign Assets:** [Launch Poster 1 ↗](./HireMate_Launch_Poster_1.jpg) • [Launch Poster 2 ↗](./HireMate_Launch_Poster_2.jpg)
  
---

### 🔍 2. Product Teardowns and Turnarounds
Deep-dives into conversion bottlenecks, trust & safety friction, and strategic product pivots.

* **[BlaBlaCar — Fixing the Last-Mile Coordination Gap](./BlaBlaCar_Product_Teardown_Last_Mile_Coordination.pdf.pdf)**  
  *Product Teardown, Trust & Safety UX, and Anonymized VoIP Architecture*  
  * **About the Platform:** A community-based carpooling platform connecting drivers with empty seats to passengers looking for a ride to share travel costs.  
  * **The Problem:** In-app messaging lag at highway speeds backs users into a corner—forcing drivers and passengers to sacrifice their personal phone numbers off-platform just to coordinate crowded roadside pickups.  
  * **The Fix:** Scoped an anonymized in-app VoIP calling MVP and pinpoint location sharing flow, backed by MoSCoW backlog prioritization, persona journey mapping, and verified App Store user complaints.  
  * **Public Breakdown:** [View Discussion on LinkedIn ↗](https://lnkd.in/p/gtzZGCg5)
   
* **[CRED — Deal of the Day Checkout Optimization](./CRED_Checkout_Optimization.pdf.pdf)**  
  *Funnel Teardown, Quantitative User Research, and Multi-Item Cart UX*  
  * **About the Platform:** A high-trust credit card payment and rewards platform in India that offers curated e-commerce drops and daily flash promotions via the CRED Store.  
  * **The Problem Statement:** CRED users find it tedious to navigate back to the 'Deal of the Day' section repeatedly for unlocking prices and checking out items one by one. Because unlocking immediately triggers a single-item payment bottom sheet (`Pay instantly`), purchasing multiple deals forces users through redundant payment authorizations and address confirmations.  
  * **The Friction Proof:** Primary survey data revealed 50% of users abandoned purchasing additional daily deals specifically due to repetitive transaction fatigue, with 50% noting repeated detail entry actively deterred them from buying more.  
  * **The Fix:** Replaced the immediate buy-now lock-in with an "Add to Deal Cart" aggregation drawer and unified single-swipe checkout, eliminating redundant navigation loops and recovering lost basket size.  
  * **Validation & Impact:** 72.7% of surveyed shoppers confirmed a unified checkout directly improved their purchase experience, with 66.7% expressing high willingness to purchase multiple daily deals in a single transaction.
