# Sortie style guide

## Naming conventions

### Files
- Modules: `kebab-case.ts` — `cc-detection.ts`, `digest-formatter.ts`
- Types: `types.ts` inside each module folder
- Tests: `module-name.test.ts` next to the file being tested
- Config: `user_prefs.json` (snake_case for JSON files)

### Functions
- Actions (do something): verb + noun — `classifyEmail()`, `sendDigest()`, `writePrefs()`
- Checks (return boolean): `is` / `has` prefix — `isVipSender()`, `hasCcOnlyRule()`
- Formatters: `format` prefix — `formatDigestEmail()`, `formatAuditEntry()`
- Getters: `get` prefix — `getUserPrefs()`, `getAuditLog()`

### Variables
- Email objects: always `email` (not `msg`, `message`, `mail`)
- User prefs: always `prefs` (not `config`, `settings`, `user`)
- Audit entries: always `entry` (not `log`, `record`, `item`)
- Decision objects: always `decision` (not `result`, `output`, `classification`)

---

## TypeScript patterns

### Always type your decision objects
```typescript
interface SortieDecision {
  category: 'urgent' | 'vip' | 'action' | 'fyi' | 'delete' | 'security'
  priority: 'A' | 'B' | 'C' | 'trash'
  action: 'flag' | 'move' | 'delete' | 'alert' | 'categorize' | 'nothing'
  reason: string        // human readable, goes in audit log
  rule: string          // which rule fired: 'classifier' | 'cc_detection' etc
  confidence: number    // 0.0 to 1.0, from AI classifier
}
```

### Always write the audit entry before the action
```typescript
// RIGHT — log it first, then do it
const entry = buildAuditEntry(email, decision)
await writeAuditLog(entry)
await executeAction(email, decision)

// WRONG — action first, log after (if action throws, no log)
await executeAction(email, decision)
await writeAuditLog(entry)
```

### Never put AI prompts inline
```typescript
// WRONG
const prompt = `You are an email classifier. The user is ${prefs.name}...`

// RIGHT — prompts live in /sortie/classifier/prompts/
import { buildClassifierPrompt } from './prompts/classifier-prompt'
const prompt = buildClassifierPrompt(prefs, email)
```

### Never put email content in logs
```typescript
// WRONG
entry.body = email.body

// RIGHT — metadata only
entry.subject = email.subject
entry.sender = email.sender
// body never leaves the classifier
```

---

## Commit message format

```
type(module): short description

feat(classifier): add Entra ID security scoring
fix(rules): cc detection now handles multiple helpdesk addresses
chore(prefs): add timezone validation
docs(skill): update architecture reference
test(digest): add cron schedule tests
```

Types: `feat` `fix` `chore` `docs` `test` `refactor`
Modules: `classifier` `rules` `digest` `bot` `alerts` `prefs` `security` `connectors`

---

## What not to do

- Don't use `any` type — if you don't know the type, define an interface
- Don't catch errors silently — always log to audit with reason
- Don't store email body anywhere outside the classifier's working memory
- Don't add a new dependency without checking if Inbox Zero already has it
- Don't write a rule in code that belongs in `user_prefs.json`
- Don't call the email API from anywhere except the connectors layer
- Don't send a Teams alert without an audit log entry
- Don't hardcode any email address, keyword, or client name in code
