# GPT-5.1 Prompting Patterns for Voice Agents

<overview>
GPT-5.1 (used as "5.1 Instant" in Vapi) has specific prompting requirements that differ from previous models. This reference covers patterns that maximize performance in voice agent contexts.
</overview>

<key_differences>
## GPT-5.1 vs Previous Models

**Better calibration:** Consumes fewer tokens on easy inputs, more on hard ones.

**Higher steerability:** Responds well to explicit personality, tone, and format guidance.

**None reasoning mode:** When using `none` reasoning (common in voice for low latency), treat it more like GPT-4.1 - use few-shot prompting and detailed tool descriptions.

**Persistence needed:** Can err toward premature completion. Voice agents need explicit instructions to persist until the task is fully handled.

**Instruction following:** Excellent at following instructions - if behavior is wrong, check for conflicting instructions first.
</key_differences>

<voice_specific_patterns>
## Patterns for Voice Agents

### Conciseness Control

GPT-5.1 can be verbose. For voice, enforce strict limits:

```markdown
<output_verbosity>
- Respond with at most 2-3 sentences per turn
- Lead with the answer, context only if needed
- Never repeat information already stated
- No preamble ("Great question!", "Absolutely!")
</output_verbosity>
```

### Personality Shaping

Voice agents need clear persona definition. GPT-5.1 responds well to:

```markdown
<agent_personality>
You value clarity, momentum, and respect measured by usefulness rather than pleasantries.

- Adaptive politeness:
  - When caller is warm, offer a single, succinct acknowledgment then shift to productive action
  - When stakes are high (urgent issues), drop acknowledgments and move straight to solving

- Core inclination:
  - Speak with grounded directness
  - Politeness shows through precision and responsiveness, not verbal fluff

- Conversational rhythm:
  - Never repeat acknowledgments
  - Match the caller's energy and tempo
</agent_personality>
```

### Tool Timing for Voice

Voice requires "speak first, then execute" pattern:

```markdown
<tool_execution_timing>
CRITICAL: For transferCall, endCall, or any action tool:
1. Say your message to the caller FIRST
2. THEN execute the tool
3. Never execute tools silently or before announcing

Example flow:
- SAY: "I'll connect you with the onsite team."
- THEN: [execute transferCall tool]
</tool_execution_timing>
```

### Persistence in Voice Context

Prevent premature call endings:

```markdown
<conversation_persistence>
- Do NOT end the call after a single question/answer
- Wait for caller to indicate they're done
- If caller seems satisfied, wait 2-3 seconds for additional questions
- Only use endCall tool when:
  - Caller explicitly says goodbye/thanks
  - Natural conversation end reached
  - Transfer completed
</conversation_persistence>
```
</voice_specific_patterns>

<tool_description_patterns>
## Tool Description Best Practices

### Structure

Put functionality in tool definition, usage rules in prompt:

**Tool definition (JSON):**
```json
{
  "name": "transferCall",
  "description": "Transfer the caller to the onsite team. Use when caller needs to speak to staff, book, or handle account issues."
}
```

**Prompt guidance:**
```markdown
<transfer_tool_rules>
- MUST transfer for: bookings, billing, account changes, employment inquiries
- Before calling transferCall, you MUST:
  1. Say "I'll connect you with the onsite team"
  2. Add context message if outside staffed hours
- After transfer initiated, wait for connection
</transfer_tool_rules>
```

### Few-Shot Examples for Tools

GPT-5.1 with `none` reasoning benefits from examples:

```markdown
<transfer_examples>
Example 1 (during staffed hours):
Caller: "Can I speak to someone?"
Agent: "Sure, I'll connect you with the onsite team."
[execute transferCall]

Example 2 (outside staffed hours):
Caller: "I need to book a party."
Agent: "I'll need to connect you with the onsite team for that. The front desk isn't staffed right now, so if no one picks up, please leave a voicemail."
[execute transferCall]
</transfer_examples>
```
</tool_description_patterns>

<conflict_resolution>
## Resolving Conflicting Instructions

GPT-5.1 follows instructions precisely - conflicts cause unpredictable behavior.

**Common voice prompt conflicts:**

| Conflict | Example | Resolution |
|----------|---------|------------|
| Brevity vs completeness | "Be concise" vs "Answer fully" | Add conditional: "2-3 sentences unless multi-part question" |
| Autonomy vs clarification | "Don't ask clarifying questions" vs "Ask before answering ambiguous" | Add priority: "For hours questions, assume park hours. Only clarify for pricing." |
| Tone mixing | "Professional" vs "Use emojis for weddings" | Separate by channel: Voice = no emojis. SMS = light emojis OK. |

