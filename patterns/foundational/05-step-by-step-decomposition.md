# 🪜 Step-by-Step Decomposition
**Category:** Foundational | **Difficulty:** Beginner | **Impact:** High

## When To Use
Use decomposition when a task is too complex to solve in a single prompt but doesn't warrant a full multi-prompt pipeline. This is the "break it into pieces" pattern — useful when a task has clear sequential dependencies where each step's output informs the next.

## The Problem

```
❌ Naive Prompt
```
```
Create a complete onboarding email sequence for new users of our
AI-powered resume builder SaaS product.
```

**What goes wrong:** The model tries to do everything at once: writes 5-7 emails, guesses at the product features, invents user segments, picks arbitrary timing, and produces a superficially complete but strategically hollow sequence. Email 1 has no connection to email 2. The "sequence" is really just 5 independent emails wearing a trench coat.

**Why it fails:** The task has hidden dependencies: you need to understand the user journey before picking email topics, you need to define success metrics before writing CTAs, and you need to map product features before you can introduce them in a logical order. Dumping everything into one prompt skips this prerequisite thinking.

## The Pattern

```
✅ Engineered Prompt
```
```
I need a 5-email onboarding sequence for our AI resume builder.
Work through this step by step, completing each step fully before
moving to the next:

STEP 1 — USER JOURNEY MAP:
Define the key milestones a new user hits in their first 14 days:
- What does "activation" look like? (first value moment)
- What are the 3 most common drop-off points?
- What action predicts long-term retention?

STEP 2 — SEQUENCE STRATEGY:
Based on the journey map from Step 1, define for each email:
- Trigger (time-based or behavior-based)
- Goal (which milestone it drives toward)
- Success metric (open rate, click, conversion)

STEP 3 — CONTENT FRAMEWORK:
For each email from Step 2, outline:
- Subject line (2 options: curiosity-based and value-based)
- Core message in one sentence
- Primary CTA
- Social proof element to include

STEP 4 — FULL EMAIL DRAFTS:
Now write each email using the framework from Step 3. Each email
should be under 150 words, mobile-optimized, and reference the
specific user milestone it targets.

STEP 5 — SEQUENCE VALIDATION:
Review the complete sequence and check:
- Does each email logically follow the previous one?
- Is there a clear escalation in commitment asked of the user?
- Would this sequence make sense if a user opens only emails 1, 3, 5?

Product context: [product description, key features, target user]
```

## Why It Works

Decomposition converts a single high-complexity task into multiple lower-complexity subtasks, each within the model's reliable capability range. This matters because model performance degrades non-linearly with task complexity — a task twice as complex doesn't just get slightly worse answers, it gets dramatically worse answers.

Each step creates context that the model uses for subsequent steps. The journey map (Step 1) isn't just an exercise — it becomes the foundation that Step 2 references. This creates a "reasoning chain" where later outputs are grounded in earlier analysis rather than generated from scratch.

The validation step (Step 5) is crucial and often overlooked. It forces the model to review its own work as a coherent whole, catching issues that are invisible at the individual step level (like emails that contradict each other or miss the overall narrative arc).

## Real-World Example

**Scenario:** Analyzing a restaurant's operational efficiency for a consulting engagement.

```
Analyze this restaurant's operational data to identify the top 3
efficiency improvements. Work through each step completely.

STEP 1 — DATA INVENTORY:
List every data point available and categorize as:
- Revenue data (covers, average check, revenue by daypart)
- Cost data (food cost, labor cost, overhead)
- Operational data (table turn time, kitchen ticket times, waste)
- Staff data (hours, roles, scheduling patterns)

STEP 2 — BENCHMARK COMPARISON:
For each metric from Step 1, compare against industry benchmarks
for a [cuisine type] restaurant doing [$X annual revenue]:
- Flag any metric more than 10% worse than benchmark
- Flag any metric more than 20% better (possible data error or
  unsustainable practice)

STEP 3 — ROOT CAUSE ANALYSIS:
For each flagged metric from Step 2, identify the most likely
root cause:
- Is it a people problem (training, staffing levels)?
- Is it a process problem (workflow, layout, procedures)?
- Is it a product problem (menu complexity, ingredient sourcing)?

STEP 4 — RECOMMENDATIONS:
For the top 3 issues by impact, provide:
- Specific change to implement
- Expected improvement (quantified)
- Implementation timeline
- Investment required vs. ROI
- Biggest risk or obstacle

Restaurant data:
[operational data]
```

## Common Mistakes

1. **Steps that don't build on each other.** If Step 3 doesn't reference Steps 1-2, you have a list of independent tasks, not a decomposition. Each step should explicitly use outputs from previous steps.

2. **Too many steps.** Beyond 5-6 steps in a single prompt, the model starts losing coherence. If you need more, switch to a [Prompt Chaining Pipeline](../advanced/14-prompt-chaining-pipelines.md) with separate prompts per stage.

> [!WARNING]
> **Skipping the validation step.** Without a final review step, the model never checks its own work holistically. The individual steps may each be solid but collectively miss the mark. Always include a "review the whole thing" step at the end.

## Related Patterns
- [Chain-of-Thought](01-chain-of-thought.md) — Focuses on reasoning steps within a single analytical task, while decomposition breaks the entire task into subtasks.
- [Prompt Chaining Pipelines](../advanced/14-prompt-chaining-pipelines.md) — When decomposition within a single prompt isn't enough, chain multiple prompts with code in between.
- [Self-Consistency Checking](../intermediate/06-self-consistency-checking.md) — Run the decomposed task multiple times and compare to catch errors in complex analyses.
