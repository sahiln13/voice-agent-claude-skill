# Vapi Evaluation Rubric Types

<overview>
Vapi supports multiple rubric types for evaluating call success. This reference covers when to use each type and how to configure them.
</overview>

<rubric_types>
## Available Rubric Types

Vapi's SuccessEvaluationPlan supports these rubric types:

| Type | Best For | Output |
|------|----------|--------|
| PassFail | Binary success/failure | pass or fail |
| Checklist | Multi-criteria scoring | criteria met count |
| NumericScale | Quality rating (1-5) | number |
| DescriptiveScale | Labeled quality levels | level name |
| PercentageScale | Percentage score | 0-100% |
| LikertScale | Agreement scale | strongly disagree to strongly agree |
| Matrix | Multi-dimensional | matrix scores |
| AutomaticRubric | Let model decide | varies |
</rubric_types>

<passfail>
## PassFail Rubric

**Use when:** Behavior is either correct or incorrect. No middle ground.

**Best for:**
- Factual accuracy (hours, pricing)
- Critical tool usage (did transfer happen?)
- Policy compliance (did agent refuse exception?)

**Configuration:**
```json
{
  "analysisPlan": {
    "successEvaluationPlan": {
      "enabled": true,
      "rubric": "PassFail"
    }
  }
}
```

**Eval prompt pattern:**
```markdown
Evaluate if the agent's response was correct.

Pass criteria (ALL must be true):
- Agent stated hours as 8am-8pm for Monday
- Agent did NOT mention timezone
- Agent did NOT offer additional information

Fail criteria (ANY triggers fail):
- Agent stated wrong hours
- Agent mentioned timezone
- Agent asked unnecessary follow-up questions

Output: "pass" or "fail" only
```

**Example use cases:**
- Did agent give correct Saturday hours?
- Did agent transfer for booking request?
- Did agent refuse to grant policy exception?
</passfail>

<checklist>
## Checklist Rubric

**Use when:** Multiple criteria should be evaluated independently.

**Best for:**
- Comprehensive response evaluation
- Multi-part answers
- Quality with multiple dimensions

**Configuration:**
```json
{
  "analysisPlan": {
    "successEvaluationPlan": {
      "enabled": true,
      "rubric": "Checklist"
    }
  }
}
```

**Eval prompt pattern:**
```markdown
Evaluate the agent's response against this checklist.

Check each criterion:
[ ] Contains correct hours (8am-8pm for weekdays)
[ ] Does not mention timezone
[ ] Response is 3 sentences or fewer
[ ] Tone is warm and professional
[ ] Does not ask unnecessary follow-up

Score: [X/5 criteria met]
Pass threshold: 4/5
```

**Example use cases:**
- Full response quality check
- Multi-part question coverage
- Comprehensive workflow compliance
</checklist>

<numericscale>
## NumericScale Rubric

**Use when:** Quality exists on a spectrum from poor to excellent.

**Best for:**
- Tone/naturalness evaluation
- Overall call quality
- Subjective assessments

**Configuration:**
```json
{
  "analysisPlan": {
    "successEvaluationPlan": {
      "enabled": true,
      "rubric": "NumericScale"
    }
  }
}
```

**Eval prompt pattern:**
```markdown
Rate the agent's response quality on a 1-5 scale:

5 - Excellent: Perfect response, all criteria met, natural tone
4 - Good: Minor issues, still fully acceptable
3 - Acceptable: Noticeable issues but functional
2 - Poor: Significant issues, needs improvement
1 - Fail: Complete failure to address query

Provide score as single number: 1, 2, 3, 4, or 5
Pass threshold: 4 or higher
```

**Example use cases:**
- Overall conversation quality
- Agent naturalness/human-likeness
- Customer experience rating
</numericscale>

<descriptivescale>
## DescriptiveScale Rubric

**Use when:** Quality levels need descriptive labels rather than numbers.

**Best for:**
- Nuanced quality assessment
- Human-readable reports
- Training/feedback purposes

**Configuration:**
```json
{
  "analysisPlan": {
    "successEvaluationPlan": {
      "enabled": true,
      "rubric": "DescriptiveScale"
    }
  }
}
```

**Eval prompt pattern:**
```markdown
Rate the agent's response using these levels:

Excellent: Fully addresses query with perfect format and tone
Good: Addresses query well with minor format/tone issues
Acceptable: Addresses query adequately but has noticeable issues
Poor: Partially addresses query or has significant issues
Fail: Does not address query or violates critical constraints

Output single level name only.
Pass threshold: "Acceptable" or higher
```

**Example use cases:**
- Human review calibration
- Quality tier classification
- Stakeholder reporting
</descriptivescale>

<percentagescale>
## PercentageScale Rubric

**Use when:** Fine-grained scoring is needed.

**Best for:**
- Weighted criteria evaluation
- Granular quality tracking
- Trend analysis over time

**Configuration:**
```json
{
  "analysisPlan": {
    "successEvaluationPlan": {
      "enabled": true,
      "rubric": "PercentageScale"
    }
  }
}
```

