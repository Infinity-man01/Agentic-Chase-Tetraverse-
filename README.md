# 🍮 Meeting-to-Action Bot for Student Teams

**Team:** Rasamalai  
**Track:** AI Agents & Automation

## 📌 Overview

Meeting-to-Action Bot is an AI-powered automation system designed to help student teams convert meeting discussions into structured summaries and actionable tasks.

Important decisions, responsibilities, and deadlines often get lost after team meetings. Our solution uses **n8n, AI agents, and Telegram** to automate meeting analysis, task extraction, and follow-up management.

Users can send meeting transcripts through a Telegram bot and receive structured meeting summaries along with trackable action items.

## 🎯 Key Features

- **Transcript Input:** Accept meeting transcripts through Telegram messages and `.txt` files.
- **AI-Powered Analysis:** Extract key decisions, action items, assigned owners, and deadlines.
- **Structured Storage:** Automatically record meetings, decisions, action items and open questions in Excel (`.xlsx`) files, one per table.
- **Meeting Summaries:** Send formatted meeting summaries directly to the Telegram group.
- **Unassigned Task Detection:** Identify tasks without assigned owners.
- **Ambiguity Handling:** Flag missing deadlines, unclear responsibilities, and incomplete information.

## ⚙️ Technology Stack

| Technology | Purpose |
|---|---|
| n8n | Workflow automation |
| Telegram Bot API | User interaction and notifications |
| AI / LLM | Meeting analysis and information extraction |
| Excel (`.xlsx`) | Structured storage, one file per table |
| JSON | Structured AI output and data exchange |

## 🔄 Workflow Architecture

1. A team member sends a meeting transcript through Telegram.
2. n8n receives and processes the transcript.
3. An AI agent analyzes the conversation.
4. Important decisions, action items, owners, and deadlines are extracted.
5. Incomplete or ambiguous information is flagged.
6. Decisions, action items and open questions are written to Excel files.
7. A formatted meeting summary is sent back to the Telegram group.

## 🏗️ Project Structure

```text
Meeting-to-Action-Bot/
│
├── docs/
│   ├── architecture.md
│   └── workflow.md
│
├── workflows/
│   └── meeting-to-action.json
│
├── samples/
│   └── sample-transcript.txt
│
├── screenshots/
│
└── README.md
```

## 🚀 Implementation Status

- [x] Telegram bot setup (bot credential configured)
- [x] n8n workflow configuration — Phases 1–6
- [x] AI-based transcript analysis (Phase 3)
- [x] Structured action item extraction (Phase 3)
- [x] Excel storage, one `.xlsx` per table (Phase 5)
- [x] Consolidated Telegram summary with message splitting (Phase 6)
- [x] Missing information, storage and processing error handling (Phase 6)
- [x] End-to-end testing (test execution `#53`, all writes verified)

## 📦 Current Build (Phases 1–6)

**Workflow:** `Rasamalai - Meeting-to-Action MVP` — **inactive, not published**

Built in n8n 2.46.4, timezone `Asia/Kolkata`.

- Telegram Trigger accepts pasted text and `.txt` documents.
- Every input is normalized to one transcript contract:
  `chat_id`, `user_id`, `message_id`, `source_type` (`text` | `txt`), `transcript`, `meeting_timestamp`.
- `AI Agent` (NVIDIA Nemotron + Structured Output Parser + Think Tool) returns
  summary, decisions, action items and open questions as JSON.
- `Validate AI Output` normalizes and validates that JSON: enums, owners, and
  deadline resolution (absolute dates, weekdays, relative phrases, vague wording).
- Storage writes four Excel files per meeting:
  `<meeting_id>_Meetings.xlsx`, `_Decisions.xlsx`, `_ActionItems.xlsx`, `_OpenQuestions.xlsx`.
- `Build Telegram Summary` verifies every write, then sends **one** consolidated
  summary (split at 3800 characters for Telegram's length limit).
- Storage failures are reported explicitly and are never presented as success.
- Rejections and failures are handled deterministically with Telegram replies:
  empty message, unsupported media, bot commands, too-short text, non-`.txt` file,
  download failure, unreadable or empty extracted text, invalid AI output.
- The AI Agent never sends Telegram messages and never writes files; those stay in
  deterministic nodes.

Exported workflow: [`workflows/meeting-to-action.json`](workflows/meeting-to-action.json)
(credential references removed before publishing).

- Architecture and data contract: [`docs/architecture.md`](docs/architecture.md)
- Node reference, validation log and test steps: [`docs/workflow.md`](docs/workflow.md)
- Test input: [`samples/sample-transcript.txt`](samples/sample-transcript.txt)

## 🧪 Expected Output

The system is designed to generate:

- Meeting summary
- Key decisions
- Action items with assigned owners
- Associated deadlines
- Unassigned tasks
- Open questions requiring clarification

## 🔮 Future Enhancements

- Individual task notifications to team members.
- Automated requests for assigning missing task owners.
- Direct transcript retrieval from Google Drive.
- Enhanced task tracking and follow-up automation.

## 👥 Team members

Aritra Pal, Tithi Panda, Pranjal Roy, Tiasha Dey 

Built as part of the AI Agents & Automation hackathon challenge.

---

*Turning meeting discussions into meaningful actions.*
