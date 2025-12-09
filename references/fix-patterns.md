# Fix Patterns for Voice Agent Prompts

<overview>
This reference provides reusable patterns for fixing common voice agent prompt issues. Each pattern shows the anti-pattern, the fix, and when to use it.
</overview>

<adding_specificity>
## Pattern 1: Adding Specificity

**When to use:** Agent behavior is inconsistent or unpredictable because instructions are vague.

### Anti-Pattern: Vague Instruction
```markdown
# BAD
Keep responses brief.
```

### Fix: Explicit Constraint
```markdown
# GOOD
- **Length**: Limit responses to 1-3 sentences maximum.
- Exception: For multi-part questions (3+ parts), you may use 4-5 sentences.
- Never provide information the caller didn't ask for.
```

### More Examples

**Vague → Specific (hours):**
```markdown
# BAD
Tell them today's hours.

# GOOD
When asked about "hours" without specification:
1. Extract current day from {{"now" | date: "%A", "America/New_York"}}
2. Provide hours for THAT day only:
   - Monday-Thursday: 8am-8pm
   - Friday: 8am-9pm
   - Saturday: 11am-9pm
   - Sunday: 11am-7pm
```

**Vague → Specific (transfer):**
```markdown
# BAD
Transfer when you can't help.

# GOOD
You MUST transfer for:
- Bookings, reservations, payments
- Account changes or billing
- Employment inquiries
- Policy exceptions or complaints
- Questions not in the knowledge base
```
</adding_specificity>

<adding_conditionals>
## Pattern 2: Adding Conditionals

**When to use:** Agent needs to behave differently based on time, channel, or context.

### Anti-Pattern: One-Size-Fits-All
```markdown
# BAD
I'll connect you with the onsite team.
```

### Fix: Scenario Branching
```markdown
# GOOD
**SCENARIO 1 - Staffed Hours:**
When within Business Hours AND within Staffed Hours:
"Sure, I'll connect you with the onsite team."

**SCENARIO 2 - Open But Unstaffed:**
When within Business Hours BUT NOT within Staffed Hours:
"I'll connect you. The front desk isn't staffed right now, so if no one picks up, please leave a voicemail."

**SCENARIO 3 - Closed:**
When outside Business Hours:
"We're closed right now. I can transfer you to voicemail. The team is available Monday-Friday 5-8pm, weekends 1-7pm."
```

### Time-Based Conditional Template
```markdown
**Know the context:**
The date and time is {{"now" | date: "%A, %B %d, %Y, %I:%M %p", "America/New_York"}}.

Extract from current time:
- Current day: {{"now" | date: "%A", "America/New_York"}}
- Current hour: {{"now" | date: "%H", "America/New_York"}}

Determine:
- Is it within [Service 1] hours?
- Is it within [Service 2] hours?
- Is the front desk staffed?
```

### Channel-Based Conditional Template
```markdown
{% if transport.conversationType == "chat" %}
**SMS-SPECIFIC RULES:**
[SMS-only instructions]
{% else %}
**VOICE-SPECIFIC RULES:**
[Voice-only instructions]
{% endif %}
```
</adding_conditionals>

<adding_examples>
## Pattern 3: Adding Examples

**When to use:** Agent needs concrete guidance for nuanced behaviors. GPT-5.1 with `none` reasoning mode especially benefits from examples.

### Anti-Pattern: Instructions Without Examples
```markdown
# BAD
Transfer the caller appropriately based on the situation.
```

### Fix: Concrete Examples
```markdown
# GOOD
**Transfer Examples:**

1. "Can I speak to someone?" (Mon 6pm - staffed)
   → "Sure, I'll connect you with the onsite team."
   [execute transferCall]

2. "Can I speak to someone?" (Mon 2pm - not staffed)
   → "I'll connect you. The front desk isn't staffed right now, so if no one picks up, please leave a voicemail."
   [execute transferCall]

3. "I want to book a party" (closed)
   → "I'll need to connect you for that. We're closed right now, but I can transfer you to leave a voicemail."
   [execute transferCall]
```

