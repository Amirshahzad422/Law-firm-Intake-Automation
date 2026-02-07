# Law Firm CRM Automation — Implementation Guide

**For:** First-time use of GHL, Make.com, Retell AI  
**Goal:** Build the Week 1 demo end-to-end with zero prior experience  
**Next:** After this, we draft a separate document to share with the client to start work.

---

## 1. What This Project Is (Plain English)

A **lead** fills a "Contact Us" form on the law firm’s site. An **AI** calls them, asks questions, books a consultation, and saves notes. When the consultation is **scheduled**, the system creates a **client + matter** in the law firm’s software (Clio), adds a row to a **tracker** in SharePoint, creates a **folder** for the client, and fills **Word templates** (e.g. Retainer, Initial Memo) with the collected data.

You will **not** write code in an IDE for the main flow. You will work mainly in **web dashboards** (GHL, Make.com, Retell AI, Clio, Microsoft 365). APIs are used **by** those platforms (and by Make.com) — you configure them, not build them from scratch.

---

## 2. Quick Glossary

| Term | Meaning |
|------|--------|
| **GHL / GoHighLevel** | CRM + marketing platform. Has forms, workflows, contacts, pipelines, calendars. You build forms and workflows in the browser. |
| **Make.com** | Automation tool that connects apps. You build "scenarios" (flows) in the browser: when X happens, do Y. It calls APIs for you. |
| **Retell AI** | Voice AI: an AI agent that makes/answers phone calls. You configure the agent and prompts in their web dashboard; Make.com triggers calls via API. |
| **Clio Manage** | Law practice management: matters, contacts, documents. You use it in the browser; Make.com talks to it via Clio’s API. |
| **SharePoint** | Microsoft app for lists, sites, files. "Intake Tracker" = a list. You work in browser or via Make.com + Microsoft Graph API. |
| **OneDrive** | Cloud storage (often tied to SharePoint). Client folders and generated docs go here. |
| **Webhook** | A URL that receives data when something happens (e.g. "form submitted"). Make.com gives you webhook URLs; GHL sends data to them. |
| **API** | How software talks to another software. You use APIs **through** Make.com (and sometimes GHL/Retell), not by writing raw API code in an IDE. |
| **Pipeline** | In GHL, a funnel of stages (e.g. New Lead → Consult Scheduled). Moving a contact to "Consult Scheduled" can trigger Make.com. |
| **MVA** | Motor Vehicle Accident. "MVA-specific data" = structured answers (incident date, injury type, etc.) that go into GHL custom fields and then Clio. |

---

## 3. Full Workflow Diagram (Demo — Step A to D)

