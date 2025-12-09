# Workflow: Optimize Prompt from Eval Results

<required_reading>
**Read these reference files NOW:**
1. references/common-failures.md
2. references/fix-patterns.md
3. references/prompt-structure.md
</required_reading>

<objective>
Systematically analyze failing evals, identify root causes, and make targeted prompt improvements that fix failures without introducing regressions.
</objective>

<process>

<step_1>
## Step 1: Gather Eval Results

Collect all failing eval cases. For each failure, document:
- Eval ID and category
- Input that triggered failure
- Expected behavior
- Actual behavior
- Specific criteria that failed

**Example failure log:**
```
EVAL-017: hours-saturday-park
Category: factual_accuracy
Input: "What time do you open Saturday?"
Expected: "11am"
Actual: "8am" (agent gave weekday hours)
Failed criteria: Contains "11" and "am"
```

**Action:** Create a complete list of failures before proceeding.
</step_1>

<step_2>
## Step 2: Categorize Failures by Root Cause

Group failures into root cause categories:

**Knowledge gaps:**
- Agent lacks information to answer correctly
- Fix: Add to Context section or FAQs

**Ambiguous instructions:**
- Prompt is unclear about what to do
- Fix: Add specificity to Task section

**Missing conditional logic:**
- Agent doesn't branch correctly (time, day, channel)
- Fix: Add explicit conditionals with Liquid syntax

**Competing instructions:**
- Two parts of prompt conflict
- Fix: Resolve conflict, add priority rules

**Wrong tool usage:**
- Agent calls wrong tool or at wrong time
- Fix: Clarify tool triggers in Task section

**Constraint violations:**
- Agent breaks length/tone/format rules
- Fix: Strengthen constraints in Rules section

**Example pattern detection:**
- Agent needs example to follow
- Fix: Add example to Examples section

**Action:** Assign root cause to each failure.
</step_2>

<step_3>
## Step 3: Prioritize Fixes

Prioritize fixes by:
1. **Impact:** How many evals does this fix?
2. **Risk:** How likely to cause regressions?
3. **Effort:** How complex is the change?

**Fix priority matrix:**
| Impact | Risk | Effort | Priority |
|--------|------|--------|----------|
| High   | Low  | Low    | P0 - Do first |
| High   | Low  | High   | P1 - Plan carefully |
| Low    | Low  | Low    | P2 - Quick wins |
| Any    | High | Any    | P3 - Extreme caution |

**Action:** Order fixes by priority.
</step_3>

<step_4>
## Step 4: Design Minimal Fixes

For each failure, design the MINIMAL change that fixes it:

**Rule: One change per commit. One failure category per fix.**

**Fix patterns by root cause:**

**Knowledge gap fix:**
```markdown
# Before
**Pricing**: "Day pass is $15 per dog."

# After (add missing info)
**Pricing**: "Day pass is $15 per dog, and $10 for each additional dog."
```

**Ambiguous instruction fix:**
```markdown
# Before
When asked about hours, provide today's hours.

# After (add specificity)
When asked about "hours" without specification, assume they are asking about the dog park and bar hours for TODAY. Provide today's hours only.
If they explicitly ask about daycare, drop-off, or drop-in hours, provide daycare hours.
```

**Missing conditional fix:**
```markdown
# Before
Transfer callers when they ask for staff.

# After (add time-awareness)
**SCENARIO 1 - Within Staffed Front Desk Hours:**
When BOTH conditions are true:
- Within Dog Park and Bar Hours (Mon-Thu 8am-8pm, ...)
- AND within Staffed Front Desk Hours (Mon-Fri 5pm-8pm, ...)
YOU MUST SAY: "Sure, I'll connect you with the onsite team."

**SCENARIO 2 - Outside Staffed Hours:**
...
```

**Constraint violation fix:**
```markdown
# Before
Keep responses concise.

# After (make constraint explicit)
- **Length**: Limit responses to 1–3 sentences maximum.
- **No Unnecessary Follow-ups**: Do NOT ask clarifying or follow-up questions after you've already answered the caller's question. Do NOT proactively ask "Anything else?" after answering.
```
</step_4>

