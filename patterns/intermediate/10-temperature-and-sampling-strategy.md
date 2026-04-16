# 🌡️ Temperature and Sampling Strategy
**Category:** Intermediate | **Difficulty:** Intermediate | **Impact:** High

## When To Use
Use temperature and sampling strategy when you need to control the randomness vs. determinism tradeoff. This pattern applies to any production prompt where you've tuned the text but the outputs are either too repetitive or too unpredictable. Temperature is the most impactful parameter most prompt engineers never touch.

## The Problem

```
❌ Naive Prompt (with default temperature ~1.0)
```
```
Generate a subject line for an email promoting our resume
review service to job seekers.
```

**What goes wrong:** At the default temperature, outputs swing between safe-but-boring ("Improve Your Resume Today") and creative-but-unhinged ("Your Resume Is a Crying Orphan in the Storm of ATS Systems"). There's no consistency across runs — you can't use this in production because the quality variance is too high.

**Why it fails:** Temperature controls the probability distribution over tokens. At high temperature, unlikely tokens get boosted, creating variety but also nonsense. At the default temperature, you get the full entropy of the model, which includes both its best and worst ideas.

## The Pattern

```
✅ Engineered Prompt (temperature = 0.3 for extraction, 0.7 for creative)
```
```
# For data extraction (temperature = 0.0 - 0.3):
Extract the following fields from this job posting.
Return exactly the JSON schema specified. Do not add commentary.

# For creative generation with guardrails (temperature = 0.7):
Generate 5 email subject lines for our resume review service.

Requirements:
- Under 50 characters each
- At least 2 should use numbers ("3 mistakes...", "87% of...")
- At least 1 should use a question format
- None should use ALL CAPS or exclamation marks
- None should use the word "revolutionary" or "transform"

Rate each on a 1-5 scale for:
- Curiosity gap (does it make you want to open?)
- Clarity (do you know what the email is about?)
- Authenticity (does it sound like a person, not a marketer?)

Recommend the top 2 with reasoning.

# For deterministic analysis (temperature = 0.0):
Compare these two candidate resumes against the job description.
Score each candidate on each requirement. This must produce
identical results if run multiple times.
```

## Why It Works

Temperature mathematically controls the softmax function that converts model logits into token probabilities. At temperature 0, the model always picks the highest-probability token (greedy decoding) — maximum determinism, minimum creativity. At temperature 1.0, the original probability distribution is preserved. Above 1.0, the distribution flattens, making unlikely tokens more probable.

The key insight is matching temperature to task type:
- **Extraction/analysis (0.0-0.3):** You want the same answer every time. There's one correct extraction of a salary from a job posting.
- **Structured creative (0.5-0.8):** You want variety within guardrails. Subject lines need creativity but within constraints.
- **Brainstorming (0.8-1.0):** You want maximum diversity of ideas, accepting some will be bad.
- **Never above 1.0** in production — the quality/creativity tradeoff becomes pure noise above this point.

Combining temperature with top_p (nucleus sampling) gives even finer control. Low temperature + high top_p = focused but not repetitive. Low temperature + low top_p = highly deterministic, potentially repetitive.

## Real-World Example

**Scenario:** Building an AI-powered career coaching chatbot with different modes.

```
# System prompt for career coaching bot
# Temperature varies by conversation phase:

PHASE: ASSESSMENT (temperature = 0.2)
When analyzing the user's resume or career history, be precise
and consistent. Use the same evaluation framework every time.
Extract skills, quantify experience, identify gaps.

PHASE: BRAINSTORMING (temperature = 0.8)
When generating career pivot ideas or unconventional job search
strategies, be creative. Suggest unexpected connections between
the user's experience and target roles. Think laterally.

PHASE: ACTION PLANNING (temperature = 0.3)
When creating specific action plans with deadlines and milestones,
be precise and practical. No speculative ideas — only actionable
steps with clear criteria for completion.

---
Implementation note: In production, dynamically set temperature
based on the detected conversation phase. Use a classifier to
determine which phase the conversation is in, then adjust the
API parameter accordingly.
```

## Common Mistakes

1. **Using the same temperature for everything.** A data extraction pipeline and a creative writing tool need fundamentally different sampling strategies. Make temperature a per-task configuration, not a global default.

2. **Chasing creativity with temperature instead of prompt design.** If your output is boring at temperature 0.7, the problem is usually the prompt, not the temperature. Fix the prompt first, then fine-tune temperature.

> [!WARNING]
> **Setting temperature to 0 and expecting perfectly deterministic outputs.** Even at temperature 0, some API implementations have minor non-determinism due to floating point operations and batching. If you need byte-identical outputs, you need to cache results, not rely on temperature alone. Also, temperature 0 can cause repetition loops in long-form generation — use 0.1-0.2 instead for production analytical tasks.

## Related Patterns
- [Evaluation-Driven Iteration](../production/16-evaluation-driven-iteration.md) — Use systematic evaluation to find the optimal temperature for your specific use case.
- [Multi-Model Routing](../production/18-multi-model-routing.md) — Different models have different optimal temperature ranges for the same task.
- [Constraint Setting](07-constraint-setting.md) — Constraints reduce output variance more reliably than temperature alone for creative tasks.
