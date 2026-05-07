# AI Hospital Management System

**An intelligent, multi-agent automation system for hospital operations using n8n, OpenClaw, and LLM-powered decision-making.**

---

## 📋 Project Overview

This project automates critical hospital workflows using AI agents built with n8n (workflow automation), OpenClaw (LLM agent framework), and OpenRouter (LLM provider). The system currently implements two core agents:

1. **Agent 1: Patient Triage** - Automated emergency assessment and priority routing
2. **Agent 2: Appointment Scheduler** - Intelligent booking system with conflict resolution (planned)

Additionally, an **OpenClaw-powered Q&A chatbot** provides patient support via Telegram.

---

## 🏗️ Architecture

### System Design

```
┌─────────────────────┐
│   Patient Input     │
│  (Web Form/Chat)    │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────────────────────────────────┐
│              n8n Workflow Engine                │
│                                                 │
│  ┌──────────────┐      ┌──────────────┐        │
│  │   Agent 1    │      │   Agent 2    │        │
│  │   Triage     │      │  Scheduler   │        │
│  └──────┬───────┘      └──────┬───────┘        │
│         │                     │                │
└─────────┼─────────────────────┼────────────────┘
          │                     │
          ▼                     ▼
┌─────────────────────┐  ┌──────────────┐
│  OpenRouter API     │  │ Google       │
│  (LLM Provider)     │  │ Sheets/      │
│                     │  │ Airtable     │
└─────────────────────┘  └──────────────┘
          │
          ▼
┌─────────────────────┐
│  Telegram Bot       │
│  Notifications      │
└─────────────────────┘
          +
┌─────────────────────┐
│  OpenClaw Agent     │
│  (Patient Q&A)      │
└─────────────────────┘
```

---

## 🎯 Agent 1: Patient Triage (WORKING)

### Workflow Logic

**Input:** Patient registration form with 6 fields
- Full Name (required)
- Age (required)
- Phone Number (required)
- Chief Complaint (required, textarea)
- Heart Rate BPM (optional)
- Blood Oxygen SpO2 % (optional)

**Processing:**
1. **Red Flag Detection** (Code Node)
   - Scans complaint text for 15+ life-threatening keywords
   - Keywords: chest pain, not breathing, unconscious, seizure, stroke, severe bleeding, cardiac arrest, etc.
   - If detected → immediate P1 priority, skip LLM

2. **IF Routing** (Conditional Node)
   - TRUE branch (red flag) → Set priority P1 → Telegram notification
   - FALSE branch (no red flag) → LLM triage → Priority assessment → Notification

3. **LLM Triage** (HTTP Request to OpenRouter)
   - Model: `nousresearch/hermes-3-llama-3.1-70b:free`
   - Analyzes complaint + vitals
   - Returns JSON: `{priority: "P1/P2/P3", reasoning: "...", recommendedAction: "..."}`

4. **Notifications** (Telegram Node)
   - P1 (Critical) - Immediate attention required
   - P2 (Urgent) - Attend within 4 hours
   - P3 (Routine) - Standard queue

**Current Status:** ✅ Red flag path working, ⏳ LLM path pending

### Priority Levels

| Priority | Description | Response Time | Trigger Conditions |
|----------|-------------|---------------|-------------------|
| P1 | Critical | < 5 minutes | Red flag keywords OR HR > 120 OR SpO2 < 92% |
| P2 | Urgent | < 4 hours | Moderate symptoms, stable vitals |
| P3 | Routine | Standard queue | Minor complaints, normal vitals |

---

## 🗓️ Agent 2: Appointment Scheduler (PLANNED)

### Features
- Patient-facing booking form
- Availability checker (hardcoded slots: 9AM-5PM, 30min intervals)
- Conflict detection
- LLM-powered alternative slot suggestions
- Database integration (Google Sheets/Airtable)
- Confirmation notifications via Telegram

**Status:** Not yet implemented

---

## 🤖 OpenClaw Patient Q&A Bot

### Purpose
Handles common patient queries via Telegram chat:
- "What's my appointment status?"
- "What are visiting hours?"
- "How do I prepare for my procedure?"
- "Can I reschedule my appointment?"

### Implementation
- **SOUL.md** - Agent identity and personality
- **SKILL.md** - Tools: `check_appointment_status`, `check_triage_status`
- **Integration** - Connects to same database as Agent 2

**Status:** Configured but not yet tested

---

## 🛠️ Tech Stack

