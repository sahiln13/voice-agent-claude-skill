# Voice Agent Constraints

<overview>
Voice agents have unique constraints that don't apply to text-based AI. This reference covers the specific requirements for voice interactions in Vapi.
</overview>

<core_constraints>
## Core Voice Constraints

### Response Length
```markdown
**Voice maximum:** 1-3 sentences per turn
**Absolute maximum:** 4-5 sentences for complex multi-part questions

Why: Callers can't scroll back. Long responses cause confusion and frustration.
```

### Speaking Pace
```markdown
- Speak slowly for: addresses, phone numbers, websites, dates
- Pause 1-2 seconds after asking questions
- Allow caller to interrupt (barge-in support)
```

### Pronunciation
```markdown
Numbers: Say digits individually for addresses
         "one-eight-five-nine-five" not "eighteen thousand five hundred..."

Phone: Natural grouping with pauses
       "three-one-seven... seven-eight-five... seven-eight-seven-two"

Websites: Spell out dots, omit www
          "example dot com"

Email: Use "at" symbol
       "info at example dot com"

Acronyms: Say full words or intuitive pronunciation
          "distemper combo" not "D-A-P-P"
          "pounds" not "L-B-S"
```

### Timezone Handling
```markdown
NEVER mention timezone abbreviations to callers
- Bad: "We're open 8am to 8pm ET"
- Good: "We're open 8am to 8pm"

Model processes in specific timezone, caller hears local time.
```
</core_constraints>

<tool_timing>
## Tool Timing

**Critical rule: Speak THEN Execute**

Voice has a specific timing requirement for tools:

```markdown
WRONG ORDER:
[execute transferCall]
"I'll connect you now."  ← Caller never hears this

RIGHT ORDER:
"I'll connect you with the onsite team."  ← Caller hears this
[1 second pause]
[execute transferCall]
```

**Tool-specific timing:**

| Tool | Speak First | Then Execute |
|------|------------|--------------|
| transferCall | "I'll connect you..." | Transfer happens |
| endCall | "Thanks for calling!" | Call ends |
| knowledge base | (silent) | Query executes |
| n8n tools | (silent) | Tool executes |

**Prompt pattern:**
```markdown
**Transfer Protocol:**
Step 1: SAY your message to the caller
Step 2: Wait 1 second
Step 3: THEN execute the transferCall tool
You MUST NOT execute transferCall before speaking.
```
</tool_timing>

<conversation_flow>
## Conversation Flow

### Opening
```markdown
Voice: Short, professional greeting
"Hello, this is [Name] from [Business]. How can I help?"

SMS: Slightly warmer
"Hi! This is [Name] from [Business]. How can I help?"
```

### Turn-Taking
```markdown
After answering:
- WAIT for caller's response
- Do NOT ask "Anything else?" proactively
- Only prompt for more if:
  - Silence detected (platform prompts you)
  - Complex multi-part question answered (3+ parts)

Pattern to enforce:
"After answering a question, simply WAIT for the caller's next response."
```

### Closing
```markdown
Caller says goodbye → "Thanks for calling!" → [endCall]
Caller says "that's all" → "Thanks for calling!" → [endCall]
Natural end → "Thanks for calling!" → [endCall]

Don't say:
- "Have a great day!" (too casual for some brands)
- "Is there anything else I can help with?" (unless prompted)
```

### Interruptions
```markdown
"Caller interrupts mid-response → Stop immediately and listen"

Voice agents must support barge-in. When caller speaks:
1. Stop current response immediately
2. Process their new input
3. Respond to what they said
```
</conversation_flow>

<channel_differences>
## Voice vs SMS Differences

| Aspect | Voice | SMS |
|--------|-------|-----|
| Length | 1-3 sentences | 2-4 sentences OK |
| Formatting | Plain speech | Light markdown OK |
| Phone numbers | Speak with pauses | (XXX) XXX-XXXX |
| Addresses | Spell out, no zip | Full with zip OK |
| Transfers | transferCall tool | "Please call us..." |
| Tone | Professional, brief | Slightly more casual |
| Follow-up | Wait for caller | Can end with question |

### Channel-Specific Instructions
```markdown
{% if transport.conversationType == "chat" %}
**SMS RULES:**
- You CANNOT transfer (transferCall doesn't exist)
- Direct complex requests to phone
- Format phone: (XXX) XXX-XXXX
- OK to use line breaks for readability
{% else %}
**VOICE RULES:**
- Limit to 1-3 sentences
- Speak numbers slowly
- Say address without zipcode
- Pause after questions
{% endif %}
```
</channel_differences>

<edge_cases>
## Edge Case Handling

### Audio Quality Issues
```markdown
Garbled audio:
1st attempt: "Sorry, I didn't catch that—could you repeat?"
2nd attempt: "I'm still having trouble hearing you, so I'll connect you with the onsite team."
→ [Transfer]
```

### Silence/No Response
```markdown
Platform detects silence → You receive prompt to check in

"Hello, are you still there?"
or
"Is there anything else I can help with?"

Extended silence → Transfer to voicemail/team
```

### Caller Frustration
```markdown
Angry caller detection → Immediate empathy + escalation

Voice:
"I'm sorry for the issue. Let me connect you with the onsite team."
[Transfer]

SMS:
"I'm sorry for the issue. Please call us at (XXX) XXX-XXXX and someone at the front desk can help."
```

### Off-Topic Requests
```markdown
Unrelated questions (programming, math, politics, etc.):

"I'm here to help with [Business] questions only."

Do NOT engage with the off-topic content.
Do NOT explain why you can't help.
Just redirect.
```
</edge_cases>

<pronunciation_guide_template>
## Pronunciation Guide Template

Add these rules for any voice prompt:

```markdown
### Pronunciation & Format Rules

- **Address**: Say "[spoken address format]" (do NOT include zipcode)
- **Website**: "[spoken website]" (omit "www")
- **Email**: "[spoken email format]"
- **Phone**: Say numbers slowly with natural grouping
- **Vaccines**: Say "[full name]" instead of acronyms
- **Weight**: Say "pounds" (never "L-B-S")
- **Currency**: Say "dollars" (never "D-O-L-L-A-R-S")
- **Timezone**: NEVER mention timezone abbreviations

Examples:
- 18595 Main Street → "one-eight-five-nine-five Main Street"
- www.example.com → "example dot com"
- info@example.com → "info at example dot com"
- (317) 785-7872 → "three-one-seven, seven-eight-five, seven-eight-seven-two"
- DHPP vaccine → "distemper combo"
- 25 lbs → "twenty-five pounds"
```
</pronunciation_guide_template>

<testing_checklist>
## Voice Constraint Checklist

Before deploying, verify:

**Length:**
- [ ] Response length limit explicitly stated (1-3 sentences)
- [ ] Multi-part question exception defined
- [ ] "Anything else" usage constrained

**Pronunciation:**
- [ ] Address format specified
- [ ] Phone number format specified
- [ ] Website/email format specified
- [ ] Acronym handling defined
- [ ] Timezone suppression stated

**Tool Timing:**
- [ ] "Speak THEN execute" rule for transfers
- [ ] "Speak THEN execute" rule for call ending
- [ ] Wait/pause instructions included

**Turn-Taking:**
- [ ] Opening greeting defined
- [ ] Wait-for-response rule stated
- [ ] Interruption handling included
- [ ] Closing phrases defined

**Edge Cases:**
- [ ] Audio quality handling
- [ ] Silence handling
- [ ] Angry caller handling
- [ ] Off-topic handling
</testing_checklist>
