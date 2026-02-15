# Agent Work Estimation Skill

An [Agent Skill](https://agentskills.io) that fixes how AI coding agents estimate task duration. Instead of anchoring to human developer timelines ("this would take 2-3 days"), the agent estimates from its own operational units — **tool-call rounds** — and converts to wallclock time only at the end.

## The Problem

AI coding agents systematically overestimate task duration because they anchor to human developer timelines absorbed from training data. A task an agent can complete in 30 minutes gets estimated as "2-3 days" because that's what a human developer forum post would say.

## The Solution

This skill forces the agent to think in **rounds** (one tool-call cycle: think → write code → execute → verify → fix), estimate round counts per module, apply risk coefficients, and only convert to human wallclock time at the very end.

## Installation

### Using `npx skills` (Recommended)

The [skills CLI](https://github.com/vercel-labs/skills) is the standard package manager for the open agent skills ecosystem. It works with Claude Code, Cursor, Codex CLI, and [35+ other agents](https://skills.sh).

```bash
# Install to your current project
npx skills add ZhangHanDong/agent-estimation

# Install globally (available across all projects)
npx skills add ZhangHanDong/agent-estimation -g

# Install for a specific agent
npx skills add ZhangHanDong/agent-estimation -a claude-code

# Non-interactive
npx skills add ZhangHanDong/agent-estimation -g -a claude-code -y
```

### Manual Installation (Claude Code)

Clone the repo into your Claude Code skills directory:

```bash
# Personal (all projects)
git clone git@github.com:ZhangHanDong/agent-estimation.git \
  ~/.claude/skills/agent-estimation

# Project-specific
git clone git@github.com:ZhangHanDong/agent-estimation.git \
  .claude/skills/agent-estimation
```

## Usage

Once installed, the skill activates automatically when you ask Claude (or another agent) to estimate, scope, or plan work. You can also invoke it directly:

```
/agent-estimation
```

### Example Prompts

- "Estimate how long it would take to build a CLI tool that converts JSON to YAML"
- "How many rounds would it take to add JWT auth to this API?"
- "Scope out the work for adding dark mode to this app"

### Example Output

```markdown
### Task: CLI JSON-to-YAML Converter

#### Module Breakdown

| # | Module              | Base Rounds | Risk | Effective Rounds | Notes                      |
|---|---------------------|-------------|------|------------------|----------------------------|
| 1 | Arg parsing + I/O   | 1           | 1.0  | 1                | clap, one-shot             |
| 2 | JSON→YAML core      | 1           | 1.0  | 1                | serde, trivial             |
| 3 | Schema validation   | 3           | 1.3  | 4                | jsonschema crate, edge cases|
| 4 | Error handling + UX | 2           | 1.0  | 2                | polish                     |

#### Summary

- **Base rounds**: 7
- **Integration**: +1 round
- **Risk-adjusted total**: 8 rounds
- **Estimated wallclock**: ~24 minutes (at 3 min/round)

#### Biggest Risks
1. Schema validation edge cases with nested structures
```

## How It Works

The skill teaches the agent a three-layer estimation framework:

### Core Units

| Unit        | Definition                                              | Scale                                |
|-------------|---------------------------------------------------------|--------------------------------------|
| **Round**   | One tool-call cycle: think → write → execute → verify → fix | ~2-4 min wallclock                   |
| **Module**  | A functional unit built from multiple rounds            | 2-15 rounds                          |
| **Project** | All modules + integration + debugging                   | Sum of modules x integration factor  |

### Estimation Procedure

1. **Decompose** the task into independently buildable modules
2. **Estimate rounds** per module using calibrated anchors (1-2 for boilerplate, 3-5 for moderate, 5-10 for exploratory, 8-15 for high uncertainty)
3. **Apply risk coefficients** (1.0 low → 2.0 very high) based on documentation quality, platform quirks, and integration unknowns
4. **Add integration rounds** (10-20% of base total)
5. **Convert to wallclock** only at the end (default: 3 min/round)

### Anti-Patterns Prevented

- **Human-time anchoring**: "A developer would take about 2 weeks..." → Blocked
- **Padding by vibes**: Adding time "just to be safe" without rationale → Blocked
- **Complexity ≠ volume**: 500 lines of boilerplate ≠ hard; 1 line of CGEvent API ≠ easy
- **Forgetting integration cost**: Modules work alone but break together
- **Ignoring user-side bottlenecks**: Manual permission grants, device testing, etc.

## Files

| File                       | Description                                             |
|----------------------------|---------------------------------------------------------|
| `SKILL.md`                 | Main skill definition (the agent reads this)            |
| `calibration-examples.md`  | Real-world calibration examples across project sizes    |
| `evals.json`               | Test prompts to validate estimation accuracy            |

## Compatibility

This skill follows the [Agent Skills](https://agentskills.io) open standard and works with any compatible agent, including:

- [Claude Code](https://claude.com/claude-code)
- [Cursor](https://cursor.com)
- [Codex CLI](https://github.com/openai/codex)
- [GitHub Copilot](https://github.com/features/copilot)
- [And many more](https://skills.sh)

## License

MIT
