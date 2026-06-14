# GHL Build Guide — Service Pro Plumbing Voice Agent (Starter Plan)

**What this builds:** A post-call automation that fires every time Alex (your AI Voice Agent) finishes a call. It classifies urgency, emails the plumber, and sends SMS alerts to the plumber and customer.

---

## Before You Start

Have these ready:
- Plumber's email address
- Plumber's mobile number (for SMS alerts)
- Your GHL phone number set up with Conversation AI

---

## Part 1 — Configure Conversation AI Custom Variables

Alex needs to collect and store three pieces of data during each call. Set these up first so they're available as merge tags in your workflow.

1. Go to **Settings → Conversation AI**
2. Select the phone number you're using for Service Pro Plumbing
3. Find the **Custom Variables** section
4. Add these three variables:

| Variable Key        | Description                                      |
|---------------------|--------------------------------------------------|
| `service_requested` | What the caller needs (e.g. burst pipe, clogged drain) |
| `urgency_level`     | Must be exactly: `Emergency` or `Standard`       |
| `service_address`   | Property address where service is needed         |

5. Save

---

## Part 2 — Build the Workflow

### Step 1 — Create a new workflow

1. Go to **Automations → Workflows**
2. Click **+ Create Workflow**
3. Select **Start from Scratch**
4. Name it: `Alex — Post-Call Alerts (Starter Plan)`

---

### Step 2 — Set the Trigger

1. Click the trigger node at the top
2. Search for and select **Voice AI / Conversation AI**
3. Set the trigger event to: **Call Ended** (or "Call Status: Completed")
4. Save the trigger

---

### Step 3 — Add an If/Else branch

1. Click **+** to add an action below the trigger
2. Search for and select **If/Else**
3. Configure the condition:
   - **Field:** `conversation.custom_data.urgency_level`
   - **Operator:** `equals`
   - **Value:** `Emergency`
4. This creates two branches: **Yes (Emergency)** and **No (Standard)**

---

### Step 4 — Build the Emergency branch (Yes side)

Click **+** under the **Yes** branch and add the following actions in order:

#### Action 1 — Add Tag
- Type: **Add Contact Tag**
- Tag: `emergency`, `plumbing-lead`

#### Action 2 — Email to Plumber
- Type: **Send Email**
- To: *(plumber's email address)*
- Subject:
  ```
  🚨 EMERGENCY: New Plumbing Lead — {{contact.name}}
  ```
- Body:
  ```
  🚨 EMERGENCY CALL — Action Required

  Name: {{contact.name}}
  Phone: {{contact.phone}}
  Service Address: {{conversation.custom_data.service_address}}
  Issue: {{conversation.custom_data.service_requested}}
  Urgency: Emergency

  Call Summary:
  {{conversation.call_summary}}

  ⚠️ This caller flagged their situation as an emergency. Please call back immediately.
  ```

#### Action 3 — SMS to Plumber
- Type: **Send SMS**
- To: *(plumber's mobile number)*
- Message:
  ```
  🚨 EMERGENCY AI Call

  Name: {{contact.name}}
  Phone: {{contact.phone}}
  Service: {{conversation.custom_data.service_requested}}
  Address: {{conversation.custom_data.service_address}}

  ⚠️ Call back immediately.
  ```

#### Action 4 — SMS to Customer
- Type: **Send SMS**
- To: `{{contact.phone}}`
- Message:
  ```
  Hi {{contact.first_name}}, thanks for calling Service Pro Plumbing.

  We received your emergency request and our team has been notified. Someone will call you back shortly.

  Reply STOP to opt out.
  ```

---

### Step 5 — Build the Standard branch (No side)

Click **+** under the **No** branch and add the following actions in order:

#### Action 1 — Add Tag
- Type: **Add Contact Tag**
- Tag: `standard-inquiry`, `plumbing-lead`

#### Action 2 — Email to Plumber
- Type: **Send Email**
- To: *(plumber's email address)*
- Subject:
  ```
  📞 New Plumbing Lead — {{contact.name}}
  ```
- Body:
  ```
  📞 New Lead from AI Voice Agent

  Name: {{contact.name}}
  Phone: {{contact.phone}}
  Service Address: {{conversation.custom_data.service_address}}
  Issue: {{conversation.custom_data.service_requested}}
  Urgency: Standard

  Call Summary:
  {{conversation.call_summary}}

  Follow up at your earliest convenience.
  ```

#### Action 3 — SMS to Plumber
- Type: **Send SMS**
- To: *(plumber's mobile number)*
- Message:
  ```
  📞 New AI Call

  Name: {{contact.name}}
  Phone: {{contact.phone}}
  Service: {{conversation.custom_data.service_requested}}
  Address: {{conversation.custom_data.service_address}}
  Urgency: Standard

  Follow up when available.
  ```

#### Action 4 — SMS to Customer
- Type: **Send SMS**
- To: `{{contact.phone}}`
- Message:
  ```
  Hi {{contact.first_name}}, thanks for calling Service Pro Plumbing.

  We received your request for {{conversation.custom_data.service_requested}} and a team member will be in touch soon.

  Reply STOP to opt out.
  ```

---

## Part 3 — Activate

1. Click **Save** in the top right
2. Toggle the workflow status to **Published / Active**
3. Confirm it is linked to your Conversation AI phone number

---

## Final Workflow Structure (visual reference)

```
[Trigger: Voice AI Call Ended]
             ↓
  [If/Else: urgency = Emergency?]
       ↓ YES            ↓ NO
  [Tag: emergency]   [Tag: standard-inquiry]
  [Email → Plumber]  [Email → Plumber]
  [SMS → Plumber]    [SMS → Plumber]
  [SMS → Customer]   [SMS → Customer]
```

---

## Test Checklist

- [ ] Make a test call to the Conversation AI number
- [ ] Say something non-urgent — confirm standard email + SMS fires to plumber, confirmation SMS fires to customer
- [ ] Make a second test call and say "I have a flooding emergency" — confirm emergency email + SMS fires with correct subject line
- [ ] Verify `{{contact.name}}` and `{{contact.phone}}` are populated (not blank) in the messages
- [ ] Verify `{{conversation.custom_data.service_requested}}` is populated — if blank, check that custom variables are configured in Conversation AI settings
