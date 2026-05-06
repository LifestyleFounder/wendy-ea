---
name: morning
description: Daily morning briefing — email triage, calendar overview, news in your niche, and the top 3 strategic priorities for the day. Personalized to YOUR business via CLAUDE.md. Run this first thing every day.
---

# Morning Briefing

Your daily command center. Runs 4 modules and presents a single unified briefing in the terminal.

---

## Config (read from CLAUDE.md)

This skill assumes the user has run `/wendy-ea:onboard` and a personalized `CLAUDE.md` exists. Read these from it:

| Field | Used For |
|-------|----------|
| `USER_FIRST_NAME` | Personalize the greeting |
| `TIMEZONE` | Calendar conversions |
| `INDUSTRY` | News module — what to search for |
| `CURRENT_GOAL` | Strategic Focus — tie priorities back to this |
| `Email Label System` | Categorization for Module 1 |
| `KEY_PEOPLE` | Recognize team threads |

**If `CLAUDE.md` doesn't exist, stop and tell the user to run `/wendy-ea:onboard` first.**

---

## Process

### Module 1: Email Triage

**Read and categorize every unread email into the user's label system.**

1. Search Gmail: `is:unread in:inbox newer_than:2d`
2. **Read the full content** of every email using `read_email`
3. Categorize into the label system from CLAUDE.md (default 10 labels: Priority, Clients, Leads, Team, Calendar, Finance, Tech & Tools, Newsletters, Collabs, Noise)
4. **Apply the label** in Gmail using `batch_modify_emails` (look up label IDs with `list_email_labels` if needed)
5. **Draft replies** for Priority, Clients, Leads, and Collab emails — present each draft for the user to approve before sending
6. **Mark Noise as read** to clear the inbox

**Output for briefing:**

```
## Email Triage

### Priority ([count])
- **[From]** — [Subject] — [why it matters] — Draft ready

### Clients ([count])
- **[Name]** — [Subject] — [action needed]

### Leads ([count])
- **[Name]** — [Subject] — [signal: hot/warm/cold] — Draft ready

### Calendar ([count])
- **[Subject]** — [date/time]

### Other
- Team: [count] | Tech: [count] | Finance: [count] | Newsletters: [count] | Noise: [count, archived]
```

**Only show categories that have emails.** Skip empty categories.

**CRITICAL:** Do NOT use `-category:` Gmail filters — they hide payment and notification emails. Always use `is:unread in:inbox`.

---

### Module 2: Calendar

**Pull today and tomorrow from Google Calendar.**

If Google Calendar MCP is connected:
1. List events for today (in the user's timezone from CLAUDE.md)
2. List events for tomorrow (so they can prep)
3. For each meeting, surface: who, what, when, prep notes if any

**Output for briefing:**

```
## Calendar

### Today ([day, date])
- **[time]** — [meeting] — with [person]
- ...

### Tomorrow ([day, date]) — heads up
- **[time]** — [meeting] — with [person]
```

If no Google Calendar MCP, skip this module silently (don't show empty section).

---

### Module 3: News & Trends

**Check for relevant news in the user's industry from the last 24-48 hours.**

Use the `INDUSTRY` field from CLAUDE.md to determine search topics. Example searches:
- "[industry] news this week"
- "[industry] tools updates"
- AI / Claude / automation news (relevant to most knowledge workers)

For EACH item:
```
### [Headline]
**What:** [2-3 sentences]
**Why it matters:** [How this affects their workflow or business]
**Content angle:** [Optional — one specific post/video idea]
```

Only include 3-5 items max. Skip if nothing relevant. Don't pad.

---

### Module 4: Strategic Focus

**Surface the 3 most important things to focus on today.**

Pull context from:
1. `CURRENT_GOAL` from CLAUDE.md
2. Module 1 — urgent emails requiring action
3. Module 2 — today's meetings
4. The user's stated priorities

**Output for briefing:**

```
## Today's Strategic Focus

### 1. [Most important thing]
**Why:** [ties to {CURRENT_GOAL}]
**Action:** [specific next step]

### 2. [Second priority]
**Why:** [context]
**Action:** [specific next step]

### 3. [Third priority]
**Why:** [context]
**Action:** [specific next step]
```

These should be **strategic, not tactical**. Not "reply to 5 emails" but "Close the warm lead from Tuesday — they hit the ICP perfectly and align with the $30K MRR goal."

---

## Final Output

Combine all modules into one clean briefing:

```
# Morning Briefing — [Date] ([Day of Week])

Good morning, [USER_FIRST_NAME]. Here's your day.

## Email Triage
[Module 1 output]

## Calendar
[Module 2 output, or skip if no MCP]

## News & Trends
[Module 3 output]

## Today's Strategic Focus
[Module 4 output]

---
Next actions:
- /wendy-ea:manage-email — Re-run email triage on new mail
- /wendy-ea:summarize — Summarize any link or article
```

---

## CRITICAL RULES

1. **Read `CLAUDE.md` first.** Personalize everything to this specific user.
2. **Actually read every email.** Use `read_email` on each one. Don't scan subjects.
3. **Apply labels in Gmail.** Don't just report — actually organize.
4. **Draft replies for action-required emails.** Present for approval before sending.
5. **Mark Noise as read.** Clear the inbox of digest junk.
6. **Skip empty modules.** No filler, no padding.
7. **If a module fails, note the error and continue.** One broken integration shouldn't kill the whole briefing.
8. **Strategic focus = tied to goals.** Not a to-do list. What moves the needle today.