```
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│  STEP A — INGEST (GoHighLevel)                                                           │
├─────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                          │
│   [ Lead ]  ──fills──>  [ Contact Us Form ]  ──submits──>  [ GHL Contact Created ]      │
│                              (GHL)                         + Pipeline: "New Lead"       │
│                                                                                          │
│   WHERE YOU WORK: GHL dashboard (browser)                                                │
│   OUTPUT: Contact in GHL with: name, email, phone, custom fields                         │
└─────────────────────────────────────────────────────────────────────────────────────────┘
                                          │
                                          │ Form Submit = TRIGGER
                                          ▼
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│  STEP B — THE CALL (GHL Workflow + Make.com + Retell AI)                                 │
├─────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                          │
│   [ GHL Workflow ]                                                                       │
│   Trigger: Form submitted                                                                 │
│   Action:  Send webhook ──────────────────────┐                                          │
│           (contact data + custom values)       │                                          │
│                                               ▼                                          │
│   [ Make.com — Scenario 1 ]                                                            │
│   • Receive webhook (GHL payload)                                                         │
│   • Map: name, phone, email, custom fields                                               │
│   • HTTP module: POST to Retell API ──────────┐     API: Retell "Create Phone Call"      │
│   • Wait for Retell webhook (call ended)      │     https://api.retellai.com              │
│   • Parse transcript + extracted data        │                                          │
│   • Update GHL contact (notes, MVA fields)   │                                          │
│   • Book appointment → GHL Calendar          │                                          │
│   • Move pipeline → "Consult Scheduled"       │                                          │
│                                               ▼                                          │
│   [ Retell AI ]  (you work in BROWSER dashboard)                                         │
│   • Agent: prompt + voice + tools                                                        │
│   • Tools: "check_availability" (GHL/calendar), "book_appointment", "save_notes"        │
│   • Asks MVA questions → returns structured JSON                                         │
│   • Calls lead’s phone (outbound)                                                         │
│   • When call ends → Retell sends webhook to Make.com with transcript + data             │
│                                                                                          │
│   WHERE YOU WORK: GHL (workflow), Make.com (scenario), Retell (agent config) — all web   │
│   APIS USED: Retell API (from Make.com), GHL API (from Make.com for contact/calendar)   │
└─────────────────────────────────────────────────────────────────────────────────────────┘
                                          │
                                          │ Trigger: Opportunity status = "Consult Scheduled"
                                          ▼
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│  STEP C — THE HAND-OFF (Make.com Scenario 2)                                             │
├─────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                          │
│   [ Make.com — Scenario 2 ]                                                            │
│   Trigger: GHL webhook — "Opportunity status changed to Consult Scheduled"               │
│            (or: GHL workflow sends webhook when stage = Consult Scheduled)               │
│                                                                                          │
│   Action 1:  Clio API  ──>  Create Person (contact)                                     │
│   Action 2:  Clio API  ──>  Create Pending Matter (link to Person)                      │
│             Map GHL custom fields (incl. MVA) → Clio matter custom fields                │
│   Action 3:  Microsoft Graph API  ──>  Create item in SharePoint list "Intake Tracker"   │
│             (client name, matter ID, date, status)                                       │
│                                                                                          │
│   WHERE YOU WORK: Make.com (browser). Clio/SharePoint = used via APIs only.              │
│   APIS: Clio Manage API (OAuth), Microsoft Graph (SharePoint list)                      │
└─────────────────────────────────────────────────────────────────────────────────────────┘
                                          │
                                          │ Trigger: New item in SharePoint "Intake Tracker"
                                          ▼
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│  STEP D — THE DOCS (Make.com Scenario 3)                                                │
├─────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                          │
│   [ Make.com — Scenario 3 ]                                                            │
│   Trigger: SharePoint "Intake Tracker" — new item created (webhook or polling)           │
│                                                                                          │
│   Action 1:  Microsoft Graph API  ──>  Create folder in OneDrive/SharePoint              │
│             e.g. /Clients/[ClientName]/  (or /Intake/[ClientName]/)                     │
│   Action 2:  Fill Word templates (Retainer, Initial Memo) with lead/matter data          │
│             Save filled documents into the new client folder                             │
│             (Graph API for files, or Power Automate for Word merge)                      │
│                                                                                          │
│   WHERE YOU WORK: Make.com (browser). Word templates prepared in advance (client side).  │
│   APIS: Microsoft Graph (OneDrive/SharePoint, files). Word: Graph or Power Automate.     │
└─────────────────────────────────────────────────────────────────────────────────────────┘
```

**Summary of where you work:**

| Platform    | Where you work        | Main task                                      |
|------------|------------------------|-----------------------------------------------|
| GHL        | Browser (app.gohighlevel.com) | Form, workflow, custom fields, calendar      |
| Make.com   | Browser (make.com)    | 3 scenarios: webhooks + API calls             |
| Retell AI  | Browser (retellai.com)| Agent, prompts, voice, tools (e.g. book call) |
| Clio       | Browser (for reference); API via Make.com only | —                                    |
| SharePoint/OneDrive | Browser (for reference); API via Make.com | —                            |

---

## 4. Platform-by-Platform: Where to Work & What to Do

### 4.1 GoHighLevel (GHL)

- **What it is:** CRM with forms, workflows, contacts, pipelines, calendars.  
- **URL:** https://app.gohighlevel.com (client gives you login).  
- **Where you work:** 100% in the **browser**. No IDE.  
- **Docs:** https://help.gohighlevel.com  

**Step-by-step (Step A + B trigger):**

1. Log in → select the correct sub-account (agency may have multiple).
2. **Form (Step A):**  
   - **Marketing → Websites** or **Funnels** → open site → add form, **or** **Settings → Form Builder** and create standalone form.  
   - Add fields: First Name, Last Name, Email, Phone, Legal Issue Type (dropdown), Preferred Date/Time, Notes, and MVA fields (Incident Date, Injury Type, Insurance Company, etc.).  
   - Save. Copy the **form ID** or embed code.
3. **Custom fields:**  
   - **Settings → Custom Fields** (Contacts). Create fields for: Call Notes, Appointment Date/Time, MVA Incident Date, MVA Injury Type, MVA Insurance, Pipeline Stage, etc.  
   - These will be filled by Make.com after the Retell call.