| Component | Technology | Purpose | Reasoning |
|-----------|-----------|---------|-----------|
| **Workflow Engine** | n8n (Docker) | Visual workflow automation | No-code/low-code, easy integration, free |
| **LLM Provider** | OpenRouter | API gateway for LLMs | Unified interface, free tier available |
| **LLM Model** | Hermes 3 Llama 3.1 70B | Reasoning and decision-making | Free, good medical reasoning capability |
| **Agent Framework** | OpenClaw (v2026.5.6) | LLM agent orchestration | Modern agent framework, Telegram integration |
| **Notifications** | Telegram Bot API | Real-time alerts | Free, reliable, mobile push notifications |
| **Database** | Google Sheets/Airtable | Appointment storage | Free tier, API access, easy setup |
| **Runtime** | WSL2 Ubuntu 22.04 | Development environment | Docker support, Linux tooling |
| **Version Control** | Git/GitHub | Code repository | Standard for project tracking |

---

## 🚧 Issues Encountered & Solutions

### Issue 1: OpenClaw Safety Overrides
**Problem:** OpenClaw's built-in safety layers prevented medical triage instructions. When prompted to assess patient symptoms, it returned generic health disclaimers instead of structured JSON outputs.

**Impact:** 4+ hours lost attempting to bypass safety filters in SOUL.md and SKILL.md.

**Solution:** **Pivoted architecture.** Moved triage workflow to n8n (which has no safety overrides), reserved OpenClaw for patient Q&A chatbot (low-risk use case).

**Learning:** When working under tight deadlines, don't fight the framework—redesign around its constraints.

---

### Issue 2: n8n Code Node Syntax Changes
**Problem:** Documentation used outdated `$input.item.json` syntax, which threw `Unexpected identifier 'item'` errors.

**Solution:** Updated to current n8n syntax: `$input.first().json` for single-item access.

**Impact:** 15 minutes debugging, but critical for workflow execution.

---

### Issue 3: IF Node Boolean Comparison
**Problem:** Data from Code node returned boolean `true`, but IF node compared it to string `"true"`, causing all data to route through FALSE branch.

**Error Message:** `Wrong type: 'red true' is a string but was expecting a boolean`

**Solution:** Enabled "Convert types where required" toggle in IF node to auto-cast string to boolean.

**Learning:** Always check data types between nodes, especially when crossing from Code (JavaScript) to IF (n8n internal logic).

---

### Issue 4: Telegram Chat ID vs Phone Number
**Problem:** Entered phone number in Telegram node's "Chat ID" field. Received error: `Bad Request: chat not found`

**Solution:** Retrieved numeric Chat ID using `@userinfobot` on Telegram, replaced phone number with Chat ID.

**Learning:** Telegram Bot API requires chat_id (not phone), which is only obtainable after user messages the bot first.

---

### Issue 5: Edit Fields Node Empty Input
**Problem:** Set node (Edit Fields) showed "No fields - node executed, but no items were sent on this branch" despite data flowing through IF node.

**Root Cause:** IF node wasn't properly connected to Set node on the canvas (visual connection existed but data path was broken).

**Solution:** Deleted and recreated the connection from IF node's TRUE output to Set node.

**Learning:** Visual connections in n8n don't always guarantee data flow—test nodes individually after connecting.

---

## 📊 Current Progress

### Day 1 Achievements ✅
- [x] WSL2 + Ubuntu 22.04 environment setup
- [x] Docker installation and n8n deployment
- [x] OpenClaw installation and onboarding
- [x] OpenRouter API configuration
- [x] Telegram bot creation and pairing
- [x] Agent 1 patient intake form (6 fields)
- [x] Red Flag keyword detection (15 patterns)
- [x] IF routing logic (TRUE/FALSE branches)
- [x] P1 priority assignment and formatting
- [x] Telegram notification delivery (end-to-end tested)
- [x] GitHub repository initialization
- [x] Folder structure created (workflows, openclaw-agents, docs, assets)

### Day 1 Completion: ~75%
**Working:** Red flag detection → P1 assignment → Telegram alert  
**Pending:** LLM triage for non-red-flag cases

---

## 🗺️ Day 2 Roadmap

### Session 1: Complete Agent 1 (60-75 min)
- [ ] Add HTTP Request node to FALSE branch (call OpenRouter)
- [ ] Parse LLM JSON response (extract priority, reasoning)
- [ ] Add Telegram nodes for P2/P3 notifications
- [ ] End-to-end test: "headache, mild fever" → P2 → notification

### Session 2: Build Agent 2 (90-120 min)
- [ ] Create appointment scheduling form
- [ ] Implement availability checker (Code node with hardcoded slots)
- [ ] Add LLM slot suggestion logic
- [ ] Integrate Google Sheets for booking storage
- [ ] Send confirmation via Telegram

