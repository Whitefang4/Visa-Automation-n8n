# 🛂 Visa Automation System

> WhatsApp-powered visa application processing — fully automated via n8n, AI, and Google Sheets.

[![n8n](https://img.shields.io/badge/n8n-Cloud-FF6D5A?style=flat-square&logo=n8n)](https://n8n.io)
[![Twilio](https://img.shields.io/badge/Twilio-WhatsApp-F22F46?style=flat-square&logo=twilio)](https://twilio.com)
[![Groq](https://img.shields.io/badge/Groq-AI%20Parsing-00A67E?style=flat-square)](https://groq.com)
[![Google Sheets](https://img.shields.io/badge/Google%20Sheets-Database-34A853?style=flat-square&logo=google-sheets)](https://sheets.google.com)
[![Live App](https://img.shields.io/badge/Live%20App-chatvisa--bridge-1A56DB?style=flat-square&logo=vercel)](https://chatvisa-bridge.lovable.app/)

---

## 📋 Overview

Visa Automation is a fully automated visa application processing system built on **WhatsApp**. Applicants simply message a WhatsApp number, upload their documents, and receive real-time status updates — no manual handling required for standard applications.

**What it does automatically:**
- ✅ Registers new applicants
- ✅ Requests and validates documents
- ✅ Parses documents using AI (Groq / LLaMA)
- ✅ Tracks payment status
- ✅ Notifies applicants when visa is ready

---

## 🌐 Live

| Resource | URL |
|----------|-----|
| Frontend | (https://chatvisa-bridge.lovable.app/) |
| WhatsApp Number | +1 415 523 8886 |
| Join Code | `join double-bite` |

---

## 🛠️ Tech Stack

| Layer | Technology |
|-------|------------|
| Workflow Engine | n8n Cloud |
| WhatsApp Gateway | Twilio Sandbox |
| AI Document Parsing | Groq (LLaMA via Basic LLM Chain) |
| Database | Google Sheets |
| Frontend | Lovable (React) |
| Trigger | Webhook (POST) |

---

## 🏗️ Architecture

```
WhatsApp User
     │
     ▼
Twilio (+1 415 523 8886)
     │  POST
     ▼
n8n Webhook ──► JavaScript Parser ──► Google Sheets Lookup
                                              │
                                    ┌─────────┴─────────┐
                               New User             Existing User
                                    │                     │
                               Append Sheet          Switch (Status)
                                    │                     │
                               Welcome SMS    ┌───────────┼───────────┬──────────┐
                                         AWAITING_DOC  doc_process  Payment  visa_avail
                                              │              │           │         │
                                         Request Doc    AI Parsing   Payment   Visa Ready
                                                         (Groq)      Request     SMS
                                                             │
                                                      Update Sheet
```

**Two parallel flows:**
- **Main Flow** — triggered by inbound WhatsApp messages via Twilio webhook
- **Admin Flow** — triggered by Google Sheets row updates (manual admin overrides)

---

## 🔄 Workflow Nodes

| # | Node | Purpose |
|---|------|---------|
| 1 | **Webhook** | Receives POST from Twilio on every incoming WhatsApp message |
| 2 | **Code in JavaScript** | Parses and normalizes the Twilio payload |
| 3 | **Get row(s) in sheet** | Looks up the sender's phone number in Google Sheets |
| 4 | **If** | Branches: new applicant vs existing |
| 5 | **Switch (Rules)** | Routes by status: `AWAITING_DOC`, `doc_process`, `Payment`, `visa_avail` |
| 6 | **Check Doc (HTTP Request)** | Calls external API to validate uploaded document |
| 7 | **lm_parsing** | First-pass AI parsing of document content |
| 8 | **Wait** | Pauses for async document processing |
| 9 | **If1** | Checks parsing result validity |
| 10 | **lm_parsing2 + Basic LLM Chain** | Deep AI analysis using Groq Chat Model |
| 11 | **Send SMS/WhatsApp** | Sends status update back to the applicant |
| 12 | **Update / Append Sheet** | Saves updated applicant status to Google Sheets |
| 13 | **Google Sheets Trigger** | Fires on admin sheet update → pushes notification to applicant |

---

## 📊 Application Status Flow

```
New Message
    │
    ├── NEW USER ──────────────► Append row → Send welcome message
    │
    └── EXISTING USER
              │
              ├── AWAITING_DOC ──► "Please upload your documents"
              │
              ├── doc_process ───► Check Doc API → AI Parse → Update status
              │
              ├── Payment ───────► Send payment instructions
              │
              └── visa_avail ────► "Your visa is ready!" 🎉
```

---

## 🚀 Setup & Deployment

### Prerequisites

- [ ] n8n Cloud account
- [ ] Twilio account with WhatsApp Sandbox
- [ ] Google Sheets with applicant data
- [ ] Groq API key
- [ ] Lovable frontend deployed

---

### Step 1 — Configure Twilio Sandbox

1. Login at [console.twilio.com](https://console.twilio.com)
2. Go to **Messaging → Try it out → Send a WhatsApp message → Sandbox Settings**
3. Set **"When a message comes in"**
4. Set method to **POST** → **Save**

> ⚠️ Make sure you use the **production URL** (`/webhook/`) not the test URL (`/webhook-test/`)

---

### Step 2 — Activate n8n Workflow

1. Open the workflow in n8n Cloud
2. Toggle to **Active / Published**
3. Confirm the Webhook node shows the production URL

---

### Step 3 — Test It

1. Open WhatsApp → message **+1 415 523 8886**
2. Send: `join double-bite`
3. Send any message
4. Check **n8n → Executions** — you should see a triggered run


---


## 📄 License

MIT — feel free to fork and adapt for your own automation use cases.

---
