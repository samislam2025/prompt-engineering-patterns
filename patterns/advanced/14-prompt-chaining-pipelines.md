# ⛓️ Prompt Chaining Pipelines
**Category:** Advanced | **Difficulty:** Advanced | **Impact:** Critical

## When To Use
Use prompt chaining when a task is too complex for a single prompt but requires programmatic control between steps — data transformation, conditional logic, external API calls, or quality gates that need code-level validation between model calls.

## The Problem

```
❌ Naive Prompt
```
```
Here's a raw job posting URL. Scrape it, extract the key
requirements, compare them against this resume, identify gaps,
and generate a tailored cover letter that addresses each gap
with specific examples from the resume.
```

**What goes wrong:** The model tries to do everything in one pass and fails at multiple points. It can't actually scrape URLs. It conflates requirements with nice-to-haves. The gap analysis is shallow because it hasn't properly structured the comparison. The cover letter references gaps it didn't actually identify, and the whole thing is 2000 tokens of mediocrity instead of a focused, accurate output at each stage.

**Why it fails:** This task has at least 4 distinct subtasks with different optimal approaches, different context needs, and different quality criteria. Cramming them into one prompt means no subtask gets adequate attention, and errors in early stages cascade into garbage outputs for later stages.

## The Pattern

```
✅ Engineered Prompt (Pipeline Architecture)
```
```python
# STAGE 1: Extract requirements (structured extraction)
# Temperature: 0.0 | Model: fast/cheap model
prompt_1 = """
Extract from this job posting:
{
  "must_have": ["skill1", "skill2"],
  "nice_to_have": ["skill1", "skill2"],
  "years_required": int,
  "role_level": "junior|mid|senior|lead",
  "key_responsibilities": ["resp1", "resp2"],
  "company_values": ["value1", "value2"]
}

Job posting:
{job_posting_text}
"""

# STAGE 2: Resume analysis (structured extraction)
# Temperature: 0.0 | Model: fast/cheap model
# Runs in PARALLEL with Stage 1
prompt_2 = """
Extract from this resume:
{
  "skills": [{"name": "X", "level": "expert|proficient|basic",
              "evidence": "specific project or metric"}],
  "years_experience": int,
  "key_achievements": ["achievement with metric"],
  "career_narrative": "one sentence summary of career arc"
}

Resume:
{resume_text}
"""

# STAGE 3: Gap analysis (analytical reasoning)
# Temperature: 0.2 | Model: strongest available
# Depends on outputs from Stages 1 and 2
prompt_3 = """
Compare these job requirements against this candidate profile.

JOB REQUIREMENTS: {stage_1_output}
CANDIDATE PROFILE: {stage_2_output}

For each must-have requirement:
| Requirement | Candidate Has? | Evidence | Gap Severity |
|-------------|---------------|----------|-------------|

For each gap with severity "High":
- Suggest how to address it in a cover letter using
  transferable skills from the candidate's background.

Output the gap analysis and addressing strategies.
"""

# STAGE 4: Cover letter generation (creative writing)
# Temperature: 0.7 | Model: strongest available
# Depends on all previous stages
prompt_4 = """
Write a cover letter that:
1. Opens with a specific connection between the candidate's
   experience and the company's values: {stage_1.company_values}
2. Addresses EACH high-severity gap from this analysis:
   {stage_3_output}
   For each gap, use the suggested addressing strategy.
3. Highlights the top 2 achievements: {stage_2.key_achievements}
4. Closes with specific interest in the role responsibilities.

Constraints:
- Maximum 200 words
- No generic phrases
- Every claim must reference specific evidence from the resume
"""
```

## Why It Works

Pipeline architecture provides three critical advantages over monolithic prompts:

**Specialization:** Each stage uses the optimal model, temperature, and prompt structure for its specific subtask. Extraction stages use temperature 0 and can use cheaper models. The creative stage uses higher temperature and the strongest model. This optimizes both quality and cost.

**Error isolation:** If Stage 1 misidentifies a requirement, you can catch and fix it before it corrupts the gap analysis and cover letter. In a monolithic prompt, early errors cascade silently. With a pipeline, you can add validation between stages.

**Parallel execution:** Independent stages (1 and 2) run simultaneously, reducing latency. A well-designed pipeline identifies which stages can parallelize and which must be sequential.

The pipeline also enables human-in-the-loop workflows. After Stage 3 (gap analysis), you can present the results to the user for review before generating the cover letter. This makes the AI's reasoning transparent and correctable.

## Real-World Example

**Scenario:** Multi-location restaurant performance analysis pipeline.

```python
# PIPELINE: Weekly Restaurant Operations Report

# STAGE 1: Data normalization (per location, parallel)
# For each location, normalize POS data into standard schema
prompt_per_location = """
Normalize this POS export into our standard format:
{standard_schema}

Raw data from {location_name}:
{raw_pos_data}
"""

# STAGE 2: Individual location analysis (parallel)
prompt_location_analysis = """
Analyze this location's performance for the week:
{normalized_data}

Output:
- Revenue vs. forecast (% over/under)
- Top 3 items by contribution margin
- Bottom 3 items (candidates for menu removal)
- Labor cost as % of revenue
- Any anomalies (unusual spikes or drops)
"""

# STAGE 3: Cross-location comparison (sequential, needs all Stage 2)
prompt_comparison = """
Compare performance across all locations:
{all_location_analyses}

Identify:
- Which location is the best performer and why?
- Are any issues systemic (appearing in 3+ locations)?
- Which location-specific best practices should be shared?
"""

# STAGE 4: Executive summary (sequential, needs Stage 3)
prompt_executive = """
Write a 5-bullet executive summary for the restaurant group
owner. They have 30 seconds to read this.

Detailed analysis: {stage_3_output}

Format:
🟢 [Best news — what's working]
🟡 [Watch item — emerging trend to monitor]
🔴 [Action needed — biggest issue requiring decision]
📊 [Key metric] vs [target]
📋 [One specific recommendation for next week]
"""
```

## Common Mistakes

1. **Passing raw model output between stages without validation.** Always parse and validate intermediate outputs. If Stage 1 returns malformed JSON, catch it and retry before feeding garbage to Stage 2.

2. **Not running independent stages in parallel.** If Stage 1 and Stage 2 don't depend on each other, run them concurrently. This can halve your pipeline latency.

> [!WARNING]
> **Over-engineering the pipeline.** Not every multi-step task needs a pipeline. If you can solve it in 2-3 stages with a single well-structured prompt using [Step-by-Step Decomposition](../foundational/05-step-by-step-decomposition.md), that's simpler and cheaper. Reserve pipelines for tasks where you genuinely need code logic, external calls, or different model configurations between stages.

## Related Patterns
- [Step-by-Step Decomposition](../foundational/05-step-by-step-decomposition.md) — The single-prompt version of chaining; use when you don't need code between steps.
- [Output Formatting](../foundational/04-output-formatting.md) — Reliable formatting is what makes pipeline handoffs work.
- [Multi-Model Routing](../production/18-multi-model-routing.md) — Route different pipeline stages to different models based on complexity and cost.
