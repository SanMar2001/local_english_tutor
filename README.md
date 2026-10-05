# English Tutor Local AI

> A local English tutoring system designed to support multiple learners on the same machine, with independent conversations, assessments, learning goals, and progress.

## 📋 Description

English Tutor Local AI is a local conversational English tutoring system designed to help one or more learners improve their English through written and spoken interaction.

The application runs entirely on the user's machine and combines:

- Conversational interaction with a local LLM.
- Real-time audio communication.
- Local speech-to-text transcription.
- Persistent learner profiles and learning history.
- Semantic memory.
- Adaptive assessment.
- Personalized learning plans.
- Progress tracking.
- A React frontend.
- A FastAPI backend.

The system is intended to support multiple learners using the same local installation.

Each learner must have an independent learning profile and independent learning history.

The application must remain **fully local at runtime**. No external AI APIs or paid cloud services should be required.

---

# 👥 Learners

The system must support multiple learners on the same machine.

A learner represents a person using the tutor.

Each learner should eventually have independent:

- Conversations.
- Messages.
- Assessments.
- Current proficiency level.
- Target proficiency level.
- Learning goals.
- Learning plan.
- Progress.
- Recurring errors.
- Vocabulary history.
- Learning history.

Conceptually:

```text
                    English Tutor
                         │
          ┌──────────────┼──────────────┐
          │              │              │
       Learner A      Learner B      Learner C
          │              │              │
     Conversations   Conversations   Conversations
     Assessments     Assessments     Assessments
     Goals           Goals           Goals
     Progress        Progress        Progress
```

The first implementation does **not** require authentication.

User identification may initially be handled through local learner selection.

Authentication, passwords, sessions, OAuth, JWT, or other access-control mechanisms should only be introduced when they become necessary for the application.

Do not implement authentication as part of the initial project foundation.

---

# 🎯 Learning Objective

The tutor must not assume a predefined English level or learning path.

Instead, the system should first evaluate the learner's current proficiency and then build a personalized learning plan based on the results.

The learning process should be adaptive and learner-specific.

---

## Initial Assessment

The tutor should be capable of conducting an adaptive diagnostic assessment covering, when appropriate:

- Grammar.
- Vocabulary.
- Reading comprehension.
- Written expression.
- Conversational ability.
- Fluency.
- Pronunciation and intelligibility.
- Listening comprehension.
- Ability to explain, argue, and develop ideas.

The assessment should not rely exclusively on isolated questions.

The tutor should adapt subsequent questions and activities based on previous answers in order to obtain a more representative evaluation.

The tutor should estimate the learner's proficiency using the **CEFR framework (A1–C2)**.

The assessment should produce:

- Estimated overall CEFR level.
- Estimated level by skill.
- Main strengths.
- Main weaknesses.
- Recurring errors or patterns.
- Vocabulary gaps.
- Areas requiring further evaluation.
- Confidence level of the assessment.

The tutor must distinguish between an **estimated level** and a formally certified CEFR level.

The application must never present its evaluation as an official language certification.

---

# 🎯 Learning Goal

After the initial assessment, the tutor should communicate the estimated level to the learner and ask:

> What level would you like to reach?

The learner should be able to choose a target level such as:

```text
A1 → A2
A2 → B1
B1 → B2
B2 → C1
C1 → C2
```

The learner should also be able to define a more specific objective, such as:

- Professional English.
- Technical English.
- Academic English.
- Conversational fluency.
- Job interviews.
- Presentations.
- Writing.
- Travel.
- General English.

The current level and target should belong to the learner's learning context rather than being treated as global application settings.

---

# 📚 Personalized Learning Plan

Once the current and target levels are established, the tutor should generate a personalized learning plan.

The plan should define:

- Priority skills.
- Grammar topics.
- Vocabulary domains.
- Conversation topics.
- Pronunciation goals.
- Listening activities.
- Writing activities.
- Reading activities.
- Recurring weaknesses to monitor.
- Suggested progression.
- Criteria for determining when a topic has been sufficiently mastered.

The plan must be **adaptive rather than static**.

