# Demo Implementation Plan — Step-by-Step (Full Detail)

**Scope:** Week 1 demo only (Steps A–D). Skeleton deadline: **February 9th** (without SharePoint).  
**Access assumed:** GHL, Make.com, Clio dummy account. Retell and Clio OAuth when Jared provides; SharePoint/M365 when ready.  
**Use this doc:** Do each step in order. Open the screenshots when a step says "Reference screenshot." Every click, menu, and field name is written so you can follow without prior platform experience.

---

## Current direction (where we are now)

| Component | Status | Next |
|-----------|--------|------|
| Phase 1 (GHL) | ✅ Done | — |
| Make.com Scenario 1 | ✅ Webhook + Retell Create a Phone Call (2 modules) | Test call (needs US/CA phone) |
| Retell agent | 🔄 Created, configured in Make.com | **Retell enhancements** (Phase 4 below) |
| Make.com Scenario 2 | ✅ Retell call ended → Update GHL (tested via Postman) | Change stage to **Intake Call Booked** |
| Make.com Scenario 3 | ✅ Clio Person + Matter (A1, C2 mapped) | Add SharePoint list item (need access) |
| Phase 4 (Retell) | ⚠️ Needs: calendar check, book call, C2 collection | Multiple custom functions/webhooks |
| Phase 5 (SharePoint) | ❌ Need access | Create list item in MVA Client List |
| Phase 6 (Scenario 4) | ❌ Not started | Intake Tracker → Folder + Word docs |

---

## Quick start — Where to go now

| If you have… | Go to |
|--------------|-------|
| Scenario 1 (Webhook + Retell) configured | **Phase 4** — Retell proper configuration (prompt, webhook, test) |
| Retell webhook URL not set | **Phase 4** — Create Scenario 2, copy webhook URL, paste in Retell |
| Retell configured + tested | **Phase 4** — Make.com Scenario 2 (call ended → Update GHL) |
| Time in parallel | **Phase 2** (Clio dummy) — practice areas, A1, C2 |

---

## Glossary (Terms Explained)

| Term | Meaning |
|------|---------|
| **Form field** | A field the lead sees and fills on the Contact Us form (e.g. First Name, Email). |
| **Custom field** | A field stored on the contact in GHL but NOT on the form. Used to store data that comes from elsewhere (e.g. Retell call → Make.com → GHL). The lead never sees or fills these. |
| **Webhook** | A URL that receives data when something happens (e.g. form submitted). Make.com gives you a webhook URL; GHL sends form data to it. |
| **Pipeline stage** | A step in the lead journey (e.g. New Leads → Consult Scheduled). Moving a contact to "Consult Scheduled" can trigger Make.com. |
| **Consent checkbox** | SMS consent checkbox. See section 1.2. |
| **Merge tag** | GHL placeholder like `{{contact.first_name}}` that gets replaced with real data when the workflow runs. |

---

## Overview: What We're Building (Demo Only)

1. **Step A:** Contact Us form in GHL → lead submits → contact created in GHL.  
2. **Step B:** GHL workflow fires → sends data to Make.com → Retell AI calls lead → checks calendar → books Intake Call on GHL calendar → updates GHL contact and moves to **"Intake Call Booked"** (not Consult Scheduled).  
3. **Step C:** When opportunity = **"Consult Scheduled"** (staff manually moves for demo) → Make.com creates Person + Pending Matter in Clio (with A1 + C2 custom field sets) and adds a row to SharePoint MVA Client List.  
4. **Step D:** When new item in MVA Client List → Make.com creates client folder in OneDrive/SharePoint and fills Word templates (Retainer, Initial Memo) with lead data.

**Jared feedback (2026-02-08):** Retell call success → move to **Intake Call Booked**. Consult Scheduled triggers Clio + SharePoint; staff manually moves for demo.

**Reference:** Open **GHL_Demo_Setup/Demo_Workflow_Outline.png** to see this flow visually.

---

## Two Make.com Scenarios + Retell — Clear Breakdown

These are **two separate Make.com scenarios**. Retell sits in the middle.

| Scenario | Trigger | What it does | Retell's role |
|----------|---------|--------------|---------------|
| **Scenario 1** | GHL form submitted | Receives GHL data → tells Retell to call the lead | Retell **receives** the call request from Make.com (via Retell API) |
| **Scenario 2** | Retell call ended | Receives Retell data → updates GHL contact + moves to **Intake Call Booked** | Retell **sends** the call-ended payload to Make.com (via Retell webhook URL) |

### Scenario 1 — GHL Form → Retell Create a Phone Call

**Flow:** GHL workflow → Make.com webhook URL → Make.com Retell module → Retell places the call.

| Step | Where | What to configure |
|------|-------|-------------------|
| 1 | GHL | Workflow: Form submitted → **Webhook** action → paste Scenario 1 webhook URL |
| 2 | Make.com Scenario 1 | **Module 1:** Custom webhook (receives GHL payload). Copy this URL → use in GHL (step 1). |
| 3 | Make.com Scenario 1 | **Module 2:** Retell AI → **Create a Phone Call**. Map: `to_number` = `1.phone`, `metadata.ghl_contact_id` = `1.contact_id`, plus Dynamic Variables from `1.xxx` |
| 4 | Retell | **Nothing.** Retell is called by Make.com via API. Use same Agent ID in the Retell module. |

**Scenario 1 does not update GHL.** It only initiates the call.

---

### Scenario 2 — Retell Call Ended → GHL Update

**Flow:** Retell call ends → Retell sends webhook → Make.com Scenario 2 receives → Make.com updates GHL.

| Step | Where | What to configure |
|------|-------|-------------------|
| 1 | Make.com Scenario 2 | **Module 1:** Custom webhook (receives Retell "call ended" payload). Copy this URL. |
| 2 | Retell | **Webhook Settings → Agent Level Webhook URL** → paste Scenario 2 webhook URL (from step 1). |
| 3 | Make.com Scenario 2 | **Module 2:** GoHighLevel → Update contact (use `metadata.ghl_contact_id` from Retell payload to identify contact). |
| 4 | Make.com Scenario 2 | **Module 3:** GoHighLevel → Update opportunity / move to **Consult Scheduled**. |

