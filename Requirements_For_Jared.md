# Requirements to Start — Law Firm CRM Automation (Demo)

**To:** Jared  
**From:** Amir  
**Purpose:** Align on access and credentials so we can start the Week 1 demo build.

---

## Opening

Jared,

I’ve gone through the demo outline and your tech stack in detail and have a clear picture of the full workflow: form → AI call → GHL update → Clio + SharePoint → client folder and docs. I’ve mapped out how we’ll implement each step (including triggers, APIs, and where each tool is used) so we can move forward without confusion.

It’s been a day since we last spoke, and I used this time to plan the implementation end-to-end. The requirements below are based on that plan and on the tech stack you shared. If you’re open to it, I’m happy to discuss the tech stack too—if you’d prefer different tools for any part of the flow, we can adjust before we start.

Based on your tech stack, here’s what I need from you to move forward with the demo.

---

## 1. GoHighLevel (GHL)

| What I need | Why I need it |
|-------------|----------------|
| **Admin login** (or invite as admin) to the sub-account where the law firm’s CRM lives | To create the Contact Us form, workflows, custom fields, pipeline stages, and calendar. I’ll do all build in the browser (no code). |
| **Permissions:** Forms, Workflows, Custom Fields/Values, Calendars, Pipelines, Opportunities, Webhooks | So I can add the form, trigger a workflow on form submit, send data to Make.com, update contacts after the AI call, and move opportunities to “Consult Scheduled.” |

**How to provide:** Either send login credentials via a secure method you prefer, or add my email as an admin user to the correct GHL location/sub-account.

---

## 2. Make.com

| What I need | Why I need it |
|-------------|----------------|
| **Access to a Make.com account** where the scenarios will live (either your account or a shared team) | I’ll build three scenarios in the browser: (1) receive GHL form → trigger Retell call → update GHL, (2) on “Consult Scheduled” → create Clio Person + Pending Matter + SharePoint list item, (3) on new Intake Tracker item → create client folder and generate docs. All orchestration happens here. |
| **Enough operations** for the demo (e.g. 1,000+ per month) | Each form submit and each automation step consumes operations. We’ll run multiple test runs; I want to avoid hitting limits during the demo week. |

**How to provide:** Invite my email to your Make.com team/account with scenario edit rights, or share the login to the account you want used for this project.

---

## 3. Retell AI

| What I need | Why I need it |
|-------------|----------------|
| **API key** (from Retell dashboard) | Make.com will use this to trigger outbound calls when the form is submitted. I’ll use the “Create phone call” (or equivalent) API from a Make.com HTTP module. |
| **A phone number** (purchased or connected in Retell) for outbound calls | So the AI can call the lead’s number. I’ll configure the agent in the Retell dashboard (prompt, MVA questions, booking flow). |
| **Access to the Retell dashboard** (same account as the API key) | To create and edit the voice agent: system prompt, conversation flow, tools (e.g. book appointment), and the “call ended” webhook URL that sends transcript and extracted data back to Make.com. |

**How to provide:** API key from Retell (Settings/API), confirm the outbound number, and either invite my email to the Retell account or share dashboard access so I can configure the agent.

---

## 4. Clio Manage

| What I need | Why I need it |
|-------------|----------------|
| **Clio API access** so Make.com can create contacts and matters | When an opportunity moves to “Consult Scheduled,” Make.com will create a Person (contact) and a Pending Matter in Clio and map MVA custom fields from GHL to the matter. |
| **OAuth 2.0 setup:** Either an existing Clio API app (Client ID + Client Secret) or confirmation that you’ll create one and share the credentials | Make.com uses OAuth to connect to Clio. I’ll add the connection in Make.com; I need the app credentials (or a pre-connected Clio connection you’ve authorized). |
| **Sandbox / test environment** (if available) | So we can test Person + Pending Matter creation without touching live client data. If you only have production, I’ll use clearly marked test data. |

**How to provide:** Create an app in Clio’s developer portal (or use an existing one) and share Client ID and Client Secret securely, or authorize a Clio connection in Make.com and ensure I have access to that connection.

---

## 5. SharePoint — “Intake Tracker” list

| What I need | Why I need it |
|-------------|----------------|
| **Site URL** where the Intake Tracker list lives | Make.com will create a new list item here when a consultation is scheduled (Step C). I need the correct site so the Microsoft Graph API calls go to the right place. |
| **List name or List ID** (“Intake Tracker”) | To target the correct list in the API. I’ll create items with: client name, matter ID (from Clio), date, status, and any other columns you want. |
| **Permission** for the account/connection used by Make.com to create list items | The Microsoft 365 connection in Make.com (see below) must have access to this site and list. |