The tutor should continuously use the learner's performance to determine whether to:

- Continue with the current topic.
- Increase difficulty.
- Review a previous topic.
- Introduce a prerequisite.
- Revisit a recurring weakness.
- Introduce a new topic.
- Reassess the learner's level.

Different learners should be able to have completely different learning plans.

---

# 📈 Progress Reassessment

The tutor should periodically reassess the learner instead of assuming that progress is linear.

The estimated level should be updated when sufficient evidence suggests that the learner's proficiency has changed.

Progress should be evaluated using multiple signals rather than a single score.

Potential metrics include:

- `grammar_accuracy`
- `lexical_range`
- `lexical_diversity`
- `fluency`
- `wpm`
- `filler_ratio`
- `comprehension`
- `complexity`
- `pronunciation`
- `task_success`
- `recurring_errors`

These metrics should support the tutor's reasoning but should not be treated as precise measurements of language proficiency.

Progress must be associated with the corresponding learner.

---

# 💬 Conversations

Conversations are part of each learner's history.

A conversation belongs to exactly one learner.

A conversation contains messages exchanged between the learner and the tutor.

Conceptually:

```text
Learner
   │
   ├── Conversation
   │      ├── Message
   │      ├── Message
   │      └── Message
   │
   ├── Conversation
   │      ├── Message
   │      └── Message
   │
   └── ...
```

Messages should retain enough information to reconstruct the conversational context when required.

The conversation system should eventually support both:

- Text interaction.
- Spoken interaction.

The initial implementation does not need to support audio.

---

# 🧠 Long-Term Objective

The ultimate objective is to create an **adaptive language-learning system**, rather than a generic conversational chatbot.

The tutor should continuously answer:

```text
Where am I now?
       ↓
Where do I want to go?
       ↓
What should I work on next?
       ↓
Am I actually improving?
```

The system should continuously update the third answer based on evidence from the learner's performance.

Each learner should have their own answer to these questions.

---

# 🛠 Technology Stack

| Layer                  | Technology                      |
| ---------------------- | ------------------------------- |
| Backend                | Python 3.11+ + FastAPI          |
| ORM / Persistence      | SQLAlchemy                      |
| Database               | SQLite                          |
| Frontend               | React + Vite + TypeScript       |
| Audio / Communications | WebRTC + PeerJS                 |
| Speech-to-Text         | Whisper.cpp + GGUF              |
| LLM                    | Ollama                          |
| Vector Database        | ChromaDB                        |
| Infrastructure         | Native Windows or WSL2 + Docker |

The exact LLM model may change during development.

The LLM integration should remain sufficiently abstract to allow the local model to be replaced without redesigning the application.

---

# 💻 Target Environment

The primary development environment is:

- Windows.
- Ryzen 7 5800XT.
- RTX 3060 12 GB.
- 32 GB RAM.
- Python 3.11+.
- Node.js.
- Ollama.

The application must remain usable without a GPU.

GPU acceleration should be treated as an optimization rather than a hard dependency.

---

# 🏗 Architecture

The high-level architecture is:

```text
┌─────────────────────────────┐
│           Browser           │
│      React / Vite / TS      │
└──────────────┬──────────────┘
               │
               │ HTTP / WebRTC
               ▼
┌─────────────────────────────┐
│        FastAPI Server       │
│                             │
│  API / Conversations /      │
│  Learning / Audio Pipeline  │
└──────┬──────────┬───────────┘
       │          │
       │          │
       ▼          ▼
┌───────────┐  ┌───────────────┐
│Whisper.cpp│  │    Ollama     │
│    STT    │  │      LLM      │
└───────────┘  └───────┬───────┘
                       │
                       ▼
              ┌────────────────┐
              │ Learning Engine│
              └───────┬────────┘
                      │
             ┌────────┴─────────┐
             ▼                  ▼
       ┌───────────┐      ┌───────────┐
       │  SQLite   │      │ ChromaDB  │
       │ Learners/ │      │ Semantic  │
       │ Progress  │      │  Memory   │
       └───────────┘      └───────────┘
```

The architecture should evolve as implementation progresses.

