# Workflow: Surgical Prompt Edit

<required_reading>
**Read these reference files NOW:**
1. references/prompt-structure.md
2. references/voice-constraints.md
</required_reading>

<objective>
Make a targeted, minimal change to fix a specific behavior without disrupting the rest of the prompt. Surgical edits are for when you know exactly what's wrong and need to fix just that.
</objective>

<process>

<step_1>
## Step 1: Define the Problem Precisely

Before touching anything, articulate:
- **Observed behavior:** What is the agent doing wrong?
- **Expected behavior:** What should it do instead?
- **Trigger conditions:** When does this happen? (time, input, channel)
- **Scope:** Is this ONE issue or multiple?

**Good problem statement:**
```
Observed: Agent says "Today we're open 8am to 8pm" on Saturdays
Expected: Agent says "Today we're open 11am to 9pm" on Saturdays
Trigger: Any "hours" question on Saturday
Scope: Single issue - incorrect hours
```

**Bad problem statement:**
```
The agent is giving wrong information sometimes.
```

**Action:** Write a precise problem statement before proceeding.
</step_1>

<step_2>
## Step 2: Read the Full Prompt

Read the ENTIRE prompt file. Do not skim. Look for:
- Where is the relevant information?
- Are there multiple places this could be defined?
- Is there conflicting information elsewhere?
- Are there conditionals that should apply?

**Common locations for different issues:**
| Issue Type | Check These Sections |
|------------|---------------------|
| Wrong facts | Context, FAQs, Business Context |
| Wrong workflow | Task, Workflow subsections |
| Wrong tool usage | Task, Tool sections |
| Wrong tone/format | Identity, Voice Best Practices |
| Wrong conditional | Task, Transfer Handling |
| Wrong example | Examples section |
</step_2>

<step_3>
## Step 3: Identify the Root Cause

Why is the agent doing the wrong thing?

**Root cause possibilities:**

1. **Missing information:** The correct info isn't in the prompt
   → Solution: Add the missing info

2. **Ambiguous instruction:** Agent could interpret it either way
   → Solution: Add specificity/examples

3. **Conflicting instruction:** Two parts say different things
   → Solution: Resolve conflict, add priority rule

4. **Missing conditional:** Logic doesn't account for this case
   → Solution: Add the conditional branch

5. **Stale information:** Info was correct but is now outdated
   → Solution: Update the info

6. **Overly general instruction:** Applies too broadly
   → Solution: Add exceptions/constraints

**Action:** Identify the single root cause. If multiple, handle one at a time.
</step_3>

<step_4>
## Step 4: Design the Minimal Fix

Write the fix BEFORE editing. The fix should:
- Change as few lines as possible
- Add rather than rewrite
- Preserve existing formatting
- Not touch unrelated sections

**Fix design template:**
```
Target section: [section name]
Current text:
"""
[exact text to change]
"""

New text:
"""
[exact replacement]
"""

Change rationale: [why this fixes the issue]
Risk assessment: [what could break]
```

**Example - Adding missing Saturday hours context:**
```
Target section: Context > Hours > IMPORTANT

Current text:
"""
- When asked about "hours" without clarification, assume they mean dog park and bar hours for TODAY.
"""

New text:
"""
- When asked about "hours" without clarification, assume they mean dog park and bar hours for TODAY. Use the correct hours based on today's day of the week:
  - Monday-Thursday: 8am-8pm
  - Friday: 8am-9pm
  - Saturday: 11am-9pm
  - Sunday: 11am-7pm
"""

Change rationale: Agent needs the hours in context when answering, not just in the Hours section
Risk assessment: Low - additive change, hours already defined elsewhere
```
</step_4>

<step_5>
## Step 5: Check for Dependencies

Before applying, verify:

**Forward dependencies:** Will this change break something later in the prompt?
- Search for references to the section you're changing
- Check if any examples depend on current behavior

**Backward dependencies:** Does this change need earlier context?
- Is the information you're adding already defined elsewhere?
- Could there be conflicts?

**Channel dependencies:** Does this apply to both voice and SMS?
- Check for `{% if transport.conversationType %}` blocks
- Make sure your change goes in the right branch (or both)

**Time dependencies:** Does this involve time-aware logic?
- Check for time extraction: `{{"now" | date: ...}}`
- Verify your change handles all time scenarios
</step_5>

