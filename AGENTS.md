# OpenCode — Multi-Agent Architecture Configuration

A role-based multi-agent system for AI-powered software development. One
orchestrator coordinates many specialists — each with a specific role, model,
and tool set — to understand your project and make precise, safe changes.

## Project Structure
.
├── AGENTS.md # This file — instructions for AI agents
├── CLAUDE.md # Claude-specific instructions (deprecated by AGENTS.md)
├── MULTI_AGENT_ARCHITECTURE.md # Full architecture reference & model routing
│
├── agents/ # Primary agent definitions (17 agents)
│ ├── orchestrator.md # Main orchestrator (DEFAULT mode)
│ ├── orchestrator-max.md # Deep context (MAX mode)
│ ├── orchestrator-plan.md # Plan-only (PLAN mode)
│ ├── orchestrator-fast.md # Fast/lightweight (FAST mode)
│ ├── planner.md # Standalone planning agent
│ ├── file-picker.md # Fuzzy file discovery
│ ├── code-searcher.md # Mechanical ripgrep search
│ ├── researcher-web.md # Web research
│ ├── researcher-docs.md # Documentation research
│ ├── thinker.md # Deep reasoning (no tools)
│ ├── editor.md # Code implementation
│ ├── code-reviewer.md # Code review (standard/fast/multi-perspective)
│ ├── basher.md # Terminal command runner
│ ├── tmux-cli.md # Interactive CLI testing
│ ├── browser-use.md # Browser automation
│ ├── general-agent.md # General purpose
│ ├── librarian.md # Repository explorer
│ ├── security-reviewer.md # Security review
│ ├── debugger.md # Debugging specialist
│ ├── refactorer.md # Safe refactoring
│ ├── test-runner.md # Test execution
│ └── context-pruner.md # Context management (hidden)
│
├── testing-agents/ # Experimental/testing agents
│ ├── architect.md # Spec-driven architect
│ ├── tech-lead.md # Orchestrator for specialists
│ ├── developer.md # Implementation agent
│ ├── code-reviewer.md # Review agent
│ ├── debugger.md # Debugging agent
│ ├── refactorer.md # Refactoring agent
│ ├── test-engineer.md # Test engineering
│ ├── docs-writer.md # Documentation writer
│ └── security-auditor.md # Security audit
│
├── guidelines/ # Coding behavior guidelines
│ └── AGENTS.md # General-purpose coding rules
│
├── skills/ # Skill definitions (GitNexus integration)
│ ├── gitnexus-cli/
│ ├── gitnexus-debugging/
│ ├── gitnexus-exploring/
│ ├── gitnexus-guide/
│ ├── gitnexus-impact-analysis/
│ ├── gitnexus-pr-review/
│ └── gitnexus-refactoring/
│
└── .claude/skills/gitnexus/ # Claude-specific GitNexus skills

text

## Architecture Philosophy

Instead of using one model for everything, work is delegated to specialized
agents — each optimized for their role. This keeps the main conversation
context clean and allows each agent to use the best model for its task.

### Agent Hierarchy
┌─────────────────────────┐
│ Orchestrator / │
│ Build Agent │
│ (primary, full tools) │
└───────────┬─────────────┘
│
┌────────┼────────┐
│ │ │
▼ ▼ ▼
Planners/ Context Researchers
Thinkers Gatherers
│ │ │
▼ ▼ ▼
Editors/ Validators/ Testers
Implementors Reviewers

text

## Agent Categories

### Analysis-Focused (Read-Only, No Edits)

| Agent | Tools | Model | Temp | Purpose |
|-------|-------|-------|------|---------|
| `file-picker` | list, glob, read, grep | `deepseek-v4-flash` | 0.1 | Fuzzy file discovery |
| `code-searcher` | grep | `minimax-m2.5` | 0 | Mechanical code search |
| `researcher-web` | websearch, webfetch | `qwen3.6-plus` | 0.3 | Web research |
| `researcher-docs` | webfetch | `minimax-m2.5` | 0.2 | Documentation research |
| `thinker` | NONE | `mimo-v2.5-pro` | 0.1 | Deep reasoning |
| `librarian` | bash | `minimax-m2.5` | 0.1 | Clone & explore repos |

### Review-Focused (Read-Only Text Feedback)

| Agent | Tools | Model | Temp | Purpose |
|-------|-------|-------|------|---------|
| `code-reviewer` | read-only | `minimax-m2.7` | 0.1 | Code review |
| `security-reviewer` | read-only | `deepseek-v4-pro` | 0 | Security review |

### Execution-Focused (Can Edit)

