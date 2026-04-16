# 🎲 Dynamic Few-Shot Selection
**Category:** Advanced | **Difficulty:** Advanced | **Impact:** Very High

## When To Use
Use dynamic few-shot selection when you have a library of examples but context window space is limited, and different inputs benefit from different examples. This is the production evolution of static few-shot prompting — instead of hardcoding the same 3 examples, you programmatically choose the most relevant examples for each input.

## The Problem

```
❌ Naive Prompt
```
```
Classify this customer support ticket. Here are some examples:

Example 1: "My payment failed" → Billing
Example 2: "The app crashes on login" → Technical Bug
Example 3: "How do I export data?" → Feature Question

Now classify: "The dashboard shows wrong revenue numbers"
```

**What goes wrong:** The static examples don't cover this input's category well. "Wrong revenue numbers" could be a data accuracy issue, a technical bug, or a reporting feature question. The examples bias the model toward "Technical Bug" (closest match to "app crashes") when it might actually be a data integrity issue requiring escalation. With 50+ ticket categories and only 3 example slots, static selection leaves most categories unrepresented.

**Why it fails:** Static few-shot examples optimize for the average case, not the specific input. When the input doesn't resemble any of the examples, the model is essentially doing zero-shot classification with irrelevant context consuming tokens.

## The Pattern

```
✅ Engineered Prompt (with dynamic example retrieval)
```
```python
# Step 1: Maintain an example bank with embeddings
example_bank = [
    {
        "text": "My subscription was charged twice this month",
        "category": "Billing - Duplicate Charge",
        "embedding": embed("My subscription was charged twice...")
    },
    {
        "text": "The revenue dashboard shows last month's data as current",
        "category": "Data Accuracy - Stale Data",
        "embedding": embed("The revenue dashboard shows last month's...")
    },
    # ... 200+ categorized examples
]

# Step 2: For each new input, find the 3 most similar examples
input_text = "The dashboard shows wrong revenue numbers"
input_embedding = embed(input_text)
top_3 = find_nearest(input_embedding, example_bank, k=3)

# Step 3: Build the prompt with relevant examples
prompt = f"""
Classify this customer support ticket into one of our categories.
Here are examples of similar tickets and their correct categories:

{format_examples(top_3)}

IMPORTANT: These examples show similar tickets, but the new ticket
may belong to a DIFFERENT category. Classify based on the actual
content, not just similarity to examples.

Classify this ticket:
"{input_text}"

Output:
Category: [category name]
Confidence: [HIGH/MEDIUM/LOW]
Reasoning: [one sentence explaining why this category]
"""
```

## Why It Works

Dynamic selection ensures that the examples the model sees are always relevant to the specific input, maximizing the information-per-token ratio. When classifying a data accuracy ticket, the model sees 3 examples of data accuracy issues — not generic billing and crash examples that waste context space.

The embedding-based retrieval creates a semantic similarity search: examples are chosen not by keyword match but by meaning similarity. This handles cases where the input uses different vocabulary than the examples but addresses the same underlying issue.

The "IMPORTANT" caveat in the prompt prevents a subtle failure mode: with highly similar examples all from the same category, the model can over-anchor and always output that category. The caveat ensures the model considers whether the input genuinely matches or merely resembles the examples.

This pattern scales to thousands of examples without increasing prompt size — the retrieval layer acts as a smart filter, keeping prompt length constant while drawing from an ever-growing knowledge base.

## Real-World Example

**Scenario:** Dynamic example selection for a career coaching AI that provides role-specific resume advice.

```python
# Example bank organized by role type
advice_bank = [
    {
        "input_role": "Senior Data Engineer at Stripe",
        "advice": "Lead with pipeline scale metrics (events/day, "
                  "data volume). Stripe cares about reliability — "
                  "emphasize uptime and incident response experience.",
        "embedding": embed("senior data engineer fintech payments")
    },
    {
        "input_role": "Product Manager at a health tech startup",
        "advice": "Health tech PMs need to show regulatory awareness. "
                  "Mention HIPAA, FDA if relevant. Show you can ship "
                  "fast within compliance constraints.",
        "embedding": embed("product manager health tech startup")
    },
    # ... 500+ role-specific advice examples
]

# For each new user query
user_target_role = "ML Engineer at a food delivery company"
relevant_advice = find_nearest(embed(user_target_role), advice_bank, k=3)

prompt = f"""
A career coaching client is targeting this role: {user_target_role}

Here's advice we've given for similar roles:
{format_examples(relevant_advice)}

Using these as reference (but tailoring to the specific role),
provide resume optimization advice for someone targeting
{user_target_role}.

Focus on:
1. Which metrics and achievements to highlight
2. Industry-specific keywords that matter
3. What this specific type of company looks for vs. generic advice
"""
```

## Common Mistakes

1. **Selecting examples only by surface similarity.** If all your top-3 examples are about dashboards but the ticket is actually about billing (displayed on a dashboard), surface similarity misleads. Consider including at least one diverse example to prevent over-anchoring.

2. **Not refreshing the example bank.** If your examples are 6 months old and product features have changed, the model is learning from stale patterns. Regularly update examples from recently-resolved tickets.

> [!WARNING]
> **Ignoring the retrieval quality.** Your classification is only as good as your retrieval. If the embedding model can't distinguish between "payment failed" (billing) and "payment data looks wrong" (data accuracy), even perfect few-shot prompting won't help. Test your retrieval separately from your classification, and upgrade your embedding model if retrieval quality is the bottleneck.

## Related Patterns
- [Few-Shot Examples](../foundational/02-few-shot-examples.md) — The static version of this pattern. Start here, upgrade to dynamic when you have enough examples.
- [Context Window Management](../intermediate/09-context-window-management.md) — Dynamic selection is a context management strategy: choose what fills your limited space.
- [Evaluation-Driven Iteration](../production/16-evaluation-driven-iteration.md) — Measure whether dynamic selection actually improves accuracy over static examples.