The diagram represents the intended system, not a requirement to implement every component immediately.

---

# 🗄️ Data Model Direction

The system should eventually represent relationships similar to:

```text
User
 │
 ├── Conversation
 │      └── Message
 │
 ├── Assessment
 │
 ├── LearningGoal
 │
 ├── LearningPlan
 │
 ├── Progress
 │
 ├── ErrorPattern
 │
 └── VocabularyHistory
```

These are conceptual entities.

They do **not** need to be implemented all at once.

The database model should evolve according to actual application requirements.

Do not create speculative tables merely because they appear in this diagram.

---

# 📁 Intended Project Structure

The repository starts with only the project documentation and configuration files.

The intended application structure is:

```text
english-tutor/

├── server/
│   ├── main.py
│   ├── models/
│   ├── database/
│   ├── api/
│   ├── audio/
│   ├── llm/
│   ├── learning/
│   └── memory/
│
├── client/
│   ├── src/
│   │   ├── App.tsx
│   │   ├── components/
│   │   ├── hooks/
│   │   └── ...
│   └── package.json
│
├── data/
│
├── docs/
│
├── requirements.txt
│
└── docker-compose.yml
```

This structure is an architectural target, not a requirement to create every directory immediately.

Do not create empty directories or placeholder files merely to match this structure.

Create components only when required by the current implementation milestone.

---

# 🧠 Development Philosophy

Development must happen incrementally through small, independently understandable milestones.

Examples:

```text
Initialize FastAPI
        ↓
Configure SQLAlchemy + SQLite
        ↓
Create learner and conversation models
        ↓
Implement basic conversation API
        ↓
Implement Ollama client
        ↓
Implement text conversation
        ↓
Implement assessment engine
        ↓
Implement personalized learning plan
        ↓
Implement progress tracking
        ↓
Implement Whisper transcription
        ↓
Integrate audio pipeline
        ↓
Implement semantic memory
        ↓
Build frontend
```

The agent must **not attempt to implement the entire application in one session**.

A milestone should be small enough to understand, implement, verify, and hand off independently.

---

# 🤖 Agent Behavior

The development agent must:

1. Understand the current task before modifying code.
2. Inspect the repository before making changes.
3. Identify the minimum files required.
4. Implement only the current task.
5. Verify the implementation.
6. Report what changed.
7. Stop when the task is complete or when continuing would require a significant decision.

The agent must not proactively implement future milestones.

The agent should prefer simple solutions that can evolve later over speculative abstractions.

---

# 🛑 When To Stop

The agent should stop the current session when:

### 1. The current milestone is complete

Do not automatically continue to the next milestone.

### 2. The task becomes substantially larger than expected

If the current task reveals major dependencies or unrelated work, stop and create a handoff.

### 3. A significant architectural decision is required

Examples:

- Changing the database architecture.
- Replacing a technology.
- Introducing a major dependency.
- Changing the communication architecture.
- Redesigning an established data model.

The agent should explain the alternatives and request human input.

### 4. The same problem repeatedly fails

Do not continuously modify unrelated code in an attempt to make the problem disappear.

Document the problem and stop.

### 5. The context becomes unreliable

If the current session has accumulated enough context that continuing would risk inconsistent decisions or forgotten requirements, stop and create a handoff.

A clean new session is preferable to degraded reasoning.

---

# 🔄 Handoff System

Development sessions must support explicit handoffs.

A handoff is a compact, self-contained description of the current development state that allows a new session to continue without depending on the previous conversation.

When a handoff is required, create:

```text
docs/handoff.md
```

The handoff should contain:

```markdown
# Development Handoff

## Current Milestone

## Objective

## Status

Complete / Partially complete / Blocked

## What Was Implemented

## Files Created

## Files Modified

## Current Behavior

## Verification

## Problems / Limitations

## Decisions Made

## Open Questions

## Next Recommended Step

## Important Context For Next Session
```

The handoff must be:

- Concise.
- Factual.
- Self-contained.
- Focused on information needed by the next session.

Do not copy the conversation into the handoff.

Do not document irrelevant history.

---

