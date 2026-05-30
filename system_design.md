# System Design — Agentic AI Scheduling Assistant

## 1. Design Goals

| Goal | Rationale |
|---|---|
| **Minimal latency** | Scheduling interactions must feel real-time (<3s response) |
| **Fault tolerance** | Calendar API failures should not crash the conversation |
| **Extensibility** | New calendar providers (Outlook, iCal) should be addable without refactoring |
| **Auditability** | Agent reasoning steps must be logged for enterprise compliance |

---

## 2. Why Multi-Agent over Single-Agent?

A single monolithic LLM agent handling all tasks would:
- Have a bloated prompt context → higher token cost, slower inference
- Mix concerns (calendar parsing + conflict reasoning + message drafting)
- Be harder to debug (which step failed?)

**Multi-agent separation of concerns:**

```
Coordinator  →  single responsibility: orchestration + user communication
Availability →  single responsibility: calendar data + timezone logic
Conflict     →  single responsibility: reasoning + message generation
```

Each agent has a focused system prompt, smaller context window, and can be optimised independently.

---

## 3. Why LangChain?

| Alternative | Reason Not Chosen |
|---|---|
| Raw Gemini API calls | No built-in tool use, memory, or agent loop management |
| AutoGen | Heavier framework, less control over agent hand-off logic |
| CrewAI | Less mature at project start (Aug 2025), limited tool integration |

**LangChain advantages used:**
- `AgentExecutor` with custom tool definitions for Calendar API calls
- `ConversationBufferMemory` for maintaining multi-turn context
- Structured output parsing with `PydanticOutputParser`
- LangSmith tracing for debugging agent reasoning chains

---

## 4. Why Gemini over GPT-4?

| Factor | Gemini 1.5 Pro | GPT-4o |
|---|---|---|
| Context window | 1M tokens | 128K tokens |
| Cost | Lower at volume | Higher |
| Google API integration | Native | Requires bridging |
| JSON output reliability | Strong | Strong |

Gemini's 1M token context window is advantageous for passing full calendar datasets without chunking.

---

## 5. FastAPI Architecture

```
POST /schedule          ← Primary endpoint (new scheduling request)
POST /reschedule        ← Conflict-triggered reschedule
GET  /availability      ← Query free/busy for participants
POST /confirm           ← Final slot confirmation → write to Calendar API
WS   /chat              ← WebSocket for real-time conversational UI
```

**Session management:** Each conversation session gets a unique `session_id`. Agent memory and tool call history are tied to the session, enabling multi-turn context without re-sending full history.

---

## 6. Conflict Resolution Algorithm

```
Priority Score = f(seniority_weight, meeting_type, notice_period, recurrence_count)

IF conflict detected:
  1. Score all conflicting participants' events by Priority Score
  2. Lowest-score event → candidate for reschedule
  3. Availability Agent → find next 3 optimal alternative slots
  4. Conflict Agent → draft reschedule request (via Gemini) with:
       - Reason for reschedule
       - Proposed alternatives with time zone awareness
       - Apology framing calibrated to seniority delta
  5. Coordinator Agent → send request + await response
  6. IF accepted → write new event to Calendar API
     IF rejected → recurse with next candidate slot
```

---

## 7. Time Zone Handling

All internal slot representations use **UTC timestamps**. Conversion to local time happens only at:
- Input parsing (user's stated time → UTC)
- Output rendering (UTC → participant's local time)

```python
# Pseudocode — not production code
def normalise_to_utc(time_str: str, timezone: str) -> datetime:
    local_dt = datetime.fromisoformat(time_str)
    tz = pytz.timezone(timezone)
    return tz.localize(local_dt).astimezone(pytz.utc)
```

This eliminates DST-related bugs and ensures conflict detection works correctly across regions.

---

## 8. Error Handling Strategy

| Error Type | Handling |
|---|---|
| Google Calendar API rate limit | Exponential backoff with jitter (max 3 retries) |
| Gemini API timeout | Fallback to simplified slot suggestion without NL message |
| No available slot in 7-day window | Inform user; expand to 14-day window with user consent |
| Participant calendar not accessible | Mark as "unknown availability"; proceed with available participants |

---

## 9. Scalability Considerations

- **Stateless FastAPI workers** — session state held in Redis (in production design)
- **Async throughout** — all Calendar API and LLM calls use `asyncio` / `httpx`
- **Agent parallelism** — Availability Agent queries for all participants concurrently (not sequentially)

```
Sequential:   P1_query → P2_query → P3_query  →  ~3× latency
Parallel:     P1_query ┐
              P2_query ├→ merge results         →  1× latency
              P3_query ┘
```

---

## 10. Testing Strategy

| Test Type | Coverage |
|---|---|
| Unit | Timezone normalisation, conflict detection logic, slot ranking |
| Integration | Google Calendar API mock (50+ simulated conflict scenarios) |
| End-to-end | Full conversation flow from NL input → Calendar write |
| Edge cases | No availability in window, all-day events, recurring meetings, cross-DST conflicts |
