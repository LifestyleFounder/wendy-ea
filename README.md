# Wendy EA — Your AI Executive Assistant

> **Came from the workshop? You're in the right place.**
> Run these three commands in Claude Code and Wendy is yours:
>
> ```
> /plugin marketplace add LifestyleFounder/wendy-ea
> /plugin install wendy-ea
> /wendy-ea:onboard
> ```
>
> She'll interview you for ~8 minutes — then she knows your business, your voice, your schedule, and your tools.
> No more generic AI. Your assistant, fully customized.

---

## What is Wendy?

A Claude Code plugin that gives you a personalized executive assistant in 10 minutes.

Wendy reads your inbox, runs your morning briefing, summarizes anything, and drafts replies in your voice. She knows your business, your audience, your schedule, and your tools — because the first thing she does is interview you about all of it.

Built for coaches, consultants, and solo founders who want an EA without hiring one.

---

## What's inside

| Skill | Command | What it does |
|-------|---------|-------------|
| **Onboard** | `/wendy-ea:onboard` | Interview you about your business, build a personalized CLAUDE.md |
| **Morning Briefing** | `/wendy-ea:morning` | Email triage, calendar, news in your niche, today's top 3 priorities |
| **Manage Email** | `/wendy-ea:manage-email` | Inbox triage, draft replies in your voice, label and archive |
| **Summarize** | `/wendy-ea:summarize` | Summarize any URL, article, video, or pasted text |

---

## Install (60 seconds)

In Claude Code:

```
/plugin marketplace add LifestyleFounder/wendy-ea
/plugin install wendy-ea
```

Then restart Claude Code or run `/reload-plugins`.

---

## Quick Start (10 minutes)

### Step 1 — Onboard Wendy

```
/wendy-ea:onboard
```

She'll interview you for ~8 minutes about your business, audience, voice, schedule, and tools. At the end, she saves a personalized `CLAUDE.md` to your current folder. Every other skill reads from this file.

### Step 2 — Try her out

```
/wendy-ea:summarize
```

Easiest skill to test — works with no setup. Paste any URL or text and get a clean summary.

### Step 3 — Set up Gmail + Calendar (for the full experience)

`/wendy-ea:morning` and `/wendy-ea:manage-email` need access to your Gmail and Google Calendar via MCP servers. See the **MCP Setup** section below.

---

## MCP Setup

Wendy uses Claude Code's MCP (Model Context Protocol) servers to actually read your inbox and calendar. You only need this for `morning` and `manage-email` — `summarize` and `onboard` work without any setup.

### Gmail (required for /morning, /manage-email)

The simplest option is the official Google Workspace MCP:

1. Visit https://developers.google.com/gmail/api/quickstart and create OAuth credentials
2. Add the MCP to your `~/.claude/settings.json`:

```json
{
  "mcpServers": {
    "gmail": {
      "command": "npx",
      "args": ["-y", "@gongrzhe/server-gmail-autoauth-mcp"]
    }
  }
}
```

3. Restart Claude Code. First run will open a browser for Google OAuth.

### Google Calendar (optional, for /morning)

Same pattern — add a Calendar MCP to settings.json. Wendy detects whether it's connected and skips the calendar module if not.

---

## How it works

```
~/wendy-ea-plugin/                       (the plugin, installed by Claude Code)
├── .claude-plugin/plugin.json
├── skills/
│   ├── onboard/SKILL.md                 (the interview)
│   ├── morning/SKILL.md                 (daily briefing)
│   ├── manage-email/SKILL.md            (inbox triage)
│   └── summarize/SKILL.md               (content summarizer)
└── templates/
    └── CLAUDE.md.template               (filled in by /onboard)

your-folder/                             (wherever you run Wendy)
└── CLAUDE.md                            (your personalized config — generated)
```

When you run any Wendy skill, she reads `CLAUDE.md` from your current folder so every action is personalized to YOUR business. Re-run `/wendy-ea:onboard` anytime to update.

---

## Re-onboarding

Things change — new offer, new goal, new team member. Just run:

```
/wendy-ea:onboard
```

It overwrites your `CLAUDE.md` with the new answers.

---

## Privacy

Wendy runs entirely inside your Claude Code session. Your business info lives only in `CLAUDE.md` on your machine. Nothing is sent to Lifestyle Founders or any third party. The plugin has no telemetry.

---

## Troubleshooting

**"Skill not found"** — Run `/reload-plugins` after install. If still missing, run `/plugin list` to confirm wendy-ea is installed.

**"`/morning` says CLAUDE.md not found"** — You haven't run `/wendy-ea:onboard` yet, or you're running from a different folder than where you onboarded. Either re-run onboard from this folder, or move CLAUDE.md to `~/.claude/CLAUDE.md` for global access.

**"Gmail MCP errors"** — Check your OAuth credentials are valid and that you've completed the browser auth flow. Run `npx @gongrzhe/server-gmail-autoauth-mcp` directly to debug.

**"Empty news module"** — Wendy uses WebSearch which is built into Claude Code. If you're hitting rate limits, wait an hour and retry.

---

## Built by

Wendy is a free template from **[Lifestyle Founders Group](https://lifestylefounder.com)** — Dan Harrison's coaching company for established experts who want to build profitable, lean businesses with AI.

Originally built as Dan's personal EA. Generalized so any coach or founder can install it in 10 minutes.

If you want the full system Dan uses (5 AI agents — EA, Media Buyer, Social Media Director, Video Editor, Tech Officer), check out the [Lifestyle Founders Skool community](https://skool.com/lfg).

---

## License

MIT — fork it, ship it, make it your own.
