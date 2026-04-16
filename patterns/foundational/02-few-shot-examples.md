# 🎯 Few-Shot Examples
**Category:** Foundational | **Difficulty:** Beginner | **Impact:** Very High

## When To Use
Use few-shot examples when you need consistent output format, tone, or reasoning style that's hard to describe in words alone. Especially effective when the task has subjective quality criteria — showing is faster and more reliable than telling.

## The Problem

```
❌ Naive Prompt
```
```
Write a professional LinkedIn post about my career transition from
restaurant management to tech. Make it engaging and authentic.
```

**What goes wrong:** "Engaging and authentic" means different things to every person and every model run. You'll get a generic, corporate-sounding post full of buzzwords like "thrilled to announce" and "exciting new chapter." The tone varies wildly between runs. There's no consistent voice.

**Why it fails:** Adjectives like "engaging" and "authentic" are underconstrained. The model falls back on the most common patterns in its training data for LinkedIn posts, which are overwhelmingly the polished-but-hollow style that performs worst for actual engagement.

## The Pattern

```
✅ Engineered Prompt
```
```
Write a LinkedIn post about my career transition from restaurant
management to tech. Match the tone and structure of these examples:

EXAMPLE 1:
"I spent 6 years making sure 200 covers went out perfectly every
Friday night. Now I make sure 200,000 API calls go through without
breaking. Different kitchen. Same pressure. Same obsession with
the details that nobody sees but everybody feels.

Restaurant management taught me more about systems thinking than
any CS course could. When your line cook calls in sick at 4pm on
Valentine's Day, you learn real-time resource allocation fast."

EXAMPLE 2:
"Hot take: The best product managers I know aren't from Stanford.
They're from restaurants, retail floors, and call centers.

Why? Because they've spent years watching real humans interact
with real systems under real pressure. No A/B test needed — you
see the UX failure in real time when a customer's face falls."

MY CONTEXT:
- 8 years managing a high-volume restaurant (300+ covers/night)
- Just completed a product management bootcamp
- Starting my first PM role at a B2B SaaS company
- I want to connect my food industry experience to my new role
```

## Why It Works

Few-shot examples function as implicit instructions that are far more information-dense than explicit descriptions. When you provide examples, you're simultaneously communicating tone (conversational, direct), structure (short paragraphs, concrete details first), format (no hashtag spam, no emoji overload), reasoning style (analogy-based), and quality bar (specific details over vague claims).

The model performs in-context learning — it identifies the common patterns across your examples and generalizes them to the new input. This is why 2-3 examples work better than 1: with a single example, the model might copy surface features (specific words, exact structure). With multiple examples, it extracts the underlying pattern.

Critically, few-shot examples also set the quality floor. The model will rarely produce output significantly worse than your worst example. Choose examples that represent your minimum acceptable quality, not your aspirational best.

## Real-World Example

**Scenario:** Generating consistent job description summaries for a career coaching platform.

```
Summarize job descriptions in this format for job seekers.
Match the style of these examples exactly:

INPUT: [Full JD for Senior Data Engineer at Stripe]
OUTPUT:
💰 $180-220K | 📍 Remote (US) | 🏢 Stripe (Fintech)
THE ROLE: Build and maintain data pipelines processing billions
of payment events. You'll own the infrastructure that powers
Stripe's internal analytics and ML models.
ACTUALLY NEED: 5+ years with Spark/Airflow at scale, strong SQL,
experience with data quality frameworks. Python required.
NICE BUT NOT REQUIRED: Kafka experience, dbt, prior fintech.
RED FLAGS TO WATCH: "Fast-paced" appears 4x — likely
under-resourced team. No mention of on-call rotation despite
infrastructure role.

INPUT: [Full JD for Product Manager at Notion]
OUTPUT:
💰 $160-200K | 📍 SF or NYC | 🏢 Notion (Productivity)
THE ROLE: Own the collaboration features roadmap. You'll talk to
enterprise customers weekly and translate pain points into
shipping product.
ACTUALLY NEED: 3+ years PM experience, enterprise B2B background,
strong user research chops. Must be comfortable with SQL.
NICE BUT NOT REQUIRED: Prior experience with productivity tools,
technical background.
RED FLAGS TO WATCH: "Wear many hats" + Series C = role scope
may be undefined. Ask about team size in interview.

Now summarize this job description:
[New JD to summarize]
```

## Common Mistakes

1. **Using examples that are too similar.** If both examples are about engineering roles, the model may struggle with a marketing role. Include variety across your examples to show the range of acceptable outputs.

2. **Providing examples with inconsistent quality.** If one example is detailed and another is shallow, you're giving the model permission to be shallow. Curate examples to your quality bar.

> [!WARNING]
> **Providing too many examples (5+).** More examples consume context window and can actually degrade performance by confusing the model about which patterns to prioritize. 2-3 well-chosen examples outperform 5-6 mediocre ones every time.

## Related Patterns
- [Role Assignment](03-role-assignment.md) — Combine with few-shot to set both the persona and the output format simultaneously.
- [Output Formatting](04-output-formatting.md) — When you need strict structure without the overhead of full examples, output formatting is lighter weight.
- [Dynamic Few-Shot Selection](../advanced/15-dynamic-few-shot-selection.md) — Programmatically choose which examples to include based on the input, maximizing relevance.
