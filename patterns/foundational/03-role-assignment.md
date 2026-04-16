# 🎭 Role Assignment
**Category:** Foundational | **Difficulty:** Beginner | **Impact:** High

## When To Use
Use role assignment when the quality of the output depends on domain expertise, perspective, or communication style. Particularly effective when you need the model to prioritize certain types of knowledge over others, or when the same question would get different (and unequally useful) answers from different experts.

## The Problem

```
❌ Naive Prompt
```
```
Review my resume and give me feedback.
```

**What goes wrong:** You get generic advice that tries to please everyone: "Consider adding more action verbs," "Quantify your achievements," "Tailor to the job description." It's the career advice equivalent of "eat healthy and exercise" — technically true, practically useless. The model doesn't know whether to optimize for ATS systems, human recruiters, hiring managers, or the candidate's confidence.

**Why it fails:** Without a role, the model averages across all possible advisors in its training data. You get a blend of career counselor platitudes, resume-writing service upsells, and Reddit advice threads. No single perspective dominates, so no perspective is useful.

## The Pattern

```
✅ Engineered Prompt
```
```
You are a senior technical recruiter at a FAANG company who has
reviewed 10,000+ resumes and conducted 2,000+ phone screens.
You are blunt, specific, and care more about saving the candidate
time than being polite.

Your evaluation framework:
- You spend 6 seconds on initial scan (what do you notice?)
- You look for: quantified impact, technical depth signals,
  career progression logic
- You reject for: buzzword stuffing, vague descriptions,
  unexplained gaps, formatting that wastes space
- You advance candidates who make your job easy — clear story,
  obvious fit, easy to pitch to the hiring manager

Review this resume for a Senior Product Manager role at a
mid-stage B2B SaaS company:

[Resume]

Structure your feedback as:
1. 6-SECOND SCAN: What's your gut reaction? Would you keep reading?
2. TOP 3 STRENGTHS: What makes this candidate stand out?
3. TOP 3 PROBLEMS: What would make you reject or hesitate?
4. SPECIFIC FIXES: For each problem, give exact language the
   candidate should use instead.
```

## Why It Works

Role assignment activates specific clusters of knowledge and communication patterns in the model. When you say "senior technical recruiter," the model shifts its probability distribution toward recruiter-specific vocabulary, evaluation frameworks, and decision patterns. It's not pretending to be a recruiter — it's accessing the subset of its training data that represents how recruiters think and communicate.

The specificity of the role matters enormously. "Career advisor" gives you generic advice. "Senior technical recruiter at a FAANG company who has reviewed 10,000+ resumes" gives you pattern-matched expertise from a very specific slice of the model's knowledge. The numbers aren't arbitrary — they signal the experience level and therefore the sophistication of the feedback.

Adding behavioral traits ("blunt, specific, cares more about saving time than being polite") further constrains the output distribution. Without this, the model defaults to diplomatic, hedge-everything communication. With it, you get actionable directness.

## Real-World Example

**Scenario:** Getting operational feedback on a restaurant's menu pricing strategy.

```
You are a restaurant operations consultant who specializes in
menu engineering and has helped 50+ independent restaurants
improve profitability. You've worked with restaurants doing
$1M-$5M annual revenue.

You think in terms of:
- Food cost percentages (target: 28-32%)
- Menu psychology and price anchoring
- Contribution margin, not just markup
- Cover counts and average check optimization
- The relationship between menu complexity and kitchen efficiency

Analyze this menu and pricing:

[Menu with prices and food costs]

Provide:
1. MARGIN ANALYSIS: Flag any items with food cost above 35% or
   below 20% (likely underpriced or quality concern).
2. PRICING PSYCHOLOGY: Identify 3 pricing mistakes based on
   menu layout and price positioning.
3. MENU ENGINEERING MATRIX: Categorize each item as Star (high
   popularity + high margin), Plow Horse (high popularity + low
   margin), Puzzle (low popularity + high margin), or Dog (low
   both).
4. TOP 3 CHANGES: Specific price adjustments or menu modifications
   to improve average contribution margin by 10%+.
```

## Common Mistakes

1. **Assigning roles that conflict with the task.** Asking a "creative writing professor" to write marketing copy creates tension. The role should naturally align with the output you need.

2. **Using roles that are too generic.** "You are an expert" does almost nothing. "You are a conversion copywriter who has written 500+ landing pages for B2B SaaS companies" does a lot. Specificity is the lever.

> [!WARNING]
> **Stacking contradictory roles.** "You are a supportive career coach who is also brutally honest and critical" creates internal conflict in the prompt. Pick one dominant communication style. If you need multiple perspectives, use separate prompts for each role.

## Related Patterns
- [System Prompt Architecture](../advanced/11-system-prompt-architecture.md) — Role assignment becomes the identity layer of a multi-part system prompt.
- [Few-Shot Examples](02-few-shot-examples.md) — Show the role in action with examples rather than just describing it.
- [Constraint Setting](../intermediate/07-constraint-setting.md) — Add guardrails to keep the role focused on what matters.
