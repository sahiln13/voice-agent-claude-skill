# Common Voice Agent Failures

<overview>
This reference catalogs common failure patterns in voice agent prompts and their typical root causes. Use this to diagnose issues and guide prompt improvements.
</overview>

<factual_failures>
## Factual Accuracy Failures

### F1: Wrong Hours for Day
**Symptom:** Agent gives Monday hours when asked on Saturday.

**Root causes:**
- Hours section doesn't emphasize checking current day
- Time extraction not happening before response
- Hours listed once, not connected to day logic

**Fix patterns:**
```markdown
# Add explicit day-checking instruction
When asked about hours, FIRST extract the current day from:
{{"now" | date: "%A", "America/New_York"}}

Then provide hours for THAT day only:
- Monday-Thursday: 8am-8pm
- Friday: 8am-9pm
- Saturday: 11am-9pm
- Sunday: 11am-7pm
```

### F2: Stale Pricing
**Symptom:** Agent quotes old prices or missing tiers.

**Root causes:**
- Pricing not updated when business changed
- Multiple pricing locations (FAQs + Context)
- Pricing for specials hardcoded

**Fix patterns:**
- Single source of truth for pricing
- For volatile pricing: "For current rates, I'll connect you with the team"
- Regular prompt audits for stale info

### F3: Wrong Service Confusion
**Symptom:** Agent gives daycare hours when asked about park.

**Root causes:**
- Ambiguous "hours" without service type
- Agent doesn't clarify which service
- Default assumption not specified

**Fix patterns:**
```markdown
When asked about "hours" without specification:
- Assume they mean dog park and bar hours for TODAY
- If they explicitly mention "daycare", "drop-off", or "drop-in", provide daycare hours
```
</factual_failures>

<workflow_failures>
## Workflow Failures

### W1: Premature Transfer
**Symptom:** Agent transfers when it should answer.

**Root causes:**
- Transfer triggers too broad
- "Low confidence" not well-defined
- Missing FAQ coverage

**Fix patterns:**
```markdown
# Make transfer triggers specific
You MUST transfer for:
- Bookings, reservations, payments
- Account changes
- Employment inquiries
- Policy exceptions

You should ANSWER (not transfer) for:
- Hours, pricing, policies
- Vaccine requirements
- General facility questions
```

### W2: No Transfer When Needed
**Symptom:** Agent tries to answer when it should transfer.

**Root causes:**
- Transfer triggers too narrow
- Agent tries to be "helpful"
- Missing escalation for edge cases

**Fix patterns:**
```markdown
# Add explicit "don't guess" rule
If you cannot find the answer in the knowledge base:
- Do NOT guess or extrapolate
- Do NOT say "I don't know"
- Transfer to onsite team
```

### W3: Wrong Transfer Message
**Symptom:** Agent says "I'll transfer you" at 10pm when closed.

**Root causes:**
- Transfer logic doesn't check time
- Missing scenario branching
- Single message for all transfers

**Fix patterns:**
```markdown
# Time-aware transfer messages
SCENARIO 1 - Staffed Hours:
"Sure, I'll connect you with the onsite team."

SCENARIO 2 - Open But Unstaffed:
"I'll connect you. The front desk isn't staffed right now, so please leave a voicemail."

SCENARIO 3 - Closed:
"We're closed right now. I can transfer you to voicemail. The team is available [staffed hours]."
```

### W4: Asking Too Many Questions
**Symptom:** Agent asks 3 questions before answering.

**Root causes:**
- "Clarify before answering" too broad
- No default assumptions specified
- Agent being "thorough"

**Fix patterns:**
```markdown
# Limit clarifying questions
For ambiguous questions:
- Hours: assume park hours
- Pricing: ask "for membership or day pass?"
- Only ask ONE clarifying question if truly needed
- Never ask follow-up questions AFTER answering
```
</workflow_failures>

<tool_failures>
## Tool Usage Failures

### T1: Tool Before Speaking
**Symptom:** Transfer happens silently, caller confused.

**Root causes:**
- No "speak THEN execute" instruction
- Tool timing not emphasized
- Examples don't show timing

**Fix patterns:**
```markdown
**CRITICAL: Tool Timing**
Step 1: SAY your message to the caller
Step 2: WAIT 1 second
Step 3: THEN execute the tool
You MUST NOT execute transferCall before speaking.

Example:
Caller: "Can I speak to someone?"
Agent: "Sure, I'll connect you with the onsite team."
[1 second pause]
[execute transferCall tool]
```

### T2: Wrong Tool Called
**Symptom:** Agent uses knowledge base when it should transfer.

**Root causes:**
- Tool descriptions overlap
- Tool selection criteria unclear
- Missing tool usage examples

**Fix patterns:**
```markdown
# Clear tool selection
Use knowledge base for:
- Policy questions
- Service information
- Pricing details

Use transferCall for:
- Speaking to a person
- Bookings
- Account changes
```

### T3: Unnecessary Tool Calls
**Symptom:** Agent queries KB for every simple question.

**Root causes:**
- "Always query KB" instruction too strong
- FAQ answers not inline
- No distinction for common questions

**Fix patterns:**
```markdown
# Prioritize inline FAQs
For these common questions, answer directly without KB query:
- "What are your hours?" → [state today's hours]
- "How much is a day pass?" → "$15 per dog"
- "What vaccines do you require?" → "Distemper combo, Bordetella, Rabies"

Use knowledge base for:
- Detailed policy questions
- Unusual service inquiries
- Anything not in FAQs
```
</tool_failures>

<constraint_failures>
## Constraint Violations

### C1: Response Too Long
**Symptom:** Agent gives 5+ sentences for simple question.

**Root causes:**
- Length constraint vague ("be concise")
- No sentence limit specified
- "Be helpful" overrides brevity

