# 🤖 AI Support Triage Agent

An AI-powered email support triage automation built with n8n and Google Gemini that automatically classifies, responds to, and escalates incoming support emails.

## 🚀 Live Status
![Status](https://img.shields.io/badge/Status-Live%20in%20Production-brightgreen)
![Platform](https://img.shields.io/badge/Platform-n8n%20Cloud-orange)
![AI](https://img.shields.io/badge/AI-Google%20Gemini%202.0%20Flash-blue)

---

## 📌 What It Does

Every support email that lands in the inbox goes through this automated pipeline:

1. **Monitors** a Gmail support inbox every minute via polling
2. **Filters** out automated/own emails to prevent loops
3. **Extracts** and cleans email content for AI processing
4. **Classifies** the email using Google Gemini AI into categories
5. **Routes** based on whether human review is required
6. **Auto-replies** to simple FAQ emails instantly without human intervention
7. **Escalates** complex issues to a human agent with full context
8. **Logs** every processed email to Google Sheets for tracking
9. **Reports** a daily analytics summary every morning at 8 AM

---

## 🏗️ Workflow Architecture

```
Gmail Trigger (polls every 1 minute)
        │
        ▼
Filter Out Sent/Own Emails (IF node)
        │
        ▼ (true branch only)
Extract & Clean Email Content (Code node)
        │
        ▼
AI Classification & Draft Response (HTTP Request → Gemini API)
        │
        ▼
Parse AI Response (Code node)
        │
        ▼
Route by Human Required (Switch node)
        │
        ├── Auto Reply Branch (requires_human: false)
        │         │
        │         ▼
        │   Send AI-Drafted Reply (Gmail → Reply to message)
        │         │
        │         ▼
        │   Append Row in Google Sheets
        │
        └── Human Required Branch (requires_human: true)
                  │
                  ▼
          Send Internal Alert Email (Gmail → Send message)
                  │
                  ▼
          Append Row in Google Sheets
```

---

## 🧠 AI Classification Categories

| Category | Description |
|---|---|
| `billing` | Payment, invoice, subscription questions |
| `technical` | Bugs, errors, integration issues |
| `general` | Pricing, FAQs, product information |
| `refund` | Refund and chargeback requests |
| `spam` | Irrelevant or automated emails |

## ⚡ Priority Levels

| Priority | Description |
|---|---|
| `high` | Urgent issues requiring immediate attention |
| `medium` | Standard support requests |
| `low` | General inquiries, no urgency |

---

## 🛠️ Tech Stack

| Tool | Purpose |
|---|---|
| **n8n Cloud** | Workflow automation platform |
| **Google Gemini 2.0 Flash** | AI email classification and response generation |
| **Gmail API (OAuth2)** | Email monitoring, auto-reply, and alert sending |
| **Google Sheets API** | Real-time logging and analytics storage |

---

## 📊 Performance Metrics

- ✅ Auto-resolves ~60% of incoming emails without human intervention
- ⚡ Average response time under 2 minutes
- 📬 Processes up to 10 emails per poll cycle
- 📈 Daily analytics summary delivered every morning at 8 AM
- 🔄 Running continuously with 99%+ uptime on n8n Cloud free tier

---

## 📁 Repository Structure

```
ai-support-triage-agent/
│
├── workflow.json              # Main triage workflow (import into n8n)
├── daily-summary.json         # Daily analytics workflow (import into n8n)
├── workflow-screenshot.png    # Visual of the n8n canvas
└── README.md                  # This file
```

---

## 🔧 How to Import & Run

### Prerequisites
- [n8n Cloud account](https://n8n.cloud) (free tier works)
- Gmail account with OAuth2 credentials
- [Google Gemini API key](https://aistudio.google.com) (free)
- Google Sheets document with these columns:
  `Timestamp | From | Subject | Category | Priority | AI Response Sent | Human Required | Resolution`

### Import Steps

1. Download `workflow.json` from this repository
2. Open your n8n instance
3. Click **"+"** → **"New Workflow"**
4. Click **"..."** menu → **"Import from file"**
5. Select `workflow.json`
6. Update the following credentials:
   - **Gmail OAuth2** → connect your Gmail account
   - **Gemini API key** → paste in HTTP Request node headers
   - **Google Sheets** → update with your Sheet URL
7. Click **"Publish"** to activate

---

## 🔐 Security Notes

- Gmail credentials stored as n8n OAuth2 credentials (never in workflow JSON)
- Gemini API key stored directly in HTTP Request headers (move to n8n credentials for production)
- Google Sheets access scoped to specific spreadsheet only
- No user data stored beyond Google Sheets log
- Workflow filters own emails to prevent reply loops

---

## 📈 What I Learned Running This in Production

- **Prompt specificity matters** — vague auto-reply rules caused Gemini to over-escalate. Explicit case lists fixed classification accuracy significantly.
- **Empty fields break downstream nodes** — Gmail's simplified output sometimes returns empty `from` fields for automated emails. Added fallback parsing logic in the Code node.
- **Boolean routing in n8n** — Switch node needs "Convert types where required" enabled when routing on boolean values parsed from JSON strings.
- **Free tier limits are real** — n8n Cloud free tier gives 1,000 executions/month. With polling every minute, a single email triggers ~5 node executions. Planning executions budget is important.

---

## 🗓️ Daily Summary Report (Separate Workflow)

A second workflow (`daily-summary.json`) runs every day at 8:00 AM and sends an email digest containing:

- Total emails processed
- Auto-resolved vs human-required count
- Automation rate percentage
- Priority breakdown (high / medium / low)

---
