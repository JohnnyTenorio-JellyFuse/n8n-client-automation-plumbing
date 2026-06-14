# Client Voice Agent Automation — Plumbing (Two Service Tiers)

Two n8n workflows representing the back-end automation for "Alex," an AI voice agent receptionist built for a plumbing company, sold as two product tiers: **Starter** and **Growth**. Both are built on the same dual-path webhook architecture used in [the Monica voice agent backend](../n8n-voice-agent-backend) — this repo shows how that template scales down (Starter) or up (Growth) depending on what a client needs and pays for.

The client name ("Service Pro Plumbing") and contact details throughout are placeholder/test data — no real client information is included.

---

## Starter Plan

**File:** `workflows/Starter Plan for Plumbing.json`

Post-call processing only — a single webhook that fires after each call ends.

```
Webhook
  → Parse & Normalize Call   (extract caller name, phone, service, urgency)
  → Urgency Classifier        (Emergency vs. Standard)
  → [Emergency branch]  → SMS to business + SMS to customer (emergency templates)
                         → Append row to Google Sheets ("Plumber Call Sheet")
  → [Standard branch]   → SMS to business + SMS to customer (standard templates)
                         → Append row to Google Sheets
  → Respond to ElevenLabs
```

**Urgency classification** keys off phrases like *emergency*, *urgent*, *asap*, *flood*, and *no heat* — anything matching routes to the emergency branch with faster SMS turnaround and different message copy for both the business owner and the customer.

This tier is intentionally simple: no calendar integration, no CRM — just call logging and instant SMS alerts. It's the entry point for a client who wants "don't miss a call" coverage without a full booking system.

---

## Growth Plan

**File:** `workflows/Growth Plan for Plumbing.json`

Full dual-path architecture — the same pattern as the Monica voice agent backend, adapted for a plumbing client's needs.

```
                          Webhook to ElevenLabs
                                  │
                          Parse Information
                                  │
              ┌───────────────────┴───────────────────┐
              ▼                                         ▼
   PATH A — Tool Call (live)                 PATH B — Post-Call Transcript
   "Is this time slot free?"                  Full extraction:
              │                               name, email, phone, address,
              ▼                               problem description, urgency
   Format Calendar Search Dates               (Emergency / Urgent / Routine),
              │                               booking date/time, ZIP validation
              ▼                                         │
   Google Calendar — Check Availability                 ▼
              │                              Log to Airtable CRM
              ▼                                         │
   Process Calendar Check                  ┌────────────┴────────────┐
              │                             ▼                          ▼
              ▼                   New Lead (not booked)      Scheduled (booked)
   Respond to ElevenLabs                    │                          │
                                  Send SMS (Request Booking)  Send SMS (Confirmation)
                                                               + Create Calendar Event
```

**What's different from the Monica template:**
- **Service area validation** — incoming addresses are checked against a ZIP code list for the plumber's actual service area (Stafford, VA and surrounding) before a booking is confirmed.
- **Three urgency levels** instead of two (Emergency / Urgent / Routine), feeding into different SMS templates and response priority.
- **SMS-first communication** (via Twilio) rather than email — appointment confirmations and booking requests go out as text messages, which fits how plumbing customers expect to communicate.

---

## Client Delivery Materials

- **`prompts/Alex - GHL Conversation AI.md`** — the system prompt for the voice agent itself (persona, call-handling instructions, urgency classification rules, what data to collect). This is what gets pasted into the voice platform's Conversation AI configuration.
- **`docs/GHL Setup Guide - Plumber Starter.md`** — a step-by-step build guide for configuring the Starter plan's automation inside GoHighLevel: setting up custom variables, building the workflow, and wiring up SMS/email alerts. Written for a non-technical operator to follow.
- **`docs/architecture.md`** — internal notes on both tiers, including known transcript-parsing quirks shared with the Monica template (spoken-digit phone numbers, date extraction, etc.).

---

## Setup Notes

To run either workflow you'll need:

- An **ElevenLabs Conversational AI** agent with a system prompt matching `prompts/Alex - GHL Conversation AI.md`
- **Twilio** for SMS (both plans)
- **Starter:** a Google Sheets credential for call logging
- **Growth:** Google Calendar (availability + booking) and Airtable (CRM) credentials

All credentials, base IDs, and account-specific identifiers have been removed and replaced with `{{PLACEHOLDER}}` values.