**Scenario 2 does not initiate calls.** It only runs when Retell tells it the call ended.

---

### Retell — What it needs

| Item | Where it comes from | Used for |
|------|---------------------|----------|
| Agent ID | Retell dashboard | Make.com Scenario 1 → Retell module (Override Agent ID) |
| From Number | Retell dashboard | Make.com Scenario 1 → Retell module (outbound caller ID) |
| Webhook URL | Make.com Scenario 2 (module 1) | Retell → Webhook Settings → Agent Level Webhook URL |

Retell **does not** need Scenario 1's webhook URL. GHL sends to Scenario 1; Retell receives the call request from Make.com's API call.

---

## Order of Execution (Phases) — Follow This Sequence

**After Phase 1, do Phase 3 next.** Phase 2 (Clio) can run in parallel or after Phase 3.

| Order | Phase | What | When | Why this order |
|-------|-------|------|------|----------------|
| **1** | Phase 1 | GHL: Form + Custom Fields + Workflow + Pipeline/Calendar | ✅ Done | Foundation — form creates leads, workflow sends to webhook |
| **2** | Phase 3 | Make.com Scenario 1: Webhook + parse GHL + (Retell when ready) + update GHL | **Next** | GHL workflow needs Make.com webhook URL. Build webhook + parse now; add Retell when Jared gives access |
| **3** | Phase 2 | Clio (dummy): Practice areas + Matter numbering + Custom field sets A1 & C2 | In parallel or after Phase 3 | Needed for Phase 5. No dependency on Phase 3 |
| **4** | Phase 4 | Retell AI: Agent + prompt + webhook to Make.com | When Jared shares access | Completes Scenario 1 (call + update GHL) |
| **5** | Phase 5 | Make.com Scenario 3: Consult Scheduled → Clio Person + Matter + SharePoint | After Clio OAuth + Phase 3/4 | Triggers when lead moves to Consult Scheduled |
| **6** | Phase 6 | Make.com Scenario 4: Intake Tracker item → folder + docs | When SharePoint/M365 ready | Triggers on new Intake Tracker row |
| **7** | Phase 7 | End-to-end test | After Phases 1–6 | Final verification |

---

# PHASE 1: GoHighLevel (GHL) — Form, Custom Fields, Workflow

**Platform:** GoHighLevel (GHL)  
**URL:** https://app.gohighlevel.com  
**Login:** Use the credentials Jared sent (or the link he shared). Select the **location** for this project (sub-account).  
**Reference:** **GHL_Demo_Setup/GHL_MAIN_Pipeline.png** and **GHL_Demo_Setup/GHL_Kimball_Law_Funnel.png** — these show what’s already set up; you add the form and workflow.

---

## 1.1 — Open GHL and confirm pipeline and funnel

1. In your browser, go to **https://app.gohighlevel.com** and log in.  
2. In the left sidebar, find **Locations** (or the sub-account selector). Select the location Jared gave you (e.g. the one in the link: `.../location/C4Zs4L5cOXo5ktQzLHtw/dashboard`).  
3. **Pipeline:** In the left menu, click **Opportunities** (or **Sales** → **Opportunities** / **Pipeline**). You should see the **MAIN Pipeline** with stages: New Leads, Attempting Contact, Intake Call Booked, Check w/ Lawyer, **Consult Scheduled**, Pending En…  
   - **Reference screenshot:** **GHL_Demo_Setup/GHL_MAIN_Pipeline.png** — match these stage names. If "Consult Scheduled" is missing, add it (use the pipeline settings / "+ Add stage").  
4. **Funnel:** In the top navigation, click **Funnels**. Open the funnel named **"Kimball Law (PI) Funnel"**.  
   - **Reference screenshot:** **GHL_Demo_Setup/GHL_Kimball_Law_Funnel.png** — you should see **Funnel Steps**: Landing Page, Calendar Page, Thank You Page.  
5. **Calendar:** In the left menu, go to **Calendars** (or **Settings** → **Calendars**). Find the calendar type **"Intake Call"** (1 hr, Round Robin).  
   - **Reference screenshot:** **GHL_Demo_Setup/GHL_Intake_Call_Calendar.png** — if it shows **Inactive**, click the row, then use **Edit** (pencil) or **Settings** (wrench) and set status to **Active** (or ask Jared to activate it). Note the **calendar ID** if you need it later (e.g. for Make.com booking).

---

## 1.2 — Create the Contact Us form

1. In the top navigation bar, click **Forms** (next to Funnels, Websites, etc.).  
2. Click the **"+ Create Form"** or **"Add Form"** button (usually blue, top right).  
3. Choose **"Blank Form"** or **"Contact Form"** and name it **"Contact Us"** (or "Demo Contact Us").  
4. Add these fields (use **Add Field** / drag from the left panel):

| Field name | Type | Required? | Purpose |
|------------|------|-----------|---------|
| First Name | Single line text | Yes | Lead's first name. |
| Last Name | Single line text | Yes | Lead's last name. |
| Email | Email | Yes | For follow-up and documents. |
| Phone | Phone | Yes | Retell will call this number. |
| Legal Issue Type | Dropdown | Yes | Options: **Personal Injury / MVA**, **Civil Litigation**, **Family**, **Other**. Helps Retell know what to ask. |
| Preferred Date/Time | Single line or Date/Time | No | When lead wants consultation. Retell can also ask during call. |
| Any Additional Notes | Multi-line text | No | Free text for anything else. |  
5. **Do NOT add C2 fields to the form.** C2 fields (Date of incident, Location, Seatbelted, Brief description, Injuries) are collected by **Retell during the call**, not by the lead. Add them only as **GHL Custom Fields** in step 1.3. Make.com will populate them when Retell sends data after the call.  
6. **Submit button:** Set label to **"Submit"** or **"Book Your Free Consultation"**.  
7. **Consent checkbox (TnC):** See full explanation below.  
8. **Save** the form. Copy the **Form ID** (often in URL or in form settings) — you’ll need it for the workflow.  
9. **Connect form to funnel:** Funnels → Kimball Law (PI) Funnel → Landing Page → Edit → add Form block → select Contact Us. Save.

---

### Consent checkbox — Full explanation

