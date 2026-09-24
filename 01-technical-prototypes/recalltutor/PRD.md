 # Product Requirement Document (PRD): RecallTutor
**Product:** RecallTutor  

**System:** Deterministic Agentic State & Granular Question-Type Memory Architecture  

**Author:** Uditanshu Singh 

**Status:** Functional Prototype (In Progress / Active Testing)

---

## 1. Problem Statement & Genesis

### 1.1 The Genesis
While preparing for the GMAT Focus Edition (Quantitative & Data Insights sections), studying high-variance topics like Conditional Probability and Bayes' Theorem revealed a critical flaw in standard LLM tutoring sessions: context amnesia and semantic retrieval failure.

### 1.2 The Core Breakdown: Naive Embeddings vs. Session State
Standard LLM memory architectures rely on vector embeddings or conversational history dumps. When a user returns after a 7-day spaced repetition interval:

1. **Granular State Loss:** Vector search matches high-level semantic keywords, but drops granular session state. The model remembers *that* a broad topic was discussed, but forgets the exact question subtypes practiced (e.g., remembering you practiced "Speed, Time & Distance," but forgetting whether you specifically struggled with "two trains traveling in opposite directions" versus "boats and streams").
2. **Hallucination of Practice History:** Because it lacks discrete state tracking, the LLM hallucinates question variants that were never introduced in the prior chat, disrupting targeted revision.
3. **Loss of Pedagogical Calibration:** The model forgets the learner's specific misconception patterns and preferred explanation frameworks (e.g., forgetting that you understand problems faster using concrete numbers like "assume a group of 100 people" rather than abstract mathematical formulas).
4. **Cognitive Re-Prompting Tax:** The learner is forced to burn cognitive bandwidth during the first 5–10 minutes of a session acting as an "instruction manager"—correcting the model's assumptions and reconstructing what was covered a week prior.

### 1.3 The Solution
RecallTutor replaces stateless chat and naive RAG with a deterministic state extraction engine. At the end of each study session, a state extraction prompt isolates the exact question subtype practiced, specific error traps, and pedagogical preferences into a structured JSON schema.

When the learner returns for revision, the runtime engine injects a compact state block into the system prompt. This eliminates question-type hallucination and gives the learner an immediate, targeted drill within a strict token budget.

---

## 2. Target Persona & High-Stakes Constraints

### 2.1 Target Persona
* **Profile:** High-Stakes Exam Aspirant (GMAT Focus Edition: Quantitative & Data Insights).
* **Behavior:** Highly analytical, time-constrained learner using spaced repetition across multi-day intervals.
* **Core Pain Point:** In GMAT Focus prep, missing a question is rarely about the broad topic (e.g., "Probability"); it happens at the granular sub-mechanics level (e.g., falling for boundary constraints or base-rate fallacies in Data Sufficiency). Standard LLMs forget these specific traps, making them unreliable for serious revision.

### 2.2 Core User Stories
* **US-1 (Targeted Spaced Revision):** *As a student returning after 7 days, I want the tutor to test me on the exact trap I failed last week (e.g., Data Sufficiency base-rate traps) without me having to remind it.*
* **US-2 (Zero Re-Prompting Friction):** *As a time-constrained learner, I want to jump straight into problem-solving without spending the first 5 minutes of a chat acting as an "instruction manager."*
* **US-3 (Explanation Style Retention):** *As a learner who grasps concepts visually, I want the tutor to default to my preferred explanation framework (e.g., 2x2 contingency tables with concrete numbers) without needing to request it every session.*

---

## 3. Product Goals & Non-Goals

### 3.1 Product Goals
* **Zero Question-Type Hallucination:** Ensure deterministic recall of specific problem sub-types and past error patterns across multi-day revision intervals.
* **Preserve Pedagogical Modality:** Retain how the learner prefers concepts broken down (e.g., tabular 2x2 grids and concrete populations over abstract algebraic formulas) without requiring manual reminders.
* **Token & Cost Efficiency:** Pass memory as a compact summary under 200 tokens, keeping API costs low and response times fast instead of resending long chat logs.
* **UI Stability:** Ensure the app catches and handles malformed AI outputs gracefully so bad data never crashes the user interface.

