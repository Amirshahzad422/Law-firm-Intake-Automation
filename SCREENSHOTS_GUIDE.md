# Screenshots guide — What Jared shared and how to use them

**Purpose:** Understand what each screenshot is for, which platform it’s from, and how it ties into the Week 1 demo.

---

## 1. What Jared shared and for what

Jared sent **two groups** of screenshots:

1. **GHL demo setup** (pipeline, funnel, calendar) — So you know what’s already built in GHL and what you need to add (form, workflow, calendar embed).
2. **Clio production reference** (practice areas, matter numbering, custom field sets) — So you can **recreate the same structure in the dummy Clio account** and then wire Make.com to it. He said: *“You can add all these fields, practice areas, etc except for the last screenshot (custom field sets), I'm just showing you everything the current production Clio account has. In the next set of screenshots, I'll show you the only actual custom field sets you need to create for the demo.”*  
   The **custom field sets to create for the demo** are mainly **A1 General intake** and **C2 Civil intake** (ignore C1 Civil update). Z1 and Z2 are also shown; create them if useful for the demo.

So: **everything is for the demo** — either to align your build (GHL) or to replicate production Clio in the dummy account so Make.com can create Persons/Matters with the right fields.

---

## 2. By folder — What each screenshot is and how to use it

### Clio/ (10 screenshots — from **Clio Manage**, production Kimball Law)

| Screenshot | What it shows | Platform | How you use it in the demo |
|------------|----------------|----------|-----------------------------|
| **Clio_Practice_Areas_List_1.png** | List of practice areas (Adoption, Civil Litigation, Personal Injury, etc.). | Clio Manage | In **dummy Clio**: add the same practice areas (or the subset you need). When Make.com creates a Pending Matter, you’ll assign a practice area (e.g. Personal Injury). |
| **Clio_Practice_Areas_List_2.png** | Same list, different view/pagination. | Clio Manage | Same as above — reference for practice area names. |
| **Clio_Practice_Area_Enablement.png** | Which practice areas are Enabled/Disabled; Personal Injury is Primary. | Clio Manage | In dummy Clio: enable the practice areas you need for the demo (at least Personal Injury as primary if doing MVA/civil intake). |
| **Clio_Matter_Numbering.png** | Matter number format: `[year] - [matter number] - [client summary name] - [client first name]`. | Clio Manage | In dummy Clio: set the same matter numbering scheme so matters created by Make.com follow this format. |
| **Clio_Custom_Field_Sets_Table.png** | Table of custom field sets (A1 General intake, C1 Civil update, C2 Civil intake, Z1, Z2, etc.). | Clio Manage | **Reference** — shows what production has. For the demo you only need to **create** the sets Jared asked for (see below). |
| **Clio_Field_Set_A1_General_Intake.png** | Edit modal: A1 General intake — fields: *Narrative notes from intake*, *Matter subtypes by practice area*. Default = On. | Clio Manage | **Requirement.** In dummy Clio: create custom field set **A1 General intake fields** with these fields (text one-line or multi-line). When Make.com creates a Matter, attach this set and map GHL/Retell data (notes, subtype) into these fields. |
| **Clio_Field_Set_C1_Civil_Update.png** | Edit modal: C1 Civil update — *Date of incident*, *Update calls*, *Last client update*, *Damages notes*. | Clio Manage | **Ignore for demo.** Jared: *“You can ignore the C1 Civil update fields.”* |
| **Clio_Field_Set_C2_Civil_Intake.png** | Edit modal: C2 Civil intake — *Date of incident or accident*, *Time of incident/accident*, *Location of incident*, *Seatbelted*, *Brief description of incident*, *Injuries reported at intake*. | Clio Manage | **Requirement.** In dummy Clio: create **C2 Civil intake** with these fields. Retell should ask and collect these (if you have time); otherwise map from GHL custom fields. When Make.com creates the Matter, attach this set and populate these fields. |
| **Clio_Field_Set_Z1_All_Clients_Consults.png** | Edit modal: Z1 All clients and consults — *Conflict check status*, *Primary office location*, *Conflict check date*, *Communication preference*, *Consult booked*, *Client retained*, *Legal Aid Certificate*, etc. | Clio Manage | **Optional for demo.** Create in dummy Clio if you want Matters to have these fields; map from GHL/Retell where relevant (e.g. Consult booked = yes when “Consult Scheduled”). |
| **Clio_Field_Set_Z2_How_Did_You_Hear.png** | Edit modal: Z2 How did you hear about us — *Google/Search Engine*, *Referral*, *Facebook*, *Yellow Pages*, etc. | Clio Manage | **Optional for demo.** Create if the form or Retell captures “how did you hear about us” and you want it in Clio. |

**Summary:**  
- **Must create in dummy Clio:** Practice areas (at least Personal Injury), matter numbering, **A1 General intake**, **C2 Civil intake**.  
- **Ignore:** C1 Civil update.  
- **Optional:** Z1, Z2.  
- When **Make.com** creates a Person and Pending Matter, it should attach these custom field sets and map GHL/Retell data into them.

