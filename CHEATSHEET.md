# 📋 Prompt Engineering Patterns — Quick Reference Cheatsheet

Use this as a scannable reference when you know what you need but want to quickly find the right pattern.

---

## 🧱 Foundational Patterns

| # | Pattern | One-Line Description | When To Use | Difficulty |
|---|---------|---------------------|-------------|------------|
| 🔗 01 | [Chain-of-Thought](patterns/foundational/01-chain-of-thought.md) | Force the model to show its reasoning before concluding | Multi-step reasoning, math, complex analysis | Beginner |
| 🎯 02 | [Few-Shot Examples](patterns/foundational/02-few-shot-examples.md) | Show 2-3 examples of desired output to set quality bar | Consistent tone, format, or style across outputs | Beginner |
| 🎭 03 | [Role Assignment](patterns/foundational/03-role-assignment.md) | Give the model a specific expert persona and behavioral traits | Need domain expertise or specific communication style | Beginner |
| 📋 04 | [Output Formatting](patterns/foundational/04-output-formatting.md) | Define exact schema/structure for model output | Parsing outputs programmatically or comparing results | Beginner |
| 🪜 05 | [Step-by-Step Decomposition](patterns/foundational/05-step-by-step-decomposition.md) | Break complex task into sequential subtasks in one prompt | Task too complex for one pass but doesn't need a pipeline | Beginner |

## ⚡ Intermediate Patterns

| # | Pattern | One-Line Description | When To Use | Difficulty |
|---|---------|---------------------|-------------|------------|
| 🔄 06 | [Self-Consistency Checking](patterns/intermediate/06-self-consistency-checking.md) | Analyze from multiple frameworks, then synthesize | High-stakes decisions where wrong answers have consequences | Intermediate |
| 🚧 07 | [Constraint Setting](patterns/intermediate/07-constraint-setting.md) | Add explicit boundaries on length, scope, tone, and content | Model goes off-track in predictable, repeatable ways | Intermediate |
| 🚫 08 | [Negative Examples](patterns/intermediate/08-negative-examples.md) | Show what NOT to do with annotated bad examples | Persistent failure modes that instructions alone can't fix | Intermediate |
| 📐 09 | [Context Window Management](patterns/intermediate/09-context-window-management.md) | Structure long inputs to prevent "lost in the middle" failures | Working with large documents or many data points | Intermediate |
| 🌡️ 10 | [Temperature & Sampling](patterns/intermediate/10-temperature-and-sampling-strategy.md) | Match randomness parameters to task type | Outputs too repetitive (temp too low) or chaotic (too high) | Intermediate |

## 🔬 Advanced Patterns

| # | Pattern | One-Line Description | When To Use | Difficulty |
|---|---------|---------------------|-------------|------------|
| 🏗️ 11 | [System Prompt Architecture](patterns/advanced/11-system-prompt-architecture.md) | Multi-section system prompt with identity, capabilities, boundaries | Building any persistent AI application or chatbot | Advanced |
| 🛡️ 12 | [Adversarial Guardrails](patterns/advanced/12-adversarial-guardrails.md) | Defense-in-depth against prompt injection and misuse | User-facing prompts with untrusted input | Advanced |
| ✅ 13 | [Chain of Verification](patterns/advanced/13-chain-of-verification.md) | Model verifies its own claims against source material | Factual accuracy matters and hallucination is a real risk | Advanced |
| ⛓️ 14 | [Prompt Chaining Pipelines](patterns/advanced/14-prompt-chaining-pipelines.md) | Multi-prompt pipeline with code logic between stages | Task needs different models, validation, or branching | Advanced |
| 🎲 15 | [Dynamic Few-Shot Selection](patterns/advanced/15-dynamic-few-shot-selection.md) | Programmatically choose examples based on input similarity | Large example bank, limited context, varying input types | Advanced |

## 🏭 Production Patterns

| # | Pattern | One-Line Description | When To Use | Difficulty |
|---|---------|---------------------|-------------|------------|
| 📊 16 | [Evaluation-Driven Iteration](patterns/production/16-evaluation-driven-iteration.md) | Systematic scoring framework replacing gut-feel tuning | Moving from "works in testing" to "works at scale" | Advanced |
| 🧪 17 | [Regression Testing](patterns/production/17-regression-testing-prompts.md) | Automated test suite that catches prompt regressions | Prompts in production where changes could break things | Advanced |
| 🔀 18 | [Multi-Model Routing](patterns/production/18-multi-model-routing.md) | Route requests to optimal model by complexity and cost | Balancing quality, latency, and API spend | Advanced |
| 🔄 19 | [Graceful Degradation](patterns/production/19-graceful-degradation.md) | Fallback hierarchy when AI fails: cache → template → error | User-facing system where downtime impacts real people | Advanced |
| 📦 20 | [Prompt Versioning](patterns/production/20-prompt-versioning.md) | Track, compare, and roll back prompt versions like code | Prompts are core product and changes need auditability | Advanced |

---

## Quick Decision Guide

**"My output is wrong"** → Start with [Chain-of-Thought (#1)](patterns/foundational/01-chain-of-thought.md) + [Self-Consistency (#6)](patterns/intermediate/06-self-consistency-checking.md)

**"My output format is inconsistent"** → [Output Formatting (#4)](patterns/foundational/04-output-formatting.md) + [Few-Shot Examples (#2)](patterns/foundational/02-few-shot-examples.md)

**"My output tone is off"** → [Role Assignment (#3)](patterns/foundational/03-role-assignment.md) + [Negative Examples (#8)](patterns/intermediate/08-negative-examples.md)

**"I'm going to production"** → [System Prompt Architecture (#11)](patterns/advanced/11-system-prompt-architecture.md) + [Adversarial Guardrails (#12)](patterns/advanced/12-adversarial-guardrails.md) + [Evaluation (#16)](patterns/production/16-evaluation-driven-iteration.md)

**"I need to optimize costs"** → [Multi-Model Routing (#18)](patterns/production/18-multi-model-routing.md) + [Context Window Management (#9)](patterns/intermediate/09-context-window-management.md)