### 3.2 Non-Goals (Scope Boundaries for MVP)
* **Full Conversation Transcript Archiving:** The system will not store, embed, or re-inject thousands of raw conversational tokens; it persists only structured state tags.
* **Subjective Essay & Long-Form Writing Evaluation:** While the underlying Gemini model can answer queries across any academic subject, RecallTutor’s state and memory tracking are intentionally optimized for question-and-answer problem-solving (math, data insights, logic), not grading subjective essays.

### 3.3 Autonomy Envelope & Human-in-the-Loop (HITL) Boundaries
* **Autonomous Actions:** Generating chat titles, running background exchange extractions (`learnFromExchange`), and pruning memory FIFO tokens.
* **Human-in-the-Loop (HITL) Actions:** Advancing problem difficulty, validating solution accuracy checkpoints, and marking topics as mastered (the agent cannot skip checkpoints without user confirmation).

---


## 4. System Architecture & Information Flow

### 4.1 System Layout & Component Topology
```text
┌─────────────────────────── RecallTutor Client (Mobile-First Web App) ───────────────────────────┐
│                                                                                                  │
│  ┌──────────────────────┐  ┌────────────────────────────────────┐  ┌──────────────────────────┐  │
│  │ Left Nav Drawer      │  │ Main Interaction View              │  │ Right Memory Sidebar     │  │
│  ├──────────────────────┤  ├────────────────────────────────────┤  ├──────────────────────────┤  │
│  │ • [+ New Chat]       │  │ • Top Header (Avatar & Profile)    │  │ "What I Remember"        │  │
│  │ • Recents List       │  │ • Doc Upload (Syllabus/Notes)      │  │                          │  │
│  │   (Tab-based routing │  │ • Conversational Chat Stream       │  │ 1. Weak Spots            │  │
│  │    without remounts) │  │ • Input Bar                        │  │ 2. Mastered Topics       │  │
│  │                      │  │                                    │  │ 3. Preferred Explanation │  │
│  └──────────┬───────────┘  └─────────────────┬──────────────────┘  └────────────▲─────────────┘  │
└─────────────┼────────────────────────────────┼──────────────────────────────────┼─────────────┘
              │                                │                                  │
              │ (First Turn: Auto-Title)       │ (User Message)                   │ (Reactive Query:
              ▼                                ▼                                  │  conversation_id =
     [ Utility LLM Prompt ]          [ Runtime LLM Engine ]                       │  activeConversationId)
       (3–5 Word Summary)                      │                                  │
              │                                │ (Turn Completion)                │
              │                                ▼                                  │
              │                      [ learnFromExchange ]                        │
              │                      (Background Extractor)                       │
              │                                │                                  │
              │                                │ (Silent Fallback on Parse Error) │
              ▼                                ▼                                  │
      ┌────────────────────────────────────────────────────────────────────────┐  │
      │                     Supabase Backend (PostgreSQL)                      │  │
      │  ┌──────────────────────────────┐    ┌──────────────────────────────┐  │  │
      │  │     conversations Table      │◄───┤    student_memories Table    ├───┴──┘
      │  │ (id, user_id, title, dates)  │ FK │ (id, conversation_id, topic, │   
      │  │                              │    │  category, notes)            │   
      │  └──────────────────────────────┘    └──────────────────────────────┘   
      └─────────────────────────────────────────────────────────────────────────┘
```

### 4.2 Step-by-Step Data Pipelines

#### 4.2.1. Starting a Session and Naming the Chat
* **Clean State Reset:** When the student clicks `+ New chat`, the screen resets completely. The chat window clears, and the memory sidebar empties out so there is zero leftover data from past topics.
* **Dynamic Auto-Titling:** Once the student sends their very first message, a quick background prompt reads that message, summarizes it into a 3 to 5-word title (e.g., *"Bayes' Theorem & Traps"*), and applies that title to the left sidebar—all without the page reloading or blinking.

