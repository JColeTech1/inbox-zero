# Sortie layers — what to touch and what not to touch

## The layered cake

```
┌─────────────────────────────────────┐
│         SORTIE LAYER                │  ← you build here
│  /sortie/*  /config/*  CLAUDE.md    │
├─────────────────────────────────────┤
│      INBOX ZERO EXTENSION POINTS    │  ← hook in here only
│  webhooks, action API, scheduler    │
├─────────────────────────────────────┤
│       INBOX ZERO CORE               │  ← never touch
│  apps/web/*  packages/*             │
└─────────────────────────────────────┘
```

---

## Inbox Zero files — DO NOT EDIT

If you find yourself editing these, stop and find another way:

- `apps/web/app/` — Next.js app router pages
- `apps/web/utils/` — Inbox Zero utility functions
- `apps/web/providers/` — email provider integrations
- `packages/tinybird/` — analytics
- `packages/loops/` — email sending
- Any file not inside `/sortie/` or `/config/`

**Why:** When Inbox Zero releases updates you want to be able to pull them
in without merge conflicts. If you've edited their files, every update
becomes a manual conflict resolution job.

---

## Inbox Zero extension points — use these

These are the hooks Sortie uses to plug in without modifying core:

### 1. Webhook / email processing hook
Inbox Zero fires a webhook when a new email arrives.
Sortie registers a handler that receives the email and runs the classifier.

File to look at: `apps/web/app/api/google/webhook/route.ts`
How to extend: add your handler call AFTER the existing processing, not instead of it.

### 2. Action API
Inbox Zero exposes functions to act on emails.
Sortie calls these — never replicates them.

Key functions to use:
- `moveToFolder(emailId, folder)`
- `applyLabel(emailId, label)`
- `flagEmail(emailId)`
- `deleteEmail(emailId)`
- `sendEmail(to, subject, body)`

### 3. Scheduler / cron hooks
Inbox Zero has scheduled job support.
Register Sortie's digest cron here rather than running a separate process.

### 4. User/auth context
Inbox Zero manages OAuth tokens.
Sortie reads the current user context from Inbox Zero's session —
never stores its own copy of credentials.

---

## Decision tree — where does new code go?

```
Is this feature already in Inbox Zero?
  YES → configure it via their settings, don't rewrite
  NO  ↓

Does it need to know about a specific user's preferences?
  YES → reads from user_prefs.json → lives in /sortie/prefs/ or relevant module
  NO  ↓

Does it classify or score an email?
  YES → /sortie/classifier/ or /sortie/security/
  NO  ↓

Does it make a decision based on rules?
  YES → /sortie/rules/
  NO  ↓

Does it take an action on an email?
  YES → calls Inbox Zero action API → orchestration code in /sortie/rules/
  NO  ↓

Does it send something to the user?
  Teams message → /sortie/alerts/ or /sortie/bot/
  Email digest  → /sortie/digest/
  NO  ↓

Is it a new connector for a new email provider?
  YES → /sortie/connectors/
```

---

## CLAUDE.md — keep this updated

The `/CLAUDE.md` file in the root of the repo is what Claude Code reads
first when you open the project. Keep it current. Every time you add a
new module to `/sortie/`, add one line to CLAUDE.md describing it.

Minimum CLAUDE.md content:
- What Sortie is (one paragraph)
- The layer rule (never edit Inbox Zero core)
- List of active Sortie modules with one-line descriptions
- Current phase of build
- What's working vs what's in progress
- Any known gotchas or things NOT to do
