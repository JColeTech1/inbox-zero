# Sortie architecture reference

## Data flow — how an email moves through Sortie

```
New email arrives in Gmail / Outlook
         ↓
Inbox Zero webhook fires (we hook into this, don't replace it)
         ↓
sortie/classifier/
  - loads user_prefs.json for this user
  - builds prompt: ai_context + email metadata (sender, subject, snippet)
  - calls Ollama (or Claude/GPT if configured)
  - returns: { category, priority, action, reason, confidence }
         ↓
sortie/rules/
  - applies deterministic overrides on top of AI decision
  - CC detection: is user CC-only? is helpdesk in TO/CC? → delete
  - VIP match: is sender in vip_senders? → folder A + flag
  - keyword match: urgent_keywords in subject/body? → folder A + alert
  - returns final: { action, folder, reason, rule }
         ↓
Inbox Zero action executor (we call their API, don't replace it)
  - move to folder A/B/C
  - apply flag/category/color
  - delete if rule says so
         ↓
sortie/alerts/ (if action = alert)
  - sends Teams message with subject + sender only
  - never sends email body
         ↓
sortie/audit logger
  - writes log entry to local audit.log
  - always runs, even if action = "nothing"
         ↓
sortie/digest/ (runs on cron at user sign_on_time)
  - reads audit.log since last digest
  - formats plain email summary
  - sends via Graph API or Gmail API to user's inbox
```

## How the Teams bot fits in

```
User types in Teams
         ↓
Azure Bot Service webhook → sortie/bot/
         ↓
Intent parser (what did they ask for?)
  "what did you do today?" → read audit.log → reply
  "add X to my VIPs"      → write user_prefs.json → reply
  "show urgent emails"    → query Inbox Zero → reply
  "my sign-on time is 8am"→ write user_prefs.json → reply
         ↓
Teams webhook reply
```

## Connector layer — Gmail vs Outlook

Sortie uses an adapter pattern. The classifier and rules engine never
talk to email APIs directly. They receive a normalized email object:

```typescript
interface SortieEmail {
  id: string
  sender: string
  subject: string
  snippet: string        // first 200 chars only
  recipients_to: string[]
  recipients_cc: string[]
  timestamp: string
  thread_id: string
  labels: string[]       // existing labels/folders
}
```

Gmail connector → normalizes Gmail API response → SortieEmail
Outlook connector → normalizes Graph API response → SortieEmail

This means the classifier and rules engine work identically on both.
To add a new email provider → add a new connector, nothing else changes.

## Where Inbox Zero ends and Sortie begins

Inbox Zero provides:
- OAuth flows for Gmail and Outlook
- Email fetching and webhook subscriptions
- The action API (flag, move, label, delete)
- The rules engine UI (we use the backend, not the UI)
- Scheduling hooks

Sortie adds:
- The AI classifier with per-user context injection
- The deterministic rules layer on top
- The audit logger
- The digest email formatter and scheduler
- The Teams bot
- The user_prefs.json system
- The security email scoring (Entra ID)

## Environment variables

```
# AI backend (pick one)
OLLAMA_BASE_URL=http://localhost:11434/api
OLLAMA_MODEL=llama3
# ANTHROPIC_API_KEY=         # optional cloud fallback
# OPENAI_API_KEY=            # optional cloud fallback

# Gmail (for personal testing)
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=

# Outlook / M365 (fill in when Alex gives access)
MICROSOFT_CLIENT_ID=
MICROSOFT_CLIENT_SECRET=
MICROSOFT_TENANT_ID=

# Teams bot (fill in when bot is set up)
TEAMS_BOT_ID=
TEAMS_BOT_PASSWORD=
TEAMS_WEBHOOK_URL=

# Digest schedule
DIGEST_CRON=                  # overridden per user by sign_on_time in prefs
```
