# Workflow: Audit Voice Agent Prompt

<required_reading>
**Read these reference files NOW:**
1. references/prompt-structure.md
2. references/voice-constraints.md
3. references/common-failures.md
</required_reading>

<objective>
Review an existing voice agent prompt for anti-patterns, missing elements, and improvement opportunities. Produce an actionable audit report with prioritized recommendations.
</objective>

<process>

<step_1>
## Step 1: Read the Complete Prompt

Read the entire prompt file without making changes. Take notes on:
- Overall structure and organization
- Length (line count)
- Sections present/missing
- First impressions of clarity

**Expected sections for a well-structured Vapi prompt:**
1. Dynamic Variables
2. Channel Detection (if multi-channel)
3. Identity (Role, Tone, Personality)
4. Task (Workflow, Opening, Core Logic)
5. Common Services/Scenarios
6. Edge Case Handling
7. Transfer Handling (if applicable)
8. Context (Business Info, Hours, FAQs)
9. Rules of Engagement
10. Best Practices (Voice/SMS specific)
11. Examples

**Action:** Complete read-through before any analysis.
</step_1>

<step_2>
## Step 2: Check Structural Issues

Audit the prompt structure:

**Organization checklist:**
- [ ] Clear section separation
- [ ] Logical flow (identity → task → context → rules → examples)
- [ ] Channel-specific content properly branched
- [ ] No orphaned/disconnected sections

**Structural anti-patterns:**
| Issue | Symptom | Impact |
|-------|---------|--------|
| Scattered info | Same topic in 3+ places | Agent inconsistent |
| Buried constraints | Important rules at bottom | Agent ignores them |
| Missing conditionals | No time/channel branching | Wrong behavior |
| Monolithic sections | 100+ line sections | Agent loses focus |

**Document:** List all structural issues found.
</step_2>

<step_3>
## Step 3: Check Voice-Specific Requirements

Voice agents have unique constraints. Verify:

**Brevity:**
- [ ] Response length explicitly constrained (1-3 sentences)
- [ ] "Be concise" is insufficient - needs specific limits
- [ ] Follow-up question guidance (when to ask, when not to)

**Pronunciation:**
- [ ] Address pronunciation guide (digits vs words)
- [ ] Phone number format (grouping, pacing)
- [ ] Website/email spoken format
- [ ] Acronym handling (spell out or pronounce?)
- [ ] Unit formatting (say "pounds" not "L-B-S")

**Pacing:**
- [ ] Pause instructions after questions
- [ ] Speaking pace guidance
- [ ] Wait-for-response reminders

**Tool timing:**
- [ ] Clear "speak THEN execute" instructions
- [ ] Tool triggers well-defined
- [ ] Post-tool-call behavior specified
</step_3>

<step_4>
## Step 4: Check Channel Handling

If multi-channel (voice + SMS):

**Channel branching:**
- [ ] Uses `{% if transport.conversationType %}` correctly
- [ ] Voice-only features gated (transferCall, audio handling)
- [ ] SMS-specific responses defined
- [ ] Phone number formatting differs (spoken vs written)

**Common channel issues:**
| Issue | Impact |
|-------|--------|
| Transfer mentioned in SMS | Agent promises impossible action |
| No SMS-specific closing | Agent tries to "end call" in text |
| Same response for both | Missing channel-appropriate formatting |

**Document:** List all channel handling issues.
</step_4>

<step_5>
## Step 5: Check Information Accuracy

Audit factual content:

**Hours:**
- [ ] All service types covered
- [ ] Day-specific hours correct
- [ ] Holiday hours handling defined
- [ ] Timezone handling (should be suppressed for callers)

**Pricing:**
- [ ] All tiers/packages documented
- [ ] Add-on pricing included
- [ ] Stale pricing flagged
- [ ] "Contact for current pricing" for volatile info

**Policies:**
- [ ] Requirements clearly stated (vaccines, age, etc.)
- [ ] Exceptions process defined
- [ ] What agent CAN vs CANNOT do

**Contact info:**
- [ ] Phone number correct
- [ ] Address complete
- [ ] Website/email accurate
- [ ] Social media if relevant
</step_5>

<step_6>
## Step 6: Check Workflow Logic

Audit the core workflows:

**Opening:**
- [ ] Greeting appropriate for channel
- [ ] Outbound vs inbound handled differently (if applicable)
- [ ] Time-of-day awareness (if relevant)

