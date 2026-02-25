# LLM Evaluation Framework — Call Intelligence Tasks

> **Version 6.0 — February 2026**
> Both layers use `expected_outcome` (golden truth) as the judge's primary reference.
> **Layer 1 (Task Metrics):** score 0.0–1.0, compared against `expected_outcome`.
> **Layer 2 (Standard Benchmarks):** binary 0 or 1, optional, also uses `expected_outcome` as ground truth.

## How Scoring Works

| Layer | What it covers | Score type | Reference |
|---|---|---|---|
| Layer 1 — Task Metrics | QA, Entity, Text, Translation | 0.0 – 1.0 continuous | `expected_outcome` |
| Layer 2 — Standard Benchmarks | Faithfulness, Hallucination, Relevancy… | 0 or 1 binary | `expected_outcome` |

**Judge inputs per test case:** `transcript` · `model_output` · `expected_outcome` · `metric` · `config`

# Task 1 — QA Analysis

## What This Task Does

The model reads a **call transcript** and a **QA scorecard** (a list of questions like "Did the agent greet the customer?" or "Did the agent offer a resolution?"). For each question it must:
- Assign a **score** (e.g. 5/5 or 0/5)
- Give a **reason** quoting what happened in the call

The judge then compares the model's output to the `expected_outcome` — the correct score and reason that a human QA analyst would give.

## Metrics Overview

| # | Metric | Full Name | Score Type |
|---|---|---|---|
| 1 | Score Correctness | Question Score Accuracy | 0.0 – 1.0 (weight 70%) |
| 2 | Calibration Score | Score Gap Accuracy | 0.0 – 1.0 (weight 20%) |
| 3 | Reason Quality | Evidence-Backed Reasoning | 0.0 – 1.0 (weight 10%) |
| 4 | Structure Check | Response Structure Compliance | 0 or 1 (prerequisite) |
| 5 | False Positive Rate | Compliance False Pass Rate | % (blocker if > 3%) |

**Weights total: 70 + 20 + 10 = 100%**

## Metric 1 — Question Score Accuracy
**Weight: 70%**

**What it checks:** Out of all questions in the scorecard, how many did the model score correctly — matching the score in `expected_outcome`?

This is the most important metric. Getting the score right is the core job of the model.

