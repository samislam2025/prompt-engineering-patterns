# 📋 Output Formatting
**Category:** Foundational | **Difficulty:** Beginner | **Impact:** High

## When To Use
Use output formatting when the downstream consumer of the model's output is a system (API, database, UI component) or when humans need to scan and compare outputs quickly. Essential for any production prompt where the output feeds into a pipeline.

## The Problem

```
❌ Naive Prompt
```
```
Extract the key information from this job posting and summarize it.
```

**What goes wrong:** The model returns a wall of prose: "This is a Senior Data Engineer position at Acme Corp based in San Francisco, offering a salary range of..." Every run produces a different structure. Some include salary, some don't. Some list requirements as bullets, some embed them in paragraphs. Parsing this programmatically is a nightmare. Even for humans, comparing 10 job summaries in different formats is exhausting.

**Why it fails:** "Summarize" is a format-free instruction. The model chooses whatever structure feels natural for that particular input, which means output structure varies with input structure. No consistency, no parseability.

## The Pattern

```
✅ Engineered Prompt
```
```
Extract information from this job posting into the exact JSON
format below. If a field is not mentioned in the posting, use null.
Do not infer or guess values — only extract what is explicitly stated.

{
  "title": "string — exact job title as written",
  "company": "string",
  "location": {
    "city": "string | null",
    "state": "string | null",
    "remote_policy": "remote | hybrid | onsite | null"
  },
  "compensation": {
    "salary_min": "number | null — annual USD",
    "salary_max": "number | null — annual USD",
    "equity": "boolean | null"
  },
  "requirements": {
    "years_experience": "number | null — minimum stated",
    "must_have": ["string — only explicitly required skills"],
    "nice_to_have": ["string — explicitly listed as preferred"]
  },
  "red_flags": ["string — any concerning language patterns"],
  "posted_date": "string | null — ISO 8601 format"
}

Job posting:
[job posting text]
```

## Why It Works

Explicit output schemas solve two problems simultaneously. First, they eliminate structural ambiguity — the model knows exactly what to produce, so it allocates attention to extraction accuracy rather than format decisions. Second, they create a contract between the prompt and downstream code, making the output reliably parseable.

The field-level type annotations (`"string | null"`) serve as inline instructions. They tell the model what to do when information is missing (return null) rather than leaving it to guess. The comments after each field (`"— annual USD"`) resolve unit ambiguity that would otherwise cause inconsistencies.

Specifying "do not infer or guess" is critical for extraction tasks. Without this, models will hallucinate plausible values — a common failure mode where the model "fills in" a salary range that seems reasonable but isn't in the source text.

## Real-World Example

**Scenario:** Structuring restaurant inspection feedback for a multi-location operations dashboard.

```
You are documenting a restaurant location inspection. Output your
findings in this exact markdown format. Every section is required.

## Inspection Summary
| Field | Value |
|-------|-------|
| Location | [store name and address] |
| Date | [YYYY-MM-DD] |
| Inspector | [name] |
| Overall Score | [X/100] |

## Critical Issues (Immediate Action Required)
- [ ] [Issue description] — **Deadline:** [date]

## Warnings (Resolve Within 7 Days)
- [ ] [Issue description]

## Observations (No Action Required)
- [Observation]

## Scores by Category
| Category | Score | Notes |
|----------|-------|-------|
| Food Safety | /25 | |
| Cleanliness | /25 | |
| Staff Compliance | /25 | |
| Customer Experience | /25 | |

## Follow-Up Actions
1. [Action] — **Owner:** [name] — **Due:** [date]

Inspection notes:
[raw inspection notes]
```

## Common Mistakes

1. **Defining format but not edge cases.** What happens when a field is empty? When there are zero critical issues? Specify these: "If no critical issues found, write 'None identified' under the heading."

2. **Over-structuring simple outputs.** Not everything needs JSON. For a 3-field extraction, a simple `Field: Value` format is clearer and more token-efficient than nested JSON.

> [!WARNING]
> **Not validating outputs match the schema.** In production, always validate the model's output against your expected format programmatically. Models occasionally break format, especially with complex nested structures. Build parsing with graceful error handling, not blind trust.

## Related Patterns
- [Few-Shot Examples](02-few-shot-examples.md) — When the format is complex, showing a completed example is faster than describing the schema.
- [Constraint Setting](../intermediate/07-constraint-setting.md) — Add constraints to individual fields within your format (word limits, allowed values, etc.).
- [Prompt Chaining Pipelines](../advanced/14-prompt-chaining-pipelines.md) — Output formatting enables reliable handoff between pipeline stages.
