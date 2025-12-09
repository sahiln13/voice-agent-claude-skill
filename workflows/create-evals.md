# Workflow: Create Evaluation Suite

<required_reading>
**Read these reference files NOW:**
1. references/eval-criteria.md
2. references/rubric-types.md
</required_reading>

<objective>
Build a comprehensive evaluation suite that tests voice agent behavior across all critical scenarios. Evals should catch regressions and validate improvements.
</objective>

<process>

<step_1>
## Step 1: Identify the Prompt to Evaluate

Locate and read the voice agent prompt file. Identify:
- Which location/client is this for?
- What channel(s) does it support? (voice, SMS, both)
- What are the core tasks? (answering questions, transfers, bookings, etc.)
- What tools does it use? (transferCall, endCall, n8n, knowledge base)

**Action:** Read the prompt file completely before proceeding.
</step_1>

<step_2>
## Step 2: Extract Testable Behaviors

From the prompt, extract every discrete behavior that should be tested:

**Categories to extract:**

1. **Factual Responses**
   - Hours (by day, by service type)
   - Pricing (day passes, memberships, add-ons)
   - Requirements (vaccines, age limits, policies)
   - Contact info (address, phone, website, email)

2. **Workflow Compliance**
   - Opening greeting format
   - Channel-specific behavior (voice vs SMS)
   - Time-aware logic (open/closed, staffed/unstaffed)
   - Tool usage triggers (when to transfer, when to use knowledge base)

3. **Edge Cases**
   - Off-topic refusal
   - Angry caller handling
   - Unclear audio handling
   - Spam/sales deflection

4. **Constraints**
   - Response length (1-3 sentences)
   - Pronunciation rules
   - Things the agent must NOT say
   - Timezone suppression

**Action:** Create a list of 20-50 specific behaviors to test.
</step_2>

<step_3>
## Step 3: Design Eval Cases

For each behavior, create an eval case with:

```yaml
eval_case:
  id: "hours-monday-park"
  category: "factual_accuracy"
  scenario: "Caller asks about park hours on Monday"
  input: "What are your hours on Monday?"
  context:
    day: "Monday"
    time: "10:00 AM"
    channel: "voice"
  expected_behavior: "Agent states Monday park hours are 8am-8pm"
  success_criteria:
    - Contains "8" and "am" (opening time)
    - Contains "8" and "pm" (closing time)
    - Does NOT mention timezone
  rubric: "PassFail"
```

**Eval case structure:**
- `id`: Unique identifier for tracking
- `category`: factual_accuracy | workflow_compliance | edge_case | constraint
- `scenario`: Human-readable description
- `input`: What the caller says
- `context`: Time, day, channel, any relevant state
- `expected_behavior`: What the agent should do
- `success_criteria`: Specific, measurable criteria
- `rubric`: PassFail | Checklist | NumericScale (1-5)
</step_3>

<step_4>
## Step 4: Prioritize by Risk

Not all evals are equal. Prioritize by:

**P0 - Critical (test first, fail = block)**
- Hours accuracy
- Transfer routing
- Vaccine requirements
- Pricing accuracy

**P1 - Important (test always, fail = investigate)**
- Channel-specific behavior
- Opening greeting
- Tool usage triggers
- Edge case handling

**P2 - Nice to have (test regularly, fail = backlog)**
- Pronunciation
- Response length
- Tone consistency

**Action:** Assign priority to each eval case.
</step_4>

<step_5>
## Step 5: Create Test Transcripts

For each eval case, create a realistic test transcript:

**Voice transcript format:**
```
CALLER: [What the caller says]
AGENT: [Expected agent response]
[TOOL: toolName if applicable]
```

**SMS transcript format:**
```
USER: [What the user texts]
AGENT: [Expected agent response]
```

**Include variations:**
- Different phrasings of the same question
- Common misspellings/mishearings
- Follow-up questions
- Edge case inputs
</step_5>

<step_6>
## Step 6: Define Success Rubrics

For each eval, specify how to evaluate success:

**PassFail rubric:**
```
Pass if ALL criteria met:
- [Criterion 1]
- [Criterion 2]

Fail if ANY:
- [Failure condition 1]
- [Failure condition 2]
```

**Checklist rubric:**
```
Score = count of criteria met:
- [ ] Contains correct hours
- [ ] Does not mention timezone
- [ ] Under 3 sentences
- [ ] Warm/friendly tone
```

**NumericScale rubric (1-5):**
```
5: Perfect response, all criteria met
4: Minor issue, still acceptable
3: Noticeable issue, needs improvement
2: Significant issue, problematic
1: Complete failure
```
</step_6>

<step_7>
## Step 7: Output Eval Suite

Create the eval suite document with:

1. **Summary section:**
   - Total eval count by category
   - Priority distribution
   - Coverage gaps identified

2. **Eval cases section:**
   - All eval cases with full details
   - Organized by category

3. **Baseline expectations:**
   - Target pass rate per category
   - Acceptable regression threshold

**File format:** Create as `evals/{location-slug}-eval-suite.md`
</step_7>

</process>

<templates>
## Eval Suite Template

```markdown
# Eval Suite: {Location Name}

## Summary
- Total evals: {count}
- P0 (Critical): {count}
- P1 (Important): {count}
- P2 (Nice to have): {count}

## Coverage
- Factual accuracy: {count} evals
- Workflow compliance: {count} evals
- Edge cases: {count} evals
- Constraints: {count} evals

## Baseline Targets
- P0 pass rate: 100%
- P1 pass rate: 95%
- P2 pass rate: 85%

---

## P0 - Critical Evals

### EVAL-001: {name}
- **Category:** factual_accuracy
- **Input:** "{caller input}"
- **Context:** {day}, {time}, {channel}
- **Expected:** {expected behavior}
- **Rubric:** PassFail
- **Pass criteria:**
  - {criterion 1}
  - {criterion 2}
- **Fail if:**
  - {failure condition}

---

## P1 - Important Evals
...

## P2 - Nice to Have Evals
...
```
</templates>

<success_criteria>
This workflow is complete when:
- [ ] Prompt has been fully analyzed
- [ ] 20+ testable behaviors identified
- [ ] Eval cases created for each behavior
- [ ] Priority assigned to each eval
- [ ] Test transcripts written
- [ ] Success rubrics defined
- [ ] Eval suite document created
- [ ] Coverage gaps documented
</success_criteria>