### Example Template
```markdown
## Examples

**Scenario Category 1:**
1. Input: "[caller says X]"
   Context: [time, channel, state]
   Output: "[agent response]"
   Action: [tool if applicable]

2. Input: "[caller says Y]"
   ...
```
</adding_examples>

<resolving_conflicts>
## Pattern 4: Resolving Conflicts

**When to use:** Agent behavior is inconsistent because two instructions contradict.

### Anti-Pattern: Conflicting Rules
```markdown
# BAD - Conflict between brevity and completeness
Be concise and brief.
...
Provide complete and thorough answers.
...
Err on the side of helpfulness.
```

### Fix: Priority Rules
```markdown
# GOOD - Clear priority
**Response Guidelines (in order of priority):**
1. Accuracy: Information must be correct (never guess)
2. Brevity: Limit to 1-3 sentences (never exceed without cause)
3. Completeness: Within brevity limits, answer fully
4. Helpfulness: If equally accurate and brief, prefer the more helpful option

For multi-part questions (3+ parts), you may extend to 4-5 sentences.
```

### Conflict Resolution Template
```markdown
# When [Rule A] and [Rule B] conflict:
- Prioritize [Rule A] when [condition]
- Prioritize [Rule B] when [other condition]
- Default: [which rule wins]
```

### Common Conflict Resolutions
```markdown
# Brevity vs Completeness
"1-3 sentences unless the question has 3+ distinct parts"

# Autonomy vs Clarification
"Answer directly for [common topics]. Clarify only for [specific ambiguous cases]."

# Helpfulness vs Boundaries
"Be helpful within defined boundaries. For requests outside boundaries, redirect (not refuse)."
```
</resolving_conflicts>

<strengthening_constraints>
## Pattern 5: Strengthening Constraints

**When to use:** Agent occasionally violates rules because constraints aren't emphatic enough.

### Anti-Pattern: Weak Constraint
```markdown
# BAD
Try to avoid mentioning the timezone.
```

### Fix: Strong Constraint
```markdown
# GOOD
- DO NOT mention timezone abbreviations (ET, EST, Eastern Time) in any response.
- Hours are in local time for the caller. They don't need to know the timezone.

BAD: "Today we're open 8am to 8pm ET."
GOOD: "Today we're open 8am to 8pm."
```

### Constraint Strengthening Template
```markdown
**CRITICAL: [Constraint Name]**

- DO NOT [prohibited action]
- NEVER [prohibited pattern]
- If [trigger], then [required action] - no exceptions

Example of what NOT to do:
"[bad example]"

Example of correct behavior:
"[good example]"
```

### Common Constraints to Strengthen
```markdown
# Off-topic refusal
**CRITICAL: OFF-TOPIC REFUSAL**
Do NOT answer questions unrelated to [Business]:
- [List of off-topic categories]
If asked, say: "I'm here to help with [Business] questions only."

# Policy exceptions
**CRITICAL: NO EXCEPTIONS**
- Do NOT provide policy exceptions for any reason
- Do NOT promise to "check" or "see what we can do"
- Only the onsite team can grant exceptions

# Tool timing
**CRITICAL: SPEAK BEFORE TOOLS**
- Say your message FIRST
- THEN execute the tool
- Never execute transferCall silently
```
</strengthening_constraints>

<adding_anti_patterns>
## Pattern 6: Adding Explicit Anti-Patterns

**When to use:** Agent does something wrong that isn't covered by positive instructions.

### Anti-Pattern: Missing Negative Guidance
```markdown
# BAD - No guidance on what NOT to do
After answering, check if they need anything else.
```

