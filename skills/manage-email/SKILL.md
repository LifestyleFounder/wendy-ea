---
name: manage-email
description: Inbox triage — scan recent emails, categorize into actionable buckets, apply Gmail labels, draft replies for important threads, and archive noise. Personalized to YOUR business via CLAUDE.md.
---

# Manage Email

On-demand inbox triage. Use anytime you want to check email or clear the queue (the `/wendy-ea:morning` skill calls this internally as Module 1).

---

## Config (read from CLAUDE.md)

| Field | Used For |
|-------|----------|
| `USER_FIRST_NAME` / `USER_FULL_NAME` | Email signatures in drafts |
| `Email Label System` | Categorization buckets |
| `KEY_PEOPLE` | Recognize team threads vs. external |
| `Brand Voice` | Tone for draft replies |
| `Connected Tools` | Whether Gmail MCP is set up |

**If `CLAUDE.md` doesn't exist, stop and prompt the user to run `/wendy-ea:onboard`.**
**If Gmail MCP isn't connected, give them setup instructions and stop.**

---

## Default Label System

The `/wendy-ea:onboard` skill creates these labels in Gmail (or maps to existing ones if found):

| Label | What Goes Here |
|-------|---------------|
| **Priority** | Urgent — needs attention today |
| **Clients** | Active client communication |
| **Leads** | Inbound interest, applications, prospects |
| **Team** | Internal team threads |
| **Calendar** | Meeting invites and confirmations |
| **Finance** | Invoices, receipts, payments, billing |
| **Tech & Tools** | Platform alerts, tool updates, system notifications |
| **Newsletters** | Reading material, industry updates |
| **Collabs** | Partnership inquiries, podcast invites |
| **Noise** | Digests, spam, low-value notifications — archive without reading |

Look up the user's actual label IDs at runtime with `list_email_labels`.

---

## Process

### Step 1: Scan Recent Emails

Search Gmail: `newer_than:2d in:inbox` — pull last 48 hours.

Use `search_emails` with `maxResults: 100`. If 100 results return, run a second pass for older items.

### Step 2: Read and Categorize

Read full content of important-looking emails using `read_email`. For obvious noise (recognized newsletter senders, community digests), the subject line is enough.

**Categorize every email into one bucket:**

#### Priority (Action Needed Today)
- New customer signups / revenue events
- Platform errors (broken automations, alerts, billing failures)
- Time-sensitive client requests
- Meeting confirmations for today

#### Clients
- DMs and emails from active paying clients
- Client questions, updates, project discussions

#### Leads
- People expressing interest in working with the user
- Application-related emails
- Prospects asking about offers/pricing

#### Team
- Threads with team members listed in CLAUDE.md `KEY_PEOPLE`
- Internal coordination

#### Calendar
- Meeting invitations and confirmations
- Booking confirmations (Calendly, Cal.com, etc.)
- Zoom/Meet links and reschedules

#### Collabs
- Partnership inquiries
- Podcast guest invitations
- Sponsorship/paid promotion offers

#### Finance
- Invoices, receipts, payments
- Billing issues, payment failures, refunds

#### Tech & Tools
- Platform notifications (errors, status changes)
- Tool updates, version changes
- Domain/DNS/security alerts

#### Newsletters
- Marketing newsletters
- Industry updates, course content

#### Noise (Archive)
- Community post notifications (not DMs)
- Weekly digests
- Cold outreach / sales pitches
- Duplicate emails from same sender
- Simple "thanks" replies that need no action

### Step 3: Present the Summary

Output a structured report:

```
## Email Report — [Date]

### Priority — Action Needed ([count])
For each:
- **From:** [Name] — [Subject]
- **Why it matters:** [1 sentence]
- **Action:** [what to do]

### Clients ([count])
For each:
- **From:** [Name] — [Subject]
- **Summary:** [1 sentence]
- **Action:** [reply / no action]

### Leads ([count])
For each:
- **From:** [Name] — [Subject]
- **Signal:** [hot / warm / cold]
- **Suggested next step:** [reply / book call / nurture]

### Calendar ([count])
For each:
- **What:** [Meeting name]
- **When:** [Date/time, in user's timezone]
- **With:** [Person]
- **Link:** [meeting URL]

### Collabs ([count])
For each:
- **From:** [Name] — [Brand/Company]
- **Ask:** [what they want]
- **Draft reply:** Ready / Needs your review

### Awareness ([count])
- [Sender — one-line summary] (no action needed)

### Noise ([count])
- [count] notifications, digests, etc — archived

### Stats
- Total emails scanned: X
- Priority: X | Clients: X | Leads: X | Calendar: X | Collabs: X | Noise: X
```

### Step 4: Draft Replies (where applicable)

For Priority, Client, Lead, and Collab emails, draft a reply in the user's voice (pull tone from CLAUDE.md `Brand Voice`).

Use `draft_email` to create each draft. **Always present each draft for review before sending.** Never auto-send.

For Collab emails, default reply pattern (adapt to voice):

```
Hey [First Name],

Thanks for reaching out — appreciate the interest.

[Personalized response based on their ask]

[Sign-off in user's voice]
[USER_FIRST_NAME]
```

If the user has a brand manager or partnership contact in `KEY_PEOPLE`, CC them on collab replies.

### Step 5: Apply Labels and Archive Noise

Use `batch_modify_emails` to label and clean up:

1. **Priority** → add `Priority` label + `STARRED`
2. **Clients** → add `Clients` label
3. **Leads** → add `Leads` label
4. **Calendar** → add `Calendar` label
5. **Team** → add `Team` label
6. **Finance** → add `Finance` label
7. **Tech & Tools** → add `Tech & Tools` label
8. **Newsletters** → add `Newsletters` label, remove `UNREAD`
9. **Collabs** → add `Collabs` label
10. **Noise** → add `Noise` label, remove `UNREAD`

**Mark as read:** All Noise, Newsletters, awareness-only emails.

**Leave unread:** Priority, Clients needing reply, hot Leads, Collabs awaiting draft review.

**Batch efficiently** — group emails by label operation and run parallel `batch_modify_emails` calls.

Present the count before executing: *"Ready to label X emails and mark Y as read. Proceed?"*

---

## Edge Cases

- **Existing threads** — Label only the most recent message in a thread.
- **Ambiguous emails** — Flag with `?` in the summary and let the user decide.
- **No priority items** — Still present the full breakdown. "Clean inbox" is good news.
- **Duplicate threads** — Group by thread, don't list each reply separately.
- **Simple thank-you DMs** — These are noise, not client communication. Mark as read.
- **First-run** — If the user's label system doesn't exist yet, create the 10 default labels with `create_label` before triaging.

---

## Tools Used

| Operation | MCP Tool |
|-----------|----------|
| Search emails | `search_emails` |
| Read email content | `read_email` |
| List labels | `list_email_labels` |
| Create label (first run) | `create_label` |
| Apply/remove labels | `batch_modify_emails` |
| Star emails | `batch_modify_emails` (add `STARRED`) |
| Mark as read | `batch_modify_emails` (remove `UNREAD`) |
| Create reply drafts | `draft_email` |

---

## CRITICAL RULES

1. **Read `CLAUDE.md` first** to load the user's voice, label system, and key people.
2. **Actually read priority/client/lead emails** with `read_email`. Don't guess from subject lines.
3. **Apply labels in Gmail** — don't just report categories.
4. **Draft, never send.** Always present drafts for approval.
5. **Use the user's voice** (from `Brand Voice` in CLAUDE.md), not generic corporate-speak.
6. **Mark noise as read** to clear the queue.
7. **Skip empty categories** in the report.