4. **Workflow (Step B trigger):**  
   - **Automation → Workflows** → Create new.  
   - **Trigger:** Form submitted → select your Contact Us form.  
   - **Actions:**  
     - Send contact data to Make.com: use **Webhook** action, URL = your Make.com webhook URL (from Scenario 1), method POST, body = contact + custom values (use merge tags).  
   - Optionally add internal steps (e.g. "Add to pipeline – New Lead").  
   - Turn workflow **On**.  
5. **Calendar:**  
   - **Settings → Calendars** → create or use existing calendar (for AI to "book" appointments).  
   - Ensure calendar is connected to the correct pipeline/location if needed.  
6. **Pipeline:**  
   - **Opportunities** or **Sales Pipeline** → create stage "Consult Scheduled".  
   - When you move a contact to this stage (from Make.com or manually), that can trigger Make.com Scenario 2 (via webhook from GHL when stage changes, or a second workflow that fires on stage change).

**API:** You don’t code GHL API in an IDE. Make.com has a **GoHighLevel** module (OAuth). You use it to update contact, create/update opportunity, and manage calendar events after the Retell call.

---

### 4.2 Make.com

- **What it is:** Visual automation (if X then Y). Connects GHL, Retell, Clio, SharePoint, OneDrive.  
- **URL:** https://www.make.com → sign in (client may invite you or give you an account).  
- **Where you work:** 100% in the **browser**. You drag modules and map data. No IDE.  
- **Docs:** https://www.make.com/en/help  

**Step-by-step (all three scenarios):**

**Scenario 1 — The Call**

1. **Create scenario** → name e.g. "GHL Form → Retell Call → Update GHL".  
2. **Trigger:** **Webhooks → Custom webhook** → create webhook → copy URL. Use this URL in the GHL workflow "Webhook" action.  
3. **Parse GHL payload:** Often GHL sends a nested object. Use **Tools → Set multiple variables** or **JSON → Parse JSON** to get `contact.name`, `contact.phone`, `contact.email`, custom fields.  
4. **Call Retell:** **HTTP → Make a request** (or Retell module if available):  
   - Method: POST  
   - URL: `https://api.retellai.com/v2/create-phone-call` (check Retell’s latest docs).  
   - Headers: `Authorization: Bearer YOUR_RETELL_API_KEY`, `Content-Type: application/json`.  
   - Body: `{ "agent_id": "...", "to_number": "+1234567890", "metadata": { "ghl_contact_id": "..." } }`.  
   - Use mapped values from step 3 for `to_number` and metadata.  
5. **Wait for call to end:** Add **Webhooks → Custom webhook** again (or use Retell’s "call ended" webhook URL in Retell dashboard pointing to this Make.com webhook).  
6. **Process result:** From webhook payload, take transcript and extracted MVA data.  
7. **Update GHL:** **GoHighLevel** module → Update contact (notes, custom fields). **GoHighLevel** → Create or update calendar event. **GoHighLevel** → Update opportunity / pipeline stage to "Consult Scheduled".  
8. **Error handling:** Add **Error handler** route; log or send yourself a message.  
9. Save and **Run once** (test with a real form submit).

**Scenario 2 — Hand-off to Clio + SharePoint**

1. **Trigger:** **Webhooks → Custom webhook**. This URL will be called when GHL opportunity moves to "Consult Scheduled" (either from a GHL workflow "when stage = Consult Scheduled → webhook" or from Scenario 1 at the end).  
2. **Clio:** **HTTP** or **Clio** module (if available). OAuth first time.  
   - Create Person: POST to Clio contacts API with name, email, phone from GHL.  
   - Create Pending Matter: POST to Clio matters API, link to Person, map MVA custom fields.  
3. **SharePoint:** **Microsoft 365** or **HTTP (Microsoft Graph)**. Create list item in the "Intake Tracker" list (site ID, list ID, and column names from client).  
4. Save and test by moving a test contact to "Consult Scheduled".

**Scenario 3 — Docs**

1. **Trigger:** **Microsoft 365 → Watch list items** (Intake Tracker list) **or** webhook if SharePoint can send one (often polling is easier).  
2. **Create folder:** **Microsoft 365** / **OneDrive** module → Create folder (path like `/Clients/{{ClientName}}/`).  
3. **Documents:** Use **Microsoft 365** to create/copy files, or use **HTTP** to Graph API. For filling Word templates, options: (a) Power Automate flow that Make.com triggers, or (b) Make.com HTTP to an endpoint that merges data into Word, or (c) simple template with placeholders replaced in Make (e.g. CSV → replace in text and upload).  
4. Save and test by adding a test item to Intake Tracker.

**APIs used inside Make.com:** Retell (HTTP), GHL (module/OAuth), Clio (HTTP + OAuth), Microsoft Graph (module or HTTP). You never write code in an IDE for these; you configure modules and map fields.

