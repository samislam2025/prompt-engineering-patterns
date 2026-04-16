# 🔀 Multi-Model Routing
**Category:** Production | **Difficulty:** Advanced | **Impact:** Very High

## When To Use
Use multi-model routing when you need to balance cost, latency, and quality across different types of requests. Not every query needs your most expensive model, and not every query can be handled by your cheapest one. Routing puts the right model on the right task.

## The Problem

```
❌ Naive Prompt
```
```
# Current setup: Everything goes to GPT-4/Claude Opus
# Monthly API cost: $4,200
# Average latency: 3.2 seconds
# Team is told: "Use the best model for everything"

Result: 70% of requests are simple lookups or formatting tasks
that a model 10x cheaper handles equally well. The remaining 30%
genuinely need the reasoning power. You're spending $2,940/month
on overkill.
```

**What goes wrong:** Every request — from "format this JSON" to "analyze this 50-page contract" — hits the same expensive, slow model. Simple tasks that could complete in 200ms take 3 seconds. Your API bill is 3-4x what it needs to be. When you try to cut costs by switching everything to a cheaper model, the complex tasks suffer.

**Why it fails:** One-model-fits-all is the "using a sledgehammer for every nail" approach. It optimizes for simplicity of implementation at the cost of both money and performance.

## The Pattern

```
✅ Engineered Prompt (Router Architecture)
```
```python
# MODEL ROUTING SYSTEM

# Tier 1: Fast/Cheap — Claude Haiku, GPT-4o-mini
# Use for: extraction, formatting, classification, simple Q&A
# Latency: <500ms | Cost: ~$0.25/1M tokens

# Tier 2: Balanced — Claude Sonnet, GPT-4o
# Use for: summarization, moderate analysis, content generation
# Latency: ~1.5s | Cost: ~$3/1M tokens

# Tier 3: Maximum — Claude Opus, GPT-4 Turbo, o1
# Use for: complex reasoning, multi-step analysis, nuanced writing
# Latency: ~3-5s | Cost: ~$15/1M tokens

class ModelRouter:
    def route(self, task):
        """Route task to optimal model tier."""

        # Rule-based routing for clear-cut cases
        if task.type in ["json_formatting", "data_extraction",
                         "simple_classification"]:
            return Tier.FAST

        if task.type in ["content_moderation", "summarization",
                         "email_drafting"]:
            return Tier.BALANCED

        if task.type in ["legal_analysis", "financial_reasoning",
                         "system_prompt_generation"]:
            return Tier.MAXIMUM

        # Complexity-based routing for ambiguous cases
        complexity_score = self.estimate_complexity(task)
        if complexity_score < 3:
            return Tier.FAST
        elif complexity_score < 7:
            return Tier.BALANCED
        else:
            return Tier.MAXIMUM

    def estimate_complexity(self, task):
        """Quick complexity estimation using a Tier 1 model."""
        response = tier_1_model.complete(f"""
        Rate the complexity of this task from 1-10:
        1-3: Simple extraction, formatting, or lookup
        4-6: Moderate analysis or content generation
        7-10: Complex reasoning, multi-step analysis, or nuanced judgment

        Task: {task.description}
        Input length: {task.input_tokens} tokens

        Output ONLY a number 1-10.
        """)
        return int(response)
```

## Why It Works

Multi-model routing is cost optimization without quality compromise. The key insight is that model capability is not a single dimension — a fast, cheap model can match or exceed an expensive model on simple tasks because the task doesn't require the additional reasoning capability.

Using a Tier 1 model as the complexity classifier is a powerful technique. The classification call costs virtually nothing (a few tokens in, one token out), completes in <200ms, and saves you from routing simple tasks to a $15/1M token model. Even if the classifier is wrong 10% of the time, the cost savings from the other 90% easily justify it.

Cascade routing is another strategy: start with Tier 1, check if the output meets quality thresholds, escalate to Tier 2 only if needed. This ensures simple tasks stay fast and cheap while complex tasks still get the reasoning power they need.

## Real-World Example

**Scenario:** AI-powered customer support for a career coaching SaaS platform.

```python
class CareerCoachRouter:
    """Route support and coaching requests to appropriate models."""

    ROUTING_RULES = {
        # Tier 1 — Instant responses
        "faq_lookup": {
            "model": "haiku",
            "examples": [
                "What are your pricing plans?",
                "How do I cancel my subscription?",
                "Do you offer refunds?"
            ],
            "sla": "500ms"
        },

        # Tier 2 — Standard coaching
        "resume_formatting": {
            "model": "sonnet",
            "examples": [
                "Can you reformat my resume bullet points?",
                "Convert my resume to a different template",
                "Add keywords from this job posting to my resume"
            ],
            "sla": "2s"
        },

        # Tier 3 — Deep analysis
        "career_strategy": {
            "model": "opus",
            "examples": [
                "Should I take this counter-offer or leave?",
                "Map my 10-year restaurant career to PM skills",
                "Evaluate these 3 job offers holistically"
            ],
            "sla": "5s"
        }
    }

    def route_with_fallback(self, query, context):
        """Route with automatic escalation on low-confidence outputs."""
        tier = self.classify(query)
        response = self.call_model(tier, query, context)

        # Quality gate: if Tier 1/2 output seems uncertain, escalate
        if tier != "opus" and self.confidence_score(response) < 0.7:
            response = self.call_model("opus", query, context)

        return response
```

## Common Mistakes

1. **Routing based only on input length.** Long inputs aren't necessarily complex. A 5000-token document for simple extraction routes to Tier 1. A 50-token question requiring deep reasoning routes to Tier 3. Length is a factor, not the factor.

2. **Not monitoring routing accuracy.** Track cases where Tier 1 outputs are poor quality — these are routing errors that should refine your classifier. Without monitoring, your router degrades silently.

> [!WARNING]
> **Not having a fallback path.** If your Tier 1 model is down or overloaded, does the system crash or gracefully escalate? Always implement cascading fallback: Tier 1 fails → try Tier 2 → try Tier 3 → return cached response or graceful error. See [Graceful Degradation](19-graceful-degradation.md).

## Related Patterns
- [Prompt Chaining Pipelines](../advanced/14-prompt-chaining-pipelines.md) — Different pipeline stages can use different model tiers.
- [Evaluation-Driven Iteration](16-evaluation-driven-iteration.md) — Measure whether routing decisions are correct by evaluating output quality per tier.
- [Graceful Degradation](19-graceful-degradation.md) — What happens when a model tier is unavailable.
