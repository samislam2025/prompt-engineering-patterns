# 📊 Evaluation-Driven Iteration
**Category:** Production | **Difficulty:** Advanced | **Impact:** Critical

## When To Use
Use evaluation-driven iteration when you're moving a prompt from "it works in my testing" to "it works reliably at scale." This pattern replaces gut-feel prompt tuning with systematic measurement, ensuring every change is an actual improvement rather than a different flavor of mediocre.

## The Problem

```
❌ Naive Prompt
```
```
I've been tweaking this prompt for a week. Each version seems
better in some ways and worse in others. I can't tell if I'm
making progress or going in circles.

Version 1: More detailed but too verbose
Version 2: More concise but misses edge cases
Version 3: Handles edge cases but format is inconsistent
Version 4: Format is better but... wait, is this worse than V2?
```

**What goes wrong:** Without a scoring framework, you're evaluating prompts based on vibes. You test with 3-5 examples, pick the version that "feels right" on those examples, and ship it. In production, it fails on the 50 edge cases you didn't test. Worse, when you iterate from there, you have no baseline — you can't tell if version 5 is better than version 3 because you never measured version 3.

**Why it fails:** Prompt engineering without evaluation is like A/B testing with no metrics. You're running experiments but not measuring results, so you can't learn systematically. Every iteration is a guess informed by anecdote rather than data.

## The Pattern

```
✅ Engineered Prompt (Evaluation Framework)
```
```markdown
## Step 1: Define your eval dataset

Create 50-100 test cases covering:
- 60% typical cases (the bread and butter)
- 20% edge cases (unusual but valid inputs)
- 10% adversarial cases (inputs designed to break things)
- 10% boundary cases (inputs at the limits of scope)

Each test case has:
- Input: The actual input the prompt will receive
- Expected output: The gold-standard correct response
- Category: Which type of case (typical/edge/adversarial/boundary)
- Priority: How important is getting this case right

## Step 2: Define scoring criteria

For a resume review prompt, example criteria:
| Criterion | Weight | Scoring |
|-----------|--------|---------|
| Accuracy | 30% | Are the identified issues real issues? |
| Completeness | 25% | Did it catch all major issues? |
| Actionability | 25% | Can the user act on the feedback? |
| Conciseness | 10% | No unnecessary padding? |
| Tone | 10% | Direct but respectful? |

## Step 3: Run evaluations systematically

For each prompt version:
1. Run ALL test cases (not a cherry-picked subset)
2. Score each output on each criterion (1-5 scale)
3. Calculate weighted average per case
4. Calculate overall score and per-category scores
5. Identify specific cases where this version regresses vs. prior

## Step 4: Iterate with evidence

When modifying the prompt, record:
- What changed and why
- Which specific test cases motivated the change
- Hypothesis: "This change should improve [criterion] on [case type]"
- Result: Did it? At what cost to other criteria?
```

## Why It Works

Evaluation frameworks convert prompt engineering from an art into an engineering discipline. They provide three things that gut-feel tuning cannot:

**Regression detection:** When you improve edge case handling, does it break typical cases? Without a test suite, you won't know until production. With one, you know immediately.

**Objective comparison:** "Version A scores 78/100, Version B scores 82/100, Version B is better" is more useful than "Version A feels more concise but Version B handles edge cases better, I'm not sure which is better overall."

**Communication with stakeholders:** When your PM asks "how much did your prompt improvements help?", having a score that went from 72 to 85 is dramatically more convincing than "it's better, trust me."

## Real-World Example

**Scenario:** Iterating on a job-matching prompt for a career coaching platform.

```python
# Eval dataset for job-matching prompt
eval_cases = [
    {
        "input": {
            "resume": "8 years restaurant management...",
            "job": "Product Manager at DoorDash"
        },
        "expected": {
            "match_score": 6,  # out of 10
            "key_strengths": ["operations at scale", "P&L ownership",
                              "domain knowledge in food delivery"],
            "critical_gaps": ["no formal PM experience",
                              "no technical background"],
            "should_apply": True,  # strong transferable skills
        },
        "category": "career_pivot",
        "priority": "high"
    },
    # ... 99 more cases across categories:
    # - direct_match (same industry + role)
    # - career_pivot (different industry or role)
    # - overqualified (senior person, junior role)
    # - underqualified (junior person, senior role)
    # - edge_case (unusual backgrounds, gaps, etc.)
]

# Scoring rubric
scoring = {
    "match_score_accuracy": {
        "weight": 0.30,
        "measure": "abs(predicted - expected) <= 1"
    },
    "strength_identification": {
        "weight": 0.25,
        "measure": "% of expected strengths identified"
    },
    "gap_identification": {
        "weight": 0.25,
        "measure": "% of expected gaps identified"
    },
    "recommendation_accuracy": {
        "weight": 0.20,
        "measure": "should_apply matches expected"
    }
}
```

## Common Mistakes

1. **Test cases that are too easy.** If your eval suite is 90% straightforward inputs, a prompt that scores 95% might still fail on 50% of real-world edge cases. Weight your eval toward the cases that actually matter.

2. **Optimizing for the eval instead of the task.** If you iterate until your prompt scores perfectly on 50 test cases, you might be overfitting to those specific cases. Hold out 20% of your test cases and check generalization periodically.

> [!WARNING]
> **Not including adversarial test cases.** If your eval only includes well-formed inputs, your prompt will fail the first time a user sends gibberish, conflicting instructions, or an injection attempt. The adversarial 10% of your eval is what separates a robust prompt from a demo-ready one.

## Related Patterns
- [Regression Testing Prompts](17-regression-testing-prompts.md) — Automated testing that runs your eval suite on every prompt change.
- [Prompt Versioning](20-prompt-versioning.md) — Track exactly which prompt version achieved which eval scores.
- [Self-Consistency Checking](../intermediate/06-self-consistency-checking.md) — Use consistency across runs as an additional eval metric.