#### 4.2.2. The Live Tutoring Exchange
* **Active Dialogue:** The student chats with the tutor, solving GMAT quantitative or logic questions, or uploading notes and mistake logs.
* **Scoped Context:** The tutor focuses strictly on this active chat so responses stay fast, without dragging in unrelated conversation history from other days.

#### 4.2.3. Extracting Memory in the Background
* After an exchange wraps up, a background prompt reviews the dialogue to isolate three key signals:
  * **Weak Spots:** The exact calculation trap or misconception the student fell into.
  * **Mastered Topics:** What the student solved cleanly and understood.
  * **Preferred Explanations:** The format that made the concept click (e.g., using a 2x2 grid with 100,000 people instead of a dry formula).
* **Crash Prevention:** If the AI outputs broken text or formatting errors instead of clean data, the app silently drops the payload (`error: null`). This prevents the user's screen from freezing, throwing red error banners, or crashing.

#### 4.2.4. Keeping Chats Isolated (No Memory Bleed)
* **Thread Scoping:** The right sidebar ("What I Remember About You") only displays the memories attached to the specific chat currently active.
* **Context Integrity:** When switching from a "Bayes' Theorem" chat to a "Time & Distance" chat, the sidebar updates instantly to show only "Time & Distance" notes. Concepts never bleed into each other across sessions.

### 4.3 The Storage Logic (How Supabase Handles It)
In relational terms, the data architecture relies on two connected tables in Supabase:
* **The Conversations List (`conversations`):** Stores the chat ID, user ID, chat title, and timestamp created.
* **The Memories List (`student_memories`):** Stores the individual atomic notes (`weakness`, `mastery`, or `preference`), linked via a foreign key (`conversation_id`) to the specific conversation it belongs to.

Because each note is explicitly tagged with that `conversation_id`, clicking a chat in the sidebar only pulls up records belonging to that unique thread. 

---

## 5. Detailed Functional Requirements (FR)

### 5.1 FR-1 Auto-Titling on First Turn
When a student sends their first prompt in a new session, the app creates the conversation row in Supabase under a temporary title, then immediately runs an async utility prompt in the background. That worker condenses the student's question into a clean 3 to 5 word label and writes it back to `conversations.title`. The left sidebar listens to this state and updates the tab text in real time, avoiding full-page reloads, tab flicker, or DOM unmounting.

### 5.2 FR-2 Socratic Tutoring Dialogue
When a student asks for help on a topic or problem, the tutor guides them step by step rather than dumping direct answers. It breaks down quant and logic problems into intermediate checkpoints, waiting for the student to confirm before advancing. Responses stream directly into the active chat viewport using Server-Sent Events to keep perceived latency low.

### 5.3 FR-3 Background Memory Extraction (`learnFromExchange`)
The moment the tutor finishes streaming an answer, an async background agent inspects the exchange and writes new takeaways to `student_memories`.
It categorizes signals into three buckets:
* `weakness` captures calculation traps, missed constraints, or flawed logic
* `mastery` captures rules or question variants solved correctly without help
* `preference` captures learning formats like 2x2 grids or testing with 100 instead of variables
Each entry stores the topic along with specific notes, tied to the active conversation ID. If the model outputs broken JSON or misses schema constraints, the handler swallows the exception with null error handling so the UI never freezes or throws crash toasts.

### 5.4 FR-4 Thread Management and Clean State Reset
Clicking "+ New chat" clears the conversation window, drops the active conversation pointer, and resets the memory sidebar back to zero. Past threads in the Recents list render as interactive button tabs rather than standard hyperlink tags. Clicking between them updates client state on the fly without remounting the sidebar or flashing empty state placeholders.

### 5.5 FR-5 Multi-Day Spaced Context Recall
When a student reopens an old session days later, the app fetches that thread's records from `student_memories` and restores their weak spots, mastered rules, and preferred modalities in the right sidebar.
When they send a new message in that thread, the engine bundles these active memory tags into the tutor's system prompt under a strict 200-token cap. The tutor immediately respects past pitfalls and teaching styles without forcing the student to re-brief the model.

