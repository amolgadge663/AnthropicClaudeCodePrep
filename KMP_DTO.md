# Question Bank — Data Contract & Kotlin Multiplatform DTOs

Reference for an AI agent (or developer) building the Android / iOS Kotlin Multiplatform
exam-prep app on top of the JSON question banks in this folder.

---

## 1. Data files

| Certification | File | Questions | single / multiple | Images | Size |
|---|---|---|---|---|---|
| Claude Certified Developer – Foundations | `CCDV-F/CCDV-F.questions.json` | 1,163 | 1,009 / 154 | 29 | 3.7 MB |
| Claude Certified Associate – Foundations | `CCAO-F/CCAO-F.questions.json` | 1,200 | 1,002 / 198 | 143 | 4.2 MB |
| Claude Certified Architect – Foundations | `CCAR-F/CCAR-F.questions.json` | 1,918 | 1,686 / 232 | 36 | 7.7 MB |
| Claude Certified Architect – Professional | `CCAR-P/CCAR-P.questions.json` | 747 | 615 / 132 | 101 | 2.8 MB |

* Each file is a **flat JSON array** of question records — no wrapper object.
* **All four files share exactly the same record schema and key order.** One DTO parses all of them.
* Every key is always present. Lists are `[]` when empty; `domain` is the only nullable field.
* Encoding UTF-8; text is plain (no HTML). `question` and `explanation` may contain `\n` line breaks — render them.
* Images are hosted under `https://amolgadge663.github.io/AnthropicClaudeCodePrep/<CERT>/images/<file>` and mirrored locally in `<CERT>/images/`.

---

## 2. Record schema

```json
{
    "id": "634",
    "certification": "CCDV-F",
    "type": "single",
    "domain": "Applications and Integration",
    "domain_inferred": false,
    "question": "A developer ... What should the developer change?",
    "options": [
        "Enable prompt caching so the server retains earlier turns for the session.",
        "Send the full alternating history of prior user and assistant messages with each request.",
        "Raise max_tokens so the model has room to recall earlier conversation turns.",
        "Add a session identifier header so the API links requests into one conversation."
    ],
    "answer": [
        "Send the full alternating history of prior user and assistant messages with each request."
    ],
    "explanation": "Correct option:\n\nSend the full alternating history ...",
    "option_feedback": [],
    "references": ["https://docs.anthropic.com/en/api/messages"],
    "images": [
        { "place": "explanation", "image": "https://amolgadge663.github.io/AnthropicClaudeCodePrep/CCDV-F/images/CCDV-F-0007_e1.png" }
    ]
}
```

| Key | JSON type | Nullable | Semantics |
|---|---|---|---|
| `id` | string | no | Sequential `"1"`, `"2"`, … **unique only within one certification file.** Use `(certification, id)` as the composite key, or prefix: `"$certification-$id"`. |
| `certification` | string | no | One of `"CCDV-F"`, `"CCAO-F"`, `"CCAR-F"`, `"CCAR-P"`. |
| `type` | string | no | `"single"` → exactly one correct option. `"multiple"` → 2 or more correct options (never more than 2 in current data, but do not hard-code). |
| `domain` | string \| null | **yes** | Exam domain label, normalised per certification (see §4). `null` = unknown. Treat `null` as "no domain filter match". |
| `domain_inferred` | boolean | no | `false` = tagged by the course author. `true` = guessed by a text classifier (CCDV-F ≈ 72 % accurate, CCAR-F ≈ 86 %). Show as "approx." in UI or ignore for strict domain statistics. |
| `question` | string | no | Question stem, plain text, may contain `\n`. |
| `options` | string[] | no | 4 or 5 answer choices in original author order. Letters A, B, C… are **not** stored; derive from index if needed. Safe to shuffle. |
| `answer` | string[] | no | The correct option **text(s)**, each guaranteed to be an exact element of `options`. 1 item for `single`, ≥2 for `multiple`. Compare by text equality, not index, so shuffling stays safe. |
| `explanation` | string | no | Full rationale, plain text with `\n`. Never empty. |
| `option_feedback` | string[] | no | `[]` **or** one string per option, index-aligned with `options` (an entry may be `""`). Present on ~2,900 questions. Show under each option in review mode when non-empty. |
| `references` | string[] | no | `[]` or absolute URLs (official docs, articles). Open externally. |
| `images` | object[] | no | `[]` or image attachments. Currently all have `place = "explanation"`, but `"question"` and `"option A"`, `"option B"`, … are valid values — handle generically. |
| `images[].place` | string | no | Where to render: `"question"`, `"explanation"`, or `"option <Letter>"`. |
| `images[].image` | string | no | Absolute HTTPS URL (PNG / WebP). Load with Coil 3 (Compose Multiplatform) or Kamel; cache to disk. |

