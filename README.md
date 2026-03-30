# PitchCraftAI (AI Agent which can automate our entire cold outreaching.)

# 🤖 AI-Powered Cold Outreach Agent — n8n + Ollama

> Three-workflow automation that validates HR contacts, researches companies, and sends personalised AI-generated emails — all from a Google Sheets list.

---

## 🚀 Overview

This n8n automation handles the full cold outreach pipeline for a list of HRs stored in Google Sheets. It runs across **three separate workflows** — each handling a distinct stage — powered by **Ollama AI** for personalisation and **Quick Email Verification** (alongside other free tools) for email validation.

No generic templates. No manual research. No wasted sends.

---

## 🗂️ Workflow Breakdown

---

### Workflow 1 — Validate Emails

> Checks every HR's email address is active before any outreach happens, using multiple free validation tools iteratively.

**Nodes:**
```
Form → Google Sheets → Filter → Loop Over Items → HTTP Request (Email Validation) → Update Row in Sheet
```

**What it does:**
- Reads HR contacts from Google Sheets
- Filters to only unvalidated rows
- Loops through each contact and hits **Quick Email Verification** and other free validation tools iteratively to verify the email
- Falls back to the next tool if one doesn't return a confident result
- Updates the sheet with the validation result (valid / invalid)

---

### Workflow 2 — Research Workflow: Add Descriptions & Industries

> Enriches each HR record with the company's industry and description using AI.

**Nodes:**
```
Trigger → Google Sheets → Filter → SplitInBatches → AI Agent → Code in JavaScript → Update Row in Sheet
                                                          ↓
                                              Ollama Chat Model
                                              Simple Memory
                                              HTTP Request
```

**What it does:**
- Triggers when Workflow 1 completes (or manually)
- Reads the validated HR list from Google Sheets
- Filters for rows missing company data
- Splits into batches for safe processing
- Sends each company to an **AI Agent** backed by **Ollama Chat Model** with **Simple Memory**
- The agent uses an **HTTP Request** to fetch company info and generates a description + industry tag
- A **JavaScript code node** formats the output
- Updates the Google Sheet with enriched company data

---

### Workflow 3 — Send Personalised Emails to the HRs

> Composes and sends a personalised email to each HR using the enriched data.

**Nodes:**
```
Form Submission Trigger → Get Row(s) in Sheet → Loop Over Items → Message → Code in JavaScript → Wait → Gmail (Send a message)
```

**What it does:**
- Triggered via a form submission (candidate fills in their details)
- Fetches matching HR rows from Google Sheets
- Loops through each HR contact
- Constructs a personalised email body using the enriched company data from Workflow 2
- A **JavaScript code node** handles final formatting and merging
- A **Wait node** adds a controlled delay between sends to avoid spam filters
- **Gmail** sends the final personalised email to each HR

---

## 🛠️ Tech Stack

| Tool | Role |
|------|------|
| **n8n** | Workflow automation & orchestration |
| **Google Sheets** | HR contact list & central data store |
| **Quick Email Verification + free tools** | Email address validation (iterative) |
| **Ollama** | Local AI model for company research & icebreaker generation |
| **Gmail** | Email delivery |

---

## ⚙️ Setup

### Prerequisites

- [n8n](https://n8n.io/) — self-hosted or cloud
- [Ollama](https://ollama.ai/) — running locally or on a server
- Google Sheets with HR data (Name, Email, Company columns at minimum)
- Disify API access (free tier available at [disify.com](https://disify.com))
- Gmail account connected to n8n

### Steps

1. **Import all three workflow `.json` files** into your n8n instance via *Workflows → Import from File*

2. **Connect Google Sheets**
   - Authenticate with your Google account in n8n credentials
   - Point each Sheets node to your HR list spreadsheet

3. **Configure Email Validation**
   - Add your [Quick Email Verification](https://quickemailverification.com) API key to the HTTP Request node in Workflow 1
   - Endpoint: `https://api.quickemailverification.com/v1/verify?email={email}&apikey={your_key}`
   - Additional free validation tools are chained iteratively in the same loop for fallback coverage

4. **Set up Ollama**
   - Ensure Ollama is running and a model is pulled (e.g. `ollama pull llama3`)
   - Update the Ollama Chat Model node with your instance URL (default: `http://localhost:11434`)

5. **Connect Gmail**
   - Authenticate Gmail in n8n credentials
   - Verify the sender address in the Send a message node

6. **Run in order**
   - ▶️ **Workflow 1** — validate all emails first
   - ▶️ **Workflow 2** — enrich with company data
   - ▶️ **Workflow 3** — trigger via form to send emails

---

## 📋 Google Sheets Structure (Recommended)

| Column | Description | Populated By |
|--------|-------------|-------------|
| `Name` | HR contact name | Manual |
| `Email` | Email address | Manual |
| `Company` | Company name | Manual |
| `Email Valid` | true / false | Workflow 1 |
| `Industry` | Company industry tag | Workflow 2 |
| `Company Description` | AI-generated summary | Workflow 2 |
| `Email Sent` | Timestamp of send | Workflow 3 |

---

## 🗺️ Roadmap

- [ ] Auto-trigger chaining between all three workflows
- [ ] Multi-step follow-up sequences
- [ ] Reply detection & auto-pause sending
- [ ] Slack notification on HR replies

---

## 📄 License

MIT — use freely and build on it.

---

> *Cold emailing without context is dead. The bar's moved — this is how you stay ahead.*