**What it is:**  
A checkbox (often TnC1, TnC2, or "Terms and Conditions") at the bottom of the form. GHL adds it for **SMS consent**.

**Why it exists:**  
US (TCPA) and Canada (CASL) require **explicit consent** before sending SMS. If the firm sends texts (e.g. appointment reminders), this checkbox is **legally required**.

**Typical text:**  
*"By checking this box, I consent to receive non-marketing text messages from [BUSINESS NAME] about [USE_CASE]. Message frequency varies, message & data rates may apply. Text HELP for assistance, reply STOP to opt out."*

**What you do:**
1. **Keep it** if the firm will send SMS (recommended for demo).
2. **Configure:** In the checkbox settings, replace `[BUSINESS NAME]` with **Kimball Law** and `[USE_CASE]` with e.g. **"your consultation and case updates"**.
3. **Where:** Scroll to the bottom of the form. Or Add Field → Terms and Conditions / Consent / TnC.
4. **If no SMS:** You can remove it. For demo, keeping it is safer for compliance.

---

## 1.3 — Create custom fields in GHL (for contacts)

1. In the left sidebar, go to **Settings** (gear icon at bottom or in menu).  
2. Click **Custom Fields** (under Contact / Contact Settings).  
3. Ensure you’re on **Contact** custom fields (not Company or other).  
4. Click **"+ Add Field"** (or **Add Custom Field**) and create the following (type = Text one-line or Text multi-line unless noted):  
   - **Call Notes** (multi-line) — for Retell call summary  
   - **Appointment Date** (text or date if available)  
   - **Appointment Time** (text)  
   - For C2 Civil intake (Make.com writes after Retell): **Date of incident or accident**, **Time of incident or accident**, **Location of incident or accident**, **Seatbelted**, **Brief description of incident**, **Injuries reported at intake**.  
   - **Reference:** **Clio/Clio_Field_Set_C2_Civil_Intake.png** — use exact names so Make.com can map to Clio.  
5. Save each field. Note the **custom field IDs** or **keys** (often shown in the list or in API docs) if Make.com/API needs them.

---

## 1.4 — Create workflow (Form Submitted → Webhook to Make.com)

1. In the left sidebar, go to **Automation** (or **Workflows**).  
2. Click **"+ Create Workflow"** (or **Add Workflow**). Name it e.g. **"Demo – Form to Make.com"**.  
3. **Trigger:** Click the trigger block. Choose **Form Submitted**. Select your **Contact Us** form. Save.  
4. **Action 1 (optional but useful):** Add an action **Add Contact to Pipeline** (or **Opportunities** → **Add to Pipeline**). Select **MAIN Pipeline** and stage **New Leads**. This creates/updates the contact as an opportunity in **New Leads**.  
   - **Reference:** **GHL_Demo_Setup/GHL_MAIN_Pipeline.png** — first stage is "New Leads."  
5. **Action 2:** Add an action **Webhooks** → **Send Webhook** (or **HTTP Request** / **Outbound Webhook**).  
   - **URL:** You will get this from **Make.com Scenario 1** (Custom webhook URL). Paste that URL here.  
   - **Method:** **POST**.  
   - **Body:** Choose **JSON** or **Form data**. Include at least: contact first name, last name, email, phone, and any custom field values. In GHL you often use **merge tags** (e.g. `{{contact.first_name}}`, `{{contact.email}}`, `{{contact.phone}}`). Add a field like `contact_id` = `{{contact.id}}` so Make.com can update the same contact later.  
   - Save the action.  
