# Alex – Service Pro Plumbing Voice Agent

## Purpose
Alex is the AI voice agent built for Service Pro Plumbing. Handles 
inbound calls, qualifies leads, and books service appointments.

## Stack
- Voice AI: ElevenLabs
- Telephony: Twilio
- Automation: n8n
- Scheduling: Calendly

## Architecture
Based on Monica's dual-path webhook template:
- Path A: Real-time tool calls during live call
- Path B: Post-call transcript processing

## Service Plan Tiers
This client is on a paid plan. Two n8n workflow configurations are available in this folder:

### Starter Plan (`Starter Plan for Plumbing.json`)
Post-call processing only (single webhook path). After each call:
- Parses and normalizes call data (caller name, phone, service requested, urgency)
- Classifies urgency: Emergency (emergency/urgent/asap/flood/no heat) vs. Standard
- Logs call to Google Sheets ("Plumber Call Sheet") with priority and status
- Sends SMS via Twilio to both the business and the customer (emergency or standard templates)

### Growth Plan (`Growth Plan for Plumbing.json`)
Dual-path architecture (same pattern as Monica's template):
- **Path A (Tool Call):** Real-time calendar availability checks during live call via Google Calendar
- **Path B (Post-Call Transcript):** Full transcript parsing with advanced extraction:
  - Name, email, phone, service address, problem description
  - Urgency levels: Emergency, Urgent, Routine
  - Booking date/time extraction (handles spoken/spelled-out dates and times)
  - Appointment status: New Lead vs. Scheduled
  - Service area ZIP code validation (Stafford, VA and surrounding areas)
  - Logs to Airtable CRM

## Notes
- Same transcript parsing quirks as Monica (dates, phone numbers as words)
- Client-specific system prompt and persona in /prompts/ folder