---

## 3. Kotlin DTOs (kotlinx.serialization)

Place in `commonMain`. Dependencies: `org.jetbrains.kotlinx:kotlinx-serialization-json`.

```kotlin
package com.example.claudeprep.data.dto

import kotlinx.serialization.SerialName
import kotlinx.serialization.Serializable

@Serializable
data class QuestionDto(
    val id: String,
    val certification: String,
    val type: String,                       // "single" | "multiple"
    val domain: String? = null,
    @SerialName("domain_inferred") val domainInferred: Boolean = false,
    val question: String,
    val options: List<String>,
    val answer: List<String>,
    val explanation: String,
    @SerialName("option_feedback") val optionFeedback: List<String> = emptyList(),
    val references: List<String> = emptyList(),
    val images: List<QuestionImageDto> = emptyList(),
)

@Serializable
data class QuestionImageDto(
    val place: String,                      // "question" | "explanation" | "option A" ...
    val image: String,                      // absolute https URL
)
```

Parser configuration (tolerant to future additive changes):

```kotlin
val BankJson = Json {
    ignoreUnknownKeys = true
    isLenient = false
    coerceInputValues = true                // null list -> default emptyList()
}

fun parseBank(jsonText: String): List<QuestionDto> =
    BankJson.decodeFromString(ListSerializer(QuestionDto.serializer()), jsonText)
```

### 3.1 Domain model (recommended, keeps UI code typed)

```kotlin
package com.example.claudeprep.domain.model

enum class Certification(val code: String, val displayName: String, val fileName: String, val examQuestionCount: Int) {
    CCDV_F("CCDV-F", "Claude Certified Developer – Foundations", "CCDV-F.questions.json", 53),
    CCAO_F("CCAO-F", "Claude Certified Associate – Foundations", "CCAO-F.questions.json", 60),
    CCAR_F("CCAR-F", "Claude Certified Architect – Foundations", "CCAR-F.questions.json", 60),
    CCAR_P("CCAR-P", "Claude Certified Architect – Professional", "CCAR-P.questions.json", 63);

    companion object {
        fun fromCode(code: String): Certification = entries.first { it.code == code }
    }
}

enum class AnswerType { SINGLE, MULTIPLE;
    companion object { fun parse(raw: String) = if (raw == "multiple") MULTIPLE else SINGLE }
}

enum class ImagePlace { QUESTION, EXPLANATION, OPTION }

data class QuestionImage(val place: ImagePlace, val optionIndex: Int?, val url: String)

data class Question(
    val key: String,                        // "$certification-$id" — globally unique
    val id: String,
    val certification: Certification,
    val type: AnswerType,
    val domain: String?,
    val domainInferred: Boolean,
    val text: String,
    val options: List<String>,
    val correctAnswers: Set<String>,        // option texts
    val explanation: String,
    val optionFeedback: List<String>,       // empty or size == options.size
    val references: List<String>,
    val images: List<QuestionImage>,
)
```

Mapper:

```kotlin
fun QuestionDto.toDomain(): Question = Question(
    key = "$certification-$id",
    id = id,
    certification = Certification.fromCode(certification),
    type = AnswerType.parse(type),
    domain = domain?.takeIf { it.isNotBlank() },
    domainInferred = domainInferred,
    text = question,
    options = options,
    correctAnswers = answer.toSet(),
    explanation = explanation,
    optionFeedback = if (optionFeedback.size == options.size) optionFeedback else emptyList(),
    references = references,
    images = images.map { it.toDomain() },
)

private fun QuestionImageDto.toDomain(): QuestionImage = when {
    place == "question" -> QuestionImage(ImagePlace.QUESTION, null, image)
    place.startsWith("option ") -> QuestionImage(ImagePlace.OPTION, place.last() - 'A', image)
    else -> QuestionImage(ImagePlace.EXPLANATION, null, image)
}
```