**Eval prompt pattern:**
```markdown
Score the response as a percentage (0-100):

Criteria and weights:
- Accuracy (40%): Is information correct?
- Brevity (20%): Is response appropriately concise?
- Tone (20%): Is tone warm and professional?
- Completeness (20%): Does it fully answer the question?

Calculate weighted score as percentage.
Output: number 0-100
Pass threshold: 80%
```

**Example use cases:**
- Detailed quality scoring
- A/B test comparisons
- Performance dashboards
</percentagescale>

<likertscale>
## LikertScale Rubric

**Use when:** Measuring agreement/satisfaction levels.

**Best for:**
- User sentiment analysis
- Agreement with statements
- Survey-style evaluation

**Configuration:**
```json
{
  "analysisPlan": {
    "successEvaluationPlan": {
      "enabled": true,
      "rubric": "LikertScale"
    }
  }
}
```

**Eval prompt pattern:**
```markdown
Rate agreement with this statement:
"The agent fully and correctly addressed the caller's question."

Strongly Disagree | Disagree | Neutral | Agree | Strongly Agree

Output single level.
Pass threshold: "Agree" or "Strongly Agree"
```

**Example use cases:**
- Satisfaction proxying
- Statement validation
- Comparative assessment
</likertscale>

<automaticrubric>
## AutomaticRubric

**Use when:** Let the model determine appropriate evaluation method.

**Best for:**
- Rapid prototyping
- When unsure which rubric fits
- General quality assessment

**Configuration:**
```json
{
  "analysisPlan": {
    "successEvaluationPlan": {
      "enabled": true,
      "rubric": "AutomaticRubric"
    }
  }
}
```

**Note:** Model will determine evaluation approach based on context. Less predictable but useful for initial exploration.
</automaticrubric>

<choosing_rubric>
## Rubric Selection Guide

| Scenario | Recommended Rubric |
|----------|-------------------|
| Did agent do X? (yes/no) | PassFail |
| How well did agent do X? | NumericScale |
| Did agent do A, B, C? | Checklist |
| What quality tier? | DescriptiveScale |
| Weighted multi-criteria | PercentageScale |
| Does response indicate X? | LikertScale |
| Not sure yet | AutomaticRubric |

### By Eval Category

| Category | Primary Rubric | Alternative |
|----------|---------------|-------------|
| Factual Accuracy | PassFail | Checklist |
| Workflow Compliance | PassFail | Checklist |
| Tool Usage | PassFail | - |
| Fallback Handling | PassFail | - |
| Response Quality | NumericScale | DescriptiveScale |
| Tone/Style | NumericScale | DescriptiveScale |
| Comprehensive Check | Checklist | PercentageScale |
</choosing_rubric>

<vapi_analysis_plan>
## Vapi Analysis Plan Configuration

Full analysisPlan structure:

```json
{
  "analysisPlan": {
    "summaryPlan": {
      "enabled": true,
      "prompt": "Summarize this call in 2-3 sentences focusing on the caller's question and how it was resolved."
    },
    "structuredDataPlan": {
      "enabled": true,
      "schema": {
        "type": "object",
        "properties": {
          "caller_intent": { "type": "string" },
          "resolution": { "type": "string" },
          "transfer_occurred": { "type": "boolean" }
        }
      }
    },
    "successEvaluationPlan": {
      "enabled": true,
      "rubric": "PassFail",
      "messages": [
        {
          "role": "system",
          "content": "You are evaluating a voice agent call..."
        }
      ],
      "timeoutSeconds": 30
    }
  }
}
```

### Key Fields

- `enabled`: Turn evaluation on/off
- `rubric`: Rubric type (see above)
- `messages`: Custom eval prompt messages
- `timeoutSeconds`: Evaluation timeout
</vapi_analysis_plan>

<custom_eval_prompts>
## Custom Evaluation Prompts

For more control, use custom messages:

```json
{
  "successEvaluationPlan": {
    "enabled": true,
    "rubric": "PassFail",
    "messages": [
      {
        "role": "system",
        "content": "You are an LLM-Judge evaluating a voice agent call transcript.\n\nEvaluate ONLY the agent's final response.\n\nPass criteria (ALL must be true):\n- Agent stated correct hours for the day asked\n- Agent did NOT mention timezone\n- Response was 3 sentences or fewer\n\nFail criteria (ANY triggers fail):\n- Wrong hours stated\n- Timezone mentioned\n- Response exceeded 3 sentences\n\nOutput exactly one word: pass or fail"
      }
    ]
  }
}
```

### Prompt Template

```markdown
You are an LLM-Judge evaluating a voice agent call.

Context:
- Transcript: {{transcript}}
- Agent's last response: {{messages[-1]}}

Pass criteria (ALL must be true):
- [Criterion 1]
- [Criterion 2]

Fail criteria (ANY triggers fail):
- [Failure condition 1]
- [Failure condition 2]

Output format: respond with exactly one word: pass or fail
- No explanations
- No punctuation
- No additional text
```
</custom_eval_prompts>