---

### GHL_Demo_Setup/ (4 screenshots — from **GoHighLevel**)

| Screenshot | What it shows | Platform | How you use it in the demo |
|------------|----------------|----------|-----------------------------|
| **GHL_MAIN_Pipeline.png** | MAIN Pipeline stages: New Leads → Attempting Contact → Intake Call Booked → Check w/ Lawyer → **Consult Scheduled** → Pending Engagement. | GHL | **Requirement.** Your workflow: form submit → add contact as opportunity in **New Leads**. After AI books appointment → move to **Consult Scheduled**. Make.com Scenario 2 triggers when status = Consult Scheduled. |
| **GHL_Kimball_Law_Funnel.png** | Kimball Law (PI) Funnel: Landing Page, Calendar Page, Thank You Page. Landing has “Book Your Free Consultation.” | GHL | **Requirement.** Create the **Contact Us / lead form** and connect it to this funnel (e.g. on Landing Page or as a step). The **Calendar Page** is where you embed the Intake Call calendar. |
| **GHL_Intake_Call_Calendar.png** | Intake Call — 1 hr, Round Robin, currently **Inactive**. | GHL | **Requirement.** Use this calendar type for “AI books appointment on GHL Calendar.” Activate it (or use an active clone). Wire Retell/Make.com to book **Intake Call** on this calendar when the lead confirms a time. |
| **Demo_Workflow_Outline.png** | Original demo workflow (Steps A–D): Form → Webhook → AI Call → GHL update → Consult Scheduled → Clio + SharePoint → Docs. | Reference | **Requirement.** This is the demo scope; use it to align GHL workflow, Make.com scenarios, and Retell behavior. |

---

### Access Platforms/ (3 screenshots)

| Screenshot | What it shows | Platform | How you use it |
|------------|----------------|----------|----------------|
| **Access_GHL_1.png** | GHL-related access (e.g. sub-account or dashboard). | GHL | Reference for where you log in and which location to use. |
| **Access_Make_2.png** | Make.com-related access (e.g. Kimball Law invite). | Make.com | Reference for accepting the Make invite and which org to use. |
| **Access_3.png** | Third access-related screenshot. | — | Reference for any other login or access. |

These are **for your help** (where to log in), not direct build requirements.

---

## 3. Platform summary

| Platform | Screenshots from | What you do in the demo |
|----------|------------------|---------------------------|
| **Clio Manage** | Clio/ folder | In **dummy Clio**: add practice areas, matter numbering, and custom field sets (A1, C2; optionally Z1, Z2). Make.com creates Person + Pending Matter and maps GHL/Retell data into these fields. |
| **GoHighLevel (GHL)** | GHL_Demo_Setup/ folder | Create form, workflow (form → webhook → pipeline), and use MAIN Pipeline + Kimball Law funnel + Intake Call calendar. Move opportunities to “Consult Scheduled” when the AI books. |
| **Make.com** | — (no screenshots; access only) | Build Scenario 1 (GHL → Retell → GHL), Scenario 2 (Consult Scheduled → Clio + SharePoint), Scenario 3 (Intake Tracker item → folder + docs). Use Clio screenshots to know which fields/sets to create and map. |
| **Retell AI** | — (Jared will share access) | Simple demo: call lead from funnel form, book Intake Call. If you have time: ask and collect **C2 Civil intake** fields so they can be sent to GHL and then to Clio. |

---

## 4. Are these only for your help or requirements for the demo?

| Type | What | Use |
|------|------|-----|
| **Requirements for the demo** | GHL pipeline stages, funnel steps, Intake Call calendar; Clio practice areas, matter numbering, **A1 General intake**, **C2 Civil intake**; demo workflow (Steps A–D). | You must build to these: same stages, same calendar, same field sets and mapping. |
| **Reference / context** | Clio custom field sets table; C1 Civil update (ignore); Z1, Z2 (optional); Access Platform screenshots. | Use to understand production and where to log in; only implement what’s needed for the demo. |

---

## 5. Quick checklist for your build

- [x] **GHL (Phase 1 done):** Contact Us form created → workflow (form submit → webhook; tested with webhook.site). Lead created in New Leads. Intake Call calendar active. Consult Scheduled stage exists. *Next:* Point webhook to Make.com Scenario 1 URL.
- [ ] **Clio (dummy):** Add practice areas (at least Personal Injury), set matter numbering, create A1 General intake and C2 Civil intake (and optionally Z1, Z2). Ignore C1.
- [ ] **Make.com:** Scenario 1 — GHL webhook → Retell call → update GHL (notes, custom fields, pipeline, calendar). Scenario 2 — Consult Scheduled → Create Person + Pending Matter in Clio (with A1/C2 sets and mapping) + SharePoint Intake Tracker. Scenario 3 — New Intake Tracker item → create folder + docs.
- [ ] **Retell:** Simple agent: book Intake Call for new leads. If time: collect C2 Civil intake fields and return structured data for GHL/Clio.
