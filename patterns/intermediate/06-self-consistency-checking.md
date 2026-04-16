# 🔄 Self-Consistency Checking
**Category:** Intermediate | **Difficulty:** Intermediate | **Impact:** Very High

## When To Use
Use self-consistency when the task has a single correct answer (or a narrow range of acceptable answers) but the model's reasoning path is unreliable. This pattern is essential for high-stakes decisions where a wrong answer has real consequences — financial analysis, medical triage, legal review, or candidate evaluation.

## The Problem

```
❌ Naive Prompt
```
```
Based on this restaurant's financial data, should the owner open a
second location? Give me a yes or no recommendation with reasoning.
```

**What goes wrong:** Run this 5 times and you'll get 3 "yes" and 2 "no" answers, each with equally convincing reasoning. The model constructs a plausible narrative to support whatever direction it drifts toward on a given run. You have no way to know which answer to trust.

**Why it fails:** Complex analytical questions involve multiple reasoning paths, and the model samples one path per generation. That path might emphasize revenue growth (→ yes) or might emphasize debt ratios (→ no). A single run gives you one perspective disguised as a comprehensive analysis.

## The Pattern

```
✅ Engineered Prompt
```
```
Analyze whether this restaurant should open a second location.
Provide THREE independent analyses, each starting from a different
analytical framework:

ANALYSIS 1 — FINANCIAL LENS:
Evaluate based purely on financial metrics: cash flow, debt-to-
equity, current margins, capital requirements, and break-even
timeline for a second location.
Recommendation: [YES/NO] with confidence [HIGH/MEDIUM/LOW]

ANALYSIS 2 — OPERATIONAL LENS:
Evaluate based on operational readiness: management depth, systems
scalability, supply chain capacity, and whether the first location
runs without owner presence.
Recommendation: [YES/NO] with confidence [HIGH/MEDIUM/LOW]

ANALYSIS 3 — MARKET LENS:
Evaluate based on market factors: local competition, demand
indicators, real estate availability, and demographic trends
in the target area.
Recommendation: [YES/NO] with confidence [HIGH/MEDIUM/LOW]

SYNTHESIS:
- How many of the 3 analyses agree? If all 3 agree, high confidence.
  If 2/3, moderate confidence. If split, the decision is genuinely
  ambiguous.
- What is the single biggest risk that could override the majority
  recommendation?
- Final recommendation with honest confidence assessment.

Financial data:
[restaurant financial data]
```

## Why It Works

Self-consistency exploits the fact that correct answers tend to be reachable from multiple reasoning paths, while incorrect answers typically depend on a specific (flawed) reasoning chain. By forcing the model to approach the same question from different angles, you're effectively running a vote across reasoning strategies.

This pattern reduces the variance in model outputs. A single analysis might be swayed by whichever data point the model attends to first. Three independent analyses, each constrained to a different framework, ensure comprehensive coverage of the relevant factors.

The synthesis step is where the real value emerges. When the analyses disagree, that disagreement itself is information — it tells you the decision is genuinely close and depends on which factors you weight more heavily. This is vastly more useful than a false-confidence single recommendation.

## Real-World Example

**Scenario:** Evaluating whether a job candidate should accept a counter-offer from their current employer.

```
My career coaching client received a counter-offer. Help me advise
them by running three independent analyses:

ANALYSIS 1 — COMPENSATION ANALYSIS:
Compare total comp (base + bonus + equity + benefits) between
the new offer and counter-offer over a 3-year horizon. Account
for vesting schedules and promotion trajectories.
Verdict: [TAKE COUNTER / LEAVE] — Confidence: [H/M/L]

ANALYSIS 2 — CAREER TRAJECTORY ANALYSIS:
Evaluate which role offers better skill development, title
progression, network expansion, and resume positioning for
their 5-year goals.
Verdict: [TAKE COUNTER / LEAVE] — Confidence: [H/M/L]

ANALYSIS 3 — RISK ANALYSIS:
Assess the counter-offer risk (data shows 50-80% of counter-offer
acceptors leave within 18 months), the new role risk (startup
stage, funding runway), and the relationship dynamics at current
company post-counter-offer.
Verdict: [TAKE COUNTER / LEAVE] — Confidence: [H/M/L]

SYNTHESIS:
Provide a final recommendation noting where the analyses agree
and where they conflict. Flag the single factor that should be
the tiebreaker.

Client situation: [details]
```

## Common Mistakes

1. **Making the analyses too similar.** If all three analyses use the same reasoning approach with slightly different framing, you haven't achieved true independence. Force genuinely different analytical frameworks.

2. **Ignoring disagreement.** When analyses conflict, the temptation is to go with the majority. But a 2-1 split where the dissenting analysis has high confidence should be treated as a yellow flag, not dismissed.

> [!WARNING]
> **Skipping the synthesis step.** Three independent analyses without synthesis is just three opinions. The synthesis — especially when analyses disagree — is where you extract signal from noise. Always force the model to reconcile its own conflicting conclusions.

## Related Patterns
- [Chain-of-Thought](../foundational/01-chain-of-thought.md) — Each analysis within the self-consistency check benefits from chain-of-thought reasoning.
- [Chain of Verification](../advanced/13-chain-of-verification.md) — After self-consistency identifies a recommendation, verification checks the facts behind it.
- [Evaluation-Driven Iteration](../production/16-evaluation-driven-iteration.md) — Use systematic evaluation to measure whether self-consistency actually improves your accuracy.
