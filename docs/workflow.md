# Workflow Reference — Rasamalai Meeting-to-Action MVP

Workflow ID: `ebk0jMAStBUcYLa8`
Status: **inactive** (`active: false`, `triggerCount: 0`) — never published.
Settings: `executionOrder: v1`, `timezone: Asia/Kolkata`, `binaryMode: separate`.
Size: 43 executable nodes + 1 canvas sticky note.

## Phase status

| Phase | Scope | Status |
|---|---|---|
| 1 | Telegram trigger, normalization, message type detection, transcript validation, rejection replies | Done |
| 2 | `.txt` upload, extension validation, file download, text extraction, failure handling, common payload | Done |
| 3 | AI Agent + NVIDIA Nemotron + Structured Output Parser + Think Tool, output validation | Done |
| 4 | Storage row preparation with deterministic ids (originally Google Sheets) | Done |
| 5 | Excel storage — one `.xlsx` per table | Done |
| 6 | Consolidated Telegram summary, message splitting, storage error handling | Done |

## Connection map

```
Telegram Trigger
  └─ Normalize Telegram Update
       └─ Message Type Detection
            └─ Transcript Validation
                 └─ Route by Transcript Status
                      ├─ 0 Ready         → Build Text Transcript Payload → Transcript Ready for AI
                      ├─ 1 Document      → Is Supported TXT File?
                      │                        ├─ true  → Telegram Get File
                      │                        │            ├─ success → Extract Text From TXT
                      │                        │            │              ├─ success → Build TXT Transcript Payload
                      │                        │            │              │              └─ Is TXT Transcript Usable?
                      │                        │            │              │                   ├─ yes → Transcript Ready for AI
                      │                        │            │              │                   └─ no  → Send Unreadable File Reply
                      │                        │            │              └─ error   → Send Unreadable File Reply
                      │                        │            └─ error   → Send Download Failed Reply
                      │                        └─ false → Send Unsupported File Reply
                      ├─ 2 Empty Message → Send Empty Message Reply
                      ├─ 3 Unsupported   → Send Unsupported Message Reply
                      └─ 4 Invalid Input → Send Invalid Input Reply

Transcript Ready for AI
  └─ AI Agent  (NVIDIA Nemotron Chat Model + Structured Output Parser + Think Tool)
       └─ Validate AI Output
            └─ Is AI Output Valid?
                 ├─ true  → Prepare Storage Records
                 │            └─ Build Meeting Row → Convert Meetings to Excel → Write Meetings Excel
                 │                 └─ Build Decision Rows → Convert Decisions to Excel → Write Decisions Excel
                 │                      └─ Build Action Item Rows → Convert Action Items to Excel → Write Action Items Excel
                 │                           └─ Build Open Question Rows → Convert Open Questions to Excel → Write Open Questions Excel
                 │                                └─ Build Telegram Summary → Send Telegram Summary
                 └─ false → Build Validation Error Response → Send Telegram Validation Error

Error outputs (onError: continueErrorOutput) on all four Convert and four Write nodes
  → Build Storage Failure Message → Send Telegram Storage Failure
```

## Storage layout

`Prepare Storage Records` emits one item holding the meeting row plus three row arrays.
Each table is then written as its own workbook:

```
<meeting_id>_Meetings.xlsx       sheet Meetings
<meeting_id>_Decisions.xlsx      sheet Decisions
<meeting_id>_ActionItems.xlsx    sheet ActionItems
<meeting_id>_OpenQuestions.xlsx  sheet OpenQuestions
```

Files are written to `<n8n data folder>/exports`. With the Docker bind mount used in
development that is `C:\Users\<user>\...\telegram n8n\exports`.

Identifiers:

```
meeting_id  = <chat_id>_<message_id>
decision_id = <meeting_id>_D<n>
task_id     = <meeting_id>_T<n>
question_id = <meeting_id>_Q<n>
```

A table with no rows still writes its file with correct headers plus one blank row, so
the write chain stays linear and every run produces the same four files.

## Validation performed

- Node configuration validated for every node added in Phases 1–6.
- Saved graph re-read: 43 executable nodes, no disconnected executable nodes, no dangling
  connection targets. The three LangChain sub-nodes are connected on non-`main`
  connection types, which is expected.
- `onError: continueErrorOutput` confirmed on all eight Convert/Write nodes; every error
  output is wired to `Build Storage Failure Message`.
- Telegram credential assigned to all 11 Telegram nodes.
- Code-node logic unit-tested outside n8n for: builders (real and empty payloads),
  summary formatting, chunking, write verification, and the failure message builder.
- Excel generation verified with n8n's own `convertJsonToSpreadsheetBinary`: correct
  sheet names, headers and rows, including the headers-only case.
- End-to-end test execution `#53` (external nodes pinned, no live messages):
  - all 25 nodes on the happy path executed, status `success`
  - four `.xlsx` files created and read back successfully
  - summary contained 4 action items, 2 flagged deadlines and 1 unassigned task
- Failure path exercised in execution `#52`: a blocked write produced the storage
  failure reply and **no** summary.

Known tooling warnings:

- Canvas top-level item ceiling warning (cosmetic, pre-existing).
- The canvas sticky note is not included in the exported JSON.

## How to test manually

1. Open the workflow in n8n and assign the Telegram credential if needed.
2. Click **Execute workflow** (this registers a temporary test webhook).
3. Send the bot:
   - a normal transcript as text,
   - the same transcript as `samples/sample-transcript.txt`,
   - a `.pdf`,
   - `/start`,
   - a sticker.
4. Inspect the execution:
   - `Telegram Get File` should show binary property `data`.
   - `Extract Text From TXT` should show `transcript` in its JSON.
   - `Transcript Ready for AI` should receive the six-field payload.
   - `Write … Excel` should show `fileName` in its JSON.
5. Open the `exports` folder and confirm the four workbooks.

## Import notes

`workflows/meeting-to-action.json` is a sanitized export:

- `credentials` blocks were removed on purpose — assign your own after import.
- Owner/project metadata was stripped from `shared`.
- Canvas sticky notes and node groups are not part of the export.
- Import via **Workflows → Import from File**, then re-select the Telegram credential.

To write files, start the container with:

```
-e N8N_BLOCK_FILE_ACCESS_TO_N8N_FILES=false
-e N8N_RESTRICT_FILE_ACCESS_TO=/home/node/.n8n/exports
```

## Open items before activation

- Workflow must stay inactive until end-to-end review is approved.
- Telegram webhook is registered only on activation (or temporary manual test).
- Files accumulate per meeting; there is no retention or cleanup policy yet.
