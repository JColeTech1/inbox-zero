# Sortie — Claude Code context

## What this project is

Sortie is an AI-powered email management assistant built on top of Inbox Zero
(MIT license). It classifies, triages, and acts on emails using local AI
(Ollama) and communicates with users through Teams messages and Outlook/Gmail
digest emails. No new apps — Teams and Outlook are the only UI.

## The one rule

Never edit files outside `/sortie/` or `/config/`.
Inbox Zero core lives in `apps/` and `packages/` — extend it, never modify it.

## Active modules

| Module | Location | Status | What it does |
|--------|----------|--------|--------------|
| classifier | /sortie/classifier/ | not started | AI email classification with user context |
| rules | /sortie/rules/ | not started | CC detection, VIP matching, keyword flags |
| digest | /sortie/digest/ | not started | Morning digest email at user sign-on time |
| bot | /sortie/bot/ | not started | Teams bot webhook handler |
| alerts | /sortie/alerts/ | not started | Urgent Teams/SMS alerts |
| prefs | /sortie/prefs/ | not started | user_prefs.json reader/writer |
| security | /sortie/security/ | not started | Entra ID digest scoring |

## Current phase

Phase 1 — connecting to email and getting read + classify working

## Email connectors

- Gmail: personal Google account (testing)
- Outlook: MDC college M365 account (testing)
- SecureNet tenant: waiting on Alex for Azure credentials

## What's working

- [ ] Gmail connected
- [ ] MDC Outlook connected
- [ ] Ollama running locally
- [ ] Inbox Zero reading emails

## Known gotchas

- MDC college email is M365 under the hood — use the Outlook connector
- Ollama must be running before starting the dev server
- Never commit `.env` — use `.env.example` as the template
- The audit log must be written BEFORE any email action is taken

## Reference docs

- Full architecture: `doc/architecture.md`
- Layer rules: `doc/layers.md`
- Style guide: `doc/style.md`
- Product vision: `doc/VISION (1).md`
- Build to-do: `doc/TODO.md`
- Skill definition: `doc/skill-reference.md`

## goodwork — end-of-session command

When the user types "goodwork", "good work", or "session done", do ALL of these:

1. **Review** what was accomplished in the session
2. **Update `doc/TODO.md`** — check off completed items `[x]`, add new tasks if discovered
3. **Update this file (`CLAUDE.md`)** — module statuses, "what's working" checklist, phase, gotchas
4. **Update other docs** if architecture or conventions changed
5. **Update memory** — edit `project_sortie_status.md` with new state and today's date
6. **Commit** all doc changes: `docs(sortie): update status after session — [what happened]`
7. **Summarize** — short bullet list of what changed and what's next
