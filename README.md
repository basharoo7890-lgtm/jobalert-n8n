# 🚀 JobAlert — Automated DevOps Job Hunter

> نظام أتمتة ذكي يراقب LinkedIn كل 6 ساعات، يُصفّي وظائف DevOps المناسبة بالذكاء الاصطناعي، ويرسل تنبيهاً فورياً على Telegram — مع نظام مراقبة للأخطاء.

---

## 📌 Overview

**JobAlert** is an n8n automation workflow that:
- Fetches LinkedIn job listings via RSS feed every 6 hours
- Filters jobs relevant to DevOps/Cloud using keyword + tool matching
- Excludes senior-level positions automatically
- Summarizes each job with an AI agent (Groq / Llama 4 Scout)
- Sends a formatted Telegram alert with a one-tap apply button
- Logs all sent jobs to Google Sheets to prevent duplicates
- Reports any workflow failures via a separate Telegram alert

---

## 🔧 Tech Stack

| Component | Tool |
|---|---|
| Automation engine | n8n (self-hosted or cloud) |
| Job source | LinkedIn via [rss.app](https://rss.app) RSS feed |
| AI model | Groq API — `meta-llama/llama-4-scout-17b-16e-instruct` |
| Database | Google Sheets |
| Notifications | Telegram Bot API |
| Scripting | JavaScript (n8n Code node) |

---

## 🗺️ Workflow Architecture

```
Schedule Trigger (6h)
        │
        ▼
Read Stored Job IDs     LinkedIn RSS Feed
  (Google Sheets)  ──►  (rss.app feed)
                               │
                               ▼
                   Transform & Clean Job Data
                   (extract title, company, ID,
                    location, strip HTML)
                               │
                               ▼
                   Job Qualification Filter (JS)
                   ├─ keyword matching: devops, cloud, k8s...
                   ├─ tool matching: docker, terraform, aws...
                   ├─ exclude: senior, lead, manager, architect
                   └─ exclude: > 6 years experience
                               │
                               ▼
                      Cool-down Period (3s)
                          /         \
                         ▼           ▼
               Analyze & Summarize   Merge (branch 0)
               (Groq AI Agent)         │
                         \            /
                          ▼          /
                          Merge ◄───
                               │
                               ▼
                       Message Builder
                   (format Telegram message)
                               │
                               ▼
                  Telegram Alert Dispatcher
                  (inline keyboard: Apply Now 🚀)
                               │
                               ▼
                    Log Sent Jobs to DB
                      (Google Sheets)
                               │
                               ▼
                    Reset Error Count (JS)

─────── Error Flow (separate workflow) ──────────

Any node failure → Global Error Handler
                       │
                       ▼
               Telegram Error Alert
               (workflow name + error message + link)
```

---

## 📂 Files

```
├── workflows/
│   ├── Alert_Job.json              # Main workflow
│   └── Global_Error_Handler.json   # Error monitoring workflow
├── README.md
├── LICENSE
└── .gitignore
```

> ⚠️ **Before importing:** all `chatId`, `credentials.id`, and `instanceId` fields have been replaced with placeholders (`YOUR_TELEGRAM_CHAT_ID`, etc.). You must reconnect your own credentials after import — see Setup below.

---

## ⚙️ Setup

### 1. Prerequisites
- n8n instance (cloud or self-hosted)
- Telegram Bot token (via [@BotFather](https://t.me/BotFather))
- Groq API key (free at [console.groq.com](https://console.groq.com))
- Google account with Sheets access
- rss.app account with a LinkedIn job search RSS feed

### 2. Import Workflows
1. Open n8n → **Workflows** → **Import from file**
2. Import `workflows/Global_Error_Handler.json` first
3. Import `workflows/Alert_Job.json`

### 3. Configure Credentials
In n8n **Settings → Credentials**, add:

| Credential | Used by |
|---|---|
| `Telegram API` | Telegram Alert Dispatcher, Error Handler |
| `Google Sheets OAuth2` | Read Stored Job IDs, Log Sent Jobs to DB |
| `Groq API` | Groq Chat Model (LLM) |

### 4. Set Your RSS Feed
In the `Read Linkedin Jobs RSS2` node, replace the URL:
```
https://rss.app/feeds/YOUR_FEED_ID.xml
```
To create a feed: go to [rss.app](https://rss.app), create a LinkedIn job search RSS, copy the feed URL.

### 5. Set Your Telegram Chat ID
In both Telegram nodes, set `chatId` to your Telegram user ID.
Get it by messaging [@userinfobot](https://t.me/userinfobot).

### 6. Configure Google Sheet
Create a Google Sheet named `JobAlert` with columns:
```
Job_ID | Job_Title | Link | Published_Date | Company | Location | SentAt | Source | Application_Status | AI_Analysis
```
Copy the Sheet ID from the URL and update both Google Sheets nodes.

### 7. Activate
1. Activate `Global_Error_Handler` first
2. Activate `Alert_Job` (v2)

---

## 🧠 AI Filtering Logic

The `Job Qualification Filter` node runs a multi-layered JavaScript filter:

```
PASS if: (has DevOps concept OR has DevOps tool)
       AND (no senior keywords)
       AND (no experience requirement > 6 years)
       AND (job ID not already in database)
```

**Concept keywords:** `devops`, `cloud`, `sre`, `platform`, `infrastructure`, `automation`, `ci/cd`

**Tool keywords:** `kubernetes`, `docker`, `terraform`, `ansible`, `jenkins`, `gitlab ci`, `aws`, `azure`, `gcp`, `linux`, `bash`, `python`

**Excluded seniority words:** `senior`, `lead`, `manager`, `head`, `architect`, `principal`

---

## 📩 Sample Telegram Message

```
المسمى الوظيفي: Junior DevOps Engineer.

الشركة: 🏢 Acme Corp.

ملخص الذكاء الاصطناعي:
الخبرة المطلوبة: خبرة مبتدئة، 1-2 سنوات
أهم التقنيات: AWS, Docker, Linux, Terraform
طبيعة الدور: أتمتة البنية التحتية وإدارة pipelines

الرابط: https://linkedin.com/jobs/view/...

[ التقديم الآن 🚀 ]
```

---

## 🔴 Error Handling

The `Global_Error_Handler` workflow (ID: `NMP14cw2WPp83gXU`) is linked in the main workflow's settings:
```json
"errorWorkflow": "NMP14cw2WPp83gXU"
```

Any failure in the main workflow triggers an immediate Telegram message containing:
- Workflow name
- Error message
- Direct link to the failed execution in n8n

---

## 🔮 Roadmap

- [ ] Add Gulf market RSS sources (Bayt, Naukrigulf)
- [ ] Support Arabic job titles in filter keywords
- [ ] Add `intermediate` / `junior` keyword boosting
- [ ] Weekly digest mode (batch alerts)
- [ ] Application tracker integration (update status in Sheets via Telegram commands)

---

## 📄 License

MIT — free to use, modify, and share.

---

*Built with n8n · Groq · Telegram · Google Sheets*
