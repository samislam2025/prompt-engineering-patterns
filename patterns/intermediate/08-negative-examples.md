# 🚫 Negative Examples
**Category:** Intermediate | **Difficulty:** Intermediate | **Impact:** High

## When To Use
Use negative examples when the model keeps producing a specific failure mode that you can't fix with instructions alone. This pattern is the "show me what NOT to do" complement to few-shot examples. Particularly effective for tone/style calibration and eliminating persistent bad habits.

## The Problem

```
❌ Naive Prompt
```
```
Write a professional but warm rejection email to a job candidate.
Don't be too corporate or too casual.
```

**What goes wrong:** "Not too corporate, not too casual" is an impossible instruction to follow consistently. The model oscillates between stilted HR-speak ("We regret to inform you that after careful deliberation...") and overly familiar language ("Hey! So unfortunately..."). Every run lands at a different point on the spectrum.

**Why it fails:** "Don't be too X" doesn't tell the model where the boundary is. It knows the extremes but can't reliably find the middle ground without examples showing where that line sits.

## The Pattern

```
✅ Engineered Prompt
```
```
Write a rejection email to a job candidate. Here's exactly the
tone I need — defined by what it is NOT:

❌ TOO CORPORATE (Don't write like this):
"Dear Mr. Johnson, After thorough evaluation of your application
materials and careful consideration by our hiring committee, we
regret to inform you that we have elected to move forward with
other candidates whose qualifications more closely align with
our current requirements."
WHY IT'S WRONG: Reads like a legal document. "Regret to inform"
is a phrase no human would say out loud. Passive voice hides
behind process.

❌ TOO CASUAL (Don't write like this):
"Hey David! Thanks so much for chatting with us — you're clearly
super talented! Unfortunately we're going a different direction
but I just KNOW you'll find something amazing. Keep crushing it! 🎉"
WHY IT'S WRONG: Insincerely enthusiastic. Exclamation points and
emoji in a rejection feel tone-deaf. "Keep crushing it" after a
rejection is patronizing.

✅ TARGET TONE (Write like this):
"David — thank you for the time you put into our interview process.
The conversation about your approach to menu cost optimization was
genuinely interesting. We've decided to move forward with another
candidate, but I'd be happy to share specific feedback if that
would be useful. Best, Sarah"
WHY IT WORKS: Direct, human, respectful. Mentions something
specific to prove the email isn't a template. Offers value
(feedback) instead of empty platitudes.

Now write a rejection email for this scenario:
[candidate context, interview details, reason for rejection]
```

## Why It Works

Negative examples exploit contrastive learning — the model understands the desired output better when it can see the boundaries of what's unacceptable. Providing a "too far left" and "too far right" example triangulates the sweet spot more precisely than any adjective-based description.

The "WHY IT'S WRONG" annotations are essential. Without them, the model might identify surface features of the bad examples (paragraph length, formality of greeting) and avoid only those, while missing the deeper issues (passive voice, insincerity). The annotations tell the model which features to avoid, not just which examples to avoid.

This pattern also reduces the prompt engineering iteration cycle. Instead of revising instructions across multiple attempts ("less formal" → "not that casual" → "somewhere in between"), you capture the failure modes you've already encountered and inoculate the prompt against them upfront.

## Real-World Example

**Scenario:** Writing product descriptions for an AI SaaS tool's landing page.

```
Write a product description for our AI resume review feature.

❌ DON'T WRITE: Hype copy
"Revolutionary AI technology that will TRANSFORM your resume
and guarantee you land your dream job! Our cutting-edge neural
networks analyze thousands of data points to create the perfect
resume every single time."
WHY IT'S BAD: Unsubstantiated claims ("guarantee"), superlatives
that erode trust ("revolutionary," "perfect"), no specifics about
what it actually does.

❌ DON'T WRITE: Feature-list dump
"Our resume tool uses NLP for keyword extraction, supports 15
file formats, integrates with 200+ ATS systems, provides
readability scoring, offers template customization, and includes
batch processing capabilities."
WHY IT'S BAD: Lists features without connecting to user outcomes.
Reads like a spec sheet, not a value proposition. Nobody cares
about "NLP for keyword extraction."

✅ TARGET STYLE:
Specific, benefit-led, honest about what the tool does and
doesn't do. Show one clear before/after. Lead with the outcome,
explain the mechanism, include a real limitation.

Product details: [feature specifications]
Target user: Mid-career professionals actively job hunting
```

## Common Mistakes

1. **Negative examples that are too obviously bad.** If your bad example is something no model would produce anyway, it doesn't help. Use actual failure outputs from your testing.

2. **Only showing one boundary.** A single negative example says "not this" but doesn't define the direction. Two boundaries (too much, too little) create a corridor the model can navigate.

> [!WARNING]
> **Providing negative examples without explaining WHY they're wrong.** The model might avoid superficial features of the bad example (same opening word, same length) while reproducing the underlying problem (same tone, same logical structure). Always annotate what specifically makes each example bad.

## Related Patterns
- [Few-Shot Examples](../foundational/02-few-shot-examples.md) — Use positive examples alongside negative ones for maximum calibration.
- [Constraint Setting](07-constraint-setting.md) — Constraints define rules; negative examples illustrate what breaking those rules looks like.
- [Adversarial Guardrails](../advanced/12-adversarial-guardrails.md) — Negative examples for safety-critical edge cases.