---

### 4.3 Retell AI

- **What it is:** Voice AI that makes/receives phone calls. You define what the AI says and what it can do (e.g. book appointment).  
- **URL:** https://www.retellai.com → dashboard.  
- **Where you work:** 100% in the **browser** (dashboard). No IDE. You may paste prompts in a text editor locally, then paste into Retell.  
- **Docs:** https://docs.retellai.com  

**Step-by-step:**

1. **Account:** Get API key and (if needed) a phone number from client. In dashboard: **API Keys** → create key; **Phone Numbers** → buy or connect number.  
2. **Create Agent:** **Agents** → Create new.  
   - **General:** Name (e.g. "Law Firm Intake"), voice, language.  
   - **Prompt:** System prompt that defines:  
     - Role (intake coordinator for law firm).  
     - Steps: greet, confirm name/contact, ask legal issue type, if MVA ask MVA questions (incident date, injury, insurance, etc.), ask preferred consultation time, then "book" and summarize.  
     - Instruction to output structured data at end (e.g. JSON or a fixed format) for MVA fields and appointment.  
   - **Tools / Function calling:** Add tools Make.com or GHL will use, or that Retell calls itself:  
     - e.g. `check_availability(date, time)` → you implement this (or stub) via Retell’s server or a small API you expose.  
     - e.g. `book_appointment(date, time, contact_id)` → same.  
     In the demo, you can start with "book_appointment" as a tool that returns success and let Make.com do the real GHL calendar create.  
   - **Webhook:** Set "Call ended" webhook URL = your Make.com Scenario 1 webhook (the one that receives call result). Retell will POST transcript and any extracted data there.  
3. **Test:** Use Retell’s "Test call" in the dashboard to your phone. Iterate on prompt until the agent asks the right questions and "books" (even if stub).  
4. **Connect to Make.com:** In Make.com Scenario 1, use `agent_id` from Retell and the "Create phone call" API so that when the form is submitted, Make.com triggers the outbound call to the lead.

**API:** Used **from Make.com** (HTTP module) to create the call. You don’t build a separate app in an IDE; you only need the API key and endpoint in Make.com.

---

### 4.4 Clio Manage

- **What it is:** Law practice management (contacts = People, cases = Matters).  
- **URL:** https://app.clio.com (or client’s Clio URL).  
- **Where you work:** You use Clio in the **browser** only to verify data. All changes are done **via Make.com** using Clio’s API.  
- **Docs:** https://docs.clio.com  

**In Make.com:**

1. **Connect Clio:** **Connections** → Add **Clio** (or HTTP + OAuth). Follow OAuth flow (client may need to approve app in Clio).  
2. **Create Person:** Use Clio module or HTTP POST to `/contacts` (or equivalent) with name, email, phone from GHL.  
3. **Create Pending Matter:** POST to matters API, link to contact ID, set type to "Pending" or equivalent, map custom fields (MVA data from GHL).  
4. **IDs:** Store Clio Person ID and Matter ID; send to SharePoint (e.g. matter ID in Intake Tracker) for Step D.

**API:** Clio REST API, OAuth 2.0. Used only inside Make.com.

---

### 4.5 SharePoint & OneDrive (Microsoft 365)

- **What it is:** SharePoint = sites and lists ("Intake Tracker"). OneDrive = document storage (client folders).  
- **URL:** https://www.office.com → SharePoint / OneDrive (client gives access).  
- **Where you work:** Browser to see lists/folders; **Make.com** does all create/update via **Microsoft Graph API** (Microsoft 365 connection).  
- **Docs:** https://learn.microsoft.com/en-us/graph/api/overview  

**In Make.com:**

1. **Connect:** **Connections** → **Microsoft 365** or **OneDrive** → sign in with client’s org account (or app-only if client sets up app).  
2. **Intake Tracker list:** Get site ID and list ID (from list URL or Graph explorer). Scenario 2: **Create list item** with columns: Client Name, Matter ID, Date, Status, etc.  
3. **Client folder:** Scenario 3: **Create folder** in the right OneDrive/SharePoint document library (e.g. `/Clients/John Doe/`).  
4. **Word docs:** Either use Power Automate (client side) triggered by Make.com, or simple merge in Make (replace placeholders in a text template and upload as .docx or PDF). Full Word merge often needs Graph or Power Automate.

**API:** Microsoft Graph. Used only inside Make.com (modules or HTTP).

---

## 5. Week 1 Execution Order (Step-by-Step)

**Day 1–2: Setup & Step A**

