# 🔄 Graceful Degradation
**Category:** Production | **Difficulty:** Advanced | **Impact:** Critical

## When To Use
Use graceful degradation when your AI system is user-facing and downtime or errors would impact real people. This pattern ensures that when things go wrong — model API outage, rate limiting, malformed input, unexpected output — users get a reasonable experience instead of a crash or nonsensical response.

## The Problem

```
❌ Naive Prompt
```
```python
# The "it works on my machine" production setup
def generate_response(user_input):
    response = openai.chat.completions.create(
        model="gpt-4",
        messages=[{"role": "user", "content": user_input}]
    )
    return response.choices[0].message.content
```

**What goes wrong:** API is down? Crash. Rate limited? Crash. Model returns empty string? Display empty page. Model returns 10,000 tokens when you expected 50? UI overflow. JSON parsing fails because the model wrapped its JSON in markdown code fences? Crash. This is a demo, not a product.

**Why it fails:** Happy-path code assumes every API call succeeds, every response is well-formed, and every output is within expected bounds. In production, these assumptions fail regularly — API error rates of 1-3% are normal, and at 1000 requests/day, that's 10-30 failures daily.

## The Pattern

```
✅ Engineered Prompt (Degradation Hierarchy)
```
```python
class ResilientAIService:
    """
    Degradation hierarchy:
    Level 0: Full quality (primary model, full prompt)
    Level 1: Reduced quality (fallback model, same prompt)
    Level 2: Cached response (nearest matching cached output)
    Level 3: Template response (pre-written, no AI)
    Level 4: Graceful error (clear message, alternative actions)
    """

    def respond(self, user_input, context):
        # LEVEL 0: Try primary model with full prompt
        try:
            response = self.call_primary_model(user_input, context)
            if self.validate_output(response):
                self.cache_response(user_input, response)
                return response
        except (APIError, Timeout, RateLimitError):
            pass

        # LEVEL 1: Try fallback model
        try:
            response = self.call_fallback_model(user_input, context)
            if self.validate_output(response):
                return response
        except Exception:
            pass

        # LEVEL 2: Try cached response
        cached = self.find_similar_cached(user_input)
        if cached and cached.similarity > 0.85:
            return f"{cached.response}\n\n_Note: This response " \
                   f"is from a similar previous question. If it " \
                   f"doesn't fully address your query, please " \
                   f"try again in a few minutes._"

        # LEVEL 3: Template response
        template = self.get_template_response(user_input)
        if template:
            return template

        # LEVEL 4: Graceful error
        return {
            "message": "I'm having trouble processing your request "
                       "right now. Here's what you can do:",
            "alternatives": [
                "Try again in a few minutes",
                "Check our help center at [link]",
                "Contact support at [email]"
            ]
        }

    def validate_output(self, response):
        """Catch malformed outputs before they reach the user."""
        if not response or len(response.strip()) < 10:
            return False  # Empty or near-empty
        if len(response) > self.MAX_RESPONSE_LENGTH:
            return False  # Runaway generation
        if self.contains_system_prompt_leak(response):
            return False  # Security check
        if not self.format_matches_expected(response):
            return False  # Wrong format
        return True
```

## Why It Works

Graceful degradation treats reliability as a first-class product feature. Users don't care whether your AI is powered by GPT-4 or a cached response — they care that it works when they need it. By defining a degradation hierarchy, you ensure that the user always gets some useful response, even under the worst conditions.

Output validation catches a class of failures that most developers ignore: responses that technically succeed (200 OK from the API) but are functionally broken — empty responses, format violations, system prompt leaks, or runaway length. Catching these in validation rather than displaying them prevents the jarring experience of a broken AI response.

The caching strategy serves dual purposes: it provides fallback responses during outages AND reduces API costs during normal operation by serving cached responses for common queries. A well-maintained cache can handle 20-40% of typical traffic without an API call.

## Real-World Example

**Scenario:** Resilient AI resume review feature for a career coaching platform.

```python
class ResumeReviewService:
    """
    The resume review feature can't just crash — users are
    preparing for interviews with time pressure. Every failure
    path must provide value.
    """

    DEGRADATION_LEVELS = {
        "full": {
            "description": "Complete AI review with personalized feedback",
            "model": "claude-opus",
            "prompt": FULL_REVIEW_PROMPT,
        },
        "quick": {
            "description": "Faster review with standard feedback",
            "model": "claude-haiku",
            "prompt": QUICK_REVIEW_PROMPT,
        },
        "checklist": {
            "description": "Automated checklist (no AI needed)",
            "checks": [
                "resume_length_check",    # Is it 1-2 pages?
                "contact_info_present",   # Phone, email, LinkedIn?
                "quantified_bullets",     # Do bullets have numbers?
                "keyword_density",        # Match against job desc
                "formatting_consistency", # Consistent date formats?
            ]
        },
        "static": {
            "description": "General resume tips (pre-written content)",
            "content": STATIC_RESUME_TIPS,
        }
    }

    def review_resume(self, resume_text, job_description=None):
        # Try full review
        result = self.try_full_review(resume_text, job_description)
        if result:
            return {"level": "full", "review": result}

        # Fall back to quick review
        result = self.try_quick_review(resume_text)
        if result:
            return {
                "level": "quick",
                "review": result,
                "note": "We're providing a streamlined review. "
                        "For the full analysis, try again shortly."
            }

        # Fall back to automated checklist
        result = self.run_checklist(resume_text)
        return {
            "level": "checklist",
            "review": result,
            "note": "Our AI review is temporarily unavailable. "
                    "Here's an automated quality check of your resume."
        }
```

## Common Mistakes

1. **Generic error messages.** "Something went wrong" tells the user nothing. Always provide context and next steps: "Our AI is temporarily unavailable. Try again in 2 minutes, or check our help center for resume tips."

2. **Silent degradation without telling the user.** If you serve a cached response instead of a fresh analysis, say so. Users who think they're getting personalized AI analysis but are actually getting a cached response will lose trust when they discover it.

> [!WARNING]
> **Not monitoring your degradation levels.** If your system degrades to Level 3 (template responses) 15% of the time and nobody notices, you have a quality problem hiding behind a reliability feature. Track how often each degradation level is triggered and alert when non-primary levels exceed thresholds.

## Related Patterns
- [Multi-Model Routing](18-multi-model-routing.md) — Routing between models is Level 0-1 of the degradation hierarchy.
- [Regression Testing Prompts](17-regression-testing-prompts.md) — Test that each degradation level works correctly.
- [Adversarial Guardrails](../advanced/12-adversarial-guardrails.md) — Output validation is a guardrail against malformed or leaked responses.
