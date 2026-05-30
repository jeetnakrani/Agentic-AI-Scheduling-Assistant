# 🤖 Agentic AI Scheduling Assistant
### Intelligent, Autonomous Meeting Coordination Across Time Zones

> **Industry Project** — LTIMindtree × NMIMS MPSTME | Aug 2025 – Oct 2025  
> *Note: Source code is proprietary to LTIMindtree. This repository documents the system architecture, design decisions, and outcomes.*

---

## 📌 Problem Statement

In today's globalized work environment, teams are spread across multiple time zones, making meeting scheduling a constant operational bottleneck.

| Pain Point | Impact |
|---|---|
| Back-and-forth communication to finalise slots | ~15 min per meeting |
| Scheduling conflicts due to overlapping commitments | 2–3 follow-up exchanges |
| Manual rescheduling across time zones | Delays & frustration |
| No intelligent conflict negotiation in existing tools | Human intervention required every time |

**Existing tools** (Calendly, Google Calendar, MS Bookings) only provide static availability views — they cannot *reason*, *negotiate*, or *autonomously resolve conflicts*.

---

## 🎯 Solution

An **Agentic AI Scheduling Assistant** powered by a multi-agent LangChain architecture and Gemini API that:

- 🔍 Scans participant calendars to detect optimal meeting slots
- 🤝 Uses multi-agent negotiation to autonomously resolve scheduling conflicts
- 📧 Sends polite, context-aware reschedule requests when clashes occur
- ✅ Provides instant confirmations or alternative time suggestions
- 🌍 Handles time zone normalisation automatically

**Result: Scheduling effort reduced from ~15 min → ~2 min (87% reduction), eliminating 2–3 follow-up exchanges per meeting.**

---

## 🏗️ System Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                        User (Conversational UI)                   │
│                     "Schedule a meeting with Alex and Sam"        │
└─────────────────────────────┬────────────────────────────────────┘
                              │ HTTP / WebSocket
                              ▼
┌──────────────────────────────────────────────────────────────────┐
│                      FastAPI Service Layer                        │
│              (Request routing, session management)                │
└──────┬──────────────────────┬───────────────────────┬────────────┘
       │                      │                       │
       ▼                      ▼                       ▼
┌─────────────┐    ┌──────────────────┐    ┌─────────────────────┐
│  Coordinator│    │  Availability    │    │  Conflict Resolution│
│    Agent    │◄──►│    Agent         │◄──►│       Agent         │
│             │    │                  │    │                     │
│ - Intent    │    │ - Calendar scan  │    │ - Slot negotiation  │
│   parsing   │    │ - Timezone norm  │    │ - Reschedule        │
│ - Orchestr. │    │ - Free/busy API  │    │   request drafting  │
│ - Final     │    │ - Slot ranking   │    │ - Priority-based    │
│   confirm.  │    │                  │    │   resolution        │
└─────────────┘    └──────────────────┘    └─────────────────────┘
       │                      │                       │
       └──────────────────────┴───────────────────────┘
                              │
                              ▼
              ┌───────────────────────────────┐
              │         Gemini API            │
              │  (Reasoning / NL Generation)  │
              └───────────────┬───────────────┘
                              │
              ┌───────────────┴───────────────┐
              │      Google Calendar API      │
              │  (Read/Write calendar events) │
              └───────────────────────────────┘
