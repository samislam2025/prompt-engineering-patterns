# 📦 Prompt Versioning
**Category:** Production | **Difficulty:** Advanced | **Impact:** Critical

## When To Use
Use prompt versioning when prompts are a core part of your product and changes need to be tracked, compared, rolled back, and audited. If you've ever asked "which version of the prompt was running when that user complained?", you need this pattern.

## The Problem

```
❌ Naive Prompt
```
```python
# The "prompts in code" anti-pattern
SYSTEM_PROMPT = """
You are a helpful assistant...
[last edited by: ???]
[last change: something about tone???]
[is this the version that fixed the billing FAQ issue or the one before?]
"""

# The Slack message approach to prompt management:
# "Hey, I updated the prompt. It should be better now."
# "Which prompt?"
# "The support one."
# "There are 4 support prompts."
# "The main one."
```

**What goes wrong:** Nobody knows which version is running. When a bug appears, you can't determine which prompt version caused it. When you want to roll back, you dig through git history (if the prompt is even in git) trying to find the last known-good version. A/B testing is impossible because you can't cleanly separate versions. Post-mortems are guesswork.

**Why it fails:** Treating prompts as casual text instead of versioned artifacts makes prompt engineering a black box. Every other part of your stack — code, infrastructure, models — has version control, deployment pipelines, and rollback capability. Prompts, which directly control user-facing behavior, have none.

## The Pattern

```
✅ Engineered Prompt (Version Control System)
```
```python
# prompts/support_agent/v3.2.1.yaml
---
metadata:
  id: support_agent
  version: "3.2.1"
  created: "2024-11-15"
  author: "sayem@aisecondacts.com"
  model: "claude-sonnet-4-6"
  temperature: 0.3
  changelog: |
    v3.2.1 - Fixed regression where enterprise customers got casual tone
    v3.2.0 - Added billing FAQ handling, reduced response length
    v3.1.0 - Rewrote tone guidelines after user complaints about formality
    v3.0.0 - Major rewrite: moved from single prompt to system prompt architecture
    v2.x   - Legacy single-prompt version (deprecated)
  eval_scores:
    accuracy: 0.87
    tone_consistency: 0.92
    guardrail_compliance: 0.98
    overall: 0.89
  approved_by: "team_lead"
  rollback_to: "v3.1.0"  # known safe version

system_prompt: |
  # IDENTITY
  You are a support agent for [Product]...
  [full system prompt]

test_suite: "tests/support_agent_v3.2.1.json"
---

# Deployment configuration
deployment:
  strategy: "canary"  # canary | blue-green | immediate
  canary_percentage: 10
  promotion_criteria:
    min_requests: 1000
    max_error_rate: 0.02
    min_satisfaction_score: 4.2
  auto_rollback:
    trigger: "error_rate > 0.05 OR satisfaction < 3.8"
    target: "v3.1.0"
```

```python
# prompt_manager.py

class PromptVersionManager:
    """
    Manage prompt versions with deployment controls.
    """

    def get_prompt(self, prompt_id, user_context=None):
        """Get the active prompt version, supporting canary deployments."""
        active_versions = self.get_active_versions(prompt_id)

        if len(active_versions) == 1:
            # Standard deployment
            return active_versions[0]

        # Canary: route percentage of traffic to new version
        canary = active_versions[-1]  # newest
        stable = active_versions[0]   # proven

        if self.in_canary_group(user_context, canary.canary_percentage):
            self.log_version_served(prompt_id, canary.version, "canary")
            return canary
        else:
            self.log_version_served(prompt_id, stable.version, "stable")
            return stable

    def rollback(self, prompt_id, reason):
        """Emergency rollback to last known-good version."""
        current = self.get_current_version(prompt_id)
        target = current.metadata.rollback_to
        self.deploy(prompt_id, target)
        self.alert(f"ROLLBACK: {prompt_id} {current.version} → {target}. "
                   f"Reason: {reason}")
```

## Why It Works

Prompt versioning treats prompts as first-class deployment artifacts, giving them the same rigor as application code. This enables capabilities that are impossible without versioning:

**Root cause analysis:** When a user reports a problem, you can check which exact prompt version they received. No more guessing.

**Safe deployment:** Canary deployments route 10% of traffic to the new version. If the new version has issues, only 10% of users are affected, and auto-rollback catches it within minutes.

**A/B testing:** With versioned prompts, you can run controlled experiments comparing prompt versions on the same traffic. "Version A converts at 15%, Version B at 22%" is actionable data.

**Compliance and audit:** For regulated industries, knowing exactly which prompt was running at any point in time is a compliance requirement, not a nice-to-have.

## Real-World Example

**Scenario:** Managing prompt versions for a career coaching chatbot across multiple features.

```yaml
# Prompt inventory for a career coaching platform

prompts:
  resume_reviewer:
    current: v4.1.0
    status: stable
    last_incident: "v3.8.0 - hallucinated salary ranges (2024-09)"
    eval_score: 0.91

  interview_prep:
    current: v2.3.0
    status: canary (v2.4.0 at 15%)
    canary_metrics:
      requests: 847/1000 needed
      error_rate: 0.01 (under 0.02 threshold)
      satisfaction: 4.5 (above 4.2 threshold)

  job_matcher:
    current: v1.5.0
    status: stable
    note: "Frozen until new eval dataset for career pivots is complete"

  cover_letter_gen:
    current: v3.0.0
    status: rollback from v3.1.0
    incident: "v3.1.0 generated letters over 400 words despite
               200-word constraint. Rolled back 2024-10-22.
               Root cause: constraint was in user prompt but
               new system prompt override took precedence."
```

## Common Mistakes

1. **Versioning the prompt text but not the configuration.** The prompt text, model, temperature, max_tokens, and system prompt are all part of the "version." Changing temperature from 0.3 to 0.7 is as impactful as rewriting the prompt.

2. **Not associating eval scores with versions.** Each version should have a recorded eval score so you can objectively compare versions and detect regressions immediately.

> [!WARNING]
> **Deploying new prompt versions without a rollback plan.** Before every deployment, define: what metrics trigger a rollback, which version you'll roll back to, and who has authority to execute the rollback. The cover letter incident in the example above shows what happens when a rollback target isn't pre-defined — precious minutes are wasted determining which version is safe.

## Related Patterns
- [Evaluation-Driven Iteration](16-evaluation-driven-iteration.md) — Eval scores are the quality metric attached to each prompt version.
- [Regression Testing Prompts](17-regression-testing-prompts.md) — Every version must pass the regression suite before deployment.
- [System Prompt Architecture](../advanced/11-system-prompt-architecture.md) — System prompts are the most important artifacts to version, as they control overall behavior.
