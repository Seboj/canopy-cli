# Canopy User Guide

A detailed reference for workflows, troubleshooting, and configuration. For quick-start setup, see the [README](../README.md).

---

## Table of Contents

- [Workflow Diagrams](#workflow-diagrams)
- [Command Reference](#command-reference)
- [Configuration Reference](#configuration-reference)
- [Usage Examples](#usage-examples)
- [Troubleshooting](#troubleshooting)
- [Recovery Quick Reference](#recovery-quick-reference)

---

## Workflow Diagrams

### Full Project Lifecycle

```
  ┌──────────────────────────────────────────────────┐
  │                   NEW PROJECT                    │
  │  /canopy:new-project                             │
  │  Questions -> Research -> Requirements -> Roadmap│
  └─────────────────────────┬────────────────────────┘
                            │
             ┌──────────────▼─────────────┐
             │      FOR EACH PHASE:       │
             │                            │
             │  ┌────────────────────┐    │
             │  │ /canopy:discuss-phase │  <- Lock in preferences
             │  └──────────┬─────────┘    │
             │             │              │
             │  ┌──────────▼─────────┐    │
             │  │ /canopy:plan-phase │    │  <- Research + Plan + Verify
             │  └──────────┬─────────┘    │
             │             │              │
             │  ┌──────────▼─────────┐    │
             │  │ /canopy:execute-phase │  <- Parallel execution
             │  └──────────┬─────────┘    │
             │             │              │
             │  ┌──────────▼─────────┐    │
             │  │ /canopy:verify-work │   │  <- Manual UAT
             │  └──────────┬─────────┘    │
             │             │              │
             │     Next Phase?────────────┘
             │             │ No
             └─────────────┼──────────────┘
                            │
            ┌───────────────▼──────────────┐
            │  /canopy:audit-milestone     │
            │  /canopy:complete-milestone  │
            └───────────────┬──────────────┘
                            │
                   Another milestone?
                       │          │
                      Yes         No -> Done!
                       │
               ┌───────▼──────────────┐
               │  /canopy:new-milestone│
               └──────────────────────┘
```

### Planning Agent Coordination

```
  /canopy:plan-phase N
         │
         ├── Phase Researcher (x4 parallel)
         │     ├── Stack researcher
         │     ├── Features researcher
         │     ├── Architecture researcher
         │     └── Pitfalls researcher
         │           │
         │     ┌──────▼──────┐
         │     │ RESEARCH.md │
         │     └──────┬──────┘
         │            │
         │     ┌──────▼──────┐
         │     │   Planner   │  <- Reads PROJECT.md, REQUIREMENTS.md,
         │     │             │     CONTEXT.md, RESEARCH.md
         │     └──────┬──────┘
         │            │
         │     ┌──────▼───────────┐     ┌────────┐
         │     │   Plan Checker   │────>│ PASS?  │
         │     └──────────────────┘     └───┬────┘
         │                                  │
         │                             Yes  │  No
         │                              │   │   │
         │                              │   └───┘  (loop, up to 3x)
         │                              │
         │                        ┌─────▼──────┐
         │                        │ PLAN files │
         │                        └────────────┘
         └── Done
```

### Validation Architecture (Nyquist Layer)

During plan-phase research, Canopy now maps automated test coverage to each phase
requirement before any code is written. This ensures that when Claude's executor
commits a task, a feedback mechanism already exists to verify it within seconds.

The researcher detects your existing test infrastructure, maps each requirement to
a specific test command, and identifies any test scaffolding that must be created
before implementation begins (Wave 0 tasks).

The plan-checker enforces this as an 8th verification dimension: plans where tasks
lack automated verify commands will not be approved.

**Output:** `{phase}-VALIDATION.md` -- the feedback contract for the phase.

**Disable:** Set `workflow.nyquist_validation: false` in `/canopy:settings` for
rapid prototyping phases where test infrastructure isn't the focus.

### Execution Wave Coordination

```
  /canopy:execute-phase N
         │
         ├── Analyze plan dependencies
         │
         ├── Wave 1 (independent plans):
         │     ├── Executor A (fresh 200K context) -> commit
         │     └── Executor B (fresh 200K context) -> commit
         │
         ├── Wave 2 (depends on Wave 1):
         │     └── Executor C (fresh 200K context) -> commit
         │
         └── Verifier
               └── Check codebase against phase goals
                     │
                     ├── PASS -> VERIFICATION.md (success)
                     └── FAIL -> Issues logged for /canopy:verify-work
```

### Brownfield Workflow (Existing Codebase)

```
  /canopy:map-codebase
         │
         ├── Stack Mapper     -> codebase/STACK.md
         ├── Arch Mapper      -> codebase/ARCHITECTURE.md
         ├── Convention Mapper -> codebase/CONVENTIONS.md
         └── Concern Mapper   -> codebase/CONCERNS.md
                │
        ┌───────▼──────────────┐
        │ /canopy:new-project  │  <- Questions focus on what you're ADDING
        └──────────────────────┘
```

---

## Command Reference

### Core Workflow

| Command | Purpose | When to Use |
|---------|---------|-------------|
| `/canopy:new-project` | Full project init: questions, research, requirements, roadmap | Start of a new project |
| `/canopy:new-project --auto @idea.md` | Automated init from document | Have a PRD or idea doc ready |
| `/canopy:discuss-phase [N]` | Capture implementation decisions | Before planning, to shape how it gets built |
| `/canopy:plan-phase [N]` | Research + plan + verify | Before executing a phase |
| `/canopy:execute-phase <N>` | Execute all plans in parallel waves | After planning is complete |
| `/canopy:verify-work [N]` | Manual UAT with auto-diagnosis | After execution completes |
| `/canopy:audit-milestone` | Verify milestone met its definition of done | Before completing milestone |
| `/canopy:complete-milestone` | Archive milestone, tag release | All phases verified |
| `/canopy:new-milestone [name]` | Start next version cycle | After completing a milestone |

### Navigation

| Command | Purpose | When to Use |
|---------|---------|-------------|
| `/canopy:progress` | Show status and next steps | Anytime -- "where am I?" |
| `/canopy:resume-work` | Restore full context from last session | Starting a new session |
| `/canopy:pause-work` | Save context handoff | Stopping mid-phase |
| `/canopy:help` | Show all commands | Quick reference |
| `/canopy:update` | Update Canopy with changelog preview | Check for new versions |

### Phase Management

| Command | Purpose | When to Use |
|---------|---------|-------------|
| `/canopy:add-phase` | Append new phase to roadmap | Scope grows after initial planning |
| `/canopy:insert-phase [N]` | Insert urgent work (decimal numbering) | Urgent fix mid-milestone |
| `/canopy:remove-phase [N]` | Remove future phase and renumber | Descoping a feature |
| `/canopy:list-phase-assumptions [N]` | Preview Claude's intended approach | Before planning, to validate direction |
| `/canopy:plan-milestone-gaps` | Create phases for audit gaps | After audit finds missing items |
| `/canopy:research-phase [N]` | Deep ecosystem research only | Complex or unfamiliar domain |

### Brownfield & Utilities

| Command | Purpose | When to Use |
|---------|---------|-------------|
| `/canopy:map-codebase` | Analyze existing codebase | Before `/canopy:new-project` on existing code |
| `/canopy:quick` | Ad-hoc task with Canopy guarantees | Bug fixes, small features, config changes |
| `/canopy:debug [desc]` | Systematic debugging with persistent state | When something breaks |
| `/canopy:add-todo [desc]` | Capture an idea for later | Think of something during a session |
| `/canopy:check-todos` | List pending todos | Review captured ideas |
| `/canopy:settings` | Configure workflow toggles and model profile | Change model, toggle agents |
| `/canopy:set-profile <profile>` | Quick profile switch | Change cost/quality tradeoff |
| `/canopy:reapply-patches` | Restore local modifications after update | After `/canopy:update` if you had local edits |

---

## Configuration Reference

Canopy stores project settings in `.canopy/config.json`. Configure during `/canopy:new-project` or update later with `/canopy:settings`.

### Full config.json Schema

```json
{
  "mode": "interactive",
  "depth": "standard",
  "model_profile": "balanced",
  "planning": {
    "commit_docs": true,
    "search_gitignored": false
  },
  "workflow": {
    "research": true,
    "plan_check": true,
    "verifier": true,
    "nyquist_validation": true
  },
  "git": {
    "branching_strategy": "none",
    "phase_branch_template": "canopy/phase-{phase}-{slug}",
    "milestone_branch_template": "canopy/{milestone}-{slug}"
  }
}
```

### Core Settings

| Setting | Options | Default | What it Controls |
|---------|---------|---------|------------------|
| `mode` | `interactive`, `yolo` | `interactive` | `yolo` auto-approves decisions; `interactive` confirms at each step |
| `depth` | `quick`, `standard`, `comprehensive` | `standard` | Planning thoroughness: 3-5, 5-8, or 8-12 phases |
| `model_profile` | `quality`, `balanced`, `budget` | `balanced` | Model tier for each agent (see table below) |

### Planning Settings

| Setting | Options | Default | What it Controls |
|---------|---------|---------|------------------|
| `planning.commit_docs` | `true`, `false` | `true` | Whether `.canopy/` files are committed to git |
| `planning.search_gitignored` | `true`, `false` | `false` | Add `--no-ignore` to broad searches to include `.canopy/` |

> **Note:** If `.canopy/` is in `.gitignore`, `commit_docs` is automatically `false` regardless of the config value.

### Workflow Toggles

| Setting | Options | Default | What it Controls |
|---------|---------|---------|------------------|
| `workflow.research` | `true`, `false` | `true` | Domain investigation before planning |
| `workflow.plan_check` | `true`, `false` | `true` | Plan verification loop (up to 3 iterations) |
| `workflow.verifier` | `true`, `false` | `true` | Post-execution verification against phase goals |
| `workflow.nyquist_validation` | `true`, `false` | `true` | Validation architecture research during plan-phase; 8th plan-check dimension |

Disable these to speed up phases in familiar domains or when conserving tokens.

### Git Branching

| Setting | Options | Default | What it Controls |
|---------|---------|---------|------------------|
| `git.branching_strategy` | `none`, `phase`, `milestone` | `none` | When and how branches are created |
| `git.phase_branch_template` | Template string | `canopy/phase-{phase}-{slug}` | Branch name for phase strategy |
| `git.milestone_branch_template` | Template string | `canopy/{milestone}-{slug}` | Branch name for milestone strategy |

**Branching strategies explained:**

| Strategy | Creates Branch | Scope | Best For |
|----------|---------------|-------|----------|
| `none` | Never | N/A | Solo development, simple projects |
| `phase` | At each `execute-phase` | One phase per branch | Code review per phase, granular rollback |
| `milestone` | At first `execute-phase` | All phases share one branch | Release branches, PR per version |

**Template variables:** `{phase}` = zero-padded number (e.g., "03"), `{slug}` = lowercase hyphenated name, `{milestone}` = version (e.g., "v1.0").

### Model Profiles (Per-Agent Breakdown)

| Agent | `quality` | `balanced` | `budget` |
|-------|-----------|------------|----------|
| canopy-planner | Opus | Opus | Sonnet |
| canopy-roadmapper | Opus | Sonnet | Sonnet |
| canopy-executor | Opus | Sonnet | Sonnet |
| canopy-phase-researcher | Opus | Sonnet | Haiku |
| canopy-project-researcher | Opus | Sonnet | Haiku |
| canopy-research-synthesizer | Sonnet | Sonnet | Haiku |
| canopy-debugger | Opus | Sonnet | Sonnet |
| canopy-codebase-mapper | Sonnet | Haiku | Haiku |
| canopy-verifier | Sonnet | Sonnet | Haiku |
| canopy-plan-checker | Sonnet | Sonnet | Haiku |
| canopy-integration-checker | Sonnet | Sonnet | Haiku |

**Profile philosophy:**
- **quality** -- Opus for all decision-making agents, Sonnet for read-only verification. Use when quota is available and the work is critical.
- **balanced** -- Opus only for planning (where architecture decisions happen), Sonnet for everything else. The default for good reason.
- **budget** -- Sonnet for anything that writes code, Haiku for research and verification. Use for high-volume work or less critical phases.

---

## Usage Examples

### New Project (Full Cycle)

```bash
claude --dangerously-skip-permissions
/canopy:new-project            # Answer questions, configure, approve roadmap
/clear
/canopy:discuss-phase 1        # Lock in your preferences
/canopy:plan-phase 1           # Research + plan + verify
/canopy:execute-phase 1        # Parallel execution
/canopy:verify-work 1          # Manual UAT
/clear
/canopy:discuss-phase 2        # Repeat for each phase
...
/canopy:audit-milestone        # Check everything shipped
/canopy:complete-milestone     # Archive, tag, done
```

### New Project from Existing Document

```bash
/canopy:new-project --auto @prd.md   # Auto-runs research/requirements/roadmap from your doc
/clear
/canopy:discuss-phase 1               # Normal flow from here
```

### Existing Codebase

```bash
/canopy:map-codebase           # Analyze what exists (parallel agents)
/canopy:new-project            # Questions focus on what you're ADDING
# (normal phase workflow from here)
```

### Quick Bug Fix

```bash
/canopy:quick
> "Fix the login button not responding on mobile Safari"
```

### Resuming After a Break

```bash
/canopy:progress               # See where you left off and what's next
# or
/canopy:resume-work            # Full context restoration from last session
```

### Preparing for Release

```bash
/canopy:audit-milestone        # Check requirements coverage, detect stubs
/canopy:plan-milestone-gaps    # If audit found gaps, create phases to close them
/canopy:complete-milestone     # Archive, tag, done
```

### Speed vs Quality Presets

| Scenario | Mode | Depth | Profile | Research | Plan Check | Verifier |
|----------|------|-------|---------|----------|------------|----------|
| Prototyping | `yolo` | `quick` | `budget` | off | off | off |
| Normal dev | `interactive` | `standard` | `balanced` | on | on | on |
| Production | `interactive` | `comprehensive` | `quality` | on | on | on |

### Mid-Milestone Scope Changes

```bash
/canopy:add-phase              # Append a new phase to the roadmap
# or
/canopy:insert-phase 3         # Insert urgent work between phases 3 and 4
# or
/canopy:remove-phase 7         # Descope phase 7 and renumber
```

---

## Troubleshooting

### "Project already initialized"

You ran `/canopy:new-project` but `.canopy/PROJECT.md` already exists. This is a safety check. If you want to start over, delete the `.canopy/` directory first.

### Context Degradation During Long Sessions

Clear your context window between major commands: `/clear` in Claude Code. Canopy is designed around fresh contexts -- every subagent gets a clean 200K window. If quality is dropping in the main session, clear and use `/canopy:resume-work` or `/canopy:progress` to restore state.

### Plans Seem Wrong or Misaligned

Run `/canopy:discuss-phase [N]` before planning. Most plan quality issues come from Claude making assumptions that `CONTEXT.md` would have prevented. You can also run `/canopy:list-phase-assumptions [N]` to see what Claude intends to do before committing to a plan.

### Execution Fails or Produces Stubs

Check that the plan was not too ambitious. Plans should have 2-3 tasks maximum. If tasks are too large, they exceed what a single context window can produce reliably. Re-plan with smaller scope.

### Lost Track of Where You Are

Run `/canopy:progress`. It reads all state files and tells you exactly where you are and what to do next.

### Need to Change Something After Execution

Do not re-run `/canopy:execute-phase`. Use `/canopy:quick` for targeted fixes, or `/canopy:verify-work` to systematically identify and fix issues through UAT.

### Model Costs Too High

Switch to budget profile: `/canopy:set-profile budget`. Disable research and plan-check agents via `/canopy:settings` if the domain is familiar to you (or to Claude).

### Working on a Sensitive/Private Project

Set `commit_docs: false` during `/canopy:new-project` or via `/canopy:settings`. Add `.canopy/` to your `.gitignore`. Planning artifacts stay local and never touch git.

### Canopy Update Overwrote My Local Changes

Since v1.17, the installer backs up locally modified files to `canopy-local-patches/`. Run `/canopy:reapply-patches` to merge your changes back.

### Subagent Appears to Fail but Work Was Done

A known workaround exists for a Claude Code classification bug. Canopy's orchestrators (execute-phase, quick) spot-check actual output before reporting failure. If you see a failure message but commits were made, check `git log` -- the work may have succeeded.

---

## Recovery Quick Reference

| Problem | Solution |
|---------|----------|
| Lost context / new session | `/canopy:resume-work` or `/canopy:progress` |
| Phase went wrong | `git revert` the phase commits, then re-plan |
| Need to change scope | `/canopy:add-phase`, `/canopy:insert-phase`, or `/canopy:remove-phase` |
| Milestone audit found gaps | `/canopy:plan-milestone-gaps` |
| Something broke | `/canopy:debug "description"` |
| Quick targeted fix | `/canopy:quick` |
| Plan doesn't match your vision | `/canopy:discuss-phase [N]` then re-plan |
| Costs running high | `/canopy:set-profile budget` and `/canopy:settings` to toggle agents off |
| Update broke local changes | `/canopy:reapply-patches` |

---

## Project File Structure

For reference, here is what Canopy creates in your project:

```
.canopy/
  PROJECT.md              # Project vision and context (always loaded)
  REQUIREMENTS.md         # Scoped v1/v2 requirements with IDs
  ROADMAP.md              # Phase breakdown with status tracking
  STATE.md                # Decisions, blockers, session memory
  config.json             # Workflow configuration
  MILESTONES.md           # Completed milestone archive
  research/               # Domain research from /canopy:new-project
  todos/
    pending/              # Captured ideas awaiting work
    done/                 # Completed todos
  debug/                  # Active debug sessions
    resolved/             # Archived debug sessions
  codebase/               # Brownfield codebase mapping (from /canopy:map-codebase)
  phases/
    XX-phase-name/
      XX-YY-PLAN.md       # Atomic execution plans
      XX-YY-SUMMARY.md    # Execution outcomes and decisions
      CONTEXT.md          # Your implementation preferences
      RESEARCH.md         # Ecosystem research findings
      VERIFICATION.md     # Post-execution verification results
```
