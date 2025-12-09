# Evaluation Criteria for Voice Agents

<overview>
This reference defines the criteria for evaluating voice agent behavior. Use these categories to build comprehensive eval suites that catch regressions and validate improvements.
</overview>

<eval_categories>
## Evaluation Categories

### 1. Factual Accuracy
Does the agent provide correct information?

**What to test:**
- Hours (by day, by service type)
- Pricing (all tiers, add-ons)
- Requirements (vaccines, age limits, policies)
- Contact info (address, phone, website, email)
- Business details (services, features, capacity)

**Eval format:**
```yaml
eval:
  id: "hours-saturday-park"
  category: factual_accuracy
  input: "What are your hours on Saturday?"
  expected: "11am to 9pm" or equivalent
  criteria:
    - Contains "11" (opening)
    - Contains "9" (closing)
    - Does NOT mention weekday hours
    - Does NOT mention timezone
```

### 2. Workflow Compliance
Does the agent follow the defined workflow?

**What to test:**
- Opening greeting format
- Question answering sequence
- Tool usage triggers
- Transfer decision logic
- Closing behavior

**Eval format:**
```yaml
eval:
  id: "transfer-booking-request"
  category: workflow_compliance
  input: "I'd like to book a party"
  context:
    time: "Monday 2pm"
    channel: "voice"
  expected_behavior:
    - Agent says transfer message
    - Agent mentions voicemail (outside staffed hours)
    - Agent executes transferCall
  criteria:
    - Contains "connect" or "transfer"
    - Contains "voicemail"
    - Tool called: transferCall
```

### 3. Tool Usage
Does the agent call the right tools at the right time?

**What to test:**
- Transfer triggers (when should transfer happen?)
- Knowledge base queries (when to search?)
- Event lookups (when to fetch events?)
- End call (when to terminate?)

**Eval format:**
```yaml
eval:
  id: "kb-query-before-answer"
  category: tool_usage
  input: "What vaccines does my dog need?"
  expected_tool: "knowledge_base_query" or similar
  expected_before_response: true
  criteria:
    - Knowledge base queried
    - Query happens before response
    - Response uses KB results
```

### 4. Fallback Handling
Does the agent escalate correctly when unsure?

**What to test:**
- Unknown questions → transfer or direct to call
- Policy exceptions → cannot provide, escalate
- Complex requests → transfer
- Audio issues → transfer after retry

**Eval format:**
```yaml
eval:
  id: "unknown-question-transfer"
  category: fallback_handling
  input: "Can I get a discount for my birthday?"
  expected_behavior: "Transfer to onsite team"
  criteria:
    - Does NOT promise discount
    - Does NOT say "I don't know"
    - Offers transfer (voice) or directs to call (SMS)
```

### 5. Constraint Compliance
Does the agent follow formatting and length rules?

**What to test:**
- Response length (1-3 sentences)
- Pronunciation formats
- Timezone suppression
- Channel-appropriate responses

**Eval format:**
```yaml
eval:
  id: "response-length"
  category: constraint_compliance
  input: "Tell me about your memberships"
  criteria:
    - Response ≤ 3 sentences
    - No timezone mentioned
    - No "L-B-S" for weight
```

### 6. Edge Case Handling
Does the agent handle unusual situations correctly?

**What to test:**
- Off-topic questions
- Angry callers
- Spam/sales calls
- Audio quality issues
- Silence

**Eval format:**
```yaml
eval:
  id: "off-topic-refusal"
  category: edge_case
  input: "Can you help me write Python code?"
  expected: "Polite refusal, redirect to business"
  criteria:
    - Does NOT engage with Python
    - Mentions "[Business] questions only"
    - Does NOT say "I can't help"
```

### 7. Channel Compliance
Does the agent behave correctly for the channel?

**What to test:**
- Voice: transfers work, short responses
- SMS: no transfers promised, phone number provided

**Eval format:**
```yaml
eval:
  id: "sms-no-transfer-promise"
  category: channel_compliance
  input: "Can I speak to someone?"
  context:
    channel: "chat"  # SMS
  criteria:
    - Does NOT mention "transfer" or "connect"
    - Provides phone number
    - Mentions staffed hours
```
</eval_categories>

<priority_levels>
## Priority Levels

### P0 - Critical
**Fail = Production blocker. Must fix before deploy.**

Examples:
- Wrong hours for today
- Transfer not happening when required
- Wrong phone number
- Safety issue (giving medical advice, etc.)

### P1 - Important
**Fail = Investigate immediately. Fix before next release.**

Examples:
- Wrong pricing tier
- Channel-specific behavior broken
- Tool timing wrong (transfer before speaking)
- Opening greeting wrong

### P2 - Moderate
**Fail = Add to backlog. Fix in next sprint.**

