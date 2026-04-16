# 📐 Context Window Management
**Category:** Intermediate | **Difficulty:** Intermediate | **Impact:** Very High

## When To Use
Use context window management when your prompts involve large inputs — long documents, multiple data sources, extensive conversation history, or any scenario where you're working near the model's context limit. Critical for production applications where input size varies unpredictably.

## The Problem

```
❌ Naive Prompt
```
```
Here are 50 customer support tickets from the last month. Analyze
them and identify the top issues, trends, and recommendations.

[Ticket 1: full text]
[Ticket 2: full text]
...
[Ticket 50: full text]
```

**What goes wrong:** The model gives disproportionate attention to tickets at the beginning and end of the list, with middle tickets getting compressed or ignored entirely. Patterns in tickets 20-35 are systematically underweighted. The "comprehensive analysis" is really a beginning-and-end analysis with guesses about the middle.

**Why it fails:** This is the "lost in the middle" problem. Transformer attention mechanisms allocate processing unevenly across long contexts. Information at the start (primacy) and end (recency) of the context receives more attention than information in the middle. With 50 tickets, the middle 30 are effectively second-class citizens.

## The Pattern

```
✅ Engineered Prompt
```
```
I need to analyze 50 customer support tickets. To ensure thorough
analysis, we'll work in structured passes:

PASS 1 — CATEGORIZATION:
For each ticket below, output ONLY a one-line summary in this format:
[Ticket #] | [Category: Bug/Feature Request/Billing/UX/Other] |
[Severity: Critical/High/Medium/Low] | [Product Area] |
[One-line summary, max 15 words]

TICKETS:
[All 50 tickets]

---

After completing the categorization, proceed to:

PASS 2 — PATTERN ANALYSIS:
Using ONLY your categorization table from Pass 1 (not the raw
tickets), answer:
1. What are the top 5 categories by frequency?
2. Which product area has the highest ratio of Critical/High
   severity tickets?
3. Are there clusters of related tickets that suggest a single
   root cause?

PASS 3 — RECOMMENDATIONS:
Based on the patterns from Pass 2, provide:
- Top 3 issues to address, ranked by (frequency × severity)
- For each: root cause hypothesis, suggested fix, effort estimate
```

## Why It Works

The multi-pass strategy converts a single overwhelming task into progressive refinement. Pass 1 forces the model to process every ticket individually (not skip the middle), producing a compressed representation that preserves the essential information. Pass 2 operates on this compressed representation, which is short enough for the model to attend to fully.

This is analogous to MapReduce: Pass 1 is the "map" (extract key information from each ticket), and Passes 2-3 are the "reduce" (aggregate and analyze the extracted data). The compression step in Pass 1 is the critical innovation — it prevents information loss from the middle of long contexts.

For very large inputs that exceed context limits entirely, this same principle extends to splitting the input across multiple API calls, with a final aggregation call that processes only the compressed summaries from each batch.

## Real-World Example

**Scenario:** Analyzing a restaurant's 6-month review history to identify operational improvement priorities.

```
I have 120 customer reviews spanning 6 months. Process them in
two phases:

PHASE 1 — EXTRACTION (Process each review individually):
For each review, extract:
| Review # | Date | Rating | Positive Keywords | Negative Keywords |
Primary Complaint (if any, max 10 words) |

Process reviews in batches of 20:

BATCH 1 (Reviews 1-20):
[reviews]

[After completing each batch, hold results and continue]

BATCH 6 (Reviews 101-120):
[reviews]

PHASE 2 — TREND ANALYSIS (using extraction table only):
1. SENTIMENT TRAJECTORY: Is the average rating trending up or
   down month over month?
2. PERSISTENT ISSUES: Which complaints appear in 3+ months?
3. QUICK WINS: Which negative keywords appear frequently but
   suggest easy fixes?
4. BRIGHT SPOTS: Which positive keywords are consistent strengths
   to protect?

Output a one-page executive summary suitable for a restaurant
owner who doesn't have time for details.
```

## Common Mistakes

1. **Assuming the model reads everything equally.** It doesn't. In contexts above ~4K tokens, middle content receives measurably less attention. Structure your prompts to account for this.

2. **Putting critical instructions in the middle of long prompts.** Your most important instructions should be at the very beginning (system prompt) or very end (just before the model's response). Never bury key instructions between large blocks of data.

> [!WARNING]
> **Cramming everything into one prompt when batching would be better.** If your input exceeds 50% of the context window, seriously consider splitting it across multiple calls. The quality improvement from batching often outweighs the added latency and cost. A reliable answer from 3 calls beats an unreliable answer from 1.

## Related Patterns
- [Prompt Chaining Pipelines](../advanced/14-prompt-chaining-pipelines.md) — The natural extension of context management: split processing across multiple model calls.
- [Dynamic Few-Shot Selection](../advanced/15-dynamic-few-shot-selection.md) — Choose which examples to include based on available context space.
- [Temperature and Sampling Strategy](10-temperature-and-sampling-strategy.md) — Context window size interacts with sampling parameters in ways that affect output quality.
