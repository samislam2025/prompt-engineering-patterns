# 🔗 Chain-of-Thought Prompting
**Category:** Foundational | **Difficulty:** Beginner | **Impact:** Very High

## When To Use
Use chain-of-thought when the task requires multi-step reasoning, math, logic, or any decision where the model needs to "show its work" to arrive at a correct answer. This is your go-to pattern whenever a direct answer consistently comes back wrong or inconsistent.

## The Problem

```
❌ Naive Prompt
```
```
Analyze this candidate's resume and tell me if they're a good fit for our Senior ML Engineer role.

Resume: [candidate resume]
Job Description: [job description]
```

**What goes wrong:** The model produces a vague "yes" or "no" with surface-level reasoning. It pattern-matches on keyword overlap (Python, TensorFlow) rather than evaluating deeper fit signals like system design experience, scale of data worked with, or leadership trajectory. The output feels like a coin flip with extra words.

**Why it fails:** Without explicit reasoning steps, the model compresses a complex multi-factor evaluation into a single forward pass. It skips the analytical work that makes the conclusion trustworthy.

## The Pattern

```
✅ Engineered Prompt
```
```
You are a senior technical recruiter evaluating ML Engineer candidates.

Analyze this candidate step by step:

1. SKILLS MATCH: List each required skill from the job description.
   For each, note whether the resume demonstrates it and at what level
   (none / basic / intermediate / expert). Cite specific evidence.

2. EXPERIENCE DEPTH: Evaluate years of relevant experience, scale of
   systems built (users, data volume, team size), and progression of
   responsibility.

3. GAPS: Identify any critical requirements from the JD that the resume
   does not address. Distinguish between hard gaps (missing must-haves)
   and soft gaps (missing nice-to-haves).

4. CULTURE SIGNALS: Note any indicators of collaboration style,
   communication ability, or leadership from the resume's project
   descriptions and achievements.

5. FINAL ASSESSMENT: Based ONLY on your analysis in steps 1-4, provide
   a fit score (1-10) and a 2-sentence recommendation with your
   top concern and top strength.

Resume: [candidate resume]
Job Description: [job description]
```

## Why It Works

Chain-of-thought forces the model to allocate compute to each reasoning step before reaching a conclusion. In transformer architectures, each token generated becomes part of the context for subsequent tokens. By requiring intermediate analysis, you're effectively giving the model a "scratchpad" — the reasoning in steps 1-4 becomes context that directly informs the quality of step 5.

This mirrors how humans make better decisions when they write out pros and cons rather than going with gut instinct. The model can't skip to a conclusion because the prompt structure requires it to generate evidence first. When that evidence contradicts an initial pattern-match, the model self-corrects because the reasoning tokens are now in context.

Research from Wei et al. (2022) showed chain-of-thought prompting improved accuracy on complex reasoning tasks by 20-50%. The key insight is that it's not the instructions that matter — it's forcing the model to generate intermediate tokens that carry reasoning information.

## Real-World Example

**Scenario:** Evaluating whether a career coaching client should pivot from restaurant management to product management.

```
Analyze whether this career pivot makes sense, step by step:

1. TRANSFERABLE SKILLS: Map each skill from 8 years of restaurant
   management to its product management equivalent:
   - Menu optimization → A/B testing and data-driven iteration
   - Staff scheduling → Resource allocation and sprint planning
   - Customer complaint resolution → User feedback triage
   - P&L management → Business metrics and KPI ownership
   - Vendor negotiations → Stakeholder management

2. SKILL GAPS: Identify what a restaurant manager typically lacks
   for PM roles (technical fluency, product analytics tools,
   roadmapping frameworks).

3. MARKET REALITY: Assess how hiring managers in tech actually
   view non-traditional PM candidates. Be honest about bias.

4. BRIDGE STRATEGY: Based on steps 1-3, recommend the most
   efficient path to a first PM role (timeline, certifications,
   portfolio projects, target company types).
```

**Output quality:** Instead of generic "yes, you have transferable skills" advice, this produces a tactical roadmap with honest assessment of both strengths and gaps. The step-by-step structure forces the model to confront the market reality (step 3) before recommending a strategy (step 4), preventing over-optimistic advice.

## Common Mistakes

1. **Making steps too granular.** Ten micro-steps create busywork without improving reasoning. Aim for 3-6 meaningful analytical stages.

2. **Forgetting to connect the chain.** Each step should explicitly build on previous ones. Adding "Based on your analysis above" to the final step forces the model to reference its own reasoning rather than generating a disconnected conclusion.

> [!WARNING]
> **Not anchoring the conclusion to the reasoning.** If you don't explicitly say "Based ONLY on steps 1-4, provide your assessment," the model may generate reasoning that points one direction and then contradict itself in the conclusion. Always tie the output to the chain.

## Related Patterns
- [Step-by-Step Decomposition](05-step-by-step-decomposition.md) — Takes chain-of-thought further by breaking the entire task into subtasks, not just reasoning steps.
- [Self-Consistency Checking](../intermediate/06-self-consistency-checking.md) — Run chain-of-thought multiple times and compare outputs to catch reasoning errors.
- [Chain of Verification](../advanced/13-chain-of-verification.md) — Adds a verification layer after the reasoning chain to fact-check the model's own conclusions.