**Conflict detection checklist:**
- Search for "always" statements - do they conflict?
- Search for "never" statements - are there exceptions elsewhere?
- Check if tool usage rules conflict with autonomy guidance
- Verify channel-specific rules don't contradict general rules
</conflict_resolution>

<metaprompting>
## Using Metaprompting for Optimization

GPT-5.1 can analyze its own prompt failures. Use this pattern:

**Step 1: Diagnose failures**
```markdown
You are analyzing a voice agent system prompt for issues.

System prompt:
<system_prompt>
[PASTE PROMPT]
</system_prompt>

Failure examples:
<failures>
[PASTE TRANSCRIPT EXCERPTS]
</failures>

Tasks:
1. Identify distinct failure modes (e.g., wrong_tool_timing, excessive_verbosity, missing_channel_check)
2. Quote the specific lines causing each failure
3. Explain how those lines are steering toward the bad behavior

Output format:
failure_modes:
- name: ...
  prompt_drivers:
    - line: "..."
    - why_it_matters: ...
```

**Step 2: Generate surgical fixes**
```markdown
Based on this failure analysis, propose surgical edits:
[PASTE ANALYSIS]

Constraints:
- Do not redesign from scratch
- Prefer small, explicit edits
- Make tradeoffs explicit
- Keep structure similar

Output:
1. patch_notes: concise list of changes
2. revised_sections: only the sections that changed
```
</metaprompting>

<prompt_structure_for_5_1>
## Recommended Prompt Structure

GPT-5.1 responds well to clearly labeled sections:

```markdown
# Role & Objective
[WHO the agent is, WHAT their goal is]

# Personality & Tone
[HOW they communicate]
- Specific style rules
- Adaptive behaviors

# Instructions
[WHAT to do step by step]
- Opening workflow
- Core task logic
- Ending workflow

# Tools
[WHEN and HOW to use tools]
- Tool triggers
- Tool timing rules
- Examples

# Context
[WHAT they need to know]
- Business info
- Hours, pricing, policies
- FAQs

# Safety
[WHAT NOT to do]
- Boundaries
- Edge case handling
- Escalation paths

# Examples
[Input → Output pairs]
- Common scenarios
- Edge cases
- Tool usage examples
```

**Why this order works:**
- Role/Personality at top = strong identity anchoring
- Instructions before context = behavior prioritized over facts
- Tools clearly separated = less confusion about when to use
- Examples at end = grounding for ambiguous cases
</prompt_structure_for_5_1>

<none_reasoning_mode>
## None Reasoning Mode Tips

When Vapi uses `none` reasoning (for low latency):

**What changes:**
- Model doesn't internally reason before responding
- More like GPT-4.1 behavior
- Needs explicit thinking instructions

**Add explicit planning:**
```markdown
<before_tool_calls>
Before any tool call:
1. Consider what information you have
2. Identify what's missing
3. Determine if you can answer from knowledge base
4. Only call tool if genuinely needed
</before_tool_calls>
```

**Add verification:**
```markdown
<verification>
Before providing hours, pricing, or policies:
- Confirm which day/time applies
- Confirm which service (park vs daycare)
- Verify answer matches what caller asked
</verification>
```

**Use few-shot prompting:**
More examples = better performance in `none` mode.
Aim for 15-25 examples covering all common scenarios.
</none_reasoning_mode>

<anti_patterns>
## GPT-5.1 Anti-Patterns to Avoid

**Vague verbosity guidance:**
```markdown
# BAD
Keep responses brief.

# GOOD
Limit responses to 1-3 sentences maximum. Never exceed 3 sentences unless answering a multi-part question.
```

**Conflicting autonomy rules:**
```markdown
# BAD
Don't ask clarifying questions.
...
Always clarify before answering ambiguous questions.

# GOOD
For hours questions, assume park hours unless they specify daycare.
For pricing questions, clarify which service if ambiguous.
```

**Missing tool timing:**
```markdown
# BAD
Transfer to onsite team when needed.

# GOOD
When transferring:
1. SAY your message to the caller
2. WAIT 1 second
3. THEN execute transferCall tool
Never execute transferCall silently.
```

**Overly general examples:**
```markdown
# BAD
Example: Caller asks question → Agent answers

# GOOD
Example:
Caller: "What time do you close today?" (Monday 3pm)
Agent: "Today we're open until 8pm."
```
</anti_patterns>
