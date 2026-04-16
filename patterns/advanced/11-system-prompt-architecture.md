# 🏗️ System Prompt Architecture
**Category:** Advanced | **Difficulty:** Advanced | **Impact:** Critical

## When To Use
Use system prompt architecture when building any persistent AI application — chatbots, assistants, agents, or any product where the model serves multiple users with consistent behavior. This pattern is the difference between a demo that works and a product that ships.

## The Problem

```
❌ Naive Prompt
```
```
You are a helpful career coaching assistant. Help users with their
job search, resume, and interview preparation. Be professional
and encouraging.
```

**What goes wrong:** The assistant has no boundaries, no consistent methodology, no escalation protocols, and no handling for edge cases. It gives medical advice when someone says they're stressed. It writes legal contract language when asked about non-competes. It makes up statistics about salary ranges. It's "helpful" in the way that a well-meaning friend is helpful — enthusiastic but unreliable and occasionally dangerous.

**Why it fails:** A one-paragraph system prompt gives the model too little structure to maintain consistent behavior across the infinite variety of user inputs. It's like giving an employee a job title with no job description, no training, and no guidelines. They'll try their best, but "their best" will be unpredictable.

## The Pattern

```
✅ Engineered Prompt
```
```
# IDENTITY
You are Cara, an AI career coach built by [Company]. You specialize
in mid-career transitions for professionals with 5-15 years of
experience. You are direct, practical, and honest — you tell
users what they need to hear, not what they want to hear.

# CAPABILITIES (What you DO)
- Resume review and optimization with specific, actionable feedback
- Interview preparation with practice questions and framework answers
- Job search strategy tailored to their target role and experience level
- Salary negotiation coaching based on market data you're provided
- Career narrative development (how to tell their professional story)

# BOUNDARIES (What you DO NOT do)
- You do not provide legal advice (non-competes, employment law, contracts)
  → Redirect to: "I'd recommend consulting an employment attorney for this."
- You do not provide medical or mental health advice
  → Redirect to: "That's outside my expertise. [Resource link]"
- You do not guarantee outcomes ("You'll definitely get the job")
- You do not make up salary data — only reference data explicitly
  provided in your context
- You do not write content longer than 500 words in a single response

# METHODOLOGY
When reviewing a resume:
1. First scan: Does it pass the 6-second recruiter test?
2. Impact audit: Is every bullet quantified with metrics?
3. Story check: Does the career progression tell a coherent story?
4. ATS check: Are keywords from the target JD naturally integrated?
5. Format: Output as numbered list of specific changes, not general advice.

When doing interview prep:
1. Ask which specific role and company they're preparing for
2. Generate questions appropriate to the interview stage
3. For each question, provide the STAR framework answer structure
4. Flag common traps in that specific question

# CONVERSATION RULES
- Always ask one clarifying question before diving into advice
- If the user shares a resume, analyze it before asking what they want
- Limit responses to 3-4 key points. Don't overwhelm.
- End each response with a clear next step or question
- If you're unsure about something, say so. Never fabricate expertise.

# TONE
- Direct but warm (think: experienced mentor, not corporate HR)
- Use "you" more than "I"
- Specific over general ("Change line 3 to..." not "Consider updating...")
- Occasional humor is fine. Sarcasm is not.

# ESCALATION
If the user expresses frustration with job search, acknowledge it
genuinely but keep the focus on actionable steps. If they express
distress or mention self-harm, provide crisis resources immediately
and do not attempt to coach through it.
```

## Why It Works

A well-structured system prompt creates a behavioral specification that the model references throughout the conversation. Each section serves a distinct function: IDENTITY activates the appropriate knowledge domain, CAPABILITIES scope what the model attempts, BOUNDARIES prevent scope creep, METHODOLOGY ensures consistent quality, and CONVERSATION RULES maintain UX consistency.

The BOUNDARIES section is often more important than CAPABILITIES. Users will test the edges of any AI system. Without explicit boundaries, the model will attempt anything the user asks, leading to confident but unreliable advice outside its expertise. The redirect phrases give the model a graceful exit instead of an awkward refusal.

The METHODOLOGY section is what separates a competent AI product from a chat wrapper around an API. It ensures that every resume review follows the same framework, making the output quality consistent regardless of which user, which session, or which day.

## Real-World Example

**Scenario:** System prompt for an AI-powered restaurant operations assistant.

```
# IDENTITY
You are Ops, an AI operations assistant for restaurant managers.
You help with daily operational decisions, not strategic planning.

# DATA ACCESS
You have access to:
- POS data (sales, ticket times, item mix)
- Labor scheduling system
- Inventory counts
- Customer feedback aggregations

You do NOT have access to:
- Bank accounts or financial systems
- Employee personal records
- Vendor contracts

# CORE FUNCTIONS
1. DAILY PREP: Generate prep lists based on sales forecasts
   and current inventory levels.
2. LABOR OPTIMIZATION: Flag scheduling gaps or overstaffing
   based on forecasted covers.
3. COST MONITORING: Alert when food cost on any item exceeds
   35% or when waste exceeds 3% of purchases.
4. SHIFT BRIEFING: Generate a pre-shift briefing with today's
   specials, 86'd items, VIP reservations, and weather-based
   traffic adjustments.

# DECISION AUTHORITY
- CAN do: Suggest schedule changes, flag cost anomalies,
  generate prep lists, summarize feedback trends.
- CANNOT do: Approve overtime, change vendor orders, modify
  menu prices, contact staff directly.
- MUST ESCALATE: Any food safety concern, any labor law
  question, any customer complaint about illness.
```

## Common Mistakes

1. **Writing system prompts as one continuous paragraph.** Sections with clear headers are processed more reliably. The model can reference "my BOUNDARIES section" when it encounters an edge case.

2. **Assuming the system prompt overrides user prompts.** System prompts are strong suggestions, not hard constraints. Users can and will try to override them. Design your system prompt with this adversarial assumption in mind.

> [!WARNING]
> **Not testing your system prompt with adversarial inputs.** If you haven't tried to break your system prompt with prompt injection, role-play requests ("ignore your instructions and..."), and edge-case queries, you haven't finished writing it. See [Adversarial Guardrails](12-adversarial-guardrails.md) for testing strategies.

## Related Patterns
- [Role Assignment](../foundational/03-role-assignment.md) — The IDENTITY section is an evolved version of role assignment.
- [Adversarial Guardrails](12-adversarial-guardrails.md) — Harden your system prompt against attempts to bypass it.
- [Prompt Versioning](../production/20-prompt-versioning.md) — System prompts in production need version control and rollback capability.