**Score types:**
- `PASS_FAIL` — answer is either full marks or zero (e.g. agent either greeted or didn't)
- `SCORE` — partial marks allowed (e.g. 3/5 for a partially followed script)

For `SCORE` type, a small margin of ±0.5 on a 5-point scale is allowed.

```
Question Score Accuracy = questions_scored_correctly / total_questions
```

> **Example:** Scorecard has 20 questions. Model scores 18 correctly.
> 18 / 20 = **0.90**

| Rating | Score |
|---|---|
| Good | >= 0.95 |
| Acceptable | 0.90 – 0.95 |
| Fail | < 0.90 |

## Metric 2 — Score Gap Accuracy
**Weight: 20%**

**What it checks:** Even when the model doesn't get the exact score right, how far off is it on average — relative to what each question's scale actually allows? A model that is consistently "close" is better than one that swings wildly.

Since each question has a different `max_score`, the gap for each question is normalized by **its own** `max_score` before averaging. This makes a 1-point gap on a 2-point question (50% off) count much more than a 1-point gap on a 10-point question (10% off).

```
Per-question normalized gap = |model_score − expected_outcome_score| / question_max_score

Average Normalized Gap = mean of all per-question normalized gaps

Score Gap Accuracy = 1 − Average Normalized Gap
```

> **Example:**
>
> | Question | model_score | expected_score | max_score | Gap | Normalized Gap |
> |---|---|---|---|---|---|
> | Q1 | 4 | 5 | 5 | 1 | 1/5 = 0.20 |
> | Q2 | 1 | 2 | 2 | 1 | 1/2 = 0.50 |
> | Q3 | 8 | 9 | 10 | 1 | 1/10 = 0.10 |
>
> Average Normalized Gap = (0.20 + 0.50 + 0.10) / 3 = **0.267**
> Score Gap Accuracy = 1 − 0.267 = **0.733**

| Rating | Score Gap Accuracy | What it means |
|---|---|---|
| Good | >= 0.90 | Within 10% of each question's scale on average |
| Acceptable | 0.80 – 0.90 | Within 10–20% off on average |
| Fail | < 0.80 | Deviates more than 20% on average |

## Metric 3 — Evidence-Backed Reasoning
**Weight: 10%**

**What it checks:** Does the model's reason actually point to something real in the transcript? A good reason says *what happened and where* — not just a vague summary.

Two things are checked per reason:
1. **Has evidence** — references a specific moment, quote, or action from the transcript
2. **Is factual** — the cited moment actually exists and is accurately described

```
Evidence-Backed Reasoning = (reasons_with_evidence + factual_reasons) / (2 × total_reasons)
```

> **Example:** Model says "Agent greeted the customer at 00:12 by saying 'Good morning, how can I help?'" — traceable in the transcript and matches the `expected_outcome` reason → scores on both dimensions.
>
> If the model says "Agent was polite" with no reference → fails the evidence check.

| Rating | Score |
|---|---|
| Good | >= 0.90 |
| Acceptable | 0.80 – 0.90 |
| Fail | < 0.80 |

## ⚠ Blocker — Compliance False Pass Rate

**What it checks:** Did the model give full marks to a question where the `expected_outcome` is zero (the agent clearly failed)?

This is the most dangerous error — it lets a non-compliant agent appear compliant.

```
False Positive Rate = (wrongly_passed_questions / total_pass_fail_questions) × 100
```

> **Example:** Agent skipped the mandatory fraud warning. `expected_outcome` = 0. Model gives 5/5 → false positive.

| Rating | Rate |
|---|---|
| Good | <= 1% |
| Acceptable | 1% – 3% |
| Blocker | > 3% |

## ⚠ Prerequisite — Response Structure Compliance
**Score: 0 or 1**

**What it checks:** Did the model return its output in exactly the same structure and keys as `expected_outcome`?

```
Response Structure Compliance = 1  if structure matches expected_outcome exactly
                              = 0  if any key is missing, renamed, or extra
```

**Expected output structure:**
```json
{
  "questions": [
    {
      "question_id": "Q1",
      "score": 5,
      "max_score": 5,
      "type": "PASS_FAIL",
      "reason": "Agent confirmed account verification per script at 01:23."
    }
  ]
}
```

> **Pass (score = 1):** All keys present — `question_id`, `score`, `max_score`, `type`, `reason` → **1**
>
> **Fail (score = 0):** Returns `"reasoning"` instead of `"reason"`, or skips `"max_score"`, or adds `"confidence"` → **0**

| Score | Meaning |
|---|---|
| 1 | Structure valid — proceed to scoring |
| 0 | Structure invalid — output cannot be scored |

## Full QA Score Formula

| Metric | Weight |
|---|---|
| Question Score Accuracy | 70% |
| Score Gap Accuracy | 20% |
| Evidence-Backed Reasoning | 10% |
| **Total** | **100%** |

```
IF Response Structure Compliance = 0  →  QA Score = invalid

IF Response Structure Compliance = 1  →

QA Score = (Question Score Accuracy   × 0.70)
          + (Score Gap Accuracy        × 0.20)
          + (Evidence-Backed Reasoning × 0.10)

Range: 0.0 – 1.0
```

> **Example:**
> - Response Structure Compliance = 1 ✓
> - Question Score Accuracy = 0.93
> - Score Gap Accuracy = 0.73
> - Evidence-Backed Reasoning = 0.85
>
> QA Score = (0.93 × 0.70) + (0.73 × 0.20) + (0.85 × 0.10)
>           = 0.651 + 0.146 + 0.085 = **0.882**

# Task 2 — Entity Analysis

## What This Task Does

The model reads a **call transcript** and a **configured entity list** (keywords and topics the business wants to track — e.g. "refund", "loan", "billing"). For each transcript it must:
- Report which **keywords** from the configured list were spoken
- Report which **topics** from the configured list were discussed
- Report **only** what is in the configured list — nothing extra

The judge compares the model's detected entities against the `expected_outcome`.

## Metrics Overview

| # | Metric | Full Name | Score Type |
|---|---|---|---|
| 1 | Keyword Precision | Keyword Detection Precision | 0.0 – 1.0 |
| 2 | Keyword Recall | Keyword Detection Recall | 0.0 – 1.0 |
| 3 | Keyword F1 | Keyword Detection F1 | 0.0 – 1.0 **(weight 47%)** |
| 4 | Topic Precision | Topic Detection Precision | 0.0 – 1.0 |
| 5 | Topic Recall | Topic Detection Recall | 0.0 – 1.0 |
| 6 | Topic F1 | Topic Detection F1 | 0.0 – 1.0 **(weight 29%)** |
| 7 | Config Adherence | Configured Entity Compliance | 0.0 – 1.0 **(weight 24%)** |
| 8 | Hallucinated Entities | Fabricated Entity Count | Count (blocker if > 2) |
| 9 | Structure Check | Response Structure Compliance | 0 or 1 (prerequisite) |

**Weights total: 47 + 29 + 24 = 100%**

## Metric 1 — Keyword Detection Precision

**What it checks:** Of all the keywords the model reported, how many actually appear in `expected_outcome`? Catches over-detection.

```
Keyword Precision = keywords_also_in_expected_outcome / total_keywords_model_detected
```

> **Example:** Model detected ["hello", "refund", "cancel"]. `expected_outcome` has ["hello", "refund", "account"].
> 2 match. "cancel" does not. Precision = 2 / 3 = **0.67**

| Rating | Score |
|---|---|
| Good | >= 0.90 |
| Acceptable | 0.85 – 0.90 |
| Fail | < 0.85 |

## Metric 2 — Keyword Detection Recall

**What it checks:** Of all keywords in `expected_outcome`, how many did the model find? Catches under-detection.

```
Keyword Recall = keywords_also_in_expected_outcome / total_keywords_in_expected_outcome
```

> **Example:** `expected_outcome` has ["hello", "refund", "account"]. Model found ["hello", "refund", "cancel"].
> "account" was missed. Recall = 2 / 3 = **0.67**

| Rating | Score |
|---|---|
| Good | >= 0.90 |
| Acceptable | 0.85 – 0.90 |
| Fail | < 0.85 |

## Metric 3 — Keyword Detection F1
**Weight: 47%**

**What it checks:** Combined score of Precision and Recall. Penalises both over-detection and under-detection.

```
Keyword F1 = 2 × (Keyword Precision × Keyword Recall) / (Keyword Precision + Keyword Recall)
```

> **Example:** Precision = 0.67, Recall = 0.67 → Keyword F1 = **0.67**

| Rating | Score |
|---|---|
| Good | >= 0.90 |
| Acceptable | 0.85 – 0.90 |
| Fail | < 0.85 |

## Metric 4 — Topic Detection Precision

**What it checks:** Of all topics the model reported, how many appear in `expected_outcome`?

```
Topic Precision = topics_also_in_expected_outcome / total_topics_model_detected
```

> **Example:** Model detected ["billing", "refund_policy"]. `expected_outcome` has ["billing", "account_management"].
> Only "billing" matches. Precision = 1 / 2 = **0.50**

| Rating | Score |
|---|---|
| Good | >= 0.88 |
| Acceptable | 0.80 – 0.88 |
| Fail | < 0.80 |

## Metric 5 — Topic Detection Recall

**What it checks:** Of all topics in `expected_outcome`, how many did the model identify?

```
Topic Recall = topics_also_in_expected_outcome / total_topics_in_expected_outcome
```

> **Example:** `expected_outcome` has ["billing", "account_management"]. Model found ["billing", "refund_policy"].
> "account_management" missed. Recall = 1 / 2 = **0.50**

| Rating | Score |
|---|---|
| Good | >= 0.88 |
| Acceptable | 0.80 – 0.88 |
| Fail | < 0.80 |

## Metric 6 — Topic Detection F1
**Weight: 29%**

**What it checks:** Combined Precision + Recall for topics.

```
Topic F1 = 2 × (Topic Precision × Topic Recall) / (Topic Precision + Topic Recall)
```

> **Example:** Precision = 0.50, Recall = 0.50 → Topic F1 = **0.50**

| Rating | Score |
|---|---|
| Good | >= 0.88 |
| Acceptable | 0.80 – 0.88 |
| Fail | < 0.80 |

## Metric 7 — Configured Entity Compliance
**Weight: 24%**

**What it checks:** Did the model report *only* entities from the configured list? Extras = compliance violation.

```
Config Adherence Score = entities_from_configured_list / total_entities_model_detected
```

> **Example:** Config is ["refund", "account", "billing"]. Model reports ["refund", "loan"].
> "loan" is not in config. Score = 1 / 2 = **0.50** → blocker.

| Rating | Score |
|---|---|
| Good | 1.00 |
| Acceptable | 0.95 – 0.99 |
| Blocker | < 0.95 |

## ⚠ Blocker — Fabricated Entity Count

**What it checks:** Did the model report an entity absent from both the transcript and `expected_outcome`?

```
Fabricated Entity Count = entities in model output absent from both transcript and expected_outcome
```

> **Example:** Transcript never mentions "insurance". Config doesn't include it. Model detects "insurance" → fabricated.

| Count | Outcome |
|---|---|
| 0 | Pass |
| 1 – 2 | Warning |
| > 2 | Blocker — do not score |

## ⚠ Prerequisite — Response Structure Compliance
**Score: 0 or 1**

**What it checks:** Did the model return output with exactly the keys and structure in `expected_outcome`?

```
Response Structure Compliance = 1  if all required keys present, no extra keys
                              = 0  if any key missing, renamed, or extra
```

**Expected output structure:**
```json
{
  "detected_keywords": ["hello", "refund"],
  "detected_topics": ["billing", "account_management"],
  "valid_entity_set": ["hello", "account", "refund", "billing", "account_management"]
}
```

| Score | Meaning |
|---|---|
| 1 | Structure valid — proceed to scoring |
| 0 | Structure invalid — do not score |

## Full Entity Score Formula

| Metric | Weight |
|---|---|
| Keyword Detection F1 | 47% |
| Topic Detection F1 | 29% |
| Configured Entity Compliance | 24% |
| **Total** | **100%** |

```
IF Response Structure Compliance = 0  →  Entity Score = invalid
IF Fabricated Entity Count > 2       →  Entity Score = invalid

IF both checks pass  →

Entity Score = (Keyword F1             × 0.47)
             + (Topic F1               × 0.29)
             + (Config Adherence Score × 0.24)

Range: 0.0 – 1.0
```

> **Example:**
> - Response Structure Compliance = 1 ✓, Fabricated Entity Count = 0 ✓
> - Keyword F1 = 0.91, Topic F1 = 0.85, Config Adherence = 1.00
>
> Entity Score = (0.91 × 0.47) + (0.85 × 0.29) + (1.00 × 0.24)
>             = 0.428 + 0.247 + 0.240 = **0.915**

# Task 3 — Text Analysis

## What This Task Does

The model reads a **call transcript** and must produce three things:

1. **Sentiment** — label every sentence as positive, neutral, or negative
2. **Summary** — identify the call purpose, extract highlights, and fill structured fields
3. **Emotion** — identify dominant emotions and their percentage breakdown

The judge compares every field against the `expected_outcome`.

## Metrics Overview

| # | Metric | Full Name | Score Type |
|---|---|---|---|
| **Sentiment** | | | |
| 1 | Label Accuracy | Per-Sentence Sentiment Accuracy | 0.0 – 1.0 (weight 60% of Sentiment) |
| 2 | Sentiment Macro F1 | Class-Balanced Sentiment F1 | 0.0 – 1.0 (weight 40% of Sentiment) |
| B1 | Missing Labels | Missing Sentiment Label Count | Count (blocker if > 2) |
| **Summary** | | | |
| 3 | Call Purpose Accuracy | Call Intent Match | 0.0 / 0.5 / 1.0 (weight 30% of Summary) |
| 4 | Highlights Coverage | Highlight Recall Rate | 0.0 – 1.0 (weight 10% of Summary) |
| 5 | Highlights Factual Accuracy | Highlight Correctness Rate | 0.0 – 1.0 (weight 10% of Summary) |
| 6 | Extracted Info Completeness | Required Field Presence Rate | 0.0 – 1.0 (weight 20% of Summary) |
| 7 | Hallucination Score | Fabrication-Free Rate | 0.0 – 1.0 (weight 30% of Summary) |
| **Emotion** | | | |
| 8 | Top Emotion Match | Dominant Emotion Correctness | 0 or 1 |
| **Prerequisites** | | | |
| P1 | Structure Check | Response Structure Compliance | 0 or 1 |

**Sentiment weights: 60 + 40 = 100% | Summary weights: 30 + 10 + 10 + 20 + 30 = 100%**

## Part 1 — Sentiment Analysis
**Contributes 30% to Text Score**

### Metric 1 — Per-Sentence Sentiment Accuracy
**Weight: 60% of Sentiment Score**

**What it checks:** How many sentences did the model label correctly — matching `expected_outcome`?

```
Sentiment Accuracy = correctly_labelled_sentences / total_sentences
```

> **Example:** 40 sentences. Model labels 35 correctly. 35 / 40 = **0.875**

| Rating | Score |
|---|---|
| Good | >= 0.88 |
| Acceptable | 0.80 – 0.88 |
| Fail | < 0.80 |

### Metric 2 — Class-Balanced Sentiment F1
**Weight: 40% of Sentiment Score**

**What it checks:** Accuracy alone can be gamed — if 80% of sentences are neutral, a model that labels everything "neutral" scores 0.80 while understanding nothing. Macro F1 holds the model accountable for each class equally.

```
For each class (positive / neutral / negative):
  Precision = correctly predicted as this class / all predicted as this class
  Recall    = correctly predicted as this class / all actually this class
  F1        = 2 × (Precision × Recall) / (Precision + Recall)

Sentiment Macro F1 = (F1_positive + F1_neutral + F1_negative) / 3
```

> **Example:** 5 positive, 30 neutral, 5 negative. Model labels all 40 as "neutral".
> Accuracy = 0.75 but Macro F1 = (0 + 0.86 + 0) / 3 = **0.29** — exposes the model.

| Rating | Score |
|---|---|
| Good | >= 0.85 |
| Acceptable | 0.75 – 0.85 |
| Fail | < 0.75 |

### ⚠ Blocker — Missing Sentiment Label Count

**What it checks:** Did the model skip labelling any sentence that `expected_outcome` has a label for?

```
Missing Label Count = sentences where model label is null/missing but expected_outcome has a valid label
```

> **Example:** 50-sentence transcript. Model skips sentences 12 and 31 → count = 2 → warning.

| Count | Outcome |
|---|---|
| 0 | Pass |
| 1 – 2 | Warning |
| > 2 | Blocker |

### Sentiment Sub-Score

```
Sentiment Score = (Per-Sentence Accuracy × 0.60)
               + (Sentiment Macro F1     × 0.40)
```

## Part 2 — Summary Analysis
**Contributes 50% to Text Score**

### Metric 3 — Call Intent Match
**Weight: 30% of Summary Score**

**What it checks:** Does the model's `call_purpose` match `expected_outcome`?

```
1.0 = exact or semantically equivalent match
0.5 = partial — correct domain but incomplete intent
0.0 = wrong intent entirely → blocker
```

> **Example:** `expected_outcome`: "Customer requesting refund for duplicate charge"
> Model: "Customer asking about billing" → **0.5** (billing correct, refund specifics missed)
> Model: "Customer wants to upgrade plan" → **0.0** → blocker

| Score | Rating |
|---|---|
| 1.0 | Good |
| 0.5 | Acceptable |
| 0.0 | Blocker |

### Metric 4 — Highlight Recall Rate
**Weight: 10% of Summary Score**

**What it checks:** Of all highlights in `expected_outcome`, how many did the model capture?

```
Highlight Recall Rate = highlights_matched_to_expected_outcome / total_highlights_in_expected_outcome
```

> **Example:** `expected_outcome` has 4 highlights. Model captures 3. 3 / 4 = **0.75**

| Rating | Score |
|---|---|
| Good | >= 0.85 |
| Acceptable | 0.75 – 0.85 |
| Fail | < 0.75 |

### Metric 5 — Highlight Correctness Rate
**Weight: 10% of Summary Score**

**What it checks:** Of highlights the model returned, are they factually accurate vs `expected_outcome`?

```
Highlight Correctness Rate = factually_accurate_highlights / total_highlights_model_returned
```

> **Example:** Model returns 3 highlights. One says "Refund of $99" but `expected_outcome` says $49 → that highlight fails. 2 / 3 = **0.67**

| Rating | Score |
|---|---|
| Good | >= 0.90 |
| Acceptable | 0.80 – 0.90 |
| Fail | < 0.80 |

### Metric 6 — Required Field Presence Rate
**Weight: 20% of Summary Score**

**What it checks:** Are all required fields from `expected_outcome.call_extracted_info` present and non-empty?

```
Required Field Presence Rate = fields_present_and_non_empty / total_required_fields_in_expected_outcome
```

> **Example:** `expected_outcome` requires 4 fields. Model fills 3, skips `follow_up`. 3 / 4 = **0.75**

| Rating | Score |
|---|---|
| Good | >= 0.90 |
| Acceptable | 0.75 – 0.90 |
| Fail | < 0.75 |

### Metric 7 — Fabrication-Free Rate
**Weight: 30% of Summary Score**

**What it checks:** Did the model introduce any facts absent from both the transcript and `expected_outcome`?

```
Hallucination Rate = (fabricated_facts / total_facts_in_model_summary) × 100
Fabrication-Free Rate = 1 − (Hallucination Rate / 100)
```

> **Example:** Model says "Agent offered a 10% discount" — absent from both transcript and `expected_outcome` → fabricated.

| Rating | Hallucination Rate | Fabrication-Free Rate |
|---|---|---|
| Good | 0% | 1.00 |
| Acceptable | 1 – 2% | 0.98 – 1.00 |
| Fail | > 2% | < 0.98 |
| Blocker | > 3% | < 0.97 |

### Summary Sub-Score

```
Summary Score = (Call Intent Match            × 0.30)
             + (Highlight Recall Rate         × 0.10)
             + (Highlight Correctness Rate    × 0.10)
             + (Required Field Presence Rate  × 0.20)
             + (Fabrication-Free Rate         × 0.30)
```

**Weights: 30 + 10 + 10 + 20 + 30 = 100%**

## Part 3 — Emotion Analysis
**Contributes 20% to Text Score**

### Metric 8 — Dominant Emotion Correctness

**What it checks:** Did the model identify the correct top emotion for the call?

```
Dominant Emotion Correctness = 1  if model's top emotion matches expected_outcome's top emotion
                             = 0  if it does not
```

> **Example:** `expected_outcome` top emotion = anger. Model says neutral → **0**

| Score | Meaning |
|---|---|
| 1 | Top emotion correct |
| 0 | Top emotion wrong |

```
Emotion Score = Dominant Emotion Correctness
```

## ⚠ Prerequisite — Response Structure Compliance
**Score: 0 or 1**

**What it checks:** Did the model return output with exactly the keys and structure in `expected_outcome`?

```
Response Structure Compliance = 1  if all required keys present, no extra keys
                              = 0  if any key missing, renamed, or extra
```

**Expected output structure:**
```json
{
  "sentiment": [
    { "sentence_id": 1, "text": "...", "label": "negative" },
    { "sentence_id": 2, "text": "...", "label": "neutral" }
  ],
  "summary": {
    "call_purpose": "Customer requesting refund for duplicate charge",
    "highlights": ["Agent confirmed duplicate charge", "Refund initiated for $49"],
    "call_extracted_info": {
      "customer_name": "Priya Sharma",
      "issue_type": "Billing",
      "resolution": "Refund approved",
      "follow_up": "Credit in 3–5 business days"
    }
  },
  "emotion": {
    "anger": 60,
    "neutral": 30,
    "joy": 10,
    "fear": 0,
    "sadness": 0
  }
}
```

| Score | Meaning |
|---|---|
| 1 | Structure valid — proceed to scoring |
| 0 | Structure invalid — do not score |

## Full Text Score Formula

| Sub-score | Metrics | Weight in Text Score |
|---|---|---|
| Sentiment Score | Label Accuracy (60%) + Macro F1 (40%) | 30% |
| Summary Score | Intent (30%) + Highlights (20%) + Fields (20%) + Fabrication (30%) | 50% |
| Emotion Score | Dominant Emotion Correctness | 20% |
| **Total** | | **100%** |

```
IF Response Structure Compliance = 0  →  Text Score = invalid
IF Call Intent Match = 0.0            →  Text Score = invalid (blocker)
IF Missing Label Count > 2            →  Text Score = invalid (blocker)
IF Hallucination Rate > 3%            →  Text Score = invalid (blocker)

IF all checks pass  →

Sentiment Score = (Per-Sentence Accuracy × 0.60)
               + (Sentiment Macro F1     × 0.40)

Summary Score   = (Call Intent Match           × 0.30)
               + (Highlight Recall Rate        × 0.10)
               + (Highlight Correctness Rate   × 0.10)
               + (Required Field Presence Rate × 0.20)
               + (Fabrication-Free Rate        × 0.30)

Emotion Score   = Dominant Emotion Correctness

Text Score      = (Sentiment Score × 0.30)
               + (Summary Score   × 0.50)
               + (Emotion Score   × 0.20)

Range: 0.0 – 1.0
```

> **Example:**
> - Structure = 1 ✓, Missing Labels = 1 ✓, Call Intent = 0.5 ✓, Hallucination Rate = 1.5% ✓
>
> **Sentiment Score:**
> Accuracy = 0.85, Macro F1 = 0.80
> = (0.85 × 0.60) + (0.80 × 0.40) = 0.510 + 0.320 = **0.830**
>
> **Summary Score:**
> Intent = 0.5, Highlight Recall = 0.80, Highlight Correctness = 0.90, Field Presence = 0.85, Fabrication-Free = 0.985
> = (0.5 × 0.30) + (0.80 × 0.10) + (0.90 × 0.10) + (0.85 × 0.20) + (0.985 × 0.30)
> = 0.150 + 0.080 + 0.090 + 0.170 + 0.296 = **0.786**
>
> **Emotion Score:**
> Top Emotion Match = 1 → **1.0**
>
> **Text Score:**
> = (0.830 × 0.30) + (0.786 × 0.50) + (1.0 × 0.20)
> = 0.249 + 0.393 + 0.200 = **0.842**

# Task 4 — Translation Analysis

## What This Task Does

The model receives a **call transcript** and must produce a **full translation** into the target language. Every sentence must be translated, domain terms preserved, and no facts added, changed, or dropped.

The judge compares the model translation against the `expected_outcome` — a human-verified reference translation.

## Metrics Overview

| # | Metric | Full Name | Score Type |
|---|---|---|---|
| 1 | Sentence Coverage | Translation Completeness Rate | 0.0 – 1.0 (weight 10%) |
| 2 | Semantic Equivalence | Sentence-Level Meaning Accuracy | 0.0 – 1.0 (weight 35%) |
| 3 | Fluency | Target Language Fluency Score | 0.0 – 1.0 (weight 10%) |
| 4 | Terminology Accuracy | Domain Term Preservation Rate | 0.0 – 1.0 (weight 20%) |
| 5 | Named Entity Preservation | Proper Noun Preservation Rate | 0.0 – 1.0 (weight 10%) |
| 6 | Meaning Drift Score | Critical Fact Preservation Rate | 0.0 – 1.0 (weight 15%) |
| P1 | Structure Check | Response Structure Compliance | 0 or 1 (prerequisite) |

**Weights total: 10 + 35 + 10 + 20 + 10 + 15 = 100%**

## Metric 1 — Translation Completeness Rate
**Weight: 10%**

**What it checks:** Did the model translate every sentence from the source? Silent skips are unacceptable.

```
Translation Completeness Rate = translated_sentence_count / total_source_sentences
```

> **Example:** 60 source sentences. Model translates 55. 55 / 60 = **0.917** — fails Good threshold.

| Rating | Score |
|---|---|
| Good | 1.00 |
| Acceptable | 0.95 – 1.00 |
| Fail | < 0.95 |

## Metric 2 — Sentence-Level Meaning Accuracy
**Weight: 35%**

**What it checks:** For each sentence, how similar is the model's translation to the `expected_outcome` sentence? Averaged across all sentences — more precise than document-level similarity, which can mask one badly mistranslated critical sentence.

```
Per-sentence score = cosine_similarity(
    multilingual_embedding(model_sentence),
    multilingual_embedding(expected_outcome_sentence)
)

Sentence-Level Meaning Accuracy = mean of all per-sentence scores
```

Use a multilingual embedding model (LaBSE or mUSE).

> **Example:**
>
> | Sentence | Similarity |
> |---|---|
> | S1 | 0.95 |
> | S2 | 0.88 |
> | S3 (mistranslated) | 0.42 |
> | S4 | 0.96 |
>
> Mean = (0.95 + 0.88 + 0.42 + 0.96) / 4 = **0.803** — S3 surfaces as the problem.

| Rating | Score |
|---|---|
| Good | >= 0.85 |
| Acceptable | 0.75 – 0.85 |
| Fail | < 0.75 |

## Metric 3 — Target Language Fluency Score
**Weight: 10%**

**What it checks:** Is the translated text grammatically correct and natural in the target language? Judge-evaluated score — semantic similarity cannot detect broken grammar.

```
1.0 = Grammatically correct, reads like a native translation
0.75 = Minor issues, clearly readable
0.50 = Noticeable errors, requires effort
0.25 = Significant issues, meaning obscured
0.0  = Incomprehensible
```

> **Example:** Model (Hindi): "एजेंट रिफंड 3-5 दिन पुष्टि किया।" — verb agreement wrong, grammatically broken → **0.25**

| Rating | Score |
|---|---|
| Good | >= 0.85 |
| Acceptable | 0.70 – 0.85 |
| Fail | < 0.70 |

## Metric 4 — Domain Term Preservation Rate
**Weight: 20%**

**What it checks:** Are regulatory codes, product names, and brand names handled exactly as `expected_outcome` handles them? These must not be semantically translated.

```
Domain Term Preservation Rate = correctly_handled_domain_terms / total_domain_terms_in_expected_outcome
```

> **Example:** `expected_outcome` keeps "KYC" as-is. Model writes "ग्राहक सत्यापन" (customer verification) → **incorrect**.

| Rating | Score |
|---|---|
| Good | >= 0.95 |
| Acceptable | 0.90 – 0.95 |
| Blocker | < 0.90 |

## Metric 5 — Proper Noun Preservation Rate
**Weight: 10%**

**What it checks:** Are customer names, agent names, and place names transliterated correctly per `expected_outcome`?

```
Proper Noun Preservation Rate = correctly_preserved_proper_nouns / total_proper_nouns_in_expected_outcome
```

> **Example:** `expected_outcome` has "प्रिया शर्मा" for "Priya Sharma". Model writes "प्रिया" (surname dropped) → **incorrect**.

| Rating | Score |
|---|---|
| Good | >= 0.95 |
| Acceptable | 0.90 – 0.95 |
| Fail | < 0.90 |

## Metric 6 — Critical Fact Preservation Rate
**Weight: 15%**

**What it checks:** Did the translation preserve all critical facts — numbers, dates, amounts, outcomes — exactly as in `expected_outcome`?

```
Meaning Drift % = (added_or_omitted_critical_facts / total_critical_facts_in_expected_outcome) × 100

Critical Fact Preservation Rate = 1 − (Meaning Drift % / 100)
```

> **Example:**
>
> | Fact in expected_outcome | Model translation |
> |---|---|
> | "refund of ₹49" | "refund of ₹49" ✓ |
> | "3–5 business days" | "7–10 days" ✗ |
> | "Agent escalated to supervisor" | "Agent resolved the issue" ✗ |
> | "Account number 9987" | "Account number 9987" ✓ |
>
> 2 of 4 drifted → Drift = 50% → Preservation Rate = **0.50** → blocker.

| Rating | Preservation Rate | Drift % |
|---|---|---|
| Good | 1.00 | 0% |
| Acceptable | 0.97 – 1.00 | 1 – 3% |
| Fail | 0.90 – 0.97 | 3 – 10% |
| Blocker | < 0.97 | > 3% |

## ⚠ Prerequisite — Response Structure Compliance
**Score: 0 or 1**

**What it checks:** Did the model return output with exactly the keys and structure in `expected_outcome`?

```
Response Structure Compliance = 1  if all required keys present, no extra keys
                              = 0  if any key missing, renamed, or extra
```

**Expected output structure:**
```json
{
  "full_translation": "...",
  "sentence_translations": [
    {
      "source_id": 1,
      "source_text": "Hello, how can I help you today?",
      "translated_text": "नमस्ते, मैं आपकी कैसे सहायता कर सकता हूँ?"
    }
  ],
  "domain_terms_handled": [
    { "term": "KYC", "handled_as": "KYC" }
  ],
  "named_entities_handled": [
    { "entity": "Priya Sharma", "handled_as": "प्रिया शर्मा" }
  ]
}
```

| Score | Meaning |
|---|---|
| 1 | Structure valid — proceed to scoring |
| 0 | Structure invalid — do not score |

## Full Translation Score Formula

| Metric | Weight |
|---|---|
| Translation Completeness Rate | 10% |
| Sentence-Level Meaning Accuracy | 35% |
| Target Language Fluency Score | 10% |
| Domain Term Preservation Rate | 20% |
| Proper Noun Preservation Rate | 10% |
| Critical Fact Preservation Rate | 15% |
| **Total** | **100%** |

```
IF Response Structure Compliance = 0        →  Translation Score = invalid
IF Domain Term Preservation Rate < 0.90     →  Translation Score = invalid (blocker)
IF Critical Fact Preservation Rate < 0.97   →  Translation Score = invalid (blocker)

IF all checks pass  →

Translation Score = (Translation Completeness Rate   × 0.10)
                 + (Sentence-Level Meaning Accuracy  × 0.35)
                 + (Target Language Fluency Score     × 0.10)
                 + (Domain Term Preservation Rate     × 0.20)
                 + (Proper Noun Preservation Rate     × 0.10)
                 + (Critical Fact Preservation Rate   × 0.15)

Range: 0.0 – 1.0
```

> **Example:**
> - Structure = 1 ✓, Domain Term Rate = 0.92 ✓, Fact Preservation = 0.98 ✓
>
> = (0.97 × 0.10) + (0.84 × 0.35) + (0.80 × 0.10) + (0.92 × 0.20) + (0.95 × 0.10) + (0.98 × 0.15)
> = 0.097 + 0.294 + 0.080 + 0.184 + 0.095 + 0.147 = **0.897**

# Final Scoring

## Option A — With Standard Benchmarks

Use this when you have run the optional Layer 2 benchmarks.

| Task / Layer | Weight |
|---|---|
| QA Score | 25% |
| Entity Score | 20% |
| Text Score | 15% |
| Translation Score | 10% |
| Benchmark Score | 30% |
| **Total** | **100%** |

```
Final Score = (QA Score          × 0.25)
            + (Entity Score      × 0.20)
            + (Text Score        × 0.15)
            + (Translation Score × 0.10)
            + (Benchmark Score   × 0.30)

Range: 0.0 – 1.0
```

## Option B — Without Standard Benchmarks

Use this when benchmarks are not run. The 30% benchmark weight is redistributed proportionally across the four tasks.

| Task | Weight |
|---|---|
| QA Score | 35% |
| Entity Score | 30% |
| Text Score | 20% |
| Translation Score | 15% |
| **Total** | **100%** |

```
Final Score = (QA Score          × 0.35)
            + (Entity Score      × 0.30)
            + (Text Score        × 0.20)
            + (Translation Score × 0.15)

Range: 0.0 – 1.0
```

## Cost-Adjusted Ranking

```
Cost Efficiency = Final Score / Cost per 1000 calls (normalized)
Model Rank      = Cost Efficiency (descending)
```

Adjust task weights in either formula based on your actual production workload split.

# Judge System Prompt

```
You are an evaluator for call center AI systems.

Inputs per test case:
  transcript:       The original call transcript
  model_output:     The LLM response being evaluated
  expected_outcome: The golden truth / human-verified reference
  metric:           The metric to score
  config:           Task configuration

SCORING RULES:

  Task Metrics (Layer 1):
    Return a continuous score between 0.0 and 1.0.
    Compare model_output to expected_outcome on the metric dimension.
    expected_outcome is your primary reference for what correct looks like.

  Standard Benchmarks (Layer 2, optional):
    Return binary 0 or 1.
    1 = criterion met at or above threshold. 0 = not met.
    Use expected_outcome as ground truth for factual verification,
    correctness checks, contextual alignment, and argument validation.

CONDUCT:
  - Cite the expected_outcome field and model_output field that justify your score.
  - Never pass an output with facts absent from both transcript and expected_outcome.
  - For PASS_FAIL questions: model giving maxScore when expected_outcome = 0 is a false positive.
  - Score strictly.

Return JSON:
{
  "metric": "<name>",
  "score": <value>,
  "expected_outcome_reference": "<relevant expected_outcome value>",
  "model_output_observed": "<what model produced>",
  "reason": "<short explanation>"
}
```

# Scorecard Template

`[!]` = Blocker — breach disqualifies the model for that task.

| Metric | Model A | Model B | Model C | Threshold |
|---|---|---|---|---|
| **QA TASK METRICS** | | | | |
| Question Score Accuracy | | | | >= 0.95 |
| Score Gap Accuracy | | | | >= 0.90 |
| Evidence-Backed Reasoning | | | | >= 0.90 |
| [!] Compliance False Pass Rate | | | | <= 1% \| > 3% blocker |
| [!] Response Structure Compliance | | | | must = 1 |
| **QA Score** | | | | 0.0 – 1.0 |
| **ENTITY TASK METRICS** | | | | |
| Keyword F1 | | | | >= 0.90 |
| Topic F1 | | | | >= 0.88 |
| [!] Configured Entity Compliance | | | | >= 0.95 \| < 0.95 blocker |
| Cross-Language Match Rate | | | | >= 0.90 |
| [!] Fabricated Entity Count | | | | 0 \| > 2 blocker |
| [!] Response Structure Compliance | | | | must = 1 |
| **Entity Score** | | | | 0.0 – 1.0 |
| **TEXT TASK METRICS** | | | | |
| Per-Sentence Sentiment Accuracy | | | | >= 0.88 |
| Sentiment Macro F1 | | | | >= 0.85 |
| [!] Missing Sentiment Labels | | | | 0 \| > 2 blocker |
| [!] Call Intent Match | | | | 1.0 \| 0.0 = blocker |
| Highlight Recall Rate | | | | >= 0.85 |
| Highlight Correctness Rate | | | | >= 0.90 |
| Required Field Presence Rate | | | | >= 0.90 |
| [!] Fabrication-Free Rate | | | | > 3% hallucination = blocker |
| Dominant Emotion Correctness | | | | = 1 |
| [!] Response Structure Compliance | | | | must = 1 |
| **Text Score** | | | | 0.0 – 1.0 |
| **TRANSLATION TASK METRICS** | | | | |
| Translation Completeness Rate | | | | 1.00 |
| Sentence-Level Meaning Accuracy | | | | >= 0.85 |
| Target Language Fluency Score | | | | >= 0.85 |
| [!] Domain Term Preservation Rate | | | | >= 0.95 \| < 0.90 blocker |
| Proper Noun Preservation Rate | | | | >= 0.95 |
| [!] Critical Fact Preservation Rate | | | | >= 0.97 \| < 0.97 blocker |
| [!] Response Structure Compliance | | | | must = 1 |
| **Translation Score** | | | | 0.0 – 1.0 |
| **STANDARD BENCHMARKS — OPTIONAL (0 or 1)** | | | | |
| Faithfulness | | | | 1 if >= 0.90 |
| Hallucination Rate | | | | 1 if <= 2% |
| Answer Relevancy | | | | 1 if >= 0.85 |
| Answer Correctness | | | | 1 if >= 0.85 |
| Contextual Precision | | | | 1 if >= 0.85 |
| Contextual Recall | | | | 1 if >= 0.85 |
| Contextual Relevancy | | | | 1 if >= 0.85 |
| Argument Correctness | | | | 1 if >= 0.85 |
| Instruction Following | | | | 1 if >= 0.95 |
| Exact Match | | | | 1 if >= 0.90 |
| Consistency Score | | | | 1 if >= 0.90 |
| **Benchmark Score** | | | | >= 0.80 recommended |
| **FINAL** | | | | |
| Final Score (with benchmarks) | | | | Option A |
| Final Score (without benchmarks) | | | | Option B |
| Cost per 1000 calls | | | | $ |
| Cost-Adjusted Rank | | | | — |
| Any Blocker Triggered? | | | | Yes / No |

# Standard Benchmarks (Optional — Layer 2)

These are general LLM quality checks. Run them to get a model-level capability baseline. All use `expected_outcome` as ground truth. Each returns binary **0 or 1**. Benchmark Score = mean of all; recommended minimum **0.80**.

### Faithfulness

Did the model only state things grounded in the transcript or `expected_outcome`?

```
Faithfulness = claims_supported / total_claims
```

> **Example:** Model says "Agent offered a $20 voucher" — absent from both transcript and `expected_outcome` → unsupported.

| Pass (1) | Fail (0) |
|---|---|
| >= 0.90 | < 0.90 |

### Hallucination Rate

What share of output facts don't exist in transcript or `expected_outcome`?

```
Hallucination Rate = (hallucinated_facts / total_facts) × 100
```

| Pass (1) | Fail (0) |
|---|---|
| <= 2% | > 2% |

### Answer Relevancy

Did the output actually answer the question asked? Use `expected_outcome` as the reference for a complete, on-topic answer.

```
Relevancy = cosine_similarity(embedding(output), embedding(question))
```

| Pass (1) | Fail (0) |
|---|---|
| >= 0.85 | < 0.85 |

### Answer Correctness

Do the model's facts match `expected_outcome`?

```
Answer Correctness = (F1_score × 0.50) + (semantic_similarity × 0.50)
```

> **Example:** `expected_outcome` lists 4 facts. Model gets 3 right, adds 1 wrong → F1 < 1.0.

| Pass (1) | Fail (0) |
|---|---|
| >= 0.85 | < 0.85 |

### Contextual Precision

Of context the model used, how much was actually needed (per `expected_outcome`)?

```
Precision = relevant_pieces_used / total_pieces_referenced
```

| Pass (1) | Fail (0) |
|---|---|
| >= 0.85 | < 0.85 |

### Contextual Recall

Of context needed (per `expected_outcome`), how much did the model use?

```
Recall = relevant_pieces_used / total_relevant_in_input
```

| Pass (1) | Fail (0) |
|---|---|
| >= 0.85 | < 0.85 |

### Contextual Relevancy

Average of Precision and Recall.

```
Contextual Relevancy = (Precision + Recall) / 2
```

| Pass (1) | Fail (0) |
|---|---|
| >= 0.85 | < 0.85 |

### Argument Correctness

Is the model's reasoning sound, compared to the reasoning in `expected_outcome`?

```
Per step: 1.0 = sound | 0.5 = partial | 0.0 = wrong
Final = mean(step_scores)
```

> **Example:** Model identifies non-compliance but gives wrong reason → step score = 0.5.

| Pass (1) | Fail (0) |
|---|---|
| >= 0.85 | < 0.85 |

### Instruction Following

Did the model follow all prompt instructions? Use `expected_outcome` as the reference for a compliant output.

```
Score = instructions_followed / total_instructions
```

| Pass (1) | Fail (0) |
|---|---|
| >= 0.95 | < 0.95 |

### Exact Match

For closed-form fields (labels, scores, IDs) — does output exactly match `expected_outcome`?

```
EM = exact_matches / total_predictions
```

> **Example:** `expected_outcome` sentiment = "negative". Model says "negative" → match.

| Pass (1) | Fail (0) |
|---|---|
| >= 0.90 | < 0.90 |

### Consistency / Robustness

Does the model give consistent answers when run multiple times on the same input? Each run is scored against `expected_outcome`; variance across those scores is measured.

```
Consistency = 1 − (std_dev(scores across N runs) / mean(scores))
```

> **Example:** 3 runs → scores 0.91, 0.90, 0.88 → low variance → consistent.

| Pass (1) | Fail (0) |
|---|---|
| >= 0.90 | < 0.90 |

### Benchmark Score

```
Benchmark Score = sum(binary scores) / 11 criteria    →    range 0.0 – 1.0
Recommended minimum: >= 0.80
```

*Framework Version 6.0 | February 2026*
*Tasks: QA Analysis · Entity Analysis · Text Analysis · Translation Analysis*