---

## 6. Non-Functional Requirements (NFRs) & System Benchmarks

### 6.1 Latency and Response Performance
The tutoring chat must feel instantaneous and natural. Time to first token for streaming responses should land under 1.2 seconds on standard broadband connections. The background memory extraction agent runs completely asynchronously after a turn wraps up. It should finish parsing and committing tags within 3 seconds, without ever delaying the student from typing or sending their next prompt.

### 6.2 Token Economics and Rate Limits
To avoid hitting API rate limits and keep response times crisp, runtime memory injection is capped strictly at 200 input tokens. The system passes only distilled tags rather than raw chat history, keeping recurring inference lean and focused. If memory tags accumulate beyond the budget, the compiler applies a simple FIFO trim that keeps the most recent notes.

### 6.3 Reliability and Silent Error Handling
Third-party models occasionally drop invalid characters or violate JSON structures. The extraction pipeline runs inside an isolated try-catch block. If parsing fails, the system returns a null error object, logs the incident quietly in the background, and drops the write. Under no circumstances should a background parsing issue trigger an error modal, red toast, or input freeze on the student's screen.

### 6.4 Data Privacy and Thread Sandboxing
All database reads and writes must strictly enforce user and session boundaries. A conversation thread can only query records where both the user ID and conversation ID match the active session. Switching chats immediately severs previous database subscriptions and clears local memory caches, ensuring zero cross-topic data leakage between different study subjects.

### 6.5 Out-of-Domain Guardrails & Offline Eval Suite
* **Out-of-Domain (OOD) Steering:** If a learner prompts the tutor for non-GMAT tasks (such as general software development, creative writing, or trivia), the system prompt steers them back to Quantitative and Data Insights prep.
* **Offline Eval Benchmark Suite:** Evaluates `learnFromExchange` extraction accuracy across a golden test set of 40+ prompt scenarios to verify schema fidelity, category classification, and prevent regressions before production deployment.

---

## 7. Agent State Architecture & Data Contracts

RecallTutor uses a deterministic state model to maintain continuity across sessions without loading unstructured chat transcripts into the LLM context.

### 7.1 Thread Session State
Every study session is treated as an isolated conversational thread.
* **Session Identifier:** A unique system ID generated when a new chat starts.
* **Student Attribution:** Associates the session with the authenticated user profile.
* **Semantic Title:** A concise 3 to 5 word label generated asynchronously from the user's initial prompt, replacing the default "New chat" tag.
* **Session Lifecycle:** Tracks creation and last active timestamps to power the Recents navigation list.

### 7.2 Discrete Memory Entity & Data Contract
Memory in RecallTutor is atomic and structured. Rather than relying on fuzzy vector embeddings, takeaways are persisted as discrete entities linked directly to the parent session. The background extraction worker enforces this exact JSON schema contract before persisting records to storage:

