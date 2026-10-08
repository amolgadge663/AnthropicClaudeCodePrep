# AnthropicClaudeCodePrep — Claude Certification Question Banks

Offline-friendly question banks for the four Anthropic Claude certification exams, compiled from
17 Udemy practice-test courses (87 practice tests) and normalised into **one JSON schema** so a
single Kotlin Multiplatform DTO can load any of them.

| Certification | Code | File | Questions | Images | Exam size |
|---|---|---|---|---|---|
| Claude Certified Developer – Foundations | `CCDV-F` | `CCDV-F/CCDV-F.questions.json` | 1,163 | 29 | 53 |
| Claude Certified Associate – Foundations | `CCAO-F` | `CCAO-F/CCAO-F.questions.json` | 1,200 | 143 | 60 |
| Claude Certified Architect – Foundations | `CCAR-F` | `CCAR-F/CCAR-F.questions.json` | 1,918 | 36 | 60 |
| Claude Certified Architect – Professional | `CCAR-P` | `CCAR-P/CCAR-P.questions.json` | 747 | 101 | 63 |
| **Total** | | | **5,028** | **309** | |

## Folder layout

```
AnthropicClaudeCodePrep/
├── README.md                      ← this file
├── KMP_DTO.md                     ← data contract + Kotlin DTOs / domain model / business rules
├── CCDV-F/
│   ├── CCDV-F.questions.json      ← flat JSON array of question records
│   └── images/                    ← CCDV-F-0007_e1.png … (referenced by absolute URL in the JSON)
├── CCAO-F/  (same shape)
├── CCAR-F/  (same shape)
└── CCAR-P/  (same shape)
```

Image URLs inside the JSON follow the folder layout exactly:
`https://amolgadge663.github.io/AnthropicClaudeCodePrep/<CERT>/images/<file>` — publish this folder as the
root of a GitHub Pages site (private/access-controlled repo, see *Content notice*) and every link resolves.

## Record format (identical in all four files)

```json
{
    "id": "361",
    "certification": "CCAO-F",
    "type": "single",
    "domain": "Prompting and Task Execution",
    "domain_inferred": false,
    "question": "A marketing associate sends Claude one combined request ... What should the associate do differently?",
    "options": ["Break the request into separate prompts ...", "Resend the same combined request ...", "Ask Claude to expand ...", "Ask Claude to focus only on ..."],
    "answer": ["Break the request into separate prompts ..."],
    "explanation": "CORRECT: Break the request into separate prompts ...",
    "option_feedback": [],
    "references": [],
    "images": [
        { "place": "explanation", "image": "https://amolgadge663.github.io/AnthropicClaudeCodePrep/CCAO-F/images/CCAO-F-0361_e1.png" }
    ]
}
```

* Every key is always present; lists are `[]` when empty; only `domain` can be `null`.
* `answer` holds the correct option **text(s)** (a list, so single- and multi-select share one type).
* `id` is unique only within a file — use `certification + id` as the global key.
* Full field semantics, Kotlin DTOs, enums, SQLDelight schema and app rules: see **[KMP_DTO.md](KMP_DTO.md)**.

## Domains

| Code | Domains |
|---|---|
| CCDV-F | Agents and Workflows · Applications and Integration · Claude Code · Eval, Testing, and Debugging · Model Selection and Optimization · Prompt and Context Engineering · Security and Safety · Tools and MCPs |
| CCAO-F | Configuration and Knowledge Management · Governance, Risk, and Responsible Use · Output Evaluation and Validation · Product and Model Selection · Prompting and Task Execution · Troubleshooting and Optimization · Workflow Integration and Solution Design |
| CCAR-F | Agentic Architecture and Orchestration · Claude Code Configuration and Workflows · Context Management and Reliability · Prompt Engineering and Structured Output · Tool Design and MCP Integration |
| CCAR-P | Claude Models, Prompting and Context Engineering · Developer Productivity and Operational Enablement · Evaluation, Testing and Optimization · Governance, Safety and Risk Management · Integration · Solution Design and Architecture · Stakeholder Communication and Lifecycle Management |

