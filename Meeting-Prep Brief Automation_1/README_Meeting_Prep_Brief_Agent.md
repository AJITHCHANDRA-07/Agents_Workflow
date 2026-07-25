# Founder Meeting-Prep Brief Agent (In Progress)

An n8n workflow that detects upcoming meetings on a founder's
calendar, pulls the relevant email history with that contact, and
delivers a short AI-generated context brief to Slack/WhatsApp before
the meeting starts — so no one walks into a meeting cold.

## Problem

Before every meeting, someone has to remember who the other person is,
what was last discussed, and what's likely being asked now. For a
busy founder juggling many relationships at once, this constant
context-switching is a real, recurring time cost. This agent
automates the "remembering" so the human only has to show up
prepared.

## How it works (current design)

1. **Trigger** — a Schedule Trigger polls Google Calendar every
   15 minutes, checking for events starting in the next ~45 minutes.
2. **Filter** — an IF node skips personal blocks/reminders and only
   continues for events with an external attendee.
3. **Field extraction** — meeting title, time, and attendee email are
   pulled into clean variables.
4. **Context retrieval** — the workflow searches Gmail for the most
   recent email thread(s) with that attendee's email address.
5. **Fallback handling** — if no prior email thread exists, the
   workflow still proceeds, flagging this as a first-time contact
   instead of failing.
6. **LLM summarization** — the meeting details + email context are
   sent to Groq's LLM with a prompt asking for a brief under 5 lines:
   who this is, what was last discussed, and 2 suggested talking
   points (or 2 opening questions, if there's no prior history).
7. **Delivery** — the generated brief is sent to Slack (and optionally
   WhatsApp via Twilio) roughly 30 minutes before the meeting starts.

## Tech Stack

- **n8n Cloud** — workflow orchestration
- **Google Calendar API** — meeting detection
- **Gmail API** — prior context retrieval
- **Groq API (LLaMA model)** — brief generation
- **Slack API / Twilio WhatsApp API** — delivery

## Status: In Progress

The core loop (calendar detection → Gmail search → LLM brief →
Slack/WhatsApp delivery) is being built and tested using the same
credential/auth patterns already solved in my
[email automation agent](https://github.com/ajithchandra-07) project.

**Planned next steps (not yet built):**
- **Topic-aware context matching** — classify each meeting (e.g.
  Finance, Marketing, Innovation) from its title/description, and use
  that to narrow the Gmail search and tailor the LLM prompt.
- **Persistent notes-log** — a simple Google Sheet of past
  interactions per contact, read alongside Gmail history, so briefs
  get richer over time instead of relying only on raw email search.
- **Push notifications instead of polling** — replace the 15-minute
  polling trigger with Google Calendar's push notification (`watch`)
  API, so the workflow reacts instantly to new/changed events instead
  of checking on a timer.

## Files in this repo

- `meeting_prep_brief_agent.json` — exported n8n workflow template
  (import via n8n's **Workflows → Import from File**)

## Setup (if you want to run this yourself)

1. Import `meeting_prep_brief_agent.json` into your own n8n instance.
2. Connect your own credentials for: Google Calendar, Gmail, Groq
   (Header Auth), Slack, and Twilio.
3. Replace placeholder values (Slack user ID, WhatsApp number) with
   your own.
4. Test using a calendar event you create yourself with a throwaway
   attendee email before pointing it at real meetings.

---

Built by Nimmala Ajithchandra —
[LinkedIn](https://www.linkedin.com/in/ajithchandra-nimmala) |
[GitHub](https://github.com/ajithchandra-07)