6. **Turn the workflow ON** (toggle at top).  
7. **Note:** You’ll fill in the webhook URL in **Phase 3** when you create Make.com Scenario 1. Until then, you can leave the URL placeholder or use a test URL (e.g. https://webhook.site) to confirm the workflow fires.

---

## 1.5 — Confirm calendar and pipeline stage for "Consult Scheduled"

1. **Calendar:** In **Calendars**, ensure **Intake Call** is **Active**.  
   - **Reference:** **GHL_Demo_Setup/GHL_Intake_Call_Calendar.png** — same calendar (1 hr, Round Robin).  
2. **Pipeline stage:** In **Opportunities** → **MAIN Pipeline**, confirm the stage **Consult Scheduled** exists (it’s the 5th column in the screenshot). Make.com Scenario 2 will move contacts to this stage after the AI books the call; Scenario 3 will trigger on this.

**Phase 1 done when:** Form exists, custom fields exist, workflow exists (trigger = Form Submitted, action = Webhook to Make.com URL), pipeline has "Consult Scheduled," Intake Call calendar is active.

---

### Phase 1 verification (completed 2026-02-02)

Steps 1.1–1.5 have been verified:

- **1.1–1.2:** Form submitted successfully; lead created in GHL.
- **1.3:** Custom fields exist (C2 fields empty on form submit — expected; Retell will populate).
- **1.4:** Workflow fired; webhook received at webhook.site. Sample payload structure and **exact Make.com mapping**:

| GHL payload field | Example value | In Make.com (after Parse JSON, module 2): map from | Use for |
|------------------|---------------|---------------------------------------------------|---------|
| `contact_id` | `LgOc0SyULhQg4AYKnxwk` | `2.contact_id` | Update GHL contact, Retell metadata |
| `first_name` | `Amir` | `2.first_name` | Clio Person, docs |
| `last_name` | `Shahzad` | `2.last_name` | Clio Person, docs |
| `email` | `verxeonteam@gmail.com` | `2.email` | Clio Person |
| `phone` | `+923185914596` | `2.phone` | Retell outbound call |
| `Legal Issue Type` | `["Personal Injury / MVA"]` | `2.Legal Issue Type` | Retell context (use first item if array) |
| `Preferred Date/Time` | `2026-02-06` | `2.Preferred Date/Time` | Calendar booking |
| `Any Additional Notes` | `I had an accident` | `2.Any Additional Notes` | Call Notes |
| `location.id` | `C4Zs4L5cOXo5ktQzLHtw` | `2.location.id` | GHL location (nested) |
| C2 fields | `""` | Empty until Retell | Retell populates after call |

- **1.5:** Intake Call calendar active; Consult Scheduled stage present.

**Next:** Go to **Phase 3 (Make.com Scenario 1)** — you need its webhook URL to replace webhook.site in your GHL workflow. Skip Phase 2 for now; do it in parallel or after Phase 3.

---

# PHASE 3: Make.com — Scenario 1 (GHL Form → Retell Call) — DO THIS NEXT

**Platform:** Make.com  
**URL:** https://www.make.com  
**Login:** Use the **Kimball Law** organization (Jared invited you).  

**Goal:** Receive GHL form data, trigger Retell to call the lead. GHL update and move to Consult Scheduled happen in **Scenario 2** (Retell call ended). Build webhook + Retell module now.

---

## Phase 3 — Deep breakdown

### Stage 3.1 — Create Scenario 1

| Step | Action | What you'll see |
|------|--------|-----------------|
| 1 | Make.com → **Scenarios** (left menu) → **"+ Create a new scenario"** | Blank scenario canvas with a "+" in the center |
| 2 | Click the scenario name (top) → rename to **"Demo 1 – GHL Form → Retell Call"** | Name updated |

---

### Stage 3.2 — Add trigger: Custom webhook

| Step | Action | What you'll see |
|------|--------|-----------------|
| 1 | Click the **+** in the center | Module picker opens |
| 2 | Search **Webhooks** → select **Custom webhook** → **Add** | Webhook module appears; a unique URL is generated |
| 3 | **Copy the Webhook URL** (click the URL or copy icon) | URL like `https://hook.eu1.make.com/xxxxx` |
| 4 | Go to **GHL** → Automation → your workflow → **Webhook** action → paste this URL, replace webhook.site | GHL now sends to Make.com |
| 5 | In Make.com webhook settings: leave **"Show only first 10"** unchecked | Every form submit will trigger the scenario |
| 6 | Click **OK** | Webhook module is saved |

---

### Stage 3.3 — Add Retell AI: Create a Phone Call

**Option A — 2 modules only (Webhook + Retell):** Map directly from Webhook (module 1). No Parse JSON needed.

| Step | Action | What you'll see |
|------|--------|-----------------|
| 1 | Click **+** after the webhook module | Module picker |
| 2 | Search **Retell AI** → select **Create a Phone Call** → **Add** | Retell module |
| 3 | **Connection:** Add connection with Retell API key | — |
| 4 | **From Number:** `+19025001641` (E.164, no spaces) | — |
| 5 | **To Number:** Map from `1.phone` (Webhook → phone) | Turn **Map ON** to pick variable |
| 6 | **Override Agent ID:** `agent_8e30e42c9b66e1de8071abd262` | — |
| 7 | **Metadata:** Key `ghl_contact_id`, Value = `1.contact_id` | Turn **Map ON**, select contact_id from Webhook |
| 8 | **Retell LLM Dynamic Variables:** Add first_name, last_name, email, phone, Legal Issue Type, Preferred Date/Time, Any Additional Notes (all from `1.xxx`) | Turn **Map ON** to pick variables |
| 9 | Save | — |

**Map toggle:** When **Map** is ON, you can only pick variables from previous modules. When OFF, you type values manually. For data from GHL, keep Map ON.

**Option B — 3 modules (Webhook + Parse JSON + Retell):** Use Parse JSON between Webhook and Retell; then map from `2.contact_id`, `2.phone`, etc.

---

### Stage 3.4 — Create Scenario 2 (Retell Call Ended → GHL Update)

Scenario 1 only initiates the call. When the call ends, **Retell** sends data to **Scenario 2**. Build Scenario 2 here, then paste its webhook URL into Retell (Phase 4).

---

### Stage 3.4.1 — Create Scenario 2

| Step | Action | What you'll see |
|------|--------|-----------------|
| 1 | Make.com → **Scenarios** (left menu) → **"+ Create a new scenario"** | Blank scenario canvas |
| 2 | Click the scenario name (top) → rename to **"Demo 2 – Retell Call Ended → Update GHL"** | Name updated |

---

### Stage 3.4.2 — Add trigger: Custom webhook (Retell call ended)

| Step | Action | What you'll see |
|------|--------|-----------------|
| 1 | Click the **+** in the center | Module picker opens |
| 2 | Search **Webhooks** → select **Custom webhook** → **Add** | Webhook module appears; a unique URL is generated |
| 3 | **Copy the Webhook URL** (click the URL or copy icon) | URL like `https://hook.eu1.make.com/xxxxx` |
| 4 | Leave **"Show only first 10"** unchecked | Every call-ended event will trigger the scenario |
| 5 | Click **OK** | Webhook module is saved |
| 6 | **Important:** Paste this URL into **Retell** → Agents → your agent → **Webhook Settings** → **Agent Level Webhook URL** (see Phase 4). Retell will POST here when a call ends. | — |

---

### Stage 3.4.3 — Add GoHighLevel: Update contact

| Step | Action | What you'll see |
|------|--------|-----------------|
| 1 | Click **+** after the webhook module | Module picker |
| 2 | Search **GoHighLevel** → select **Update a contact** (or equivalent) → **Add** | GHL module |
| 3 | **Connection:** Add GoHighLevel connection (OAuth) if needed | — |
| 4 | **Contact ID:** Map from `1.call_analysis.custom_data.ghl_contact_id` or `1.metadata.ghl_contact_id` (check Retell webhook payload structure; we passed `ghl_contact_id` in metadata from Scenario 1) | Turn **Map ON** to pick from webhook |
| 5 | **Custom Fields:** Set Call Notes, Appointment Date/Time, C2 fields (Date of incident, Location, Seatbelted, Brief description, Injuries) from Retell transcript or extracted data | Map from `1.xxx` paths in webhook payload |
| 6 | Save | — |

**Note:** Retell "call ended" payload structure may vary. Inspect the webhook run history in Make.com to see exact paths (e.g. `1.call_analysis`, `1.transcript`, `1.custom_data`).

---

### Stage 3.4.4 — Add GoHighLevel: Move to Intake Call Booked

| Step | Action | What you'll see |
|------|--------|-----------------|
| 1 | Click **+** after the Update contact module | Module picker |
| 2 | Search **GoHighLevel** → select **Update opportunity** or **Add to pipeline** / **Move opportunity** → **Add** | GHL module |
| 3 | **Pipeline:** MAIN | — |
| 4 | **Stage:** **Intake Call Booked** (NOT Consult Scheduled — get stage ID from GHL export) | — |
| 5 | **Contact:** Same contact (from `1.call_analysis.custom_data.ghl_contact_id` or metadata path) | Turn **Map ON** |
| 6 | Save | — |

**Note:** Consult Scheduled is for staff manual move (triggers Scenario 3). See To-Do.txt for Intake Call Booked stage ID.

---

**Phase 3 done when:** Scenario 1 (GHL → Retell call) and Scenario 2 (Retell call ended → GHL update) are both built. Scenario 1 webhook URL is in GHL. Scenario 2 webhook URL is in Retell (Phase 4).

---

# PHASE 2: Clio (Dummy Account) — Do in parallel or after Phase 3 — Practice Areas, Matter Numbering, Custom Field Sets

**Platform:** Clio Manage  
**URL:** https://app.clio.com (or the URL from the invite Jared sent)  
**Login:** Accept the **dummy Clio account** invite from Jared and log in. This account is empty; you’ll add practice areas and custom field sets here so Make.com can create Persons and Matters with the right structure.

**Reference screenshots:** All in the **Clio/** folder (or **Forms/** — same filenames). Open each when a step says so.

---

## 2.1 — Practice areas

1. In Clio, in the **left sidebar**, click **Settings** (gear or "Settings" at bottom).  
2. Click **Firm Preferences** (under Settings).  
3. Open the **Practice areas** tab.  
   - **Reference:** **Clio/Clio_Practice_Areas_List_1.png** and **Clio_Practice_Areas_List_2.png** — you’ll see a list like Adoption, Civil Litigation, Personal Injury, etc.  
4. Click **"+ Add practice area"** (blue button, top right).  
5. Add at least: **Personal Injury**, **Civil Litigation**, **Consultation-Potential Clients** (or similar). You can add more from the screenshot list if you want the demo to match production.  
6. Save each. You don’t need to fill **Practice area code** for the demo.

---

## 2.2 — Practice area enablement

1. Still under **Settings** → **Firm Preferences**, open the **Practice area enablement** tab.  
   - **Reference:** **Clio/Clio_Practice_Area_Enablement.png** — shows which categories are Enabled and which is Primary.  
2. Ensure **Personal Injury** is **Enabled** and set as **Primary** (for the demo we’ll create matters in this area). Enable any others you added.

---

## 2.3 — Matter numbering

1. Under **Settings** → **Firm Preferences**, open the **Matter numbering** tab.  
   - **Reference:** **Clio/Clio_Matter_Numbering.png** — format is something like: **[four digit year] - [yearly matter number] - [client summary name] - [client first name]**.  
2. Select **Custom** template (or the one that lets you build this format).  
3. Add tokens: **Four digit year**, **Yearly matter number**, **Client summary name**, **Client first name**, with separators (e.g. " - ").  
4. Set **Next matter number** (e.g. 1).  
5. Click **Update Settings** (or Save).

---

## 2.4 — Custom field sets: A1 General intake and C2 Civil intake

Jared said: *"The 'A1 General intake fields' and 'C2 Civil intake' are the important custom field sets."* Ignore C1 Civil update.

**2.4.1 — Create custom fields (if Clio requires creating fields first)**  
Some Clio setups let you add fields when creating a set; others require creating fields under **Custom fields** first.  
- Go to **Settings** → **Custom fields** → **Matter custom fields** → **Custom fields** (tab). Create any fields that don’t exist yet (see names below). Type: **Text (one-line)** or **Text (multi-line)** as needed.

**2.4.2 — A1 General intake fields**

1. Go to **Settings** → **Custom fields** → **Matter custom fields** → **Custom field sets** tab.  
   - **Reference:** **Clio/Clio_Custom_Field_Sets_Table.png** — you see a table of sets; **Clio/Clio_Field_Set_A1_General_Intake.png** — edit modal for A1.  
2. Click **"+ Add custom field set"** (or **Add custom field set**).  
3. **Name:** **A1 General intake fields** (exactly).  
4. **Custom fields:** Add these members (select from existing or create):  
   - **Narrative notes from intake**  
   - **Matter subtypes by practice area**  
5. Check **Default** (this will be applied to all new and existing Matters) if you want.  
6. Click **Save**.

**2.4.3 — C2 Civil intake**

1. Again **"+ Add custom field set"**.  
2. **Name:** **C2 Civil intake**.  
3. **Custom fields:** Add these (reference **Clio/Clio_Field_Set_C2_Civil_Intake.png**):  
   - Date of incident or accident  
   - Time of incident/accident  
   - Location of incident or acc (or "Location of incident or accident")  
   - Seatbelted  
   - Brief description of incident  
   - Injuries reported at intake  
4. **Default:** Optional. Save.

**Phase 2 done when:** Dummy Clio has practice areas (at least Personal Injury as primary), matter numbering set, and custom field sets **A1 General intake fields** and **C2 Civil intake** created.

---

# PHASE 4: Retell AI — Proper Configuration (Step-by-Step)

**Platform:** Retell AI  
**URL:** https://dashboard.retellai.com  
**Login:** Use the workspace Jared shared. Create API key (Developer role) if needed.

Jared expects a **proper configuration** in Retell. Follow the **Retell Configuration — Step-by-Step Guide** at the end of this doc.

## 4.1 — Retell requirements (Jared feedback)

| # | Requirement | How |
|---|-------------|-----|
| 1 | Check calendar availability | Retell custom function → webhook to Make.com scenario |
| 2 | Book intake call in GHL Intake Calendar | Retell custom function → webhook to Make.com scenario |
| 3 | After booked → Move to Intake Call Booked | Scenario 2 (Retell call ended) or GHL workflow |
| 4 | Collect C2 custom field data during call | Retell prompt (ask questions) → output to `call_analysis` |
| 5 | Extract data → GHL custom fields → Clio Matter | Scenario 2 maps `call_analysis.custom_analysis_data` → GHL; Scenario 3 maps GHL → Clio |

**Phone number for testing:** Retell does not support Pakistan outbound. Options: (A) Jared provides US/CA number + subscription; (B) You buy (e.g. Twilio). **Recommend:** Ask Jared for Option A.

## 4.2 — Open your agent

1. Retell → **Agents** (left sidebar).  
2. Open your **Law Firm Intake** agent.  
3. **Prompt:** Write a short system prompt: e.g. you’re an intake coordinator for a law firm; the lead just submitted a form; greet them, confirm name/phone, ask if they want to book a free consultation; if yes, ask for preferred date/time; say you’re booking it and summarize. Optionally: if they mention a car accident, ask the C2 questions (date of incident, time, location, seatbelted, brief description, injuries) and say you’ll note them.  
4. **Tools / Webhook:** Add a **Webhook** for “Call ended” — set the URL to your **Make.com Scenario 2** webhook (the one that receives Retell’s “call ended” payload). So when the call ends, Retell POSTs transcript and data to Make.com.  
5. **Test:** Use Retell’s **Test call** to your phone; refine the prompt.  
6. Copy the **Agent ID** into Make.com Scenario 1 (HTTP request to Retell) in **Phase 3.4**.

**Phase 4 done when:** Retell agent is live, “Call ended” webhook points to Make.com, and Scenario 1 triggers the call with the correct agent_id and to_number.

---

# PHASE 5: Make.com — Scenario 3 (Consult Scheduled → Clio + SharePoint)

**When:** After Jared sets up **Clio OAuth in Make.com** (he said he’d do that).

1. **Create a new scenario** — name e.g. **"Demo 2 – Consult Scheduled → Clio + SharePoint"**.  
2. **Trigger:** Either (a) **Webhooks** → **Custom webhook**, and in GHL add a second workflow: trigger = **Opportunity stage changed to Consult Scheduled** → action = Send webhook to this URL with contact/opportunity data; or (b) **GoHighLevel** → **Watch opportunities** (if available) filter by stage = Consult Scheduled.  
3. **Clio – Create Person:** Use **Clio** module (or **HTTP** + Clio API). **Create contact** (Person) with name, email, phone from trigger. Store **Person ID**.  
4. **Clio – Create Pending Matter:** **Create matter** (or equivalent). Link to **Person ID**. Set **Practice area** = Personal Injury (or the one you use). **Custom field sets:** Attach **A1 General intake fields** and **C2 Civil intake** and map:  
   - From **A1:** Narrative notes from intake, Matter subtypes by practice area (from GHL/Retell data).  
   - From **C2:** Date of incident or accident, Time, Location, Seatbelted, Brief description, Injuries reported (from GHL custom fields or Retell payload).  
   - **Reference:** **Clio/Clio_Field_Set_A1_General_Intake.png** and **Clio/Clio_Field_Set_C2_Civil_Intake.png** for exact field names.  
5. Store **Matter ID**.  
6. **SharePoint:** **Microsoft 365** (or **HTTP** Microsoft Graph) → **Create list item** in the **MVA Client List**. Site: `https://lawkimball.sharepoint.com/sites/1600BedfordHwy-Develop`. Output path: `/Documents/CLIENTS/"[LastName], [FirstName] - MVA"`. **Requires:** SharePoint access or M365 connection in Make.com (ask Jared).

**Phase 5 done when:** Moving a contact to Consult Scheduled triggers Scenario 3 and creates Person + Pending Matter in Clio (with A1 and C2 sets) and one row in MVA Client List.

---

# PHASE 6: Make.com — Scenario 4 (Folder + Word Docs) — Step-by-Step

**Trigger:** Scenario 3 sends HTTP POST to Scenario 4 webhook (same pattern as Scenario 2 → Scenario 3).  
**Goal:** Create client folder in SharePoint and fill Retainer + Initial Memo with lead data.

---

## 6.0 — Add HTTP module to Scenario 3 (do this first)

So that Scenario 4 runs when Scenario 3 finishes:

| Step | Action | Details |
|------|--------|---------|
| 1 | Open Scenario 3 | Make.com → Scenarios → your Scenario 3 |
| 2 | Create Scenario 4 | New scenario → name it "Demo 4 – Folder + Docs" |
| 3 | Add webhook trigger | Scenario 4 → Add module → Webhooks → Custom webhook → Add |
| 4 | Copy webhook URL | Copy the URL from Scenario 4's webhook module |
| 5 | In Scenario 3 | Add module **after** SharePoint Create Item |
| 6 | Add HTTP module | HTTP → Make a request → Add |
| 7 | **URL** | Paste Scenario 4's webhook URL |
| 8 | **Method** | POST |
| 9 | **Headers** | Add: `Content-Type` = `application/json` |
| 10 | **Body type** | Raw |
| 11 | **Request content** | Use JSON below (see 6.0.1 below). Use Map to pick variables from modules 2, 4, 5 |
| 12 | Save | Save Scenario 3 |

**6.0.1 — HTTP body JSON:**

```json
{
  "first_name": "{{2.first_name}}",
  "last_name": "{{2.last_name}}",
  "email": "{{2.email}}",
  "phone": "{{2.phone}}",
  "legal_issue_type": "{{2.legal_issue_type}}",
  "appointment_date": "{{2.appointment_date}}",
  "appointment_time": "{{2.appointment_time}}",
  "date_of_incident_or_accident": "{{2.date_of_incident_or_accident}}",
  "time_of_incident_or_accident": "{{2.time_of_incident_or_accident}}",
  "location_of_incident_or_accident": "{{2.location_of_incident_or_accident}}",
  "seatbelted": "{{2.seatbelted}}",
  "brief_description_of_incident": "{{2.brief_description_of_incident}}",
  "injuries_reported_at_intake": "{{2.injuries_reported_at_intake}}",
  "call_notes": "{{2.call_notes}}",
  "matter_display_number": "{{5.Display Number}}",
  "matter_id": "{{5.Matter ID}}",
  "client_id": "{{4.Contact ID}}"
}
```

If a variable is missing in your webhook, use `ifempty()` or leave it out.

---

## 6.1 — Create Scenario 4 and set trigger

| Step | Action | What you see |
|------|--------|--------------|
| 1 | Make.com → **Scenarios** | Left menu |
| 2 | **Create a new scenario** | Blue button |
| 3 | Click scenario name (top) | Rename to **"Demo 4 – Folder + Docs"** |
| 4 | Click the **+** in the center | Module picker opens |
| 5 | Search **Webhooks** | Type "Webhooks" in search |
| 6 | Click **Webhooks** | Service list |
| 7 | Click **Custom webhook** | Action |
| 8 | Click **Add** | Webhook module appears |
| 9 | **Copy the webhook URL** | Copy icon or click URL |
| 10 | **Paste this URL** in Scenario 3's HTTP module (see 6.0 above) | — |
| 11 | Click **OK** | Save webhook |

**Scenario 4 trigger:** Custom webhook. Data comes from Scenario 3 HTTP body.

---

## 6.2 — Create folder in SharePoint

| Step | Action | What you'll see |
|------|--------|-----------------|
| 1 | Scenario 4: Click **+** after webhook | Module picker |
| 2 | Search **Microsoft SharePoint** | Or "SharePoint" |
| 3 | Select **Microsoft SharePoint Online** | Service |
| 4 | Select **Create a folder** or **Create folder** | Or search "Create folder" |
| 5 | Click **Add** | —
| 6 | **Connection** | Select "KL Developers - SharePoint" (same as Scenario 3) |
| 7 | **Search Method** | Select from the list |
| 8 | **Site ID** | Select **1600 Bedford Hwy - Develop** |
| 9 | **Drive ID** or **Library** | If asked: Documents library. Or select from list |
| 10 | **Folder path** or **Parent folder** | Map: `/Documents/CLIENTS/` (or the parent path) |
| 11 | **Folder name** | Map: `{{1.last_name}}, {{1.first_name}} - MVA` |

Module 1 = webhook. Use `1.last_name`, `1.first_name` from the HTTP body.

**If "Create folder" is not available:** Use **Microsoft 365** → **OneDrive** → **Create folder** (or equivalent). Path: `/Documents/CLIENTS/{{1.last_name}}, {{1.first_name}} - MVA`.

---

## 6.3 — Download template + Upload to client folder (step-by-step)

**Note:** Copy file is not available in Make.com SharePoint. Use **Download a file** + **Upload a file** for each template.

Templates are in: `TEMPLATES/Correspondence`. Download each template, then upload it into the new client folder.

### 6.3.1 — Fix "Failed to load data" on File field

If the File dropdown shows "Failed to load data!", use **Enter manually** instead:

| Step | Action | Details |
|------|--------|---------|
| 1 | **Enter (File ID & File Path)** | Choose **File Path** |
| 2 | **Enter a File Path** | Change from "Select from the list" to **Enter manually** |
| 3 | **File path** | Type: `TEMPLATES/Correspondence/Retainer.docx` (or the exact template name Jared gave – e.g. `Retainer Agreement.docx`) |
| 4 | Save | Click OK |

**If you don't know the exact file name:** Open SharePoint in browser → go to `Documents/TEMPLATES/Correspondence` → note the exact file names (e.g. `Retainer.docx`, `Initial Memo.docx`).

---

### 6.3.2 — Download file + Upload file (only option)

Copy file is not available in Make.com SharePoint module. Use **Download a file** + **Upload a file**:

| Step | Module | What to configure |
|------|--------|-------------------|
| 1 | **Download a file** | Microsoft SharePoint Online → **Download a file** |
| 2 | Connection | KL Developers |
| 3 | Site ID | `lawkimball.sharepoint.com,ea856c5c-a7ed-4b12-b4f1-43d613dd10e9,2804c9f2-cc92-4559-8a33-a96d1ca0f08c` (or select "1600 Bedford Hwy - Develop" from dropdown) |
| 4 | Drive ID | Same as Site ID, or select "Documents (documentLibrary)" from dropdown |
| 5 | File Path | Enter manually: `TEMPLATES/Correspondence/Retainer.docx` |
| 6 | Convert to PDF | No |
| 7 | **Upload a file** | Microsoft SharePoint Online → **Upload a file** |
| 8 | Folder | Map the folder from Create folder (6.2). Use output e.g. `2.Id` if Create folder is module 2 |
| 9 | File | Map the output of Download (Data or File field) |
| 10 | File name | `Retainer - {{1.last_name}}, {{1.first_name}}.docx` |

---

### 6.3.3 — Retainer template (exact steps)

| Step | Action |
|------|--------|
| 1 | Add **Download a file** module after Create folder |
| 2 | Connection: KL Developers |
| 3 | Enter (File ID & File Path): File Path |
| 4 | Enter a File Path: Enter manually |
| 5 | Site ID: `lawkimball.sharepoint.com,ea856c5c-a7ed-4b12-b4f1-43d613dd10e9,2804c9f2-cc92-4559-8a33-a96d1ca0f08c` |
| 6 | Drive ID: Same or select Documents |
| 7 | File Path: `TEMPLATES/Correspondence/Retainer.docx` |
| 8 | Convert to PDF: No |
| 9 | Add **Upload a file** module |
| 10 | Folder: Map Create folder output (e.g. `2.Id`) |
| 11 | File: Map Download output (Data) |
| 12 | File name: `Retainer - {{1.last_name}}, {{1.first_name}}.docx` |

---

### 6.3.4 — Initial Memo template (same process)

| Step | Action |
|------|--------|
| 1 | Add another **Download a file** module |
| 2 | Same connection, Site ID, Drive ID |
| 3 | File Path: `TEMPLATES/Correspondence/Initial Memo.docx` (match exact name) |
| 4 | Convert to PDF: No |
| 5 | Add **Upload a file** module |
| 6 | Folder: Same as Retainer (Create folder output) |
| 7 | File: Map this Download output |
| 8 | File name: `Initial Memo - {{1.last_name}}, {{1.first_name}}.docx` |

---

### 6.3.6 — Checking template file names

To get exact names:

1. Open: https://lawkimball.sharepoint.com/sites/1600BedfordHwy-Develop  
2. Go to Documents → TEMPLATES → Correspondence  
3. Write down the exact Retainer and Initial Memo file names (e.g. `Retainer.docx`, `Initial Memo.docx`)

---

## 6.4 — (Merged into 6.3)

Copy/upload of both templates is done in 6.3. No separate 6.4 step needed.

---

## 6.5 — Module order in Scenario 4

1. **Webhook** (trigger) — receives data from Scenario 3 HTTP  
2. **Create folder** — `/Documents/CLIENTS/[LastName], [FirstName] - MVA`  
3. **Download a file** — Retainer template from TEMPLATES/Correspondence  
4. **Upload a file** — Retainer to client folder  
5. **Download a file** — Initial Memo template  
6. **Upload a file** — Initial Memo to client folder  

---

## 6.6 — Quick reference

| Item | Value |
|------|-------|
| SharePoint connection | KL Developers |
| Site | 1600 Bedford Hwy - Develop |
| Client folder path | `/Documents/CLIENTS/[LastName], [FirstName] - MVA` |
| Templates path | `/Documents/TEMPLATES/Correspondence` |
| Webhook variables | `1.first_name`, `1.last_name`, `1.email`, etc. |

---

**Phase 6 done when:** Scenario 3 HTTP triggers Scenario 4, folder is created, and Retainer + Initial Memo are saved in that folder.

---

# PHASE 7: End-to-End Test

1. Submit the **Contact Us** form in GHL (use your own phone/email for testing).  
2. Confirm: GHL workflow runs, Make.com Scenario 1 receives webhook.  
3. If Retell is connected: Retell calls the number; after the call, GHL contact is updated and opportunity moves to **Intake Call Booked**.
4. Confirm: Make.com Scenario 2 runs (call ended → GHL update) and contact moves to Intake Call Booked. Staff manually moves to Consult Scheduled for demo.
5. Confirm: Make.com Scenario 3 runs, Clio has new Person + Pending Matter (with A1 and C2 filled), and MVA Client List has a new row.
6. Confirm: Make.com Scenario 4 runs, client folder is created and Retainer + Initial Memo are saved.

---

## Quick reference: Screenshots to open

| When you are… | Open this file |
|---------------|----------------|
| Checking GHL pipeline stages | **GHL_Demo_Setup/GHL_MAIN_Pipeline.png** |
| Checking GHL funnel steps | **GHL_Demo_Setup/GHL_Kimball_Law_Funnel.png** |
| Checking GHL Intake Call calendar | **GHL_Demo_Setup/GHL_Intake_Call_Calendar.png** |
| Checking full demo flow | **GHL_Demo_Setup/Demo_Workflow_Outline.png** |
| Adding Clio practice areas | **Clio/Clio_Practice_Areas_List_1.png** or **Clio_Practice_Areas_List_2.png** |
| Enabling practice areas in Clio | **Clio/Clio_Practice_Area_Enablement.png** |
| Setting Clio matter numbering | **Clio/Clio_Matter_Numbering.png** |
| Creating A1 General intake in Clio | **Clio/Clio_Field_Set_A1_General_Intake.png** |
| Creating C2 Civil intake in Clio | **Clio/Clio_Field_Set_C2_Civil_Intake.png** |
| Ignore (per Jared) | **Clio/Clio_Field_Set_C1_Civil_Update.png** |

---

## Dependencies (what you need from Jared)

| Item | Used in | Jared said |
|------|---------|------------|
| Retell API key + phone number + workspace | Scenario 1, Retell agent | Done |
| Clio OAuth in Make.com | Scenario 3 | He’ll set up tomorrow |
| **Phone for testing** (US/CA) | Retell outbound — PK not supported | Option A: Jared provides; Option B: You buy (Twilio). Recommend A. |
| **SharePoint access** (guest or M365 in Make) | Scenario 3 (MVA Client List), Scenario 4 (folder + docs) | Site URL shared; direct access not provided. Ask Jared. |
| Word templates (Retainer, Initial Memo) | Scenario 4 | In `/Documents/TEMPLATES/Correspondence` |

You can complete **Phase 1 (GHL)** and **Phase 2 (Clio)** and Scenario 1 (webhook + Retell) in **Phase 3** now. Add Retell agent config (Phase 4) and Scenario 2 (call ended → GHL) when Retell is ready; add Scenarios 3 and 4 when Clio OAuth and SharePoint/M365 are ready.

---

## Retell Configuration — Step-by-Step Guide

**Prerequisite:** Build Scenario 2 in **Phase 3 — Stage 3.4** first. You need its webhook URL for Step 2 below.

Follow these steps in **Retell dashboard** for proper configuration (as Jared expects).

### Step 1 — Open your Retell agent

1. Retell → **Agents** (left sidebar).  
2. Open your **Law Firm Intake** agent (or create one: **Single Prompt Agent**, **Start from blank**).

### Step 2 — Paste Scenario 2 webhook URL

1. In Retell agent, expand **Webhook Settings**.  
2. Find **Agent Level Webhook URL** (or **Call ended webhook**).  
3. Paste the **Make.com Scenario 2** webhook URL (from Phase 3 — Stage 3.4.2).  
4. Save. When a call ends, Retell will POST the transcript and metadata to this URL.

### Step 3 — Configure the Prompt

1. In the agent, go to **Prompt**.  
2. Replace with a **multi-case prompt** that covers:  
   - **MVA:** Greet by name, ask for consultation. If yes, ask: date/time of accident, location, were they seatbelted, brief description, injuries.  
   - **Civil / Family:** Greet by name, ask for consultation. If yes, ask brief description of their situation.  
   - **Other:** Greet by name, ask what brought them in today.  
   - **All:** Summarize what you heard and thank them.

### Step 4 — Set Welcome Message (AI speaks first)

1. Find **Welcome Message** or **Greeting**.  
2. Set to **AI speaks first** (not human).  
3. Add greeting text, e.g.:  
   *"Hi, this is Kimball Law. You recently submitted a contact form — I'm calling to confirm your details and see if you'd like to book a free consultation."*

### Step 5 — Speech Settings

1. Go to **Speech** or **Voice**.  
2. Choose a voice (e.g. Kate, Cimo).  
3. Save.

### Step 6 — Test and Publish

1. Use **Test Chat** or **Test Audio** to simulate a call.  
2. Refine the prompt if needed.  
3. **Publish** the agent.

---

**Quick reference**

| Item | Value |
|------|-------|
| Agent ID | agent_8e30e42c9b66e1de8071abd262 |
| From Number | +19025001641 |
| Scenario 1 Webhook (GHL → Make.com) | https://hook.us2.make.com/jyq1z56fb4siowaoe07hq4shhs92wvbg |
