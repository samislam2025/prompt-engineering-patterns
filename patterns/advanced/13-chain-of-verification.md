# ✅ Chain of Verification
**Category:** Advanced | **Difficulty:** Advanced | **Impact:** Very High

## When To Use
Use chain of verification when factual accuracy matters and hallucination is a real risk. This pattern adds a self-checking layer where the model verifies its own claims against the source material. Essential for any application where users will make decisions based on the model's output — financial analysis, medical triage, legal research, or technical documentation.

## The Problem

```
❌ Naive Prompt
```
```
Based on this company's financial report, summarize the key metrics
and provide investment insights.

[Financial report]
```

**What goes wrong:** The model produces a confident summary that includes metrics not actually in the report. It might state "revenue grew 23% year-over-year" when the report shows 18%. It invents a gross margin figure that sounds plausible but isn't stated anywhere. The summary reads well and feels trustworthy, which makes the errors more dangerous.

**Why it fails:** Models are optimized to produce fluent, plausible text. When summarizing data, the model sometimes "smooths" numbers to be rounder, "fills in" expected metrics that aren't actually present, or confuses similar-sounding figures. Without a verification step, these errors pass through uncaught.

## The Pattern

```
✅ Engineered Prompt
```
```
Analyze this financial report in two phases:

PHASE 1 — EXTRACTION AND ANALYSIS:
Summarize the key metrics and provide analysis. For every specific
number, percentage, or claim you make, mark it with a [SOURCE]
tag indicating where in the report it came from.

Format: "Revenue was $4.2M [SOURCE: Page 3, Revenue Summary Table]"

If you want to include a metric but cannot find it explicitly
stated in the report, mark it [INFERRED] and explain your
calculation.

PHASE 2 — SELF-VERIFICATION:
Review your Phase 1 output. For each claim:

| # | Claim | Source Tag | Verification |
|---|-------|-----------|--------------|
| 1 | [claim] | [source tag] | VERIFIED: [quote from source] / UNVERIFIED: [could not confirm] / CORRECTED: [what it should be] |

After verification:
- Remove or flag any UNVERIFIED claims
- Correct any errors found
- Produce the FINAL verified summary with only confirmed information

Financial report:
[report text]
```

## Why It Works

Chain of verification leverages the model's ability to cross-reference within its own context window. By requiring source citations in Phase 1, the model is forced to ground each claim in specific source material rather than generating from its parametric memory. The Phase 2 table then creates a structured self-review where the model systematically checks each claim.

The verification table format is critical — it forces binary accountability for each claim. The model can't hand-wave with "the summary is broadly accurate." It must mark each specific claim as VERIFIED, UNVERIFIED, or CORRECTED. This granularity catches errors that a general "review your work" instruction would miss.

The [INFERRED] tag is a deliberate safety valve. Without it, the model might present inferred information as extracted fact. By providing a legitimate way to flag inferences, you reduce the incentive to fabricate sources. This mirrors best practices in journalism — it's fine to make inferences as long as they're clearly labeled.

## Real-World Example

**Scenario:** Extracting and verifying details from a job posting for a career coaching client.

```
Analyze this job posting for my client. Use the two-phase approach:

PHASE 1 — EXTRACTION:
Extract these fields from the posting. For each, cite the exact
text you're pulling from:

- Required years of experience: [X] [SOURCE: "exact quote"]
- Salary range: [X] [SOURCE: "exact quote" or INFERRED: reason]
- Must-have skills: [list] [SOURCE for each]
- Nice-to-have skills: [list] [SOURCE for each]
- Red flags: [list] [REASONING for each]
- Remote policy: [X] [SOURCE: "exact quote"]

PHASE 2 — VERIFICATION:
Check your extraction against the original posting:
1. Did you mark anything as required that the posting lists
   as "preferred" or "nice to have"?
2. Did you infer a salary range that isn't explicitly stated?
3. Are your red flags based on what's actually written, or on
   assumptions about what phrases typically mean?

Correct any errors and produce the verified extraction.

Job posting:
[full job posting text]
```

## Common Mistakes

1. **Treating verification as optional.** If verification is a "nice to have" step that the model can skip, it will skip it when the context is long or the task is complex — exactly when you need it most. Make it structurally required.

2. **Only verifying numbers.** Qualitative claims are just as prone to hallucination. "The company is expanding its AI division" might be an inference from hiring patterns, not a stated fact. Verify qualitative claims too.

> [!WARNING]
> **Trusting the model's own verification unconditionally.** Self-verification catches many errors but isn't perfect — the model can verify a hallucinated fact against its own hallucinated source. For high-stakes applications, combine chain of verification with programmatic fact-checking (regex for numbers, entity matching against source documents) and human spot-checks.

## Related Patterns
- [Chain-of-Thought](../foundational/01-chain-of-thought.md) — Verification is a specialized reasoning chain focused on accuracy rather than analysis.
- [Self-Consistency Checking](../intermediate/06-self-consistency-checking.md) — Run verification multiple times for additional confidence.
- [Evaluation-Driven Iteration](../production/16-evaluation-driven-iteration.md) — Measure how much verification actually reduces error rates in your specific use case.