### Fix: Explicit Anti-Pattern
```markdown
# GOOD - Clear on what to avoid
After answering a question:
- Simply WAIT for the caller's response
- Do NOT proactively ask "Anything else?" or "Is there anything else I can help with?"

Only ask "Anything else?" if:
- You answered a complex multi-part question (3+ distinct parts)
- The platform detects prolonged silence and prompts you
```

### Anti-Pattern Template
```markdown
**Do NOT:**
- [Specific prohibited behavior 1]
- [Specific prohibited behavior 2]

**Instead:**
- [Correct behavior 1]
- [Correct behavior 2]
```

### Common Anti-Patterns to Add
```markdown
# Response length
Do NOT:
- Provide information the caller didn't ask for
- Add extra context "just in case"
- Repeat information already stated

# Acknowledgments
Do NOT:
- Say "Great question!"
- Say "Absolutely!"
- Say "Of course!"
Just answer directly.

# Uncertainty
Do NOT say:
- "I don't know"
- "I don't have that information"
- "I'm not sure"

Instead: Transfer (voice) or direct to call (SMS)
```
</adding_anti_patterns>

<consolidating_scattered_info>
## Pattern 7: Consolidating Scattered Information

**When to use:** Same information appears in multiple places, causing inconsistency.

### Anti-Pattern: Scattered Information
```markdown
# BAD - Hours in 3 places
[In Context]
Hours: Mon-Fri 9-5

[In FAQs]
"What are your hours?" → "We're open 9 to 5 weekdays"

[In Examples]
Q: "When do you close?"
A: "We close at 5pm"
```

### Fix: Single Source of Truth
```markdown
# GOOD - Hours defined once, referenced elsewhere

## Context
### Hours
**Dog Park and Bar Hours:**
- Monday-Thursday: 8am-8pm
- Friday: 8am-9pm
- Saturday: 11am-9pm
- Sunday: 11am-7pm

## FAQs
- **Hours**: Refer to Hours section. State today's hours only.

## Examples
1. "What are your hours?" → Check Hours section, provide today's hours only.
```

### Consolidation Template
```markdown
# Define once (in Context)
### [Topic]
[Authoritative information]

# Reference elsewhere
- **[Topic]**: See [Topic] section above.
```
</consolidating_scattered_info>

<adding_tool_timing>
## Pattern 8: Tool Timing Instructions

**When to use:** Agent executes tools at wrong time or without proper announcement.

### Anti-Pattern: Implicit Tool Timing
```markdown
# BAD
Transfer the caller to the onsite team.
```

### Fix: Explicit Timing Steps
```markdown
# GOOD
**Transfer Protocol:**

Step 1: SAY your transfer message to the caller
Step 2: Wait 1 second for the message to complete
Step 3: THEN execute the transferCall tool

You MUST NOT execute transferCall before saying your message.

**Example:**
Caller: "Can I speak to someone?"
Agent: "Sure, I'll connect you with the onsite team."
[pause 1 second]
[execute transferCall tool]
```

### Tool Timing Template
```markdown
**When using [tool_name]:**

1. FIRST: [What to say/do before tool]
2. THEN: [Wait if needed]
3. FINALLY: [Execute tool]

Never: [What not to do]

Example:
Input: [trigger]
Agent action: [correct sequence]
```
</adding_tool_timing>

<quick_fix_reference>
## Quick Fix Reference

| Issue | Pattern | Key Fix |
|-------|---------|---------|
| Inconsistent behavior | Add Specificity | Replace vague with explicit limits |
| Wrong for time/channel | Add Conditionals | Scenario branching with Liquid |
| Nuanced behaviors wrong | Add Examples | Concrete input → output pairs |
| Unpredictable behavior | Resolve Conflicts | Add priority rules |
| Occasional violations | Strengthen Constraints | CRITICAL + DO NOT + examples |
| Doing wrong things | Add Anti-Patterns | Explicit "Do NOT" section |
| Conflicting answers | Consolidate | Single source of truth |
| Tool timing wrong | Add Tool Timing | Step-by-step execution sequence |
</quick_fix_reference>