<step_6>
## Step 6: Apply the Edit

Use the Edit tool with exact text matching:

```
Read the current text first
↓
Copy exact old_string (including whitespace)
↓
Create new_string with minimal change
↓
Apply edit
↓
Verify change took effect
```

**Critical: Match whitespace exactly.** If the old text has 3 spaces of indent, your old_string must have 3 spaces.

**After edit, read the section again to verify:**
- Change was applied correctly
- Formatting is preserved
- No accidental modifications
</step_6>

<step_7>
## Step 7: Test the Specific Behavior

Create a quick test for the exact issue:

**Test case:**
```
Input: "What are your hours?" (asked on Saturday)
Expected: Response mentions "11am" or "11" for opening
Expected: Response mentions "9pm" or "9" for closing
Expected: Does NOT mention "8am" (weekday hours)
```

**Verification methods:**
1. Mental trace: Walk through how the agent would process this
2. Search for conflicts: Are there other places that could override this?
3. Edge cases: Does this work for all relevant scenarios?

**If you have access to test calls:**
- Make a test call with the exact trigger
- Verify the agent responds correctly
- Check for any unexpected side effects
</step_7>

<step_8>
## Step 8: Document the Change

Record what you changed:

```markdown
## Surgical Edit: {brief description}

**Date:** {date}
**Prompt:** {file path}

**Problem:** {problem statement}
**Root cause:** {why it was happening}
**Fix:** {what you changed}

**Section:** {which section}
**Lines changed:** {approximate}

**Before:**
```
{old text}
```

**After:**
```
{new text}
```

**Testing:** {how you verified}
**Risk:** {what could break}
```
</step_8>

</process>

<common_surgical_edits>
## Common Edit Patterns

**Adding an exception:**
```markdown
# Before
No prong or shock collars allowed.

# After
No prong or shock collars allowed. Gentle leader head collars and front-clip harnesses are permitted.
```

**Adding time-awareness:**
```markdown
# Before
Tell them today's hours.

# After
Tell them today's hours based on the current day:
- Monday-Thursday: 8am-8pm
- Friday: 8am-9pm
- Saturday: 11am-9pm
- Sunday: 11am-7pm
```

**Adding channel branch:**
```markdown
# Before
Transfer angry callers to the onsite team.

# After
{% if transport.conversationType == "chat" %}
For angry texters, say: "I'm sorry for the issue. Please give us a call at (XXX) XXX-XXXX."
{% else %}
Transfer angry callers to the onsite team.
{% endif %}
```

**Adding pronunciation guide:**
```markdown
# Before
The address is 18595 Carousel Lane.

# After
**Address**: Say "one-eight-five-nine-five Carousel Lane, Westfield" (do NOT include zipcode or state abbreviation)
```

**Strengthening a constraint:**
```markdown
# Before
Keep responses brief.

# After
**Length**: Limit responses to 1–3 sentences maximum. Do NOT provide more information than directly asked for.
```
</common_surgical_edits>

<what_not_to_do>
## Surgical Edit Anti-Patterns

**Don't refactor while editing:**
```
# BAD
"While fixing the hours, I also reorganized the FAQ section for clarity"

# GOOD
"Fixed only the hours issue. FAQ reorganization is a separate task."
```

**Don't guess at fixes:**
```
# BAD
"I think the issue might be in this section, so I'll rewrite it"

# GOOD
"Traced the issue to line 342 in the FAQs section. Making targeted edit."
```

**Don't add features while fixing bugs:**
```
# BAD
"Fixed the hours issue and also added support for holiday hours"

# GOOD
"Fixed the hours issue only. Holiday hours is a separate enhancement."
```

**Don't change formatting unnecessarily:**
```
# BAD
"Changed the bullet to a numbered list while fixing the content"

# GOOD
"Preserved existing bullet format, only changed the content"
```
</what_not_to_do>

<success_criteria>
This workflow is complete when:
- [ ] Problem precisely defined
- [ ] Full prompt read and understood
- [ ] Root cause identified
- [ ] Minimal fix designed
- [ ] Dependencies checked
- [ ] Edit applied correctly
- [ ] Change verified
- [ ] Change documented
</success_criteria>
