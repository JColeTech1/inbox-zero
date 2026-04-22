# Sortie — skill reference

Sortie is built ON TOP of Inbox Zero — not inside it. The golden rule is:
never modify Inbox Zero core files. Always extend, never edit.

Read `references/architecture.md` before touching any file.
Read `references/layers.md` before adding any new feature.
Read `references/style.md` for naming and code conventions.

---

## The one rule that prevents spaghetti

```
Inbox Zero code  →  never touch
Sortie code      →  lives in /sortie/ folder only
Config           →  lives in user_prefs.json only
```

If you find yourself editing a file outside `/sortie/` or `/config/`,
stop and find the extension point instead.

---

## Project structure

```
sortie/                          ← your fork of inbox-zero
├── apps/web/                    ← Inbox Zero core (DO NOT EDIT)
├── packages/                    ← Inbox Zero packages (DO NOT EDIT)
├── sortie/                      ← ALL custom Sortie code lives here
│   ├── classifier/              ← AI email classification logic
│   ├── rules/                   ← CC detection, routing, delete rules
│   ├── digest/                  ← Morning digest email formatter
│   ├── bot/                     ← Teams bot webhook handler
│   ├── alerts/                  ← Urgent alert logic
│   ├── prefs/                   ← user_prefs.json reader/writer
│   └── security/                ← Entra ID / security email scoring
├── config/
│   └── user_prefs.json          ← per-user config (one per client)
├── .env                         ← credentials (never commit)
├── .env.example                 ← template (always keep updated)
└── CLAUDE.md                    ← project context for Claude Code
```

---

## Before writing any code

1. Read `references/architecture.md` — understand the data flow
2. Check if Inbox Zero already has the feature via `grep -r "feature_name" apps/`
3. If it exists → configure it, don't rewrite it
4. If it doesn't → add it in `/sortie/` only
5. Update `CLAUDE.md` if you add a new module

---

## The five Sortie modules — what each one does

### 1. classifier (`/sortie/classifier/`)
Reads an email, sends it to the AI with the user's `ai_context` from prefs,
returns a decision object: `{ category, priority, action, reason }`.
Never calls the email API directly — receives email objects from Inbox Zero hooks.

### 2. rules (`/sortie/rules/`)
Pure logic functions. No AI calls. Takes a decision object and applies
deterministic rules: CC detection, VIP matching, keyword flagging.
Overrides AI decision if a hard rule matches.
**Always log the reason for every rule match.**

### 3. digest (`/sortie/digest/`)
Runs on a cron schedule. Reads the audit log, formats a plain email summary,
sends via Graph API or Gmail API. Never stores email content — only metadata
(sender, subject, action taken, reason).

### 4. bot (`/sortie/bot/`)
Receives Teams webhook messages. Parses intent in plain English.
Reads and writes `user_prefs.json`. Replies via Teams webhook.
Never accesses the email API directly — calls other Sortie modules.

### 5. alerts (`/sortie/alerts/`)
Triggered by the classifier when urgency score is high.
Sends a Teams message or SMS. Includes the email subject and sender only —
never the body (privacy). Always includes an audit log entry.

---

## user_prefs.json — the source of truth for per-client config

```json
{
  "name": "string",
  "timezone": "America/New_York",
  "sign_on_time": "09:00",
  "vip_senders": ["email@domain.com"],
  "helpdesk_addresses": ["helpdesk@company.com"],
  "categories": {
    "cybersecurity": ["keyword1", "keyword2"],
    "urgent_keywords": ["asap", "urgent", "are you available"]
  },
  "delete_rules": {
    "cc_only_if_helpdesk_present": true
  },
  "ai_context": "Free text describing who this person is and what matters to them."
}
```

Rules:
- Every AI call must inject `ai_context` from prefs into the system prompt
- Bot writes to prefs when user says "add X to my VIPs" etc.
- Never hardcode client-specific logic in code — it goes in prefs

---

## Folder routing schema (A/B/C)

| Folder | Label | Criteria |
|--------|-------|----------|
| A | Urgent / VIP | VIP sender OR urgency keyword OR security risk > 0 |
| B | Action needed | Addressed to user, needs response, not urgent |
| C | FYI / archive | CC only, newsletters, low priority |
| Trash | Delete | CC-only AND helpdesk already has it |

---

## Audit log format

Every action must produce a log entry:
```json
{
  "timestamp": "ISO string",
  "email_id": "string",
  "sender": "email only",
  "subject": "string",
  "action": "flagged | moved | deleted | alerted | categorized",
  "folder": "A | B | C | trash",
  "reason": "human readable string",
  "rule": "classifier | cc_detection | vip_match | keyword_flag",
  "confidence": 0.0
}
```

Digest reads this log. Bot reads this log. Never delete log entries.

---

## What to do when you're unsure

1. Check `references/architecture.md` first
2. If still unsure — add it to `/sortie/` with a clear module name
3. Write the audit log entry first, then the logic
4. Ask: "does this belong in prefs instead of code?" — if yes, put it in prefs
5. Never add a new npm package without checking if Inbox Zero already uses it

---

## Common tasks — quick reference

| Task | Where to look | Where to add code |
|------|--------------|-------------------|
| Change how emails are classified | `sortie/classifier/` | `sortie/classifier/` |
| Add a new urgent keyword | `config/user_prefs.json` | nowhere — it's config |
| Add a new bot command | `sortie/bot/` | `sortie/bot/` |
| Change digest format | `sortie/digest/` | `sortie/digest/` |
| Add a new email connector | `references/architecture.md` | `sortie/connectors/` |
| Fix Inbox Zero bug | open an issue on their repo | do NOT patch in your fork |

---

## Reference files

- `references/architecture.md` — full data flow, how modules connect
- `references/layers.md` — what's Inbox Zero vs what's Sortie, extension points
- `references/style.md` — naming conventions, TypeScript patterns, commit format
