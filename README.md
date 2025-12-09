# Voice Agent Prompt Engineer

A Claude Code skill for optimizing Vapi voice agent prompts through systematic evaluation, surgical edits, and GPT-5.1 specific patterns.

## What This Skill Does

Building reliable voice agents is hard. This skill brings prompt engineering rigor to Vapi voice agents:

- **Create Evals** — Build comprehensive test suites that catch regressions before they hit production
- **Optimize from Evals** — Systematically fix failing evals without breaking what works
- **Surgical Edits** — Make targeted, minimal changes to fix specific behaviors
- **Audit Prompts** — Review existing prompts for anti-patterns and improvement opportunities

## Why This Exists

Voice agents have unique constraints that text-based AI doesn't:
- 1-3 sentence responses (callers can't scroll back)
- Pronunciation rules (say "one-eight-five" not "185")
- Tool timing ("speak THEN execute" for transfers)
- Channel branching (voice vs SMS behave differently)
- Time-aware logic (staffed hours, open/closed)

This skill encodes patterns learned from building production voice agents with Vapi + GPT-5.1.

## Installation

```bash
# Clone to your Claude Code skills directory
git clone https://github.com/YOUR_USERNAME/voice-agent-prompt-engineer ~/.claude/skills/voice-agent-prompt-engineer

# Create the slash command
cat > ~/.claude/commands/voice-prompt.md << 'EOF'
---
description: Optimize Vapi voice agent prompts through evals, surgical edits, and systematic improvement
argument-hint: [create evals | optimize | surgical edit | audit | prompt path]
allowed-tools: Skill(voice-agent-prompt-engineer)
---

Invoke the voice-agent-prompt-engineer skill for: $ARGUMENTS
EOF
```

## Usage

```bash
# Create an evaluation suite for your voice agent
/voice-prompt create evals for vapi/my-agent/prompt.md

# Optimize based on failing evals
/voice-prompt optimize - my hours evals are failing

# Make a surgical edit
/voice-prompt surgical edit - agent says wrong Saturday hours

# Audit an existing prompt
/voice-prompt audit vapi/my-agent/prompt.md
```

## What's Included

```
voice-agent-prompt-engineer/
├── SKILL.md                     # Router + essential principles
├── workflows/
│   ├── create-evals.md         # Build evaluation suites
│   ├── optimize-from-evals.md  # Fix failing evals systematically
│   ├── surgical-prompt-edit.md # Targeted, minimal changes
│   └── audit-prompt.md         # Review prompts for issues
└── references/
    ├── gpt-5-1-patterns.md     # GPT-5.1 specific optimizations
    ├── prompt-structure.md     # Standard Vapi prompt structure
    ├── voice-constraints.md    # Voice-specific requirements
    ├── eval-criteria.md        # Evaluation categories
    ├── rubric-types.md         # Vapi rubric types
    ├── common-failures.md      # Failure pattern catalog
    └── fix-patterns.md         # Reusable fix templates
```

## Key Principles

The skill enforces five core principles:

1. **Evals Are Your Source of Truth** — Never optimize blindly. Build evals first, track pass rates, reject regressions.

2. **Surgical Edits Over Rewrites** — Change ONE section at a time. Preserve working patterns. Add specificity, don't remove constraints.

3. **Voice Is Not Text** — 1-3 sentences max. Pronunciation guides. Pacing instructions. Tool timing matters.

4. **Prompt Structure Hierarchy** — Order matters: Dynamic Variables → Channel Detection → Identity → Task → Context → Rules → Examples

5. **Eval Types for Voice** — Different behaviors need different strategies: factual accuracy, workflow compliance, tool usage, fallback handling, tone/brevity.

## GPT-5.1 Patterns

This skill includes patterns specific to GPT-5.1 (used in Vapi as "5.1 Instant"):

- **None reasoning mode** — Treat like GPT-4.1, use few-shot prompting
- **Personality shaping** — Clear persona definition for voice
- **Tool timing** — Explicit "speak THEN execute" sequences
- **Conflict resolution** — Priority rules for competing instructions
- **Metaprompting** — Using GPT-5.1 to analyze its own prompt failures

## Example Eval

```yaml
eval:
  id: "hours-saturday-park"
  category: factual_accuracy
  input: "What are your hours on Saturday?"
  context:
    day: "Saturday"
    channel: "voice"
  expected: "11am to 9pm"
  criteria:
    - Contains "11" (opening time)
    - Contains "9" (closing time)
    - Does NOT mention timezone
    - Response ≤ 3 sentences
  rubric: PassFail
```

## Contributing

This skill was built for a specific use case (Vapi voice agents for pet care businesses) but the patterns are broadly applicable. PRs welcome for:

- Additional failure patterns
- New fix templates
- Other voice AI platform support
- Improved eval strategies

## License

MIT

---

Built with [Claude Code](https://claude.ai/claude-code)