```

---

## 🧠 Multi-Agent Design

### Agent 1 — Coordinator Agent
**Role:** Orchestrator and user-facing agent  
**Responsibilities:**
- Parse natural language scheduling requests
- Break down task into sub-goals and delegate to specialist agents
- Aggregate responses and generate final confirmation
- Maintain conversational context across turns

### Agent 2 — Availability Agent
**Role:** Calendar intelligence specialist  
**Responsibilities:**
- Query Google Calendar API for all participants' free/busy slots
- Normalise time zones using `pytz` / `zoneinfo`
- Rank candidate slots by participant preference scores
- Return structured availability windows to Coordinator

### Agent 3 — Conflict Resolution Agent
**Role:** Negotiation and rescheduling specialist  
**Responsibilities:**
- Detect scheduling conflicts (overlapping events, back-to-back meetings)
- Generate polite, context-aware reschedule request messages via Gemini
- Propose alternative slots with rationale
- Implement priority-based resolution (e.g., senior stakeholder takes precedence)

---

## ⚙️ Tech Stack

| Layer | Technology |
|---|---|
| **LLM / Reasoning** | Google Gemini API (gemini-1.5-pro) |
| **Agent Framework** | LangChain (Agents + Tools) |
| **Backend API** | FastAPI |
| **Calendar Integration** | Google Calendar API v3 |
| **Time Zone Handling** | `pytz`, `zoneinfo` |
| **Conversational UI** | Custom chatbot interface |
| **Language** | Python 3.11 |

---

## 📊 Key Results

| Metric | Before | After | Improvement |
|---|---|---|---|
| Time to schedule a meeting | ~15 min | ~2 min | **87% reduction** |
| Follow-up exchanges | 2–3 messages | 0 | **Eliminated** |
| Manual conflict resolution | Always required | Automated | **100% automated** |
| Time zone errors | Frequent | None | **Eliminated** |

---

## 🔄 Agent Interaction Flow

See [`demo_flow.md`](demo_flow.md) for a full step-by-step walkthrough of a real scheduling conversation, including how agents hand off tasks and resolve conflicts.

---

## 📐 Design Decisions

See [`system_design.md`](system_design.md) for detailed rationale behind:
- Why LangChain over raw API calls
- Why Gemini over GPT-4 for this use case
- Multi-agent vs single-agent trade-offs
- Conflict resolution priority algorithm
- FastAPI architecture choices

---

## 🚀 Execution Phases

| Phase | Description |
|---|---|
| 1. Requirement Understanding | Clarified use cases, success criteria with LTIMindtree mentors |
| 2. Data Gathering | Integrated Google Calendar API with sample enterprise calendar datasets |
| 3. Preprocessing & Integration | Time zone normalisation, availability slot parsing |
| 4. Agent Design | Built multi-agent negotiation logic with LangChain |
| 5. Conflict Resolution Module | Implemented reasoning + polite rescheduling via Gemini |
| 6. Architecture Setup | FastAPI backend + conversational UI frontend |
| 7. Testing & Validation | Simulated 50+ calendar conflict scenarios |
| 8. Final Integration & Demo | Polished assistant demoed to LTIMindtree enterprise team |

---

## 📚 Literature Survey

This project fills a gap identified in existing research:

| Paper | Key Finding |
|---|---|
| "Autonomous Agents for Multi-party Scheduling" — *Journal of Multiagent Systems, 2022* | Multi-agent negotiation is efficient in distributed scheduling |
| "Agent-based Conflict Resolution in Meeting Scheduling" — *IEEE Transactions on AI, 2021* | Rule-based conflict resolution has limitations in dynamic environments |
| "Large Language Models as Negotiators" — *arXiv, 2023* | LLMs enable human-like natural language rescheduling requests |

**Gap filled by this project:** No existing work fully integrates Calendar APIs + LLM-based agents + agent-to-agent negotiation for real-time conflict resolution.

---

## 🔮 Future Work

- **Outlook / Teams integration** — extend beyond Google Calendar
- **Preference learning** — remember user scheduling preferences over time
- **Meeting summarisation** — auto-generate agenda based on meeting context
- **Mobile push notifications** — proactive conflict alerts
- **RLHF fine-tuning** — improve negotiation quality from user feedback

---

## 👤 Author

**Jeet Nakrani**  
MTech Data Science & Business Analytics — NMIMS MPSTME (D010)  
Project Trainee @ LTIMindtree (Aug – Oct 2025)  
[LinkedIn](https://linkedin.com/in/your-profile) · [Portfolio](https://your-portfolio.com)

---

> *This repository is documentation-only. Source code is proprietary to LTIMindtree and cannot be shared publicly.*
