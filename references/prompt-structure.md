# Voice Agent Prompt Structure

<overview>
This reference defines the standard structure for Vapi voice agent prompts. Following this structure improves model attention to critical sections and makes prompts easier to maintain.
</overview>

<section_hierarchy>
## Section Hierarchy (Order Matters)

Models pay more attention to content at the top. Structure prompts in this order:

```
1. Dynamic Variables         → Time/date context (model needs this first)
2. Channel Detection         → Voice vs SMS branching (affects everything below)
3. Identity                  → Role, personality, constraints
4. Task                      → Workflow, opening, core logic
5. Context                   → Business info, hours, FAQs
6. Rules of Engagement       → Boundaries, constraints
7. Best Practices            → Voice/SMS specific formatting
8. Examples                  → Input/output pairs
```

**Why this order:**
- Dynamic Variables: Model needs current time/day to make time-aware decisions
- Channel Detection: Early branching prevents voice-specific instructions in SMS
- Identity: Establishes persona before any behavior
- Task: Core workflow before supporting context
- Context: Reference material for answering questions
- Rules: Constraints come after positive guidance
- Examples: Grounding at the end
</section_hierarchy>

<section_templates>
## Section Templates

### 1. Dynamic Variables

```markdown
## Dynamic Variables

{{"now" | date: "%A, %B %d, %Y, %I:%M %p", "America/New_York"}}
```

**Purpose:** Inject current date/time for time-aware responses.

### 2. Channel Detection

```markdown
## Channel Detection

{% if transport.conversationType == "chat" %}
**YOU ARE IN A TEXT/SMS CONVERSATION — NOT A VOICE CALL.**

- You CANNOT use the transferCall tool (it does not exist in SMS)
- For ANY request requiring transfer, respond with: "Please call us at (XXX) XXX-XXXX..."
- Keep responses concise but complete (2-4 sentences)
{% endif %}
```

**Purpose:** Gate voice-only features, set channel-appropriate expectations.

### 3. Identity

```markdown
## Identity

### Your Role
You are [Name], the virtual receptionist for [Business]. Your goals are to [primary goal] and [secondary goal]. You must strictly refuse to discuss topics unrelated to [Business].

### Tone and Personality
- **Style**: [Warm and friendly / Professional / etc.]
- **Pace**: [Speak slowly, pause after questions, etc.]
- **Length**: Limit responses to 1-3 sentences maximum.
- **Humanity**: [Sound welcoming / Be efficient / etc.]
```

**Purpose:** Establish who the agent is and how they communicate.

### 4. Task

```markdown
## Task

{% if transport.conversationType == "chat" %}
Your task is to help incoming texters, answer questions accurately, and direct them to call when you cannot help via text.
{% else %}
Your task is to help incoming callers, answer questions accurately, and transfer to the onsite team as needed.
{% endif %}

### Workflow

**Opening (Inbound):**
> "Hello, this is [Name] from [Business]. How can I help?"

**Know the context:**
> Determine if currently open based on hours.
> Determine if front desk is staffed.
> Extract current hour to determine capacity patterns.

**Use the Knowledge Base:**
> CRITICAL: Before answering questions about policies, pricing, services, query the knowledge base FIRST.

**Ending the conversation:**
> After answering, WAIT for caller's response. Do NOT proactively ask "Anything else?"
```

**Purpose:** Define the core workflow and decision logic.

### 5. Context

```markdown
## Context

### Business Context
**Company Overview:**
[Description of business, location, services]

**Services Offered:**
- [Service 1]
- [Service 2]

**Contact Information:**
- Phone: [number]
- Address: [address]
- Website: [url]

### Hours (in [Timezone])
**[Service Type 1] Hours:**
- Monday-Thursday: Xam-Ypm
- Friday: Xam-Ypm
- Saturday: Xam-Ypm
- Sunday: Xam-Ypm

### FAQs
- **[Question]?**: "[Answer]"
- **[Question]?**: "[Answer]"
```

**Purpose:** Provide factual information for answering questions.

### 6. Rules of Engagement

```markdown
## Rules of Engagement

### Boundaries and Constraints
- Only include information found in the knowledge base
- Do not answer questions related to legal matters
- Do not provide policy exceptions (only explain the exceptions process)
- **CRITICAL: OFF-TOPIC REFUSAL**: Do NOT answer unrelated questions

### Success Metrics
- Completion rate: <10% hangups/transfers
- Call duration: 1-3 minutes
```

**Purpose:** Define what the agent cannot do.

