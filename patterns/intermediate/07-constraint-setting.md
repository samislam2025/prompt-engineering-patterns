# 🚧 Constraint Setting
**Category:** Intermediate | **Difficulty:** Intermediate | **Impact:** High

## When To Use
Use constraint setting when the model consistently goes off-track in predictable ways — too verbose, too generic, wrong scope, wrong audience, or including information you didn't ask for. Constraints are the guardrails that keep the model in the lane you need.

## The Problem

```
❌ Naive Prompt
```
```
Write a cover letter for a product management role at a fintech startup.
```

**What goes wrong:** You get a full-page cover letter hitting every cliché: "I am excited to apply," "I believe my unique combination of skills," "I would welcome the opportunity to discuss." It's 400 words when the hiring manager reads 100. It mentions irrelevant experience. It's generic enough to send to any PM role at any company — which means it's compelling for none.

**Why it fails:** Without constraints, the model defaults to "more is more." It includes everything it can think of because it has no basis for exclusion. The output is comprehensive but unfocused, like a buffet when you needed a chef's tasting menu.

## The Pattern

```
✅ Engineered Prompt
```
```
Write a cover letter with these constraints:

LENGTH: Maximum 150 words. Every sentence must earn its place.

STRUCTURE:
- Opening: One sentence connecting a specific company initiative
  to your relevant experience. No "I am excited to apply."
- Body: Two concrete examples of impact with numbers.
  Only include experience from the last 5 years.
- Close: One sentence stating your specific interest. No generic
  "would welcome the opportunity" language.

EXCLUSIONS:
- Do not mention soft skills (leadership, communication, teamwork)
  without a specific example proving them.
- Do not reference the job description back at the reader — they
  wrote it, they know what it says.
- Do not use any phrase that could appear in 100 other cover letters.

TONE: Confident and specific, not formal and hedging.
Write as if talking to a peer, not petitioning a gatekeeper.

Role: Product Manager at [Company]
Key experience: [candidate background]
Company research: [specific initiative or product to reference]
```

## Why It Works

Constraints work by narrowing the output distribution. Instead of the model choosing from the full space of possible cover letters, it's choosing from the much smaller space of cover letters that meet all specified requirements. This smaller space is where the quality lives.

Exclusion constraints ("do not mention," "no generic language") are often more powerful than inclusion constraints. This is because models have strong priors toward certain patterns (like "I am excited to apply" in cover letters). Simply asking for a "unique" cover letter won't override these priors, but explicitly banning the pattern will.

Word limits force prioritization. At 150 words, the model must choose the two strongest examples instead of listing five mediocre ones. This editorial pressure typically produces better output than asking for "concise" content with no hard limit.

## Real-World Example

**Scenario:** Generating daily specials descriptions for a restaurant's social media.

```
Write today's special description for Instagram.

CONSTRAINTS:
- Exactly 2-3 sentences. No more.
- First sentence: the dish name and one sensory detail (taste,
  texture, aroma). Not "delicious" or "amazing."
- Second sentence: one specific ingredient or technique that
  makes this version special.
- Optional third sentence: pairing suggestion (drink or side).
- No hashtags in the body text (I'll add them separately).
- No superlatives (best, finest, incredible, unbelievable).
- No questions to the audience ("Who's hungry?" "Ready for this?").
- Write at a 6th grade reading level. Simple words, short sentences.

DISH: Pan-seared Chilean sea bass with miso glaze
KEY DETAILS: 48-hour miso marinade, served on crispy rice,
yuzu beurre blanc, available tonight only
```

## Common Mistakes

1. **Conflicting constraints.** "Be comprehensive AND keep it under 100 words" is a trap. Prioritize which constraint matters more and be explicit about it.

2. **Too many constraints.** Beyond 6-8 constraints, the model starts dropping some to satisfy others. If you need more, consider whether some constraints are actually format specifications (use [Output Formatting](../foundational/04-output-formatting.md)) or role attributes (use [Role Assignment](../foundational/03-role-assignment.md)).

> [!WARNING]
> **Vague constraints that the model can't operationalize.** "Make it sound natural" or "keep it professional" are too subjective to enforce. Instead: "Use contractions" (natural) or "No slang, no emoji, no exclamation marks" (professional). Constraints must be binary — the model should be able to verify whether it met them.

## Related Patterns
- [Output Formatting](../foundational/04-output-formatting.md) — Formatting is a specific type of constraint focused on structure rather than content.
- [Negative Examples](08-negative-examples.md) — When constraints alone aren't enough, show the model what violating them looks like.
- [Adversarial Guardrails](../advanced/12-adversarial-guardrails.md) — Production-grade constraints that prevent misuse and edge-case failures.
