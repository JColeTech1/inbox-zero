# Sortie — build to-do list

## Before you touch any code

- [ ] Make sure you have Git installed on your laptop (`git --version` in terminal)
- [ ] Make sure you have Node.js installed (`node --version` — needs v24+)
- [ ] Make sure you have VS Code or a code editor installed
- [ ] Download and install Claude Code (claude.ai/code)
- [ ] Download and install Ollama (ollama.ai) — for local AI
- [ ] Have your MDC college email login handy (Microsoft 365)
- [ ] Have your personal Google email login handy

---

## Phase 1 — get something working (goal: this week)

### Fork and clone the repos

- [ ] Go to github.com/elie222/inbox-zero → hit Fork → name it `sortie`
- [ ] Go to github.com/littlebearapps/outlook-assistant → Fork → name it `sortie-outlook-connector`
- [ ] Go to github.com/doobidoo/outlook-ai-agent → Fork → name it `sortie-windows-agent`
- [ ] Open terminal and run:
  ```
  git clone https://github.com/YOUR_USERNAME/sortie
  cd sortie
  ```
- [ ] Open the `sortie` folder in Claude Code

### Connect to Gmail (personal Google account)

- [ ] Go to console.cloud.google.com
- [ ] Create a new project called "Sortie Test"
- [ ] Enable Gmail API
- [ ] Create OAuth credentials (desktop app)
- [ ] Download the credentials JSON file
- [ ] Follow Inbox Zero setup docs to connect Gmail
- [ ] Confirm you can see your emails being read in the app

### Connect to Outlook (MDC college email)

- [ ] Go to portal.azure.com and sign in with your MDC email
- [ ] Register a new app called "Sortie Test"
- [ ] Add Microsoft Graph permissions: Mail.Read, Mail.ReadWrite
- [ ] Copy the client ID and secret
- [ ] Add Outlook connector config to Sortie
- [ ] Confirm you can see your MDC emails being read

### Install Ollama and connect local AI

- [ ] Run `ollama pull llama3` in terminal
- [ ] In Sortie settings set AI backend to Ollama
- [ ] Send a test email to yourself and confirm Sortie classifies it

### First real test

- [ ] Send yourself a fake urgent email ("are you available?") on Gmail
- [ ] Send yourself a fake VIP email on MDC Outlook
- [ ] Send yourself a fake CC-only email
- [ ] Check what Sortie did with each one
- [ ] Write down what worked and what didn't

---

## Phase 2 — make it Sortie (goal: week 2-3)

### user_prefs.json — per person config

- [ ] Ask Claude Code to create a `user_prefs.json` template
- [ ] Add fields: name, timezone, sign-on time, VIP list, email categories
- [ ] Create one prefs file for yourself as the test user
- [ ] Make the AI classifier read the prefs file before every decision

### Folder schema

- [ ] Define the A/B/C folder structure:
  - A = Urgent / VIP (needs response today)
  - B = Action needed (respond this week)
  - C = FYI / archive (no response needed)
- [ ] Ask Claude Code to add auto-folder-routing to Sortie
- [ ] Test that emails land in the right folders

### Morning digest email

- [ ] Ask Claude Code to add a cron job that runs at sign-on time
- [ ] Format a plain digest email showing what Sortie did overnight
- [ ] Test it sends to your Gmail at the right time
- [ ] Test it sends to your MDC Outlook at the right time

### CC detection logic (Helpdesk rule)

- [ ] Ask Claude Code to write CC detection — if user is CC'd only and another recipient is on the VIP/Helpdesk list, move to trash
- [ ] Test with a fake CC email
- [ ] Add to audit log so user can see what was deleted and why

### Audit log

- [ ] Every action Sortie takes gets logged: what email, what action, why
- [ ] Digest email includes a summary of the audit log
- [ ] User can ask the Teams bot "what did you do today?" and get the log

---

## Phase 3 — Teams bot + alerts (goal: month 2)

### Teams bot setup

- [ ] Create a bot in Azure Bot Service using your MDC account
- [ ] Connect bot to your MDC Teams
- [ ] Test you can send it a message and get a response
- [ ] Wire bot to Sortie backend via webhook
- [ ] Test commands:
  - "What did you do today?"
  - "Add [name] to my VIPs"
  - "My sign-on time is 8am"
  - "Show me urgent emails"
  - "What's in my inbox right now?"

### Urgent alerts

- [ ] When AI detects urgency (time-sensitive language) → Teams message
- [ ] Test with "are you available?" email
- [ ] Test with a fake Microsoft Entra security alert

### Security email scoring (Entra ID)

- [ ] Write a custom prompt for Entra ID digest emails
- [ ] 0 risky users + low sign-ins = "nothing to worry about, categorized"
- [ ] Any real risk detected = Teams alert to user immediately

---

## Phase 4 — hand off to Alex and Manny

- [ ] Clean up the GitHub repo — remove test credentials
- [ ] Write README.md (already drafted in sortie-docs)
- [ ] Write the Alex handoff doc (already drafted in sortie-docs)
- [ ] Demo to Manny on your MDC email
- [ ] Get Alex to register the app in SecureNet's Azure tenant
- [ ] Test on Manny's real inbox (read-only first)
- [ ] Get sign-off before enabling write actions on company email

---

## Nice to have (future)

- [ ] SMS alerts via Twilio for truly urgent emails
- [ ] Per-client onboarding flow ("tell me your name, timezone, VIPs")
- [ ] Feedback loop — user says "wrong call" and Sortie learns
- [ ] Package as an MSP product with SecureNet branding
- [ ] Multi-tenant support (one Sortie instance per client)
