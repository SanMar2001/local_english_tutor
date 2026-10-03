# English Tutor Local AI

> A personal English tutor running 100% locally, with no external AI APIs or monthly costs.

## 📋 Description

English Tutor Local AI is a local conversational English tutor designed to help a learner improve their English through written and spoken interaction.

The application runs entirely on the user's machine and combines:

- Real-time audio communication through WebRTC.
- Local speech-to-text transcription with Whisper.cpp.
- A local LLM for pedagogical reasoning and conversation.
- Persistent learning data.
- Semantic memory through a local vector database.
- A React frontend.
- A FastAPI backend.

The system must remain **fully local at runtime**. No external AI APIs or paid cloud services should be required.

---

# 🎯 Learning Objective

The tutor must not assume a predefined English level or learning path.

Instead, the system should first evaluate the learner's current proficiency and then build a personalized learning plan based on the results.

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

## Learning Goal

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

---

## Personalized Learning Plan

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

---

## Progress Reassessment

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

---

## Long-Term Objective

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

The learning strategy should evolve according to evidence collected from the learner's interactions.

---

# 🛠 Stack

| Layer                  | Technology                      |
| ---------------------- | ------------------------------- |
| Backend                | Python 3.11+ + FastAPI          |
| Frontend               | React + Vite + TypeScript       |
| Audio / Communications | WebRTC + PeerJS                 |
| Speech-to-Text         | Whisper.cpp + GGUF              |
| LLM                    | Ollama                          |
| Vector Database        | ChromaDB                        |
| Database               | SQLite + SQLAlchemy             |
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

```text
┌─────────────┐
│   Browser   │
│ React/Vite  │
└──────┬──────┘
       │
       │ WebRTC / HTTP
       ▼
┌──────────────────────────────┐
│        FastAPI Server        │
│                              │
│  API / Audio / Conversation  │
└──────┬──────────┬────────────┘
       │          │
       │          │
       ▼          ▼
┌───────────┐  ┌───────────────┐
│Whisper.cpp│  │    Ollama     │
│   STT     │  │     LLM       │
└───────────┘  └───────┬───────┘
                       │
                       ▼
               ┌───────────────┐
               │  Memory Layer │
               └───────┬───────┘
                       │
                ┌──────┴──────┐
                ▼             ▼
          ┌──────────┐   ┌──────────┐
          │  SQLite  │   │ ChromaDB │
          │ Progress │   │ Semantic │
          │  Data    │   │  Memory  │
          └──────────┘   └──────────┘
```

---

# 📁 Intended Project Structure

The repository starts with only the project documentation and configuration files.

The intended application structure is:

```text
english-tutor/

├── server/
│   ├── main.py
│   ├── audio/
│   │   ├── __init__.py
│   │   ├── transcriber.py
│   │   ├── synthesizer.py
│   │   └── pipeline.py
│   ├── memory/
│   │   ├── __init__.py
│   │   ├── database.py
│   │   ├── vector_store.py
│   │   └── schemas.py
│   ├── llm/
│   │   ├── __init__.py
│   │   └── client.py
│   └── routes/
│       ├── conversation.py
│       └── progress.py
│
├── client/
│   ├── src/
│   │   ├── App.tsx
│   │   ├── components/
│   │   └── hooks/
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

This structure is an architectural target, not a requirement to create every file immediately.

Do not create empty directories or placeholder files merely to match this structure.

Create components only when required by the current implementation milestone.

---

# 🧠 Development Philosophy

Development must happen incrementally through small, independently understandable milestones.

Examples:

```text
Initialize FastAPI project
        ↓
Configure SQLite
        ↓
Create database models
        ↓
Implement Ollama client
        ↓
Implement text conversation
        ↓
Implement learning assessment
        ↓
Implement learning plan
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
2. If `docs/handoff.md` exists, read it.
3. Inspect the repository state.
4. Verify that the handoff still describes the actual code.
5. Resolve discrepancies in favor of the actual repository state.
6. Continue only from the documented next step.

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

The objective is not to maximize the amount of code produced in a session.

The objective is to produce **small, correct, verifiable increments that can be safely continued by another session**.

---

# 🚀 Starting From This Repository

When the repository contains only this README and the initial project configuration:

1. Read and understand this README.
2. Inspect the repository.
3. Determine the smallest sensible first milestone.
4. Explain the proposed milestone.
5. Implement only that milestone.
6. Verify the implementation.
7. Stop when the milestone is complete.
8. Create `docs/handoff.md` if another session is expected to continue the work.

If the first milestone requires an architectural decision that is not specified here, stop and ask the human before implementing it.
