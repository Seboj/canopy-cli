<div align="center">

# CANOPY CLI

**A meta-prompting, context engineering and spec-driven development system for Claude Code, OpenCode, Gemini CLI, and Codex.**

**The Canopy Method — Outcomes, Decisions, Context, Flow, Signal — engineered for AI coding agents.**

[![npm version](https://img.shields.io/npm/v/canopy-cc?style=for-the-badge&logo=npm&logoColor=white&color=CB3837)](https://www.npmjs.com/package/canopy-cc)
[![GitHub stars](https://img.shields.io/github/stars/Seboj/canopy-cli?style=for-the-badge&logo=github&color=181717)](https://github.com/Seboj/canopy-cli)
[![License](https://img.shields.io/badge/license-MIT-blue?style=for-the-badge)](LICENSE)

<br>

```bash
npx canopy-cc@latest
```

**Works on Mac, Windows, and Linux.**

<br>

*"Describe what you want. Let the system build it correctly."*

<br>

[Why Canopy](#why-canopy) · [How It Works](#how-it-works) · [Commands](#commands) · [Why It Works](#why-it-works) · [User Guide](docs/USER-GUIDE.md)

</div>

---

## Why Canopy

AI coding agents are incredibly powerful — if you give them the context they need. Most people don't.

Other spec-driven tools exist, but they either over-engineer the process (sprint ceremonies, story points, retrospectives) or lack real understanding of what you're building. Canopy is different. It implements **The Canopy Method** — a post-Scrum methodology built around five elements:

- **Outcomes** — What you're building and why (not tasks, not stories)
- **Decisions** — Explicit tracking of every choice and its rationale
- **Context** — Living knowledge that grows with your project
- **Flow** — Continuous movement through stages, not time-boxed sprints
- **Signal** — Real metrics that tell you what's actually happening

The complexity is in the system, not in your workflow. Behind the scenes: context engineering, XML prompt formatting, subagent orchestration, state management. What you see: a few commands that just work.

---

## Who This Is For

People who want to describe what they want and have it built correctly — without pretending they're running a 50-person engineering org.

---

## Getting Started

```bash
npx canopy-cc@latest
```

The installer prompts you to choose:
1. **Runtime** — Claude Code, OpenCode, Gemini, Codex, or all
2. **Location** — Global (all projects) or local (current project only)

Verify with:
- Claude Code / Gemini: `/canopy:help`
- OpenCode: `/canopy-help`
- Codex: `$canopy-help`

### Staying Updated

```bash
npx canopy-cc@latest
```

<details>
<summary><strong>Non-interactive Install (Docker, CI, Scripts)</strong></summary>

```bash
# Claude Code
npx canopy-cc --claude --global   # Install to ~/.claude/
npx canopy-cc --claude --local    # Install to ./.claude/

# OpenCode (open source, free models)
npx canopy-cc --opencode --global # Install to ~/.config/opencode/

# Gemini CLI
npx canopy-cc --gemini --global   # Install to ~/.gemini/

# Codex (skills-first)
npx canopy-cc --codex --global    # Install to ~/.codex/

# All runtimes
npx canopy-cc --all --global      # Install to all directories
```

</details>

<details>
<summary><strong>Development Installation</strong></summary>

```bash
git clone https://github.com/Seboj/canopy-cli.git
cd canopy-cli
node bin/install.js --claude --local
```

</details>

### Recommended: Skip Permissions Mode

Canopy is designed for frictionless automation. Run Claude Code with:

```bash
claude --dangerously-skip-permissions
```

<details>
<summary><strong>Alternative: Granular Permissions</strong></summary>

Add this to your project's `.claude/settings.json`:

```json
{
  "permissions": {
    "allow": [
      "Bash(date:*)",
      "Bash(echo:*)",
      "Bash(cat:*)",
      "Bash(ls:*)",
      "Bash(mkdir:*)",
      "Bash(wc:*)",
      "Bash(head:*)",
      "Bash(tail:*)",
      "Bash(sort:*)",
      "Bash(grep:*)",
      "Bash(tr:*)",
      "Bash(git add:*)",
      "Bash(git commit:*)",
      "Bash(git status:*)",
      "Bash(git log:*)",
      "Bash(git diff:*)",
      "Bash(git tag:*)"
    ]
  }
}
```

</details>

---

## How It Works

> **Already have code?** Run `/canopy:map-codebase` first. It spawns parallel agents to analyze your stack, architecture, conventions, and concerns. Then `/canopy:new-project` knows your codebase — questions focus on what you're adding, and planning automatically loads your patterns.

### 1. Initialize Project

```
/canopy:new-project
```

One command, one flow. The system:

1. **Questions** — Asks until it understands your idea completely
2. **Research** — Spawns parallel agents to investigate the domain
3. **Requirements** — Extracts what's v1, v2, and out of scope
4. **Roadmap** — Creates phases mapped to requirements

**Creates:** `PROJECT.md`, `REQUIREMENTS.md`, `ROADMAP.md`, `STATE.md`, `.canopy/research/`

---

### 2. Discuss Phase

```
/canopy:discuss-phase 1
```

Your roadmap has a sentence or two per phase. That's not enough context to build something the way *you* imagine it. This step captures your preferences before anything gets researched or planned.

**Creates:** `{phase_num}-CONTEXT.md`

---

### 3. Plan Phase

```
/canopy:plan-phase 1
```

The system:

1. **Researches** — Investigates how to implement this phase
2. **Plans** — Creates 2-3 atomic task plans with XML structure
3. **Verifies** — Checks plans against requirements, loops until they pass

Each plan is small enough to execute in a fresh context window. No degradation.

**Creates:** `{phase_num}-RESEARCH.md`, `{phase_num}-{N}-PLAN.md`

---

### 4. Execute Phase

```
/canopy:execute-phase 1
```

The system:

1. **Runs plans in waves** — Parallel where possible, sequential when dependent
2. **Fresh context per plan** — 200k tokens purely for implementation
3. **Commits per task** — Every task gets its own atomic commit
4. **Verifies against goals** — Checks the codebase delivers what the phase promised

**Creates:** `{phase_num}-{N}-SUMMARY.md`, `{phase_num}-VERIFICATION.md`

---

### 5. Verify Work

```
/canopy:verify-work 1
```

Automated verification + manual user acceptance testing. If something's broken, it creates fix plans for immediate re-execution.

**Creates:** `{phase_num}-UAT.md`, fix plans if issues found

---

### 6. Repeat, Complete, Next Milestone

```
/canopy:complete-milestone
/canopy:new-milestone
```

Loop **discuss, plan, execute, verify** until milestone complete. Each milestone is a clean cycle: define, build, ship.

---

### Quick Mode

```
/canopy:quick
```

For ad-hoc tasks that don't need full planning. Same agents, same quality, faster path.

---

## Why It Works

### Context Engineering

| File | What it does |
|------|--------------|
| `PROJECT.md` | Project vision, always loaded |
| `research/` | Ecosystem knowledge |
| `REQUIREMENTS.md` | Scoped requirements with phase traceability |
| `ROADMAP.md` | Where you're going, what's done |
| `STATE.md` | Decisions, blockers, position — memory across sessions |
| `PLAN.md` | Atomic task with XML structure, verification steps |
| `SUMMARY.md` | What happened, what changed, committed to history |

### Multi-Agent Orchestration

| Stage | Orchestrator does | Agents do |
|-------|------------------|-----------|
| Research | Coordinates, presents findings | 4 parallel researchers investigate stack, features, architecture, pitfalls |
| Planning | Validates, manages iteration | Planner creates plans, checker verifies, loop until pass |
| Execution | Groups into waves, tracks progress | Executors implement in parallel, each with fresh 200k context |
| Verification | Presents results, routes next | Verifier checks codebase against goals, debuggers diagnose failures |

### Atomic Git Commits

Each task gets its own commit immediately after completion:

```bash
abc123f docs(08-02): complete user registration plan
def456g feat(08-02): add email confirmation flow
hij789k feat(08-02): implement password hashing
```

---

## Commands

### Core Workflow

| Command | What it does |
|---------|--------------|
| `/canopy:new-project [--auto]` | Full initialization: questions, research, requirements, roadmap |
| `/canopy:discuss-phase [N]` | Capture implementation decisions before planning |
| `/canopy:plan-phase [N]` | Research + plan + verify for a phase |
| `/canopy:execute-phase <N>` | Execute all plans in parallel waves |
| `/canopy:verify-work [N]` | Manual user acceptance testing |
| `/canopy:audit-milestone` | Verify milestone achieved its definition of done |
| `/canopy:complete-milestone` | Archive milestone, tag release |
| `/canopy:new-milestone [name]` | Start next version |

### Navigation

| Command | What it does |
|---------|--------------|
| `/canopy:progress` | Where am I? What's next? |
| `/canopy:help` | Show all commands and usage guide |
| `/canopy:update` | Update Canopy with changelog preview |

### Brownfield

| Command | What it does |
|---------|--------------|
| `/canopy:map-codebase` | Analyze existing codebase before new-project |

### Phase Management

| Command | What it does |
|---------|--------------|
| `/canopy:add-phase` | Append phase to roadmap |
| `/canopy:insert-phase [N]` | Insert urgent work between phases |
| `/canopy:remove-phase [N]` | Remove future phase, renumber |
| `/canopy:list-phase-assumptions [N]` | See Claude's intended approach before planning |
| `/canopy:plan-milestone-gaps` | Create phases to close gaps from audit |

### Session

| Command | What it does |
|---------|--------------|
| `/canopy:pause-work` | Create handoff when stopping mid-phase |
| `/canopy:resume-work` | Restore from last session |

### Utilities

| Command | What it does |
|---------|--------------|
| `/canopy:settings` | Configure model profile and workflow agents |
| `/canopy:set-profile <profile>` | Switch model profile (quality/balanced/budget) |
| `/canopy:add-todo [desc]` | Capture idea for later |
| `/canopy:check-todos` | List pending todos |
| `/canopy:debug [desc]` | Systematic debugging with persistent state |
| `/canopy:quick [--full]` | Execute ad-hoc task |
| `/canopy:health [--repair]` | Validate `.canopy/` directory integrity |

---

## Configuration

Canopy stores project settings in `.canopy/config.json`. Configure during `/canopy:new-project` or update later with `/canopy:settings`.

### Core Settings

| Setting | Options | Default |
|---------|---------|---------|
| `mode` | `yolo`, `interactive` | `interactive` |
| `depth` | `quick`, `standard`, `comprehensive` | `standard` |

### Model Profiles

| Profile | Planning | Execution | Verification |
|---------|----------|-----------|--------------|
| `quality` | Opus | Opus | Sonnet |
| `balanced` (default) | Opus | Sonnet | Sonnet |
| `budget` | Sonnet | Sonnet | Haiku |

### Git Branching

| Strategy | What it does |
|----------|--------------|
| `none` (default) | Commits to current branch |
| `phase` | Creates branch per phase, merges at completion |
| `milestone` | Creates one branch for milestone, merges at completion |

---

## Uninstalling

```bash
npx canopy-cc --claude --global --uninstall
npx canopy-cc --opencode --global --uninstall
npx canopy-cc --codex --global --uninstall
```

---

## License

MIT License. See [LICENSE](LICENSE) for details.

---

<div align="center">

**AI coding agents are powerful. Canopy makes them reliable.**

[canopymethod.ai](https://canopymethod.ai)

</div>
