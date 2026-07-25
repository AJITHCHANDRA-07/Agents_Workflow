# Agents_Workflow
Automate,Scale, Repeat
# AI Email Automation Agent

An n8n workflow that reads Gmail, uses an LLM to draft or auto-reply to
sensitive emails, and notifies the user through Slack and WhatsApp —
built and debugged end-to-end, not a tutorial copy-paste.

## Problem

Checking and responding to sensitive/time-sensitive emails manually is
slow, and important messages get missed if no one is watching the
inbox. This agent watches Gmail continuously, uses an LLM to decide
what a reasonable reply or draft looks like, and pushes a real-time
notification so a human never has to poll their inbox to stay on top
of it.

## How it works

1. **Trigger** — n8n's Gmail Trigger node watches the inbox for new
   incoming messages.
2. **Context extraction** — the workflow pulls the sender, subject,
   and body of the new email.
3. **LLM reasoning** — the email content is sent to Groq's LLaMA model
   with a prompt that decides: is this sensitive enough to need a
   careful reply, and if so, what should that reply say?
4. **Action** — depending on the LLM's output, the workflow either
   drafts a reply in Gmail (for review before sending) or sends an
   auto-reply directly, depending on configured sensitivity rules.
5. **Notification** — a summary of the action taken is pushed to
   **Slack** and **WhatsApp** (via Twilio), so the user is notified
   immediately without needing to check email or n8n directly.

## Tech Stack

- **n8n Cloud** — workflow orchestration
- **Gmail API** — trigger + draft/reply actions
- **Groq API (LLaMA model)** — LLM reasoning and reply generation
- **Slack API** — notification delivery
- **Twilio WhatsApp API** — secondary notification channel

## Real problems solved while building this (not just "it worked")

- **Node naming after JSON import** — n8n renames nodes inconsistently
  on import; fixed by manually re-labeling nodes after each import to
  keep references stable.
- **Expression scoping after Gmail action nodes** — data from earlier
  nodes wasn't resolving correctly inside later expressions; fixed by
  explicitly referencing node names (`$('Node Name').item.json...`)
  instead of relying on implicit `$json` scope.
- **Groq auth header format** — the API rejected malformed auth
  headers initially; fixed by matching the exact `Authorization:
  Bearer <key>` format Groq's API expects.
- **Twilio WhatsApp sandbox behavior** — messages silently failed
  outside the approved sandbox number/template rules; fixed by
  strictly using Twilio's sandbox join code and approved message
  format during testing.
- **Gmail trigger reprocessing old emails** — the trigger occasionally
  re-fired on already-processed emails after workflow edits; fixed by
  adding a processed-message tracking step to avoid duplicate actions.

## Status

**Complete and working.** This workflow runs in my personal n8n Cloud
workspace and has been tested end-to-end (trigger → LLM → Gmail
action → Slack/WhatsApp notification).

## Files in this repo

- `email_automation_agent.json` — exported n8n workflow (import via
  n8n's **Workflows → Import from File**)

## Setup (if you want to run this yourself)

1. Import `email_automation_agent.json` into your own n8n instance.
2. Connect your own credentials for: Gmail, Groq (Header Auth), Slack,
   and Twilio — credential IDs are not portable between n8n accounts,
   so each node will need to be reconnected to your own credentials
   after import.
3. Test using a throwaway/test email address before pointing it at a
   real inbox.

---

Built by Nimmala Ajithchandra —
[LinkedIn](https://www.linkedin.com/in/ajithchandra-nimmala) |
[GitHub](https://github.com/ajithchandra-07)