| Agent | Tools | Model | Temp | Purpose |
|-------|-------|-------|------|---------|
| `orchestrator` (DEFAULT) | All | `kimi-k2.6` | 0.1 | Primary orchestrator |
| `orchestrator` (MAX) | All | `mimo-v2.5-pro` | 0.1 | Deep context |
| `orchestrator` (PLAN) | All (no edit/bash) | `deepseek-v4-pro` | 0.2 | Plan-only |
| `orchestrator` (FAST) | All (limited) | `deepseek-v4-flash` | 0.2 | Fast mode |
| `planner` | All (no edit/bash) | `deepseek-v4-pro` | 0.2 | Standalone planning |
| `editor` | write_file, str_replace | `qwen3.6-plus` | 0 | Implementation |
| `refactorer` | edit, bash, read, task | `glm-5.1` | 0 | Safe refactoring |
| `general-agent` | Full tool access | `kimi-k2.6` | 0.1 | General purpose |
| `debugger` | bash, read, grep | `kimi-k2.6` | 0.1 | Debugging |

### Tool-Oriented (Specialized)

| Agent | Tools | Model | Temp | Purpose |
|-------|-------|-------|------|---------|
| `basher` | bash | `deepseek-v4-flash` | 0 | Run commands |
| `test-runner` | bash, read, grep | `minimax-m2.5` | 0.1 | Test management |
| `tmux-cli` | bash, read | `minimax-m2.5` | 0.1 | Interactive CLI testing |
| `browser-use` | bash, webfetch | `glm-5.1` | 0.2 | Browser testing |
| `context-pruner` | read-only | `minimax-m2.7` | 0.1 | Context management (hidden) |

## Workflow Sequences

### Standard Implementation Flow
User Request
→ [Orchestrator] Gather context (file-picker, code-searcher, researcher)
→ [Orchestrator] Think/Plan (thinker for complex tasks)
→ [Orchestrator] Deploy implementation (editor or str_replace directly)
→ [Validator] Typecheck + Test + Review (in parallel)
→ [Orchestrator] Fix issues from review
→ [Validator] Re-validate
→ [Orchestrator] Summarize + Suggest followups

text

### MAX Mode Flow (Best-of-N via Editor Strategy)
User Request
→ [Orchestrator] Deep context gathering
→ [Orchestrator] Deploy editor with strategic focus
→ [Editor] Implements with strategy (minimal/clean/robust/extensible/performance)
→ [Validator] Typecheck + Test + Review (multi-perspective)
→ [Orchestrator] Fix issues
→ [Orchestrator] Summarize

text

### Plan-Only Flow
User Request
→ [Orchestrator] Gather context
→ [Orchestrator] Ask clarifying questions
→ [Orchestrator] Generate spec/plan
→ [Orchestrator] Present plan to user
→ [User] Approve/Modify plan
→ [Orchestrator] Implement (follows standard flow)

text

### Debugging Flow
Bug Report
→ [Orchestrator] Gather context (file-picker + code-searcher)
→ [Orchestrator] Spawn debugger agent
→ [Debugger] Reproduce bug (basher/test-runner)
→ [Debugger] Isolate root cause (code-searcher)
→ [Debugger] Hypothesize fix
→ [Debugger] Return root cause + fix strategy
→ [Orchestrator] Implement fix (editor)
→ [Validator] Test fix + Review

text

## Delegation Rules

### When to Delegate

- **File discovery** → Always use `file-picker` or `code-searcher`
- **Web research** → Always use `researcher-web` for external APIs/libraries
- **Documentation** → Always use `researcher-docs` for framework docs
- **Complex reasoning** → Use `thinker` for architectural decisions
- **Code changes** → Use `editor` for non-trivial implementations (3+ edit sites)
- **Validation** → Always review + test after changes
- **Interactive testing** → Use `tmux-cli` for CLI apps, `browser-use` for web

### When NOT to Delegate

- Simple edits (1-2 lines, obvious) → Do directly with `str_replace`
- Answering questions → Answer directly from context
- Quick file reads → Use `read_files` directly
- Directory listings → Use `list_directory` or `glob` directly

### Responsibility Boundaries

- Never have an agent edit files it was not designed to edit
- Never use write/str_replace in read-only agents
- Never spawn agents inside agents that lack `spawn_agents` tool
- Never run destructive commands without user confirmation

## Coding Guidelines

From `/guidelines/AGENTS.md` — these apply to all code changes:

1. **Think Before Coding** — State assumptions, surface tradeoffs, ask when unclear
2. **Simplicity First** — Minimum code that solves the problem; no speculative features
3. **Surgical Changes** — Touch only what you must; match existing style
4. **Goal-Driven Execution** — Define verifiable success criteria; loop until verified

## Model Routing Strategy