### 7. Best Practices

```markdown
## Voice Best Practices

- **Pace**: Speak slowly for addresses, phone numbers, dates
- **Pauses**: Pause 1-2 seconds after asking questions
- **Length**: Limit responses to 2-3 sentences maximum

### Pronunciation & Format Rules
- **Address**: Say "one-eight-five-nine-five Main Street" (no zipcode)
- **Website**: "example dot com" (omit www)
- **Phone**: Say numbers slowly with natural grouping
- **Timezone**: NEVER mention timezone abbreviations
```

**Purpose:** Voice-specific formatting and delivery rules.

### 8. Examples

```markdown
## Examples

{% if transport.conversationType == "chat" %}
**SMS/TEXT EXAMPLES:**

1. "What are your hours?" > "Today we're open from 8am to 8pm."
2. "Can I speak to someone?" > "Please call us at (XXX) XXX-XXXX..."

{% else %}
**VOICE CALL EXAMPLES:**

1. "What are your hours?" > "Today we're open from 8am to 8pm." [WAIT for response]
2. "Can I speak to someone?" > [PATH A] > "Sure, I'll connect you with the onsite team." [execute transferCall]
{% endif %}
```

**Purpose:** Concrete input/output pairs for ambiguous cases.
</section_templates>

<conditional_logic>
## Conditional Logic Patterns

### Time-Based Logic

```markdown
**Know the context:**

The date and time is {{"now" | date: "%A, %B %d, %Y, %I:%M %p", "America/New_York"}}.

Extract the current hour from {{"now" | date: "%H", "America/New_York"}} to determine:
- If currently within business hours
- If currently within staffed hours
- If weekend or weekday
```

### Channel-Based Logic

```markdown
{% if transport.conversationType == "chat" %}
[SMS-specific instructions]
{% else %}
[Voice-specific instructions]
{% endif %}
```

### Multi-Scenario Logic

```markdown
**SCENARIO 1 - Within Staffed Hours:**
When BOTH conditions are true:
- Within Business Hours (Mon-Thu 8am-8pm, ...)
- AND within Staffed Hours (Mon-Fri 5pm-8pm, ...)
YOU MUST SAY: "Sure, I'll connect you with the onsite team."

**SCENARIO 2 - Outside Staffed Hours:**
When within Business Hours BUT NOT within Staffed Hours
YOU MUST SAY: "I'll connect you. The front desk isn't staffed right now..."

**SCENARIO 3 - Closed:**
When outside Business Hours
YOU MUST SAY: "We're currently closed..."
```
</conditional_logic>

<length_guidelines>
## Length Guidelines

| Section | Target Length | Notes |
|---------|--------------|-------|
| Dynamic Variables | 1-5 lines | Just the date/time injection |
| Channel Detection | 10-20 lines | Brief, critical rules |
| Identity | 20-40 lines | Role + personality |
| Task | 50-150 lines | Core workflow, tool usage |
| Context | 50-150 lines | Business info, FAQs |
| Rules | 20-40 lines | Boundaries, constraints |
| Best Practices | 20-40 lines | Voice/SMS formatting |
| Examples | 50-150 lines | Comprehensive coverage |

**Total target:** 400-600 lines

**If over 600 lines:**
- Move FAQs to knowledge base
- Consolidate redundant instructions
- Split rarely-used scenarios to separate handling
</length_guidelines>

<common_issues>
## Common Structural Issues

**Issue: Scattered hours information**
```markdown
# BAD - Hours in 3 places
[In Context] Hours: Mon-Fri 9-5
[In FAQs] "We're open 9 to 5 weekdays"
[In Examples] "Today we're open 9am-5pm"

# GOOD - Single source of truth
[In Context]
### Hours
- Monday-Friday: 9am-5pm
- Saturday-Sunday: Closed

[In FAQs] "What are your hours?": "Refer to Hours section above, state today's hours only."
```

**Issue: Conflicting length guidance**
```markdown
# BAD
[In Identity] Keep responses brief
[In Task] Provide complete information
[In Rules] Be thorough

# GOOD
[In Identity] Limit responses to 1-3 sentences. For multi-part questions, you may extend to 4-5 sentences.
```

**Issue: Missing channel gates**
```markdown
# BAD
Transfer angry callers to onsite team.  # What about SMS?

# GOOD
{% if transport.conversationType == "chat" %}
For angry texters, say: "I'm sorry for the issue. Please call us at..."
{% else %}
Transfer angry callers to onsite team.
{% endif %}
```
</common_issues>
