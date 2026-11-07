# Workflow Reference — Rasamalai Meeting-to-Action MVP

Workflow ID: `ebk0jMAStBUcYLa8`
Status: **inactive** (`active: false`, `triggerCount: 0`) — never published.
Settings: `executionOrder: v1`, `timezone: Asia/Kolkata`.

## Phase status

| Phase | Scope | Status |
|---|---|---|
| 1 | Telegram trigger, normalization, message type detection, transcript validation, rejection replies | Done |
| 2 | `.txt` upload, extension validation, file download, text extraction, failure handling, common payload | Done |
| 3 | AI Agent + NVIDIA Nemotron + Structured Output Parser | Not started |
| 4 | Google Sheets storage | Not started |
| 5 | Telegram group summary | Not started |
| 6 | End-to-end hardening / error workflow | Not started |

## Connection map

```
Telegram Trigger
  └─ Normalize Telegram Update
       └─ Message Type Detection
            └─ Transcript Validation
                 └─ Route by Transcript Status
                      ├─ 0 Ready         → Build Text Transcript Payload → Transcript Processing (Placeholder)
                      ├─ 1 Document      → Is Supported TXT File?
                      │                        ├─ true  → Telegram Get File
                      │                        │            ├─ success → Extract Text From TXT
                      │                        │            │              ├─ success → Build TXT Transcript Payload
                      │                        │            │              │              └─ Is TXT Transcript Usable?
                      │                        │            │              │                   ├─ yes → Transcript Processing (Placeholder)
                      │                        │            │              │                   └─ no  → Send Unreadable File Reply
                      │                        │            │              └─ error   → Send Unreadable File Reply
                      │                        │            └─ error   → Send Download Failed Reply
                      │                        └─ false → Send Unsupported File Reply
                      ├─ 2 Empty Message → Send Empty Message Reply
                      ├─ 3 Unsupported   → Send Unsupported Message Reply
                      └─ 4 Invalid Input → Send Invalid Input Reply
```

## Validation performed

- `validate_node_config` on the new Phase 2 nodes (`if`, `telegram file:get`, `extractFromFile`, `telegram sendMessage`): all valid.
- Saved graph re-read: 18 executable nodes, no disconnected nodes, no dangling connection targets.
- `onError: continueErrorOutput` confirmed on `Telegram Get File` and `Extract Text From TXT`; both error outputs wired.
- Telegram credential assigned to all 8 Telegram nodes.
- Logic simulation:

| Input | Result |
|---|---|
| `standup.txt` | download → TXT path |
| `STANDUP.TXT` | download → TXT path (extension lowercased) |
| `notes.pdf` | unsupported file reply |
| file without extension | unsupported file reply |
| valid pasted text | text path |
| sticker | unsupported message reply |
| extracted text < 30 chars | unreadable file reply |

Known tooling warnings:

- Canvas group ceiling warning (cosmetic). Only `Intake and Validation` is grouped. A TXT group was rejected by the group rules because the IF and download nodes have outward branches, so a single-exit frame is impossible.
- The Phase 1 canvas sticky note is not included in exported JSON.

## How to test manually

1. Open the workflow in n8n and assign the Telegram credential to the trigger if needed.
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
   - `Transcript Processing (Placeholder)` should receive the six-field payload.

## Import notes

`workflows/meeting-to-action.json` is a sanitized export:

- `credentials` blocks were removed on purpose — assign your own after import.
- Canvas sticky notes and node groups are not part of the export.
- Import via **Workflows → Import from File**, then re-select the Telegram credential.

## Open items before activation

- Phase 3 AI chain not built.
- Google Sheets credential and target sheet not configured.
- Workflow must stay inactive until end-to-end review is approved.
- Telegram webhook is registered only on activation (or temporary manual test).