Examples:
- Response slightly too long
- Pronunciation not following guide
- Edge case not handled optimally

### P3 - Minor
**Fail = Track. Fix when convenient.**

Examples:
- Tone slightly off
- Could be more concise
- Minor wording improvements
</priority_levels>

<rubric_types>
## Rubric Types

### PassFail
Binary success/failure. Use for critical behaviors.

```yaml
rubric: PassFail
pass_if:
  - All criteria met
fail_if:
  - Any criterion failed
```

**Best for:**
- Factual accuracy (hours, pricing)
- Critical tool usage (transfers)
- Safety constraints

### Checklist
Score based on criteria met. Use for multi-faceted behaviors.

```yaml
rubric: Checklist
criteria:
  - Contains correct information
  - Uses appropriate tone
  - Follows length guidelines
  - No prohibited content
score: count_of_criteria_met
pass_threshold: 3/4
```

**Best for:**
- Complex responses
- Multi-part answers
- Comprehensive coverage testing

### NumericScale
1-5 quality rating. Use for subjective quality.

```yaml
rubric: NumericScale
scale:
  5: Perfect response
  4: Minor issue, acceptable
  3: Noticeable issue, needs improvement
  2: Significant issue, problematic
  1: Complete failure
pass_threshold: 4
```

**Best for:**
- Tone evaluation
- Naturalness assessment
- Overall quality

### DescriptiveScale
Labeled quality levels. Use for nuanced assessment.

```yaml
rubric: DescriptiveScale
levels:
  Excellent: "Fully addresses query with perfect format"
  Good: "Addresses query with minor formatting issues"
  Acceptable: "Addresses query but has noticeable issues"
  Poor: "Partially addresses query or has significant issues"
  Fail: "Does not address query or violates constraints"
pass_threshold: Acceptable
```
</rubric_types>

<eval_design_patterns>
## Eval Design Patterns

### Pattern 1: Time-Aware Testing
Test behavior across different times:

```yaml
# Same input, different contexts
eval_group: "hours-question"
evals:
  - id: "hours-monday-10am"
    input: "What are your hours?"
    context: { day: "Monday", time: "10:00 AM" }
    expected: "8am to 8pm"

  - id: "hours-saturday-10am"
    input: "What are your hours?"
    context: { day: "Saturday", time: "10:00 AM" }
    expected: "11am to 9pm"

  - id: "hours-sunday-10pm"
    input: "What are your hours?"
    context: { day: "Sunday", time: "10:00 PM" }
    expected: "Currently closed, opens 11am"
```

### Pattern 2: Channel Comparison
Test same input across channels:

```yaml
eval_group: "transfer-request"
evals:
  - id: "transfer-request-voice"
    input: "Can I speak to someone?"
    context: { channel: "voice" }
    expected: "Transfer + tool execution"

  - id: "transfer-request-sms"
    input: "Can I speak to someone?"
    context: { channel: "chat" }
    expected: "Direct to phone number"
```

### Pattern 3: Edge Case Coverage
Test boundaries and unusual inputs:

```yaml
eval_group: "hours-edge-cases"
evals:
  - id: "hours-ambiguous"
    input: "Are you open?"
    expected: "Today's hours based on current time"

  - id: "hours-specific-service"
    input: "What are daycare hours?"
    expected: "Daycare hours, not park hours"

  - id: "hours-holiday"
    input: "Are you open on Christmas?"
    expected: "Transfer for holiday hours"
```

### Pattern 4: Negative Testing
Test what agent should NOT do:

```yaml
eval_group: "prohibited-behaviors"
evals:
  - id: "no-policy-exceptions"
    input: "Can my unvaccinated dog come in?"
    fail_if:
      - Agent says "yes" or approves exception
      - Agent promises to make exception
    pass_if:
      - Agent explains policy
      - Agent offers escalation path

  - id: "no-off-topic"
    input: "What's the capital of France?"
    fail_if:
      - Agent answers "Paris"
      - Agent engages with geography
    pass_if:
      - Agent redirects to business topics
```
</eval_design_patterns>

<baseline_targets>
## Baseline Pass Rate Targets

| Category | P0 Target | P1 Target | P2 Target |
|----------|-----------|-----------|-----------|
| Factual Accuracy | 100% | 98% | 95% |
| Workflow Compliance | 100% | 95% | 90% |
| Tool Usage | 100% | 95% | 90% |
| Fallback Handling | 100% | 95% | 85% |
| Constraint Compliance | 95% | 90% | 85% |
| Edge Case Handling | 95% | 90% | 80% |
| Channel Compliance | 100% | 95% | 90% |

**Regression threshold:** Any drop of >2% from baseline requires investigation.
</baseline_targets>
