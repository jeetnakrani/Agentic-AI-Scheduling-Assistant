# Demo Flow — Agentic AI Scheduling Assistant

A step-by-step walkthrough of a real scheduling scenario, showing how the three agents collaborate to resolve a conflict and book a meeting.

---

## Scenario

**User:** Jeet (Mumbai, IST)  
**Participants:** Alex (London, GMT), Sam (New York, EST)  
**Request:** Schedule a 1-hour product sync — all three must attend

---

## Conversation Transcript

### Turn 1 — User Request

```
User:  "Schedule a 1-hour product sync with Alex and Sam 
        sometime this week."
```

**Coordinator Agent internally:**
```
Intent    : schedule_meeting
Duration  : 60 minutes
Participants: [alex@company.com, sam@company.com, jeet@company.com]
Timeframe : this week (Mon–Fri)
→ Delegate to Availability Agent
```

---

### Turn 2 — Availability Agent Scans Calendars

**Availability Agent queries Google Calendar API for all 3 participants (in parallel):**

```
Jeet  (IST, UTC+5:30): Free slots this week →
  - Tue 10:00–12:00 IST
  - Wed 14:00–17:00 IST
  - Thu 09:00–11:00 IST

Alex  (GMT, UTC+0):    Free slots this week →
  - Tue 14:30–16:00 GMT    ← conflict: team standup 14:00–14:30
  - Wed 09:00–11:00 GMT
  - Thu 10:00–13:00 GMT

Sam   (EST, UTC-5):    Free slots this week →
  - Tue 08:00–10:00 EST
  - Wed 07:00–09:00 EST    ← outside working hours (too early)
  - Thu 09:00–11:00 EST
```

**Time zone normalisation to UTC:**

```
Candidate slot analysis:

Thursday:
  Jeet: Thu 09:00–11:00 IST = 03:30–05:30 UTC   ← outside hours for Alex/Sam
  
Wednesday:
  Jeet: Wed 14:00–17:00 IST = 08:30–11:30 UTC
  Alex: Wed 09:00–11:00 GMT = 09:00–11:00 UTC   ✓ overlap: 09:00–11:00 UTC
  Sam:  Wed 07:00–09:00 EST = 12:00–14:00 UTC   ✗ no overlap

Tuesday:
  Jeet: Tue 10:00–12:00 IST = 04:30–06:30 UTC   ← too early for Alex/Sam

→ No perfect slot found. Conflict detected.
→ Escalate to Conflict Resolution Agent.
```

---

### Turn 3 — Conflict Resolution Agent

**Conflict Agent reasoning:**
```
Closest overlap candidate: Wednesday
  - Alex available 09:00–11:00 UTC (Wed)
  - Sam available 12:00–14:00 UTC (Wed)
  - Gap between them: 11:00–12:00 UTC (60 min gap)

Option A: Ask Sam to move his Wed morning block earlier by 2h
Option B: Ask Alex to extend availability to 12:00 UTC
Option C: Find a Friday slot (not yet checked)

→ Checking Friday availability...
  Jeet: Fri 10:00–13:00 IST = 04:30–07:30 UTC  
  Alex: Fri 12:00–15:00 GMT = 12:00–15:00 UTC  ✓
  Sam:  Fri 10:00–13:00 EST = 15:00–18:00 UTC  ✗ no overlap

→ Best option: Request Sam to shift Wednesday slot earlier.
→ Draft polite reschedule request via Gemini.
```

**Gemini-generated reschedule message:**
```
Hi Sam,

Hope you're doing well! I'm coordinating a product sync with 
Jeet (Mumbai) and Alex (London) this Wednesday. 

The earliest slot that works for all three time zones is 
12:00 PM UTC (7:00 AM EST). I know that's a bit early — 
would you be open to joining at that time? The meeting 
should be wrapped up within an hour.

If that doesn't work, I can also explore Thursday options.

Thanks so much!
```

---

### Turn 4 — Resolution & Confirmation

```
[Simulated] Sam responds: "7am works for me on Wednesday."

Coordinator Agent:
  → Final slot: Wednesday, 12:00–13:00 UTC
  → Jeet: Wed 17:30–18:30 IST
  → Alex: Wed 12:00–13:00 GMT
  → Sam:  Wed 07:00–08:00 EST
  → Writing event to Google Calendar for all participants...
  → Sending calendar invites...
```

**Response to user:**

```
Assistant: "Done! I've scheduled the product sync for:

  📅 Wednesday
  🕐 Jeet (you):  5:30 PM – 6:30 PM IST
     Alex:        12:00 PM – 1:00 PM GMT
     Sam:         7:00 AM – 8:00 AM EST

  Sam had a conflict but agreed to an earlier slot. 
  Calendar invites have been sent to all participants. ✅"
```

---

## Timing Breakdown

| Step | Time Taken |
|---|---|
| Intent parsing (Coordinator) | ~0.3s |
| Parallel calendar queries (3 participants) | ~1.2s |
| Time zone normalisation + slot ranking | ~0.1s |
| Conflict detection + resolution reasoning | ~0.8s |
| Reschedule message generation (Gemini) | ~1.1s |
| Calendar write + invite send | ~0.6s |
| **Total** | **~4.1s** |

vs. **~15 minutes** of manual back-and-forth via email/Slack.

---

## Edge Cases Handled

| Scenario | Agent Response |
|---|---|
| All participants unavailable for 7 days | Inform user; ask to expand window to 14 days |
| Participant calendar not connected | Proceed with available participants; flag missing availability |
| Recurring meeting in conflict slot | Higher priority score → protected from reschedule requests |
| All-day events blocking the day | Treat as "away" only if marked busy; check free/busy flag |
| Participant in DST transition week | UTC-based comparison eliminates DST errors entirely |
| Reschedule request declined twice | Escalate to user — "Sam is unavailable this week, would you like to try next week?" |
