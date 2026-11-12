# Product Requirements Document (PRD): RecallTutor

**Product:** RecallTutor — Context-Aware AI Study Companion  
**Document Version:** 2.0 (Updated Release)  
**Product Owner:** Uditanshu Singh  
**Status:** Live Prototype  
**Live App:** [https://recall-buddy-00.lovable.app](https://recall-buddy-00.lovable.app)  
**Portfolio:** [github.com/yooess1307/product-management-portfolio](https://github.com/yooess1307/product-management-portfolio)

---

## 1. What RecallTutor Is About (Overview)

**RecallTutor** is a mobile-first, context-aware AI tutor designed to turn passive chatbot Q&A into disciplined, active studying. 

Unlike standard AI chatbots that forget who you are between sessions, easily give away answers, or follow you down distracting tangents, RecallTutor acts like a real personal tutor. It maintains a live, visible memory panel (**"What I Remember About You"**) for every study session—tracking your **Weak Spots**, **Mastered Topics**, and **Preferred Explanations**—while using gentle behavioral guardrails, interactive flashcards, and focus drills to help you genuinely master difficult subjects.

> **Builder's Note:** As a Product Manager, I defined the product vision, user personas, learning workflows, behavioral guardrails, and user interface structure. I used AI prototyping tools (Lovable) to bring these product specifications to life as a working web application.

---

## 2. The Problem We Are Solving

Students using generic AI tools (like standard ChatGPT or Gemini) to prepare for exams face four major roadblocks:

1. **Session Amnesia:** Standard chatbots treat every new session like a blank slate. Students waste time repeatedly explaining what topics they struggle with, what mistakes they usually make, and how they like concepts explained.
2. **False Sense of Mastery:** When a student gets stuck and the AI gives them a formula or step-by-step hint, the student often gets the final answer right. Generic bots assume the student has "learned" the topic—even though the student couldn't solve it on their own.
3. **Zero Focus Accountability:** Studying requires discipline. When a student gets tired and starts chatting about movies, food, or random trivia, generic chatbots happily play along instead of helping the student stay on track.
4. **Passive Reading vs. Active Recall:** Simply reading AI explanations creates an illusion of competence. Students need built-in tools to test themselves on their specific weak areas without having to manually create study cards.

---

## 3. Target User Personas

| Persona | Background & Context | Core Goals | Key Frustrations | How RecallTutor Helps |
| :--- | :--- | :--- | :--- | :--- |
| **1. The High-Stakes Exam Candidate (Primary)** | College graduates and working professionals preparing for competitive exams (GMAT, GRE, SSC CGL, CFA). | Identify recurring conceptual traps, practice unassisted problem-solving, and track progress across multi-day revision sprints. | *"I keep making the same silly mistakes in probability, and I have to re-explain my weak areas every time I open a new chat."* | Automatically logs weak spots in a live sidebar and tests them until solved without hints. |
| **2. The Structured Self-Learner** | Undergraduate students or career switchers learning technical or quantitative subjects independently. | Understand *why* a concept works through clear analogies and Socratic step-by-step guidance rather than just getting the final answer. | *"Standard AI dumps a wall of textbook formulas on me instead of letting me try first or using visual examples."* | Remembers preferred explanation styles (e.g., visual analogies) and prompts the student to attempt answers first. |
| **3. The Easily Distracted Learner** | Students studying late at night or balancing heavy workloads who are prone to tab-switching and mental fatigue. | Stay locked onto one syllabus topic for a focused 45-minute study block without burning out. | *"I start asking about geometry, get tired, and suddenly realize I've spent 30 minutes asking the AI random trivia."* | Detects off-topic drift, offers a gentle nudge, and pauses the session for a healthy breather when fatigue sets in. |

---

## 4. End-to-End User Journey

```
1. Welcome Command Center
   └── Student logs in and chooses between [💡 Brainstorm] or [🎯 Dedicated Study Topic].
          │
2. Focused Study & Socratic Dialogue
   └── Tutor asks diagnostic questions; student can also upload syllabus notes or mistake logs.
          │
3. Live Memory Tracking ("What I Remember About You")
   └── App silently updates Weak Spots, Mastered Topics, and Learning Preferences in the right sidebar.
          │
4. Smart Learning & Focus Guardrails
   ├── Gives hints when stuck (without falsely marking the topic as "Mastered").
   └── Nudges the student if they drift off-topic; pauses for a break after 4 distractions.
          │
5. Active Recall & Comeback Drills
   ├── Student clicks [Drill Weak Spots] anytime to flip interactive flashcards.
   └── After a break pause, student completes a quick 3-question warm-up quiz to unlock the chat.
```

---

## 5. Key Features & Product Improvements

### 5.1 Welcome Command Center & Intent Selection
When logging in or starting a **"+ New Chat"**, students land on a clean command center greeting them by name (*"Welcome back! What are we mastering today?"*) with two clear study modes:
* **💡 Brainstorm Mode:** Designed for freeform curiosity, analogies, and open-ended exploration with zero distraction limits.
* **🎯 Dedicated Study Topic Mode:** Designed for disciplined exam prep. A setup prompt asks *"What subject or topic are you focusing on?"* Once entered (e.g., *"Quadratic Equations"*), the session locks onto that subject and the tutor immediately opens with a diagnostic question.
* **Quick History Bridge:** A **"Browse Previous Sessions"** button opens past study threads instantly without reloading the screen.

### 5.2 Clean 3-Pane Workspace & Multi-Session History
To eliminate clutter during long study sessions, the desktop screen is divided into three independent panels (and adapts smoothly into slide-out drawers on mobile phones):
* **Left Panel (Session History):** Lists all past study sessions sorted by most recent. Every new chat is automatically given a concise **3–5 word title** based on the first question (e.g., *"Bayes Theorem Review"*). Switching between chats is instant with zero page flickering. Includes a clean user profile menu for account settings and safe logout.
* **Center Panel (Active Study Chat):** A distraction-free conversation feed with clear mathematical formatting, a permanently anchored message bar at the bottom, and a **Document Upload button** where students can drop syllabus notes or mistake logs to guide the tutor.
* **Right Panel ("What I Remember About You"):** A live, scrollable memory board divided into three color-coded categories:
  * 🔴 **Weak Spots:** Concepts where the student struggled or needed hints.
  * 🟢 **Mastered Topics:** Concepts the student solved completely on their own.
  * 🔵 **Preferred Explanations:** Discovered learning preferences (e.g., *"Prefers real-world examples over abstract definitions"*).
  * *Smart Visibility:* This panel only appears when inside an active study session, keeping the welcome screen clean.

### 5.3 Smart Memory Hygiene & Anti-False-Mastery Guardrails
To ensure the tutor's memory stays accurate and trustworthy, four behavioral rules govern how notes are saved:
1. **No False Mastery (Hint Protection):** Before marking any topic as **Mastered**, the tutor checks whether it gave a hint, formula, or leading clue in the previous message. If the student only got the answer right *because* of a hint, the topic stays under **Weak Spots** with a note: *"Solved with tutor hint; needs unassisted recall practice."*
2. **Automatic Promotion & Clean-Up:** A topic can never exist as both a *Weak Spot* and a *Mastered Topic* at the same time. The moment a student solves a weak concept independently, it automatically moves from Weak Spots to Mastered Topics.
3. **Short, Scannable Topic Tags:** Memory cards use crisp 2–4 word topic titles (e.g., *"Triangle Area Ratios"*) with detailed diagnostic feedback tucked neatly inside the note description.
4. **Memory Firewall:** Casual greetings, jokes, or off-topic banter are completely ignored by the memory tracker so the sidebar stays strictly academic.

### 5.4 Humane Distraction & Focus Guardrails (Dedicated Mode)
In **Dedicated Study Topic** mode, RecallTutor helps students stay accountable without feeling policed:
* **Academic Struggle Protection (Zero Penalty):** Saying *"I don't know"*, *"idk"*, *"I'm stuck"*, *"confused"*, or *"give me a hint"* is recognized as genuine learning effort—never as a distraction. The tutor responds with encouraging Socratic guidance.
* **Soft Reminder (2 Off-Topic Messages):** If a student brings up unrelated topics twice in a row (e.g., sports scores or food), the tutor gently nudges them back to their chosen study topic.
* **Healthy Break Pause (4 Off-Topic Messages):** If a student drifts off-topic four times, the tutor recognizes mental fatigue, pauses the chat input box, and displays a supportive break card reminding them how many times they have stepped away—without nagging or asking another study question.

### 5.5 Interactive Active-Recall Tools
RecallTutor turns saved memories into interactive practice loops:
* **Interactive Flashcard Modal:** Clicking **"Drill Weak Spots"** or **"Refresh Mastered"** in the right sidebar opens a focused 3D flip-card deck built from that session's notes.
  * Front of card: Displays the topic and a targeted recall question.
  * Back of card: Reveals the answer, formula, and explanation.
  * Self-Grading: Clicking **"Got It Unassisted"** automatically promotes a Weak Spot to Mastered Topics in real time; clicking **"Still Shaky"** keeps it in rotation for more practice.
* **Post-Break 3-Question Warm-Up Drill:** When a student is ready to return from a distraction pause and clicks **"Resume Studying"**, the chat does not unlock immediately. Instead, a quick **3-question multiple-choice warm-up quiz** pops up based on the last concept they were studying. Completing the 3 questions reactivates their working memory, unlocks the chat box, and seamlessly resumes the lesson.

### 5.6 Curriculum & Safety Boundaries
* **Guided Academic Explanations:** When explaining step-by-step math derivations, scientific concepts, or historical events, the tutor always ends with a quick comprehension check to ensure active engagement.
* **Strict Academic Focus:** If a student asks for non-academic execution guides (such as personal errands or hazardous real-world tasks), the tutor politely pivots back to the underlying academic or scientific principles.

---

## 6. Product Experience & Quality Standards

* **Zero-Interruption Performance:** Memory updates happen silently in the background after the tutor replies. Even if a background note fails to save, the student's active chat conversation is never blocked or interrupted.
* **Mobile-First Usability:** Works smoothly on smartphones—collapsing the left history menu and right memory board into easy-to-toggle slide-over drawers while keeping the chat input pinned to the thumb zone.
* **Privacy & Thread Isolation:** Each student's sessions and memory notes are completely private to their account, and memories stay isolated per study thread so different subjects never mix together.
* **Calm "Study Notebook" Design:** Uses warm paper-like tones, deep teal accents, soft amber for weak spots, and calm green for mastered topics to reduce eye strain during long study hours.

---

## 7. Success Metrics (How We Measure Impact)

| Metric | Target (90 Days) | Why It Matters |
| :--- | :--- | :--- |
| **Weak-to-Mastered Promotion Rate** | $\ge 30\%$ of logged weak spots promoted | Proves students are genuinely closing knowledge gaps rather than just chatting. |
| **Post-Break Session Resumption** | $\ge 60\%$ of paused sessions resumed | Shows that the break pause and 3-question warm-up drill feel supportive rather than frustrating. |
| **Weekly Returning Learners** | $\ge 40\%$ returning to past threads | Validates that persistent session memory brings students back for ongoing revision. |
| **Flashcard Drill Engagement** | $\ge 2$ flashcard drills started per user/week | Measures adoption of active-recall self-testing beyond passive Q&A. |

---

## 8. Product Risks & Mitigations

| Risk | Potential Impact | Product Mitigation |
| :--- | :--- | :--- |
| **Tutor misjudges confusion as distraction** | Student gets penalized for struggling with a hard problem. | Built an explicit **Academic Struggle Exemption** so phrases like *"idk"*, *"confused"*, or *"hint"* never trigger distraction warnings. |
| **Students feel restricted or policed** | Learners abandon the app if guardrails feel too rigid. | Offered **Brainstorm Mode** with zero guardrails, framed break pauses around mental health, and added a soft warning before pausing. |
| **Cluttered or contradictory memory sidebar** | Duplicate tags erode trust in the AI's intelligence. | Enforced short 2–4 word topic tags and automatic overwrite rules so a concept never appears as both weak and mastered. |

---

## 9. Product Roadmap

* **Version 1.0 (Current Live Prototype):** Dual-mode session setup, 3-pane workspace, thread-isolated memory panel, anti-false-mastery guardrails, distraction pause gate, interactive flashcards, and post-break warm-up drills.
* **Version 1.1 (Smart Document Ingestion):** Upload full PDF study guides or past exam papers to automatically extract and pre-populate a custom list of Weak Spots before starting a session.
* **Version 2.0 (Spaced Repetition & Cross-Session Mastery):** Gentle daily reminders to review weak spots at optimal memory intervals (1 day, 3 days, 7 days later) and an optional "Master Knowledge Profile" combining progress across all sessions in the same subject.