```json
{
  "category": "weakness | mastery | preference",
  "topic": "string",
  "notes": "string (max 120 chars)"
}
```
* **Thread Scope:** Tied directly to the active session ID, ensuring zero cross-subject memory bleed.
* **Taxonomy Bucket:** Restricted strictly to weakness, mastery, or preference.
* **Topic Anchor:** The specific problem type or sub-mechanic practiced (such as Bayes' Theorem or Distance and Work-Rate).
* **Pedagogical Note:** Concrete, high-signal summary of the student's trap, verified solution, or favored learning modality.
* **Captured Timestamp:** Enables chronological sorting so the agent prioritizes the freshest feedback.

### 7.3 State Lifecycle and Memory Sandboxing
* **Thread Sandboxing:** The agent only queries memory rows tied to the active session ID. Switching chats unbinds previous context immediately, ensuring clean isolation between topics.
* **Cascading Cleanup:** Deleting a study session from history automatically purges all attached memory tags, leaving no orphaned student data in storage.

---

## 8. Product Health & Success Metrics

We measure RecallTutor on three practical levels: are students sticking to their spaced revision habits (L0), is the tutor accurately remembering their past mistakes and teaching preferences (L1), and is the app responding quickly without breaking (L2)?

### 8.1 L0: Study Habits & Retention
* **7-Day Return Rate:** Do students return after a week to review concepts they previously struggled with?
  * *Target:* At least 40% of active learners return for a spaced practice session after 7 days.
* **Drill Completion Rate:** Do students work through a problem checkpoint until they reach the final answer, rather than abandoning the chat midway?
  * *Target:* At least 85% of started practice sessions are completed.

### 8.2 L1: Memory & Tutoring Quality
* **Zero-Reprompt Sessions:** Can a returning student jump straight into practice without spending the first five minutes reminding the AI what they studied or correcting its assumptions?
  * *Target:* At least 90% of returning sessions require zero briefing or setup from the student.
* **Question-Type Recall Accuracy:** When reviewing a past weak spot, does the tutor test the exact problem sub-type they struggled with (such as base-rate traps in Data Sufficiency) without inventing random, irrelevant variants?
  * *Target:* 100% accurate recall with zero hallucinated question types.
* **Explanation Style Match:** Does the tutor stick to the student's favored explanation format (such as 2x2 grids with concrete numbers instead of abstract formulas) without needing a reminder?
  * *Target:* At least 95% of explanations default to the saved preference.

### 8.3 L2: Speed & System Health
* **Memory Save Reliability:** Does the background worker parse and store takeaways into the database cleanly without dropping notes?
  * *Target:* At least 99% of memory extractions save successfully.
* **Response Speed:** How fast the tutor starts streaming its response so the back-and-forth feels natural.
  * *Target:* Under 1.5 seconds to start answering.
* **Memory Token Footprint:** Keeping saved notes short and sweet so the tutor stays focused, responses remain fast, and API costs stay low.
  * *Target:* Average under 150 tokens (strictly capped at 200 tokens).

---

## 9. Product Roadmap

The roadmap focuses on what solves our immediate study friction today (Now), what makes daily practice smoother next (Next), and bigger vision ideas down the line (Later).

### 9.1 Now: Current MVP
* **Step-by-Step Problem Solving:** The tutor walks through math and logic problems checkpoint by checkpoint, waiting for the student to confirm each step rather than dumping the full solution upfront.
* **Automatic Note-Taking in the Background:** After an exchange, a background worker quietly picks up calculation traps, mastered rules, and preferred explanation styles, saving them directly to the database.
* **Zero Topic Bleed:** Separate chat threads stay completely isolated so notes from a Bayes' Theorem session never spill into a Speed, Time & Distance chat.
* **Fast, Lightweight Memory Injection:** Re-injects saved takeaways in under 200 tokens when returning to a thread, keeping responses snappy and API costs low.
* **Instant Thread Titling:** Automatically renames new chats into clean 3-to-5-word titles after the first message without page reloads or screen flicker.

### 9.2 Next: Planned for v1.1
* **Uploading Notes & Mistake Logs (`Doc Upload`):** Letting students drop in screenshots of mock test errors or syllabus notes so the tutor knows what traps to target before the chat even starts.
* **Target Score & Exam Profile (`Avatar & Profile`):** A simple settings modal to track target test dates, baseline section scores, and default explanation preferences.
* **Click-to-Edit Memory Sidebar:** Giving students full control to click, edit, or delete any note in the "What I Remember About You" sidebar if their habits change or a note is outdated.
* **Dropped Connection Handling:** A clean retry button if the streaming response drops mid-explanation so the student never loses their place.

### 9.3 Later: Future Horizons (v2.0)
* **Master Revision Dashboard:** A single view that pulls together recurring weak spots across every past chat into one targeted practice drill.
* **Spaced Review Nudges:** Optional reminders when a topic where you made repeated errors hits the 7-day or 14-day spaced revision mark.
* **Support for Other Exams:** Expanding the question taxonomy beyond GMAT Quant and Data Insights to tests like GRE or CAT.
