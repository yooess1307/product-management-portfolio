# Product Requirements Document (PRD): RecallTutor

**Product:** RecallTutor — Context-Aware Agentic AI Study Companion  
**Document Version:** 2.0 (Production Release / Full Architectural Specification)  
**Author / Product Architect:** Uditanshu Singh  
**Prototyping & Deployment Stack:** Lovable (Full-Stack React + Vite Deployment), Supabase (PostgreSQL, Auth, Edge Functions), Gemini / OpenAI API  
**Live Application:** [https://recall-buddy-00.lovable.app](https://recall-buddy-00.lovable.app)  
**Portfolio Hub:** [github.com/yooess1307/product-management-portfolio](https://github.com/yooess1307/product-management-portfolio)

---

## 1. Executive Summary & Problem Statement

### 1.1 The Market Problem
Conversational AI interfaces applied to education suffer from four structural failure modes:
1. **Session Amnesia:** Generic LLM interfaces treat every conversational turn as an isolated event. Learners practicing for high-stakes quantitative and technical examinations (e.g., GMAT, GRE, technical certifications) waste cognitive bandwidth repeatedly explaining their conceptual bottlenecks, arithmetic pitfalls, and preferred explanation formats.
2. **Pedagogical Hallucination ("False Mastery"):** Standard conversational models mistake assisted problem-solving for independent student competency. When a tutor provides a formula or step-by-step hint and the student calculates the final line, naive AI tutors log the concept as "learned," distorting downstream revision schedules.
3. **Conversational Drift & Lack of Guardrails:** Open-ended chatbots passively follow user tangents. When learners fatigue and drift into non-academic chatter, traditional bots indulge the distraction rather than enforcing study accountability.
4. **Brittle Pipeline Coupling:** Forcing a single LLM call to stream conversational dialogue while simultaneously extracting structured JSON state records introduces parser exceptions (`AI_NoObjectGeneratedError`), freezing the UI and breaking active sessions.

### 1.2 The Solution
RecallTutor is a context-aware agentic study companion built with a **decoupled dual-pipeline architecture**. It separates low-latency conversational streaming from an asynchronous background cognitive harvester. Featuring **thread-isolated memory banks**, a **3-guardrail pedagogical state machine**, and **integrated active-recall tooling** (interactive flashcards and post-break warm-up drills), RecallTutor transforms passive chatbot Q&A into structured, disciplined revision.

> **Architecture & Attribution Note:** This product specification was architected by defining the functional logic, interaction design, pedagogical state machines, and prompt constraints. Technical database schemas and API bindings were co-scaffolded with Gemini 3.8 during prompt-driven design sprints and deployed via Lovable.

---

## 2. Target Personas & Core User Journey

### 2.1 User Personas
* **The High-Stakes Exam Candidate (Primary):** Preparing for standardized tests (GMAT, GRE, CFA). Needs targeted practice on specific syllabus weak spots and requires the tutor to remember recurring traps across multi-day study sprints.
* **The Structured Self-Learner:** Tackling technical or academic subjects (e.g., Discrete Mathematics, Microeconomics). Requires Socratic questioning rather than direct answers.
* **The Easily Distracted Student:** Prone to browser tab-switching and fatigue-induced tangents. Needs humane accountability checks that prevent drift without hostile nagging.

### 2.2 Core Interaction Journey
```
┌────────────────────────────────────────────────────────────────────────┐
│ 1. SESSION ENTRY: Post-Login Command Center or [+ New Chat] Modal      │
│    • Choose: [💡 Brainstorm] (Unconstrained) vs.                       │
│              [🎯 Dedicated Study Topic] (Locked Syllabus Anchor)       │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│ 2. DUAL-PIPELINE DIALOGUE TURN                                         │
│    • Fast Path: Socratic tutor response streams instantly (TTFT < 800ms)│
│    • Background Path: `learnFromExchange` extracts cognitive state     │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│ 3. PEDAGOGICAL STATE MACHINE & GUARDRAILS                              │
│    • Struggle Exemption: "idk" or "give hint" is flagged as learning   │
│    • Scaffolding Check: Assisted answers logged as Weakness, not Mastery│
│    • Overwrite Gate: Atomically resolves Weakness vs. Mastery conflicts │
│    • Drift Monitor: Increments drift counter on off-topic turns         │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│ 4. CLOSING THE RECALL LOOP                                             │
│    • Dedicated Flashcards Modal: 3D flip self-drill with auto-promotion │
│    • Drift = 4 Hard Pause: Input locked; requires 3-question Warm-Up   │
│      Drill completion to reactivate working memory and resume session  │
└────────────────────────────────────────────────────────────────────────┘
```

---

## 3. System Architecture & Dual-Pipeline Decoupling

To ensure zero user-facing latency penalties and prevent schema serialization crashes, RecallTutor isolates real-time conversational streaming from structured background knowledge harvesting.

```
                             User Prompt
                                  │
                                  ▼
           ┌──────────────────────────────────────────────┐
           │      Decoupled Execution Orchestrator        │
           └──────────────┬────────────────────────┬──────┘
                          │                        │
       (Synchronous / Fast Path)      (Asynchronous / Background)
                          ▼                        ▼
┌───────────────────────────────────┐  ┌───────────────────────────────────┐
│     Pipeline A: Chat Stream       │  │   Pipeline B: Cognitive Harvester │
│ • Raw conversational text stream  │  │ • 6-turn sliding window evaluation│
│ • TTFT < 800ms, total < 3.5s      │  │ • Strict 2–4 word noun tags       │
│ • Zero JSON schema enforcement    │  │ • Silent catch (`error: null`)    │
└─────────────────┬─────────────────┘  └─────────────────┬─────────────────┘
                  │                                      │
                  ▼                                      ▼
┌───────────────────────────────────┐  ┌───────────────────────────────────┐
│       Center Chatfeed UI          │  │     Pedagogical Guardrail Engine  │
│ • Mathematical Markdown / LaTeX   │  │ • Guardrail A: Hint Scaffolding   │
│ • Optimistic client-side updates  │  │ • Guardrail B: Overwrite / Merge  │
│ • Independent auto-scroll feed    │  │ • Memory Firewall: Abort on Drift │
└─────────────────┬─────────────────┘  └─────────────────┬─────────────────┘
                  │                                      │
                  ▼                                      ▼
┌───────────────────────────────────┐  ┌───────────────────────────────────┐
│        Supabase PostgreSQL        │  │    Supabase PostgreSQL Schema     │
│ • messages (thread transcript)    │  │ • conversations (state & gates)   │
│ • RLS scoped to auth.uid()        │  │ • student_memories (tagged bank)  │
└───────────────────────────────────┘  └─────────────────┬─────────────────┘
                                                         │
                                                         ▼
                                       ┌───────────────────────────────────┐
                                       │    Reactive Memory Panel (Right)  │
                                       │ • Weak Spots | Mastered | Style   │
                                       │ • Triggers Flashcard Self-Drills  │
                                       └───────────────────────────────────┘
```

---

## 4. Functional Specifications

### Module 1: Dual-State Navigation & Workspace Layout

#### 1.1 Dual-State Routing Shell
* **State 1: Command Center Landing Page (`activeConversationId === null`):**
  * Displays personalized greeting: *"Welcome back, {user}! What are we mastering today?"*
  * Renders primary intent cards: `[💡 Brainstorm]` and `[🎯 Dedicated Study Topic]`.
  * Provides a secondary `[Browse Previous Sessions]` action that smoothly slides open the left drawer without reloading the layout.
  * **Strict Visibility Invariant:** The right-hand "What I Remember" drawer is unmounted and occupies zero DOM width when `activeConversationId === null`.
* **State 2: Active Session Canvas (`activeConversationId !== null`):**
  * Mounts the full 3-column workspace with active conversation context.

#### 1.2 Desktop 3-Pane Geometry
Enforces a strict viewport wrapper (`h-screen max-h-screen w-screen overflow-hidden`) with zero browser-level window scrollbars:
* **Left Drawer (`w-72`, Session Navigation):** Pinned `+ New Chat` CTA and user profile header; independently scrollable session history. Session links use client-side interactive tab buttons (`<button>`) rather than `<a>` tags to eliminate layout flashing and context-menu remounting. Auto-titles sessions in 3–5 words derived from the first user exchange.
* **Center Pane (`flex-1`, Interactive Dialogue):** Flexible chat stream with auto-scroll containment, dynamic intent badges, and a permanently docked bottom prompt container with document attachment capabilities.
* **Right Drawer (`w-80`, Memory Workspace):** Houses the live cognitive state bank partitioned into *Weak Spots*, *Mastered Topics*, and *Preferred Explanations*. Scrolls independently without shifting center chat messages.
* **Mobile / Responsive Fallback (`< 1024px`):** Center pane scales to full width; Left and Right drawers collapse into sliding drawer sheets accessible via header toggles.

---

### Module 2: Intent Gating & Session Initialization

#### 2.1 The "+ New Chat" Modal Gate
Clicking `+ New Chat` triggers a modal gate to establish cognitive context before opening the canvas:
* **Option A: Brainstorming (`mode: 'brainstorm'`):**
  * Configured for open-ended exploration, analogies, and flexible inquiry.
  * Bypasses all topic-drift tracking and pause guardrails.
  * `anchored_topic` remains `null`.
* **Option B: Dedicated Study Topic (`mode: 'dedicated'`):**
  * Opens a focused input step: *"What subject or topic are you focusing on?"* (e.g., "SSC CGL Geometry", "Bayes' Theorem").
  * Writes to database: `mode = 'dedicated'`, `anchored_topic = [Entered Topic]`, `drift_count = 0`, `accountability_count = 0`.
  * The tutor immediately delivers an opening greeting anchored to the syllabus topic, presenting the first unassisted diagnostic question.

---

### Module 3: Pedagogical State Machine & Guardrail Matrix

```
                     [Dedicated Session Active]
                                 │
                     User Submits New Prompt
                                 │
                                 ▼
                 Is Prompt Academic Struggle?
              ("idk", "stuck", "hint", "confused")
                                 │
                ┌────────────────┴────────────────┐
               YES                               NO
                │                                 │
                ▼                                 ▼
      [Academic Exemption]             Is Prompt Semantically
     • Keep drift_count = 0              Aligned to Topic?
     • Provide Socratic Hint                      │
                                      ┌───────────┴───────────┐
                                     YES                      NO
                                      │                       │
                                      ▼                       ▼
                            [On-Topic Dialogue]      Increment drift_count + 1
                           • Reset drift_count = 0            │
                           • Update last_concept              ▼
                                                     ┌─────────────────┐
                                                     │ drift_count == 2│ ──► Soft Inline Reminder
                                                     └────────┬────────┘
                                                              │
                                                              ▼
                                                     ┌─────────────────┐
                                                     │ drift_count == 4│ ──► [Hard Pause Triggered]
                                                     └────────┬────────┘
                                                              │
                                                              ▼
                                                     • Lock Chat Input
                                                     • accountability_count + 1
                                                     • Show Break Card
                                                     • Require Warm-Up Drill
```

#### 3.1 Academic Struggle Exemption (Zero-Penalty Guardrail)
* **Problem:** Standard topic classifiers mistake expressions of learning confusion for off-topic drift.
* **Rule:** Natural expressions of difficulty—such as *"I don't know"*, *"idk"*, *"confused"*, *"give me a hint"*, *"can you explain?"*, or incomplete rough-work attempts—are classified as **valid academic struggle**.
* **System Action:** `drift_count` is locked at `0`. The tutor provides supportive Socratic scaffolding without triggering warnings.

#### 3.2 Drift Escalation & The Breather Gate
When a student in a dedicated session introduces unrelated subjects (pop culture, personal banter, non-academic queries):
* **Level 1 (Soft Nudge, `drift_count === 2`):** Appends an inline redirection to the conversational turn, guiding the student back to `anchored_topic`.
* **Level 2 (Hard Pause & Break Recommendation, `drift_count === 4`):**
  * Sets `conversations.session_status = 'paused'`.
  * Increments `conversations.accountability_count` by 1; resets `drift_count = 0`.
  * Delivers an isolated break message acknowledging fatigue: *"Even though breaks are necessary and healthy, you have stepped away from this topic {accountability_count} times this session. Let's take a proper breather away from the screen."*
  * **Single-Output Constraint:** Forbids appending any follow-up question or active recall prompt to the break recommendation message.
  * **UI Locking:** Hard-disables the chat input textarea (`disabled={true}`) with placeholder: *"Session paused for a breather. Click 'Resume studying' when ready."*
  * Mounts a primary action chip: `[▶️ Resume studying]`.

---

### Module 4: Cognitive Memory Engine & Lifecycle (`learnFromExchange`)

#### 4.1 Strict Formatting Constraints
* **Topic Formatting:** The extracted `topic` must strictly be a concise noun tag (2–4 words, e.g., "Quadratic Inequalities", "Conditional Probability"). Full sentences, conversational filler, or apologies in the `topic` field are strictly rejected.
* **Notes Formatting:** Concrete diagnostic reasoning (e.g., *"Struggles with denominator inversion in multi-stage fractions"*) is isolated to the `notes` column.

#### 4.2 Guardrail A: Scaffolding / Hint Guard (Anti-False Mastery)
* Before classifying any topic under `category: 'mastery'`, the background harvester evaluates the preceding assistant turn:
  * If the assistant provided an explicit formula, hint, leading question, or step-by-step breakdown, a correct answer represents **scaffolded completion**, not mastery.
  * **Classification:** Classified under `category: 'weakness'` with note: *"Solved with direct tutor hint; requires unassisted recall validation."*
  * **Mastery Criteria:** `category: 'mastery'` is only granted when the student solves an unassisted problem or explains the underlying principle independently.

#### 4.3 Guardrail B: Atomic Promotion & Conflict Overwrite
* **Invariant:** A single concept cannot exist simultaneously as both a `weakness` and a `mastery` within the same `conversation_id`.
* **Deduplication Logic:** When genuine mastery is achieved, the engine queries existing memories for that thread. If a record with `category: 'weakness'` matches or semantically overlaps the concept, the system atomically updates the row to `mastery` and updates the timestamp, instantly moving the concept across categories in the UI.

#### 4.4 The Memory Firewall
* If `conversations.mode === 'dedicated'` AND (`drift_count > 0` OR `session_status === 'paused'`):
  * The background harvester immediately terminates execution (`return null`).
  * Forbids writing casual conversation, banter, or off-topic tangents to `student_memories`.

---

### Module 5: Integrated Active-Recall Tooling

#### 5.1 Interactive Flashcards Modal (`FlashcardModal`)
* **Trigger:** Accessible via action chips in the memory panel header: `[Drill Weak Spots]` and `[Refresh Mastered]`.
* **Interface:** Distraction-free modal overlay reading stored records from `student_memories` matching `conversation_id` and category.
* **Card Anatomy & Interaction:**
  * Front: Concept tag and a targeted diagnostic question formulated from `notes`.
  * Flip Animation: Smooth 3D CSS perspective card-flip triggered via click or Spacebar.
  * Back: Correct answer, formula, and conceptual breakdown.
* **Self-Assessment Controls:**
  * `[Still Shaky]`: Retains the card in its current category and advances to the next item.
  * `[Got It Unassisted]`: Triggers Guardrail B (Automatic Promotion). If the card was in `weakness`, updates its row to `category: 'mastery'` with note: *"Mastered via flashcard self-drill unassisted,"* advancing the deck and reactively updating the sidebar.

#### 5.2 Post-Break Warm-Up Drill Modal (`WarmupDrillModal`)
* **Trigger:** Mandatory resume gate fired when the student clicks `[▶️ Resume studying]` on a paused session.
* **Pedagogical Goal:** Overcome cold-start friction and reactivate working memory before unlocking open-ended chat.
* **Execution Flow:**
  1. Opens a 3-question rapid-fire multiple-choice modal targeting `last_active_concept` and recorded session `weakness` items.
  2. Question Sequencing: Delivers 1 question at a time with 4 tap-to-select options (A, B, C, D).
  3. Tactile Feedback: Correct selection triggers a soft green indicator and advances after 0.5s; incorrect selection highlights in soft amber with a 1-sentence instant takeaway before advancing.
  4. Unassisted Promotion: Correct answers to previous weak concepts trigger Guardrail B promotion in the background.
  5. Handoff: Upon completing Question 3, displays a brief banner (*"Working memory reactivated!"*), closes the modal, sets `session_status = 'active'`, resets `drift_count = 0`, unlocks the chat textarea, and dispatches a background `{ action: 'RESUME_STUDY' }` payload to prompt the tutor's next problem.

---

### Module 6: Operational & Safety Guardrails

#### 6.1 Academic Methodology vs. Real-World Execution
* **Permitted Academic Scope:** Step-by-step mathematical algorithms, experimental derivations, scientific synthesis, and historical analyses belonging to standard curricula are permitted, provided they conclude with an active-recall comprehension check.
* **Prohibited Real-World Instructions:** Actionable real-world execution manuals for non-academic trades, hazardous tasks, or life-admin workflows are prohibited. If queried, the tutor pivots strictly to the underlying academic, historical, or scientific principle.

---

## 5. Data Architecture & Entity Relationship Specifications

> **Data Engineering Attribution:** Functional entity boundaries, state keys, and relational constraints were established in product specifications. SQL DDL syntax, cascading rules, and Supabase RLS security policies were scaffolded via Gemini 3.8 and deployed through Lovable.

```
┌─────────────────────────┐          ┌─────────────────────────┐
│      conversations      │          │     student_memories    │
├─────────────────────────┤          ├─────────────────────────┤
│ id (PK, UUID)           │◄────┐    │ id (PK, UUID)           │
│ user_id (FK -> auth)    │     │    │ conversation_id (FK)────┼──┐
│ title (VARCHAR)         │     │    │ user_id (FK -> auth)    │  │
│ mode (Enum)             │     │    │ topic (VARCHAR(64))     │  │
│ anchored_topic (TEXT)   │     │    │ category (Enum)         │  │
│ drift_count (INT)       │     │    │ notes (TEXT)            │  │
│ accountability_count(INT│     │    │ confidence_score (INT)  │  │
│ session_status (Enum)   │     │    │ updated_at (TIMESTAMPTZ)│  │
│ last_active_concept(TEXT│     │    └─────────────────────────┘  │
│ created_at (TIMESTAMPTZ)│     │                                 │
└─────────────────────────┘     │                                 │
                                │                                 │
                                │    ┌─────────────────────────┐  │
                                │    │        messages         │  │
                                │    ├─────────────────────────┤  │
                                └───┼─ conversation_id (FK)   │  │
                                     │ user_id (FK -> auth)    │  │
                                     │ role (Enum: user/asst)  │  │
                                     │ content (TEXT)          │  │
                                     │ created_at (TIMESTAMPTZ)│  │
                                     └─────────────────────────┘  │
                                                                  │
                                 1-to-Many Cascading Relationships ┘
```

### 5.1 Relational Schema Implementation (Supabase DDL)

```sql
-- 1. CONVERSATIONS TABLE
CREATE TABLE conversations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
    title TEXT NOT NULL DEFAULT 'New Study Session',
    mode TEXT CHECK (mode IN ('brainstorm', 'dedicated')),
    anchored_topic TEXT,
    drift_count INT NOT NULL DEFAULT 0,
    accountability_count INT NOT NULL DEFAULT 0,
    session_status TEXT NOT NULL DEFAULT 'active' CHECK (session_status IN ('active', 'paused')),
    last_active_concept TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- 2. MESSAGES TABLE
CREATE TABLE messages (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    conversation_id UUID NOT NULL REFERENCES conversations(id) ON DELETE CASCADE,
    user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
    role TEXT NOT NULL CHECK (role IN ('user', 'assistant')),
    content TEXT NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- 3. STUDENT_MEMORIES TABLE
CREATE TABLE student_memories (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    conversation_id UUID NOT NULL REFERENCES conversations(id) ON DELETE CASCADE,
    user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
    topic VARCHAR(64) NOT NULL,
    category TEXT NOT NULL CHECK (category IN ('weakness', 'mastery', 'preference')),
    notes TEXT NOT NULL,
    confidence_score INT DEFAULT 0 CHECK (confidence_score BETWEEN 0 AND 100),
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Row-Level Security (RLS) Policies
ALTER TABLE conversations ENABLE ROW LEVEL SECURITY;
ALTER TABLE messages ENABLE ROW LEVEL SECURITY;
ALTER TABLE student_memories ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can manage their own conversations"
ON conversations FOR ALL USING (auth.uid() = user_id);

CREATE POLICY "Users can manage their own messages"
ON messages FOR ALL USING (auth.uid() = user_id);

CREATE POLICY "Users can manage their own memories"
ON student_memories FOR ALL USING (auth.uid() = user_id);
```

### 5.2 Data Dictionary & Business Logic Constraints

| Entity | Field Name | Data Type | Constraints / Validation | Product Purpose & Business Rule |
| :--- | :--- | :--- | :--- | :--- |
| `conversations` | `id` | `UUID` | Primary Key, `gen_random_uuid()` | Unique thread identifier. |
| `conversations` | `mode` | `VARCHAR(16)` | `CHECK (mode IN ('brainstorm', 'dedicated'))` | Gating flag: determines whether drift tracking and pause gates are enforced. |
| `conversations` | `anchored_topic` | `TEXT` | Nullable | The active syllabus subject used as semantic anchor in dedicated sessions. |
| `conversations` | `drift_count` | `INTEGER` | `NOT NULL DEFAULT 0` | Tracks consecutive off-topic dialogue turns. Resets to 0 on on-topic exchanges. |
| `conversations` | `accountability_count`| `INTEGER` | `NOT NULL DEFAULT 0` | Running tally of break cards issued; referenced in pause messages. |
| `conversations` | `session_status` | `VARCHAR(16)` | `CHECK (status IN ('active', 'paused'))` | Controls UI lock state on the center chat input. |
| `conversations` | `last_active_concept`| `TEXT` | Nullable | Stores the most recent academic concept to generate warm-up drills upon resume. |
| `messages` | `conversation_id` | `UUID` | Foreign Key -> `conversations.id` `ON DELETE CASCADE` | Scopes chat persistence to specific sessions. |
| `messages` | `role` | `VARCHAR(16)` | `CHECK (role IN ('user', 'assistant'))` | Identifies conversational speaker. |
| `student_memories` | `topic` | `VARCHAR(64)` | `NOT NULL` | Concise 2–4 word noun phrase tag. Full sentences strictly prohibited. |
| `student_memories` | `category` | `VARCHAR(16)` | `CHECK (category IN ('weakness', 'mastery', 'preference'))` | Categorizes cognitive state for UI drawer display and flashcard routing. |
| `student_memories` | `notes` | `TEXT` | `NOT NULL` | Stores diagnostic context, specific arithmetic pitfalls, or preferred analogy styles. |

---

## 6. Technical Specifications & LLM Implementation Benchmarks

```
┌─────────────────────────────────┬───────────────────┬────────────────────────────────────────────────────────┐
│ Metric / Benchmark              │ Target Threshold  │ Operational Validation Method                          │
├─────────────────────────────────┼───────────────────┼────────────────────────────────────────────────────────┤
│ Time-to-First-Token (TTFT)      │ < 800 ms          │ Streaming via browser PerformanceObserver API.         │
│ End-to-End Chat Completion      │ < 3.5 s           │ Total network duration for average conversational turn.│
│ Background Harvester Execution  │ < 2.0 s           │ Asynchronous Edge Function execution duration.         │
│ Extraction Crash Rate           │ 0.0% User-Facing  │ Decoupled silent catch (`error: null`) pattern.        │
│ False Mastery Rejection Rate    │ 100%              │ Evaluated against hint-scaffolded test suites.         │
│ Topic Tag Noun Compliance       │ > 95%             │ Automated audit of string lengths (2–4 words).         │
│ Session Hydration Latency       │ < 200 ms          │ Indexed Supabase query fetch on thread switch.         │
└─────────────────────────────────┴───────────────────┴────────────────────────────────────────────────────────┘
```

---

## 7. Edge Cases & Exception Handling Matrix

| Scenario / Edge Case | Failure Mode / Operational Risk | Architectural Resolution & System Behavior |
| :--- | :--- | :--- |
| **API Parsing / JSON Drift** | Model outputs markdown backticks or malformed JSON during memory harvesting. | The extraction pipeline catches serialization errors silently (`error: null`), logs telemetry internally, and never interrupts the active chat stream. |
| **Context Window Exceeded** | Multi-hour sessions accumulate high token counts, increasing latency and API costs. | Conversational streaming maintains a rolling context window; memory harvesting evaluates only a 6-turn sliding window, relying on the Supabase memory bank for historical state. |
| **Simultaneous Category Conflict**| A student masters a concept previously logged as a weakness in the same thread. | Guardrail B queries the database for existing concept tags, silently overwriting or promoting the old `weakness` row rather than inserting duplicate records. |
| **Student Resumes After Long Break**| Student returns to an open session after days with diminished working memory. | Rehydration checks `session_status`. If paused, chat remains locked until the student clears the 3-question Warm-Up Drill. |
| **Rapid Tab Switching / Desync** | Fast clicking in the left drawer triggers race conditions in chat rendering. | Interactive tab elements manage local abort controllers for in-flight requests, rendering skeleton loaders until the active thread state is resolved. |

---

## 8. Success Metrics & Product KPIs

### 8.1 North Star Metric
* **Weekly Concept Progression Rate (CPR):** The ratio of stored student `weakness` concepts promoted to `mastery` via unassisted chat validation or self-directed flashcard drills within 7 days.

### 8.2 Supporting Product Metrics
* **Session Resumption Rate:** $\ge 60\%$ of sessions paused due to fatigue are resumed through the Warm-Up Drill within 24 hours.
* **Drift Decay Curve:** The average number of drift pauses per dedicated session decreases over a 14-day cohort window as students adapt to the session focus structure.
* **Recall Retention:** $\ge 75\%$ first-time accuracy on Warm-Up Drill questions derived from concepts mastered in prior sessions.

---

## 9. Product Roadmap

### Phase 1: V1 Core (Shipped / Current Live Prototype)
* Decoupled streaming chat and background memory harvesting.
* Thread-isolated Supabase session storage with RLS.
* Desktop 3-pane independent scroll layout with mobile responsive drawers.
* Intent gating: `[💡 Brainstorm]` vs. `[🎯 Dedicated Study Topic]`.
* 3-Guardrail state machine: Struggle Exemption, Scaffolding Guard, Drift Pause Gate.
* Interactive Flashcards Modal and post-break Warm-Up Drill resume gate.

### Phase 2: V2 Planned Enhancements
* **Autonomous Spaced Retrieval (`pg_cron`):** Resurfacing recorded session weak spots at optimal Leitner intervals ($T+1\text{ day}$, $T+3\text{ days}$, $T+7\text{ days}$) upon starting fresh threads.
* **Multimodal Error-Log Parsing:** Upload dropzone parsing handwritten algebraic error notes and isolating calculation mistakes into `student_memories`.
* **Cross-Session Memory Synthesis:** Optional opt-in toggle enabling global memory rehydration across distinct sessions belonging to the same subject tag.