### Session 3: OpenClaw Integration (45-60 min)
- [ ] Write SOUL.md (patient support agent identity)
- [ ] Create SKILL.md with appointment/triage tools
- [ ] Test via Telegram: "When is my appointment?"

### Session 4: Documentation (30-45 min)
- [ ] Export n8n workflows as JSON
- [ ] Take screenshots (canvas, Telegram, forms)
- [ ] Update README with final architecture
- [ ] GitHub commit and push

**Total Estimated Time:** 4-5 hours

---

## 🚀 Demo Flow

### Scenario: Emergency Patient Registration

1. **Patient submits form**
   - Name: John Doe
   - Age: 65
   - Complaint: "Severe chest pain, sweating, pain in left arm"
   - HR: 140 BPM
   - SpO2: 88%

2. **Red Flag Detection**
   - Keyword match: "chest pain"
   - Vitals check: HR > 120, SpO2 < 92
   - Result: Immediate P1 priority

3. **Notification Sent**
   - Telegram message delivered to hospital staff phone
   - Content: "🚨 EMERGENCY TRIAGE ALERT - Patient: John Doe - Token: TRG-1234 - Priority: P1 - Complaint: Severe chest pain - Please attend within 5 minutes"

4. **Outcome**
   - Patient prioritized for immediate care
   - Zero manual triage time
   - Instant notification delivery

---

## 📁 Repository Structure

```
ai-hospital-management/
├── workflows/
│   ├── Agent1_Triage.json          # n8n export (working)
│   └── Agent2_Scheduler.json       # (pending)
├── openclaw-agents/
│   ├── triage-agent/
│   │   ├── SOUL.md                 # Agent identity (created but unused)
│   │   └── skills/
│   │       └── hospital-triage/
│   │           └── SKILL.md        # Triage procedure (created but unused)
│   └── qa-agent/
│       ├── SOUL.md                 # (pending)
│       └── SKILL.md                # (pending)
├── docs/
│   ├── architecture.md             # (pending)
│   └── api-schemas.md              # (pending)
├── assets/
│   ├── workflow-canvas.png         # (pending)
│   ├── telegram-notification.png   # (pending)
│   └── form-interface.png          # (pending)
└── README.md                       # This file
```

---

## 🔧 Setup Instructions

### Prerequisites
- Windows 11 with WSL2 enabled
- Docker Desktop installed
- Node.js 18+ (for OpenClaw)
- Git installed
- Telegram account

### Step 1: Install n8n
```bash
docker run -it --rm -p 5678:5678 -v n8n_data:/home/node/.n8n n8nio/n8n
```
Access at: `http://localhost:5678`

### Step 2: Install OpenClaw
```bash
npm install -g openclaw@latest
openclaw onboard
```

### Step 3: Configure OpenRouter
1. Get API key from openrouter.ai
2. In OpenClaw onboarding, select OpenRouter provider
3. Enter API key
4. Select model: `nousresearch/hermes-3-llama-3.1-70b:free`

### Step 4: Create Telegram Bot
1. Message `@BotFather` on Telegram
2. Send `/newbot`
3. Follow prompts to create bot
4. Copy bot token
5. Get Chat ID from `@userinfobot`

### Step 5: Import Workflow
1. In n8n, click **Import from File**
2. Select `workflows/Agent1_Triage.json`
3. Add Telegram credentials (bot token)
4. Update Chat ID in Telegram node
5. Activate workflow

---

## 🎓 Key Learnings

1. **Framework Selection Matters** - When deadlines are tight, choose tools that don't fight your use case. OpenClaw's safety features made it unsuitable for medical triage, but perfect for patient chat support.

2. **Visual ≠ Working** - In n8n, visual connections don't guarantee data flow. Always test nodes individually after connecting.

3. **Data Types Are Critical** - JavaScript boolean `true` ≠ n8n string `"true"`. Enable type conversion or handle casting explicitly.

4. **Red Flag Logic First** - Rule-based safety checks (keyword detection, vital thresholds) should run before LLM calls. Faster, cheaper, and more reliable for life-threatening cases.

5. **Free Tier Constraints Drive Design** - Using free models and services forced efficient architecture—no unnecessary LLM calls, minimal API requests, lightweight database.

---

## 📝 License

This project is built for educational purposes as part of a college assignment. Not intended for production medical use without proper validation and compliance certifications.

---

## 🙏 Acknowledgments

- **n8n** - Open-source workflow automation platform
- **OpenClaw** - Modern LLM agent framework
- **OpenRouter** - Unified LLM API gateway
- **Telegram** - Bot API for notifications
- **Anthropic** - Claude for development assistance

---

**Last Updated:** May 7, 2026  
**Status:** Agent 1 (75% complete), Agent 2 (planned), OpenClaw (configured)
