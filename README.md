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
- **Task Tracking:** Automatically record action items in Google Sheets or Trello.
- **Meeting Summaries:** Send formatted meeting summaries directly to the Telegram group.
- **Unassigned Task Detection:** Identify tasks without assigned owners.
- **Ambiguity Handling:** Flag missing deadlines, unclear responsibilities, and incomplete information.

## ⚙️ Technology Stack

| Technology | Purpose |
|---|---|
| n8n | Workflow automation |
| Telegram Bot API | User interaction and notifications |
| AI / LLM | Meeting analysis and information extraction |
| Google Sheets | Action item storage and tracking |
| JSON | Structured AI output and data exchange |

## 🔄 Workflow Architecture

1. A team member sends a meeting transcript through Telegram.
2. n8n receives and processes the transcript.
3. An AI agent analyzes the conversation.
4. Important decisions, action items, owners, and deadlines are extracted.
5. Incomplete or ambiguous information is flagged.
6. Action items are stored in Google Sheets.
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

- [ ] Telegram bot setup
- [ ] n8n workflow configuration
- [ ] AI-based transcript analysis
- [ ] Structured action item extraction
- [ ] Google Sheets integration
- [ ] Telegram summary generation
- [ ] Missing information and error handling
- [ ] End-to-end testing

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