Labels were normalised across authors (`Domain 1:` prefixes removed, `&` → `and`).

## Data quality notes

* **Inferred domains.** Two source courses (the "Mock Exams 2026" series) carry no domain tags. Their
  613 questions (315 CCDV-F, 298 CCAR-F) were assigned a domain by a TF-IDF nearest-centroid classifier
  trained on the author-tagged questions of the same certification and are flagged `"domain_inferred": true`.
  Held-out accuracy: CCDV-F ≈ 72 %, CCAR-F ≈ 86 %. Treat them as approximate.
* **Explanations.** Where an author supplied only per-option rationale instead of a single explanation,
  `explanation` is the rationale of the correct option and `option_feedback` holds all of them.
* **Duplicates.** 5 questions with identical text to an earlier question in the same certification were dropped.
* **Validation.** Every record has 4–5 options, ≥1 answer that exactly matches an option, a non-empty
  explanation, and every image URL has a matching local file.
* **No personal attempt data** is included.

## Sources

| Code | Udemy course (slug) | Tests × Q |
|---|---|---|
| CCDV-F | practice-exams-claude-certified-developer-foundations-ccdv-f | 4 × 53 |
| CCDV-F | claude-certified-developer-foundations-practice-exam-pack | 6 × 53 |
| CCDV-F | claude-developer | 6 × 53 |
| CCDV-F | claude-certified-developer-foundations (Mock Exams 2026, domains inferred) | 6 × 53 |
| CCAO-F | claude-certified-associate-foundations-practice-exam-pack | 6 × 60 |
| CCAO-F | practice-exams-claude-certified-associate-foundations-ccao-f | 4 × 60 |
| CCAO-F | ccao-f-practice-exams-claude-associate-foundations-2026 | 6 × 60 |
| CCAO-F | claude-certified-associate-foundations-ccao-f-practice-exams | 4 × 60 |
| CCAR-F | anthropic-claude-certified-architect-3-full-practice-exams | 6 × 60 |
| CCAR-F | claude-certified-architect-foundations-5-practice-exams | 5 × 60 |
| CCAR-F | practice-exams-claude-certified-architect-foundations-ccar-f | 4 × 60 |
| CCAR-F | claude-certified-architect-ccaf-tests | 6 × 60 |
| CCAR-F | certified-claude-architect-foundations (Mock Exams 2026, domains inferred) | 6 × 50 |
| CCAR-F | claude-certified-architect-foundations-6-exam-simulations | 6 × 60 |
| CCAR-P | claude-certified-architect-professional-practice-exam-pack | 3 × 63 |
| CCAR-P | claude-certified-architect-professional-6-practice-exams-t | 6 × 63 |
| CCAR-P | ccar-p-claude-certified-architect-professional-mock-exams | 3 × 60 |

## Regenerating

The banks are derived from lossless per-course exports in `..\udemy_tests\<course-slug>\` (one `testN.json`
per practice test plus `raw\` API responses).

```powershell
# 1. (only when adding a course) export it — attaches to an Edge window with remote debugging enabled
& "C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe" --remote-debugging-port=9222 --user-data-dir="C:\#1_MyAIWork\udemy_tests\.edge_profile"
python C:\#1_MyAIWork\udemy_tests\udemy_scraper.py --cdp http://localhost:9222 <udemy-course-url>

# 2. rebuild all four question banks (+ domain inference, validation)
python C:\#1_MyAIWork\udemy_tests\build_bank.py
```

`build_bank.py` constants: `GIT_BASE` (image URL prefix), `CERTS` (slug → certification routing).
Changing a course set changes the sequential `id`s, so re-import rather than diff after a rebuild.

## Content notice

Questions, explanations and images are the intellectual property of the respective Udemy course authors and
were exported from the owner's own enrolled account for **personal study only**. Do not redistribute publicly;
keep the hosting repository private or access-controlled. If you intend any wider use, obtain the authors'
permission and seek appropriate advice first.
