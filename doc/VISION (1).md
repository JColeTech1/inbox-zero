# Sortie — product vision

> An AI-powered email management assistant that lives inside the tools people already use.
> No new apps. No new logins. Just smarter email.

---

## What it is

Sortie is a locally-run AI assistant that connects to your email, reads and classifies everything coming in, takes smart actions based on your preferences, and keeps you informed through Teams messages and digest emails.

You never log into Sortie. You talk to it in Teams. It emails you summaries. It works in the background.

---

## Who it's for

- Busy professionals drowning in email
- MSP clients (SecureNet) who need email triage without a new tool
- Anyone on Microsoft 365 or Google Workspace
- Global teams across different timezones

---

## The two interfaces

**Outlook / Gmail** — Sortie sends you things
- Morning digest at your sign-on time
- Urgent alert when something needs an immediate response
- Audit summary of what it did and why

**Microsoft Teams** — you talk to Sortie
- "What did you do today?"
- "Add Sarah to my VIPs"
- "My sign-on time is 8am EST"
- "Show me what's urgent right now"
- "Flag anything from Microsoft security as low priority"

That's it. No dashboard. No new app. No training required.

---

## What Sortie does

### Prioritize
- Detects VIP senders and pins their emails to the top
- Flags time-sensitive emails (urgency language detection)
- Routes everything into A / B / C folders automatically:
  - **A — Urgent / VIP**: needs response today
  - **B — Action needed**: respond this week
  - **C — FYI / archive**: no response needed

### Triage
- Deletes emails where user is CC-only and another handler already has it
- Routes emails to the right person based on sender/subject patterns
- Ignores marketing and low-priority noise automatically

### Alert
- Sends a Teams message when something is genuinely urgent
- Scores Microsoft Entra ID security digests — only alerts if there's real risk
- Morning digest email at your local sign-on time every day

### Remember
- Learns your preferences per person (VIP list, timezone, sign-on time, categories)
- Stores preferences locally in a simple config file — no shared database
- You update preferences by just telling the Teams bot in plain English

---

## How it's built

### Architecture
One backend, two frontends (that already exist).

```
Gmail / Outlook 365
       ↓
  Sortie backend
  ├── AI classifier (Ollama local or Claude/GPT)
  ├── Rules engine (per-client config)
  ├── Action executor (flag, move, delete, alert)
  └── Audit logger
       ↓
Teams bot          Digest email
(you talk to it)   (it emails you)
```

### Tech stack
- Base: Inbox Zero (open source, MIT license)
- Outlook connector: Outlook Assistant MCP (open source, MIT license)
- Windows service wrapper: doobidoo/outlook-ai-agent (open source, MIT license)
- Local AI: Ollama (free, runs on your machine)
- Cloud AI option: Claude or GPT (swappable)
- Email APIs: Gmail API + Microsoft Graph API
- Bot: Azure Bot Service + Teams channel
- Language: TypeScript / Python
- Config: local JSON files per user

### Privacy
- Runs locally on the client's machine
- Ollama means emails never leave the building
- No shared database — each user's prefs are their own
- Audit log stays local
- Cloud AI is optional, off by default

---

## Per-client configuration

Each user gets a `user_prefs.json` file:

```json
{
  "name": "Manny",
  "timezone": "America/New_York",
  "sign_on_time": "09:00",
  "vip_senders": ["client@bigco.com", "boss@securenet.com"],
  "helpdesk_addresses": ["helpdesk@securenet.com"],
  "categories": {
    "cybersecurity": ["Microsoft Entra", "security alert", "sign-in risk"],
    "urgent_keywords": ["are you available", "asap", "urgent", "today only"]
  },
  "delete_rules": {
    "cc_only_if_helpdesk_present": true
  },
  "ai_context": "Manny is a Global Admin on Microsoft 365. He receives Microsoft Entra ID protection emails. Analyze these and only flag if risk is non-zero."
}
```

This file is updated automatically when Manny tells the Teams bot his preferences.

---

## Build phases

| Phase | What gets built | Timeline |
|---|---|---|
| 1 | Fork + connect Gmail + connect MDC Outlook + local Ollama | This week |
| 2 | Folder schema + digest email + CC logic + user prefs | Week 2–3 |
| 3 | Teams bot + urgent alerts + Entra scoring | Month 2 |
| 4 | Hand off to Alex + connect SecureNet tenant + demo to Manny | Month 2–3 |
| 5 | MSP product packaging + multi-client onboarding | Future |

---

## What makes Sortie different

- Lives inside tools people already use — zero adoption friction
- Runs locally — privacy-safe for MSP clients
- Per-client AI context — knows who Manny is, who Gilda is, what matters
- Starts read-only, earns trust, then takes action
- Built on open source — no licensing costs, fully customizable
- Designed to be sold as an MSP product by SecureNet