<step_5>
## Step 5: Locate Edit Points

Find the exact location in the prompt to make each edit:

**Prompt structure reminder:**
1. Dynamic Variables
2. Channel Detection
3. Identity (Role, Tone)
4. Task (Workflow, Opening, Logic)
5. Context (Business info, Hours, FAQs)
6. Rules (Boundaries, Constraints)
7. Examples

**Match fix to section:**
- Knowledge gaps → Context/FAQs
- Ambiguous instructions → Task
- Missing conditionals → Task (with Liquid syntax)
- Competing instructions → Whichever section has lower priority
- Wrong tool usage → Task (tool sections)
- Constraint violations → Rules or Identity/Tone
- Missing examples → Examples

**Action:** Note the exact section and line range for each fix.
</step_5>

<step_6>
## Step 6: Apply Fixes Surgically

Apply each fix one at a time:

1. **Read the target section** before editing
2. **Copy the exact text** to be replaced
3. **Make the minimal change** - add don't rewrite
4. **Preserve formatting** - match existing style
5. **Test immediately** - run the specific failing eval

**Change log format:**
```
FIX-001: hours-saturday-park
Section: Context > Hours
Change: Added weekend-specific hours clarification
Before: "Saturday: 11am-9pm"
After: "Saturday: 11am-9pm (opens later than weekdays)"
Evals fixed: EVAL-017, EVAL-018
Regression risk: Low (additive change)
```

**DO NOT:**
- Reformat unrelated sections
- "Clean up" adjacent code
- Change working patterns
- Remove constraints
</step_6>

<step_7>
## Step 7: Validate No Regressions

After each fix, run the FULL eval suite:

**Regression check:**
```
Before fix:
- P0 pass: 18/20 (90%)
- P1 pass: 25/28 (89%)
- P2 pass: 12/15 (80%)

After fix:
- P0 pass: 20/20 (100%) ✓ improved
- P1 pass: 25/28 (89%) ✓ no change
- P2 pass: 12/15 (80%) ✓ no change
```

**If regression detected:**
1. STOP - do not continue
2. Revert the change
3. Analyze why regression occurred
4. Redesign fix to avoid regression
5. Try again

**Acceptable outcomes:**
- Target evals now pass
- All other evals unchanged
- OR explicit tradeoff approved by user
</step_7>

<step_8>
## Step 8: Document Changes

Create optimization report:

```markdown
# Optimization Report: {Location}

## Summary
- Evals analyzed: {count}
- Failures fixed: {count}
- Regressions: {count}
- Net improvement: {before}% → {after}%

## Changes Made

### FIX-001: {description}
- Root cause: {category}
- Evals fixed: {list}
- Change: {summary}

### FIX-002: ...

## Remaining Issues
- {List any unfixed failures and why}

## Recommendations
- {Next steps for further improvement}
```
</step_8>

</process>

<anti_patterns>
## What NOT to Do

**Don't rewrite sections:**
```
# BAD: Rewrote entire section
"I improved the hours section by reorganizing it..."

# GOOD: Targeted edit
"Added 'opens later than weekdays' to Saturday hours line"
```

**Don't fix unrelated issues:**
```
# BAD: While fixing hours, also cleaned up formatting
"Also noticed some inconsistent bullet points, fixed those too"

# GOOD: One fix at a time
"Fixing only EVAL-017 (Saturday hours) in this change"
```

**Don't add without validating:**
```
# BAD: Added new feature while fixing bug
"Added a new FAQ about pets while fixing the hours issue"

# GOOD: Stay focused
"Will address new FAQ in separate change after current fixes are validated"
```
</anti_patterns>

<success_criteria>
This workflow is complete when:
- [ ] All failures categorized by root cause
- [ ] Fixes prioritized by impact/risk
- [ ] Each fix designed as minimal change
- [ ] Fixes applied one at a time
- [ ] Each fix validated with eval run
- [ ] No regressions introduced
- [ ] Optimization report created
- [ ] Remaining issues documented
</success_criteria>