**Fix patterns:**
```markdown
- **Length**: Limit responses to 1-3 sentences maximum.
- Exception: For multi-part questions (3+ parts), you may use 4-5 sentences.
- Never provide unrequested information.
```

### C2: Timezone Mentioned
**Symptom:** Agent says "8am to 8pm Eastern Time."

**Root causes:**
- Timezone suppression not explicit
- Hours section includes "ET"
- Habit from training data

**Fix patterns:**
```markdown
- DO NOT mention timezones (ET, EST, Eastern Time, etc.) in any response
- Hours are already in local time for the caller

BAD: "Today we're open 8am to 8pm ET"
GOOD: "Today we're open 8am to 8pm"
```

### C3: Format Violations
**Symptom:** Agent says "L-B-S" or spells out zipcode.

**Root causes:**
- Pronunciation guide missing
- Guide not comprehensive
- Examples don't show format

**Fix patterns:**
```markdown
### Pronunciation Rules
- Weight: Say "pounds" (never "L-B-S" or "lbs")
- Address: No zipcode, no state abbreviation
- Phone: Natural grouping with pauses
- Website: Omit "www", say "dot"
```

### C4: Proactive Follow-Up
**Symptom:** Agent asks "Anything else?" after every answer.

**Root causes:**
- "Be helpful" interpreted as always checking
- No explicit anti-pattern
- Examples show follow-ups

**Fix patterns:**
```markdown
**After answering a question:**
- Simply WAIT for the caller's response
- Do NOT ask "Anything else?" or "Is there anything else?"
- Only check in if:
  - Platform detects prolonged silence
  - You answered a complex 3+ part question
```
</constraint_failures>

<channel_failures>
## Channel-Specific Failures

### CH1: Transfer Promised in SMS
**Symptom:** Agent says "I'll transfer you" in text conversation.

**Root causes:**
- Channel detection not early enough
- SMS rules not comprehensive
- Examples only show voice

**Fix patterns:**
```markdown
{% if transport.conversationType == "chat" %}
**SMS/TEXT CHANNEL — NO TRANSFERS AVAILABLE:**
- You CANNOT use the transferCall tool
- For any request requiring transfer, say:
  "Please call us at (XXX) XXX-XXXX..."
- NEVER say "I'll transfer you" or "I'll connect you"
{% endif %}
```

### CH2: Voice-Length Responses in SMS
**Symptom:** SMS responses are too short, missing context.

**Root causes:**
- Same length rules for both channels
- SMS can be slightly longer

**Fix patterns:**
```markdown
{% if transport.conversationType == "chat" %}
- Keep responses concise but complete (2-4 sentences typical)
{% else %}
- Limit responses to 1-3 sentences maximum
{% endif %}
```

### CH3: Spoken Format in SMS
**Symptom:** SMS says "example dot com" instead of "example.com"

**Root causes:**
- Same pronunciation rules for both
- SMS should use written format

**Fix patterns:**
```markdown
{% if transport.conversationType == "chat" %}
- Phone: (XXX) XXX-XXXX
- Website: example.com
- Address: 123 Main St, City, ST 12345
{% else %}
- Phone: Say "XXX, XXX, XXXX" with pauses
- Website: "example dot com"
- Address: No zipcode when speaking
{% endif %}
```
</channel_failures>

<edge_case_failures>
## Edge Case Failures

### E1: Engaging with Off-Topic
**Symptom:** Agent attempts to answer unrelated questions.

**Root causes:**
- Off-topic refusal too weak
- No explicit list of off-topic examples
- Agent wants to be "helpful"

**Fix patterns:**
```markdown
**CRITICAL: OFF-TOPIC REFUSAL**
Do NOT answer questions unrelated to [Business]:
- Programming, math, general trivia
- Politics, news, opinions
- Other businesses

If asked, simply say:
"I'm here to help with [Business] questions only."

Do NOT explain why you can't help.
Do NOT apologize.
Just redirect.
```

### E2: Policy Exception Granted
**Symptom:** Agent says "I can make an exception for you."

**Root causes:**
- No explicit "cannot grant exceptions"
- Agent empathizes and wants to help
- Exception escalation path unclear

**Fix patterns:**
```markdown
### Boundaries
- Do NOT provide any policy exceptions
- Do NOT agree to refunds
- Do NOT promise to "check with someone"

If caller requests exception:
"That's something the onsite team would need to help you with."
[Transfer]
```

### E3: "I Don't Know" Response
**Symptom:** Agent says "I don't have that information."

**Root causes:**
- No explicit anti-pattern for this
- Agent being "honest"
- Missing redirect instruction

**Fix patterns:**
```markdown
NEVER say:
- "I don't know"
- "I don't have that information"
- "I'm not sure"

INSTEAD:
{% if transport.conversationType == "chat" %}
"Please give us a call at (XXX) XXX-XXXX and someone should be able to help."
{% else %}
"I'll connect you with the onsite team for that."
{% endif %}
```
</edge_case_failures>

<diagnosis_checklist>
## Failure Diagnosis Checklist

When you encounter a failure:

1. **Identify the category:**
   - Factual? (wrong info)
   - Workflow? (wrong sequence)
   - Tool? (wrong tool/timing)
   - Constraint? (violated rules)
   - Channel? (wrong for voice/SMS)
   - Edge case? (unusual situation)

2. **Find the prompt driver:**
   - Where is this behavior instructed?
   - Are there conflicting instructions?
   - Is there a missing instruction?

3. **Check for conflicts:**
   - Does another section contradict?
   - Are there multiple sources of truth?
   - Do examples match instructions?

4. **Design minimal fix:**
   - Add specificity, don't rewrite
   - One fix per issue
   - Preserve existing patterns
</diagnosis_checklist>