### Model Assignments by Role

| Role | Model | Rationale |
|------|-------|-----------|
| Orchestration (DEFAULT) | Kimi K2.6 | Best all-around: SWE-bench 80.2%, agent swarm |
| Orchestration (MAX) | MiMo-V2.5-Pro | 1M context, agent-optimized |
| Orchestration (PLAN) | DeepSeek V4 Pro | Strong analytical reasoning |
| Orchestration (FAST) | DeepSeek V4 Flash | Fastest, cost-effective |
| Deep reasoning | MiMo-V2.5-Pro | Best reasoning, 1M context |
| Code editing | Qwen3.6 Plus | Strong instruction following |
| Code review | MiniMax M2.7 | Stable analytical review |
| Security review | DeepSeek V4 Pro | Strong vulnerability detection |
| Debugging | Kimi K2.6 | Agent swarm for systematic debugging |
| Refactoring | GLM-5.1 | Structured code transformations |
| File discovery | DeepSeek V4 Flash | Fast, cheap for high volume |
| Code search | MiniMax M2.5 | Fast mechanical search |
| Web research | Qwen3.6 Plus | 1M context, hybrid thinking |
| Docs reading | MiniMax M2.5 | Fast documentation reading |
| Bash execution | DeepSeek V4 Flash | Fastest command execution |

### Temperature Guidelines

- **Temperature 0** — Code editing, refactoring, bash, security review, code search
- **Temperature 0.1** — Orchestration, thinking, debugging, review, testing
- **Temperature 0.2** — Planning, fast mode, docs reading, browser testing
- **Temperature 0.3** — Web research (creative synthesis)

## Design Principles

### Role-Based Naming
Agent filenames describe **what** the agent does, not which model powers it.
Model selection lives in frontmatter only — change models without renaming files.

### Progressive Disclosure
- `AGENTS.md` (this file) — Project overview and architecture
- `MULTI_AGENT_ARCHITECTURE.md` — Complete model routing table
- `guidelines/AGENTS.md` — Detailed coding behavior rules
- Individual `agents/*.md` — Per-agent configuration and instructions

## GitNexus — Code Intelligence

This project is indexed by GitNexus as **opencode** (315 symbols, 308 relationships, 0 execution flows). Use the GitNexus MCP tools to understand code, assess impact, and navigate safely.

> If any GitNexus tool warns the index is stale, run `npx gitnexus analyze` in terminal first.

### Always Do

- **MUST run impact analysis before editing any symbol.** Before modifying a function, class, or method, run `gitnexus_impact({target: "symbolName", direction: "upstream"})` and report the blast radius (direct callers, affected processes, risk level) to the user.
- **MUST run `gitnexus_detect_changes()` before committing** to verify your changes only affect expected symbols and execution flows.
- **MUST warn the user** if impact analysis returns HIGH or CRITICAL risk before proceeding with edits.
- When exploring unfamiliar code, use `gitnexus_query({query: "concept"})` to find execution flows instead of grepping. It returns process-grouped results ranked by relevance.
- When you need full context on a specific symbol — callers, callees, which execution flows it participates in — use `gitnexus_context({name: "symbolName"})`.

### Never Do

- NEVER edit a function, class, or method without first running `gitnexus_impact` on it.
- NEVER ignore HIGH or CRITICAL risk warnings from impact analysis.
- NEVER rename symbols with find-and-replace — use `gitnexus_rename` which understands the call graph.
- NEVER commit changes without running `gitnexus_detect_changes()` to check affected scope.

### Resources

| Resource | Use for |
|----------|---------|
| `gitnexus://repo/opencode/context` | Codebase overview, check index freshness |
| `gitnexus://repo/opencode/clusters` | All functional areas |
| `gitnexus://repo/opencode/processes` | All execution flows |
| `gitnexus://repo/opencode/process/{name}` | Step-by-step execution trace |

### CLI

| Task | Read this skill file |
|------|---------------------|
| Understand architecture / "How does X work?" | `.claude/skills/gitnexus/gitnexus-exploring/SKILL.md` |
| Blast radius / "What breaks if I change X?" | `.claude/skills/gitnexus/gitnexus-impact-analysis/SKILL.md` |
| Trace bugs / "Why is X failing?" | `.claude/skills/gitnexus/gitnexus-debugging/SKILL.md` |
| Rename / extract / split / refactor | `.claude/skills/gitnexus/gitnexus-refactoring/SKILL.md` |
| Tools, resources, schema reference | `.claude/skills/gitnexus/gitnexus-guide/SKILL.md` |
| Index, status, clean, wiki CLI commands | `.claude/skills/gitnexus/gitnexus-cli/SKILL.md` |