**How to provide:** Send the SharePoint site URL and confirm the list name. If the list doesn’t exist yet, tell me the columns you want and I can align the scenario to that structure.

---

## 6. OneDrive / SharePoint — client folders and documents

| What I need | Why I need it |
|-------------|----------------|
| **Exact folder location** where new client folders should be created (e.g. a specific SharePoint document library or OneDrive path) | In Step D, when a new item is added to Intake Tracker, Make.com will create a folder (e.g. `/Clients/[ClientName]/`) and save the generated Retainer and Initial Memo there. I need the root path and naming convention you want. |
| **Permission** for the same Microsoft 365 connection to create folders and upload files | So Make.com can create the folder and write the filled Word documents (or PDFs) into it. |
| **Word templates** (Retainer Agreement, Initial Memo) with placeholders you want filled, or confirmation that you’ll provide them soon | I’ll map lead/matter data (name, date, MVA fields, etc.) into the templates. If you prefer to share templates after I’ve built the flow, we can plug them in once ready. |

**How to provide:** Share the full path (e.g. site + library + folder) and confirm the Microsoft 365 connection used in Make.com has write access there. Templates can be shared via link or email when ready.

---

## 7. Microsoft 365 (Graph API) — single connection for SharePoint + OneDrive

| What I need | Why I need it |
|-------------|----------------|
| **One Microsoft 365 connection in Make.com** that can access both the Intake Tracker list and the client folder location | Make.com’s Microsoft 365/OneDrive/SharePoint modules use this connection for: (1) creating list items in Intake Tracker, (2) creating client folders, (3) uploading generated documents. One connection keeps permissions and setup simple. |
| **Either:** You create and authorize this connection in Make.com and I use it, **or** you create an Azure AD app (Client ID, Client Secret, Tenant ID) with the right Graph permissions and I add it in Make.com | Required Graph permissions typically include: Sites.ReadWrite.All (or equivalent for the target site), Files.ReadWrite.All (for folders and files). I can send the exact permission list once we confirm SharePoint/OneDrive structure. |

**How to provide:** Easiest: you log into Make.com, add the Microsoft 365 connection, authorize it with an account that has access to the SharePoint site and document library, and ensure I can use that connection. Alternative: you create an app in Azure AD, grant the permissions, and share Client ID, Client Secret, and Tenant ID securely.

---

## 8. Optional but helpful for Week 1

| What I need | Why I need it |
|-------------|----------------|
| **NDA / IP agreement** (if you’re sending it) | You mentioned NDA; I’m happy to sign. Send it when ready so we can start without delay. |
| **Preferred way to reach you** (Upwork messages, email, or both) and rough availability | So I can send quick updates, ask one-off questions, and align on any blockers without slowing the demo. |
| **Confirmation of demo scope** (Steps A–D only for Week 1; RingCentral and full bi-directional sync later) | So we’re aligned: Week 1 = form, AI call, GHL update, Clio + SharePoint + folder + docs. No RingCentral or advanced sync in the first week. |

---

## 9. What I don’t need for the demo

- **RingCentral** — not required for Week 1; we’ll need it for the full project (call routing, availability).
- **Production client data** — I’ll use dummy/test data for the demo. No real lead or matter data needed until you’re ready.
- **LangChain or custom code** — the demo will use GHL, Make.com, Retell, Clio, and Microsoft 365 only; no separate codebase.

---

## 10. Next step

Once I have the items above (or a clear timeline for any that are pending, like Clio app or templates), I’ll start with Step A (form + workflow in GHL) and Step B (Make.com + Retell), then wire up Clio and SharePoint (Steps C and D). I’ll keep you updated as each part is in place.

If anything here is unclear or you’d prefer to provide something in a different way, tell me and we’ll adjust.

Thanks,  
Amir

---

## Short message to send when you upload this document (e.g. in Upwork chat)

Copy and paste the message below when you attach this file in the Files section:

---

**Message:**

Hi Jared,

I’ve attached a short requirements document in the Files section. It’s in plain language and lists everything I need from you to start the demo (GHL, Make.com, Retell, Clio, SharePoint, Microsoft 365), with a brief reason for each so nothing is unclear.

I’ve done a full pass on the workflow and implementation plan, so the list is based on that. Once I have these in place (or a timeline for any that are pending), I’ll begin with the form and AI call flow and then connect Clio and the docs. Please take a look when you can and let me know if you’d like to adjust anything.

Thanks,  
Amir

---
