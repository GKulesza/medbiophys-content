---
id: em_042
category: EM
difficulty: 2
type: open
version: 1
assets: []
tags: [electromedicine, ekg]
hint: Think about which electrode stabilises the amplifier reference.
estimatedMinutes: 4
---

## Question

Write one clear practice question for first-year medical biophysics students.

## Answer

Write a concise answer in plain language. State units where relevant.

---

### Field guide

| Field | Required | Purpose |
|-------|----------|---------|
| `id` | yes | Stable lowercase id (`em_042`). Same id in every locale translation. |
| `category` | yes | `EM`, `OP`, `RA`, `WL`, `MF`, or `FN` (fun/hidden). |
| `difficulty` | yes | Integer `0`–`5`. |
| `type` | yes | `open` today. Future: `multiple_choice`, `numeric`, `image_hotspot`, `graph`. |
| `version` | yes | Editorial version; bump when content meaning changes. |
| `assets` | no | Filenames in `tasks/{locale}/images/`, e.g. `[oscilloscope_01.png]`. |
| `tags` | no | Editorial labels for search/planning (not shown in app yet). |
| `hint` | no | Optional study nudge (validated, future UI). |
| `estimatedMinutes` | no | Authoring estimate for workload planning. |

### Authoring workflow

1. Copy this file into `tasks/_templates/drafts/`.
2. Fill frontmatter and both sections.
3. Run `python3 scripts/generate_tasks_content.py` to merge drafts into English and replicate to all locales.
4. Translate `question` / `answer` in each `tasks/{locale}/tasks.json` (Polish overrides can live in `tasks_content_seed.py` during early authoring).
5. Run `python3 scripts/validate_tasks_content.py`.
6. Run `python3 scripts/practice_content_report.py`.
7. Publish via normal content manifest workflow.