1. Get logins: GHL, Make.com, Retell, Clio (read), SharePoint/OneDrive (read).  
2. GHL: Create Contact Us form + all custom fields (including MVA).  
3. GHL: Create pipeline with stage "Consult Scheduled".  
4. GHL: Create workflow triggered by form submit; for now, webhook to a test URL (e.g. webhook.site) and confirm payload shape.  

**Day 3: Retell + Scenario 1 (core)**

5. Retell: Create agent, write prompt (greeting, MVA questions, booking intent), set "call ended" webhook.  
6. Make.com Scenario 1: Webhook (GHL) → parse → POST Retell create-call → webhook (Retell result) → parse → update GHL contact + pipeline stage (and calendar if ready).  
7. Test: Submit form → check Make run → check Retell call → check GHL contact updated.  

**Day 4: Clio + SharePoint (Scenario 2)**

8. Make.com Scenario 2: Trigger = webhook when "Consult Scheduled". Create Clio Person, then Pending Matter, then SharePoint list item. Map MVA fields to Clio.  
9. GHL: Ensure when Scenario 1 moves contact to "Consult Scheduled", it also calls Scenario 2’s webhook (or trigger Scenario 2 from Scenario 1’s end).  
10. Test end-to-end: Form → Call → Consult Scheduled → Clio + SharePoint.  

**Day 5: Docs (Scenario 3)**

11. Make.com Scenario 3: Trigger = new item in Intake Tracker. Create folder, then generate/fill Retainer + Initial Memo and save to folder.  
12. Test: Add test row to Intake Tracker (or run full flow) → check folder and docs.  

**Day 6–7: Test & Document**

13. Run 5+ full flows with dummy data. Fix errors.  
14. Record short demo video (form → call → Clio → folder → docs).  
15. Write 1-page "What we built" for client (for the separate client-facing doc).  

---

## 6. What You Don’t Need (To Avoid Confusion)

- **No IDE for the main flow:** Everything is in browser (GHL, Make.com, Retell, Clio, Microsoft).  
- **No coding of API from scratch:** Make.com and GHL modules/HTTP handle APIs.  
- **No RingCentral in Week 1:** That’s for the full project (call routing, availability).  
- **No LangChain or custom backend for the demo:** Keep it Make + Retell + GHL + Clio + Microsoft only.  

---

## 7. One-Page Reference: URLs & Where You Work

| What | URL | Where you work |
|------|-----|-----------------|
| GHL | https://app.gohighlevel.com | Browser only |
| Make.com | https://www.make.com | Browser only |
| Retell AI | https://www.retellai.com | Browser only |
| Clio | Client’s Clio URL | Browser (check); API via Make |
| SharePoint/OneDrive | https://www.office.com | Browser (check); API via Make |
| GHL help | https://help.gohighlevel.com | — |
| Make help | https://www.make.com/en/help | — |
| Retell docs | https://docs.retellai.com | — |
| Clio API | https://docs.clio.com | — |
| Microsoft Graph | https://learn.microsoft.com/en-us/graph/api/overview | — |

---

## 8. Payment & Timeline (Unchanged)

- **$48 CAD** = initial deposit (first 4 hours). Week 1 total ≈ **$480 CAD** (40 hrs × $12).  
- **Week 1:** Demo (Steps A–D). **Week 2:** Client reviews; you wait. **Weeks 3–10:** Full build if approved (bi-directional sync, RingCentral, etc.).  
- **Realistic full project:** ~8 weeks, 20–40 hrs/week, at $12 CAD/hr.  

---

## 9. Access Checklist (For You)

Before starting:

- [ ] GHL admin login  
- [ ] Make.com account (or invite)  
- [ ] Retell API key + phone number  
- [ ] Clio API access (OAuth app — client may need to create)  
- [ ] SharePoint "Intake Tracker" list name/site + OneDrive folder location  
- [ ] Microsoft 365 connection (Make.com) with permissions to create list items and folders  
- [ ] NDA signed, contract accepted  

---

## 10. Next Document (Client-Facing)

After this guide, the **next document** will be the one you send to the client to **start** the project. It will:

- List exact access and permissions they need to provide.  
- Confirm demo scope (Steps A–D).  
- Include a short timeline (Week 1 demo, Week 2 review).  
- Ask for one primary contact and preferred communication channel.  

Keep this implementation plan for yourself; use the next doc for client onboarding and kickoff.

---

**Document version:** 2.0 — Beginner-focused implementation guide  
**Last updated:** January 31, 2026  
**Status:** Pre-demo; use this to execute Week 1, then create client kickoff doc.