# 🔁 Continuing From a Handoff

At the beginning of a new development session:

1. Read `README.md`.
2. Read `AGENTS.md`.
3. If `docs/handoff.md` exists, read it.
4. Inspect the repository state.
5. Verify that the handoff still describes the actual code.
6. Resolve discrepancies in favor of the actual repository state.
7. Continue only from the documented next step.

Do not blindly trust a previous handoff.

The repository is the source of truth.

---

# 🧑‍💻 Human Approval

The agent must request human confirmation before:

- Introducing a major dependency.
- Changing the architecture.
- Replacing a technology defined in this README.
- Removing existing functionality.
- Changing the database schema substantially.
- Making security-sensitive architectural decisions.
- Expanding the scope of a milestone.
- Proceeding when requirements are ambiguous.

The agent should prefer asking one precise question over making a significant assumption.

---

# 🧪 Verification

Every completed milestone must be verified according to its scope.

Possible verification methods include:

- Unit tests.
- Integration tests.
- Type checking.
- Linting.
- API testing.
- Manual UI testing.
- Local execution.
- Log inspection.

The agent must not claim that something works without verifying it.

If verification cannot be performed, the agent must explicitly state that.

---

# 📚 Documentation

Documentation should be created progressively.

Do not create large documentation files before they are needed.

Possible future documentation includes:

```text
docs/
├── handoff.md
├── architecture.md
├── decisions.md
└── ...
```

Only create these documents when they provide useful information for future development.

`docs/handoff.md` should be created whenever the current session needs to hand work to a future session.

---

# 🔒 Local-Only Requirement

The application must remain local at runtime.

Do not introduce:

- OpenAI APIs.
- Anthropic APIs.
- Google AI APIs.
- Cloud transcription APIs.
- Cloud TTS APIs.
- External vector databases.
- External authentication services.

Local libraries and locally hosted models are allowed.

Internet access may be used during development for installing dependencies and consulting documentation, but application runtime must not depend on external AI services.

---

# 🎓 Pedagogical Principles

The tutor should behave as an English teacher rather than simply as a conversational chatbot.

The system should be capable of:

- Correcting grammar.
- Explaining corrections.
- Suggesting more natural phrasing.
- Introducing vocabulary appropriate to the learner's level.
- Asking follow-up questions.
- Detecting recurring mistakes.
- Adjusting difficulty.
- Encouraging extended responses.
- Tracking recurring weaknesses.
- Revisiting previously learned vocabulary.
- Adapting activities to the learner's goals.
- Measuring progress over time.

Corrections should prioritize useful feedback over correcting every minor imperfection.

The learner should be encouraged to communicate naturally rather than being constantly interrupted.

The teaching strategy should adapt to the learner's current level and goals.

Different learners may require different teaching strategies, topics, difficulty, and progression.

---

# ⚠️ Constraints

The implementation agent must not:

- Implement the entire roadmap at once.
- Invent requirements.
- Assume undocumented features exist.
- Create unnecessary abstractions.
- Add dependencies without justification.
- Refactor unrelated code.
- Continue indefinitely after completing a milestone.
- Hide unresolved problems.
- Claim tests passed when they were not executed.
- Make major architectural decisions without human approval.
- Implement authentication merely because multiple learners exist.
- Create speculative database models solely because they appear in the conceptual architecture.

The objective is not to maximize the amount of code produced in a session.

The objective is to produce **small, correct, verifiable increments that can be safely continued by another session**.

---

# 🚀 Starting From This Repository

When the repository contains only:

```text
README.md
AGENTS.md
.gitignore
```

the agent should:

1. Read and understand `README.md`.
2. Read and understand `AGENTS.md`.
3. Inspect the repository.
4. Determine the smallest sensible first milestone.
5. Explain the proposed milestone.
6. Identify any decisions requiring human approval.
7. Wait for approval.
8. Implement only the approved milestone.
9. Verify the implementation.
10. Stop when the milestone is complete.
11. Create `docs/handoff.md` only if another session is expected to continue the work.

If the first milestone requires an architectural decision that is not specified here, stop and ask the human before implementing it.