### 3.2 SQLDelight table (optional offline store)

```sql
CREATE TABLE question (
    key               TEXT NOT NULL PRIMARY KEY,   -- "$certification-$id"
    id                TEXT NOT NULL,
    certification     TEXT NOT NULL,
    type              TEXT NOT NULL,               -- single | multiple
    domain            TEXT,
    domain_inferred   INTEGER NOT NULL DEFAULT 0,
    question          TEXT NOT NULL,
    options_json      TEXT NOT NULL,               -- JSON array of strings
    answer_json       TEXT NOT NULL,               -- JSON array of strings
    explanation       TEXT NOT NULL,
    option_feedback_json TEXT NOT NULL,            -- JSON array ("[]" when none)
    references_json   TEXT NOT NULL,
    images_json       TEXT NOT NULL
);
CREATE INDEX question_cert_domain ON question(certification, domain);
```

Store list fields as JSON strings and decode with the same `BankJson`; it keeps the schema identical to the file and avoids join tables for read-mostly data.

---

## 4. Domains per certification (exact `domain` strings)

**CCDV-F** (8): `Agents and Workflows`, `Applications and Integration`, `Claude Code`, `Eval, Testing, and Debugging`, `Model Selection and Optimization`, `Prompt and Context Engineering`, `Security and Safety`, `Tools and MCPs`

**CCAO-F** (7): `Configuration and Knowledge Management`, `Governance, Risk, and Responsible Use`, `Output Evaluation and Validation`, `Product and Model Selection`, `Prompting and Task Execution`, `Troubleshooting and Optimization`, `Workflow Integration and Solution Design`

**CCAR-F** (5): `Agentic Architecture and Orchestration`, `Claude Code Configuration and Workflows`, `Context Management and Reliability`, `Prompt Engineering and Structured Output`, `Tool Design and MCP Integration`

**CCAR-P** (7): `Claude Models, Prompting and Context Engineering`, `Developer Productivity and Operational Enablement`, `Evaluation, Testing and Optimization`, `Governance, Safety and Risk Management`, `Integration`, `Solution Design and Architecture`, `Stakeholder Communication and Lifecycle Management`

Derive the domain list at runtime (`questions.mapNotNull { it.domain }.distinct().sorted()`) rather than hard-coding, so a regenerated bank never breaks the UI.

---

## 5. Business rules the app should implement

1. **Mock exam generation** — sample `Certification.examQuestionCount` questions (53 / 60 / 60 / 63) uniformly at random from the chosen certification; optionally stratify by `domain` so each domain's share matches the bank. Shuffle `options` per question; keep `correctAnswers` as texts so shuffling is safe. Shuffle only once per attempt and persist the order for review.
2. **Scoring** — `single`: selected text == the single correct text. `multiple`: selected set == `correctAnswers` set (all-or-nothing, as in the real exams). Show required selection count for `multiple` ("Select 2").
3. **Review screen** — show `explanation`, then per-option `optionFeedback` if non-empty, then `references` as external links, then `images` with `place == EXPLANATION`. Images with `place == QUESTION` render under the stem; `OPTION` images under the matching option.
4. **Domain filter / practice by domain** — treat `domain == null` as excluded from any domain filter but included in "All". Optionally let the user hide `domainInferred == true` questions from domain statistics.
5. **Progress tracking** — key everything by `Question.key` (`"CCDV-F-634"`), never by `id` alone.
6. **Loading** — files are 2.8–7.7 MB; bundle in `commonMain/resources` (Compose Multiplatform resources) or download once from GitHub Pages and cache. Parse on `Dispatchers.Default`; parsing 1,900 records takes well under a second on-device.
7. **Forward compatibility** — keep `ignoreUnknownKeys = true`; new optional keys may be added, existing keys will not be renamed or removed.

---

## 6. Content notice

Question text, explanations and images originate from third-party Udemy practice-test courses and were exported from the owner's own enrolled account for personal study. They are **not** licensed for public redistribution; keep the hosting repository private or access-controlled and do not publish the app content outside personal use without the course authors' permission.