**Core task flow:**
- [ ] Clear decision tree for common requests
- [ ] Tool usage triggers well-defined
- [ ] Escalation paths clear
- [ ] Dead-end handling

**Closing:**
- [ ] Natural conversation end detection
- [ ] End call tool usage
- [ ] "Anything else" guidelines

**Transfer logic (if applicable):**
- [ ] Clear transfer triggers
- [ ] Time-aware transfer messages
- [ ] Staffed vs unstaffed handling
- [ ] After-hours handling
</step_6>

<step_7>
## Step 7: Check Constraints and Boundaries

Audit what the agent should NOT do:

**Off-topic handling:**
- [ ] Clear refusal for unrelated topics
- [ ] Polite but firm redirects
- [ ] No "I don't know" responses (redirect instead)

**Policy boundaries:**
- [ ] Cannot make exceptions
- [ ] Cannot process payments
- [ ] Cannot modify accounts
- [ ] Cannot provide legal advice

**Safety rails:**
- [ ] Angry caller handling
- [ ] Spam/sales deflection
- [ ] Sensitive topic handling
- [ ] Emergency situations
</step_7>

<step_8>
## Step 8: Check Examples Quality

Audit the examples section:

**Coverage:**
- [ ] Examples for each common scenario
- [ ] Voice AND SMS examples (if multi-channel)
- [ ] Edge case examples
- [ ] Error handling examples

**Quality:**
- [ ] Input/output pairs clear
- [ ] Examples match actual expected behavior
- [ ] Examples show tool usage timing
- [ ] Examples demonstrate constraints

**Common example issues:**
| Issue | Impact |
|-------|--------|
| Too few examples | Agent improvises incorrectly |
| Outdated examples | Agent copies wrong patterns |
| Conflicting examples | Agent inconsistent |
| Missing tool examples | Wrong tool timing |
</step_8>

<step_9>
## Step 9: Generate Audit Report

Create a structured audit report:

```markdown
# Prompt Audit Report

## Summary
- **Prompt:** {file path}
- **Lines:** {count}
- **Overall assessment:** {Good/Needs Work/Critical Issues}
- **Priority issues:** {count}

## Scores by Category
| Category | Score | Notes |
|----------|-------|-------|
| Structure | /5 | {brief} |
| Voice Constraints | /5 | {brief} |
| Channel Handling | /5 | {brief} |
| Information Accuracy | /5 | {brief} |
| Workflow Logic | /5 | {brief} |
| Constraints/Boundaries | /5 | {brief} |
| Examples | /5 | {brief} |

## Critical Issues (Fix Immediately)
1. {issue + recommendation}
2. {issue + recommendation}

## High Priority Issues (Fix Soon)
1. {issue + recommendation}
2. {issue + recommendation}

## Medium Priority Issues (Plan to Fix)
1. {issue + recommendation}

## Low Priority Issues (Nice to Have)
1. {issue + recommendation}

## Strengths
- {What's working well}
- {Patterns to preserve}

## Recommended Next Steps
1. {First action}
2. {Second action}
3. {Third action}
```
</step_9>

</process>

<audit_checklist>
## Quick Audit Checklist

**Structure:**
- [ ] Sections clearly separated
- [ ] Logical flow
- [ ] Under 600 lines
- [ ] No scattered information

**Voice:**
- [ ] 1-3 sentence limit stated
- [ ] Pronunciation guides
- [ ] Pause instructions
- [ ] Tool timing clear

**Channel:**
- [ ] Proper branching
- [ ] Voice-only features gated
- [ ] SMS alternatives defined

**Information:**
- [ ] Hours complete and correct
- [ ] Pricing current
- [ ] Policies clear
- [ ] Contact info accurate

**Workflow:**
- [ ] Opening defined
- [ ] Core flow clear
- [ ] Closing natural
- [ ] Transfers time-aware

**Boundaries:**
- [ ] Off-topic handled
- [ ] Policies enforced
- [ ] Safety rails in place

**Examples:**
- [ ] Coverage adequate
- [ ] Quality high
- [ ] Up to date
</audit_checklist>

<success_criteria>
This workflow is complete when:
- [ ] Complete prompt read
- [ ] Structure audited
- [ ] Voice constraints checked
- [ ] Channel handling verified
- [ ] Information accuracy reviewed
- [ ] Workflow logic analyzed
- [ ] Constraints audited
- [ ] Examples evaluated
- [ ] Audit report generated
- [ ] Issues prioritized
- [ ] Next steps recommended
</success_criteria>
