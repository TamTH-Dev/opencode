# Multi-Agent Architecture (Role-Based)

## Philosophy

This multi-agent architecture: **one orchestrator, many specialists**. Instead of using one model for everything, we coordinate specialized agents that work together to understand your project and make precise changes. Each agent has a **role-based name** — model assignment belongs exclusively in frontmatter, not filenames.

## Agent Hierarchy

```
                    ┌─────────────────────────┐
                    │     Orchestrator /       │
                    │     Build Agent          │
                    │   (primary, full tools)  │
                    └───────────┬─────────────┘
                                │
          ┌─────────────────────┼─────────────────────┐
          │                     │                     │
          ▼                     ▼                     ▼
   ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
   │  Planners/   │    │  Context     │    │  Researchers │
   │  Thinkers    │    │  Gatherers   │    │              │
   └──────┬───────┘    └──────┬───────┘    └──────┬───────┘
          │                   │                   │
   ┌──────┴───────┐    ┌──────┴───────┐    ┌──────┴───────┐
   │ • thinker    │    │ • file-picker│    │ • researcher- │
   │ • planner    │    │ • code-      │    │   web         │
   │ • debugger   │    │   searcher   │    │ • researcher- │
   │              │    │ • librarian  │    │   docs        │
   └──────────────┘    └──────────────┘    └──────────────┘

          ┌─────────────────────┬─────────────────────┐
          │                     │                     │
          ▼                     ▼                     ▼
   ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
   │  Editors/    │    │  Validators  │    │  Testers     │
   │  Implementors│    │  / Reviewers │    │              │
   └──────┬───────┘    └──────┬───────┘    └──────┬───────┘
          │                   │                   │
   ┌──────┴───────┐    ┌──────┴───────┐    ┌──────┴───────┐
   │ • editor     │    │ • code-      │    │ • basher     │
   │ • refactorer │    │   reviewer   │    │ • tmux-cli   │
   │ • general-   │    │ • security-  │    │ • test-      │
   │   agent      │    │   reviewer   │    │   runner     │
   │              │    │              │    │ • browser-   │
   │              │    │              │    │   use        │
   └──────────────┘    └──────────────┘    └──────────────┘
```

## Agent Categories

### Analysis-Focused (Read-Only, No Edits)
| Agent | Tools | Model | Temperature | Purpose |
|-------|-------|-------|-------------|---------|
| file-picker | list, glob, read, grep | `opencode-go/deepseek-v4-flash` | 0.1 | Fuzzy file discovery |
| code-searcher | grep | `opencode-go/minimax-m2.5` | 0 | Mechanical code search |
| researcher-web | websearch, webfetch | `opencode-go/qwen3.6-plus` | 0.3 | Web research |
| researcher-docs | webfetch | `opencode-go/minimax-m2.5` | 0.2 | Documentation research |
| thinker | NONE | `opencode-go/mimo-v2.5-pro` | 0.1 | Deep reasoning |
| librarian | bash | `opencode-go/minimax-m2.5` | 0.1 | Clone & explore repos |

### Review-Focused (Read-Only Text Feedback)
| Agent | Tools | Model | Temperature | Purpose |
|-------|-------|-------|-------------|---------|
| code-reviewer | read-only | `opencode-go/minimax-m2.7` | 0.1 | Code review (standard/fast/multi-perspective) |
| security-reviewer | read-only | `opencode-go/deepseek-v4-pro` | 0 | Security review |

### Execution-Focused (Can Edit)
| Agent | Tools | Model | Temperature | Purpose |
|-------|-------|-------|-------------|---------|
| orchestrator (DEFAULT) | All | `opencode-go/kimi-k2.6` | 0.1 | Primary orchestrator |
| orchestrator (MAX) | All | `opencode-go/mimo-v2.5-pro` | 0.1 | Deep context orchestrator |
| orchestrator (PLAN) | All (no edit/bash) | `opencode-go/deepseek-v4-pro` | 0.2 | Plan-only orchestrator |
| orchestrator (FAST) | All (limited) | `opencode-go/deepseek-v4-flash` | 0.2 | Fast orchestrator |
| planner | All (no edit/bash) | `opencode-go/deepseek-v4-pro` | 0.2 | Standalone planning |
| editor | write_file, str_replace | `opencode-go/kimi-k2.6` | 0 | Implements changes |
| refactorer | edit, bash, read, task | `opencode-go/glm-5.1` | 0 | Safe refactoring |
| general-agent | Full tool access | `opencode-go/kimi-k2.6` | 0.1 | General purpose |
| debugger | bash, read, grep | `opencode-go/kimi-k2.6` | 0.1 | Debugging |

### Tool-Oriented (Specialized)
| Agent | Tools | Model | Temperature | Purpose |
|-------|-------|-------|-------------|---------|
| basher | bash | `opencode-go/deepseek-v4-flash` | 0 | Run commands |
| test-runner | bash, read, grep | `opencode-go/minimax-m2.5` | 0.1 | Test management |
| tmux-cli | bash, read | `opencode-go/minimax-m2.5` | 0.1 | Interactive CLI testing |
| browser-use | bash, webfetch | `opencode-go/glm-5.1` | 0.2 | Browser testing |
| context-pruner | read-only | `opencode-go/minimax-m2.7` | 0.1 | Context management (hidden) |

## Delegation Rules

### When to Delegate
1. **File discovery** → Always use file-picker or code-searcher instead of guessing file paths
2. **Web research** → Always use researcher-web for external APIs, libraries, best practices
3. **Documentation** → Always use researcher-docs for library/framework documentation
4. **Complex reasoning** → Use thinker for architectural decisions, algorithm design
5. **Code changes** → Use editor for non-trivial implementations (3+ edit sites)
6. **Validation** → Always review + test after changes
7. **Interactive testing** → Use tmux-cli for CLI apps, browser-use for web apps

### When NOT to Delegate
1. **Simple edits** (1-2 lines, obvious) → Do directly with str_replace
2. **Answering questions** → Answer directly from context
3. **Quick file reads** → Use read_files directly
4. **Directory listings** → Use list_directory or glob directly

### Responsibility Boundaries
- **Never** have an agent edit files it was not designed to edit
- **Never** use write/str_replace in a read-only agent (file-picker, code-searcher, researcher, thinker, reviewer)
- **Never** spawn agents inside agents that have no spawn_agents tool
- **Never** run destructive commands (git push, npm publish, etc.) without user confirmation

## Workflow Sequences

### Standard Implementation Flow
```
User Request
  → [Orchestrator] Gather context (file-picker, code-searcher, researcher)
  → [Orchestrator] Think/Plan (thinker for complex tasks)
  → [Orchestrator] Deploy implementation (editor or str_replace directly)
  → [Validator] Typecheck + Test + Review (in parallel)
  → [Orchestrator] Fix issues from review
  → [Validator] Re-validate
  → [Orchestrator] Summarize + Suggest followups
```

### MAX Mode Flow (Best-of-N via Editor Strategy)
```
User Request
  → [Orchestrator] Deep context gathering
  → [Orchestrator] Deploy editor with strategic focus
     → [Editor] Implements with specified strategy (minimal/clean/robust/extensible/performance)
  → [Validator] Typecheck + Test + Review (multi-perspective mode)
  → [Orchestrator] Fix issues
  → [Orchestrator] Summarize
```

### Plan-Only Flow
```
User Request
  → [Orchestrator] Gather context
  → [Orchestrator] Ask clarifying questions (ask_user)
  → [Orchestrator] Generate spec/plan
  → [Orchestrator] Present plan to user
  → [User] Approve/Modify plan
  → [Orchestrator] Implement (follows standard flow)
```

### Debugging Flow
```
Bug Report
  → [Orchestrator] Gather context (file-picker + code-searcher)
  → [Orchestrator] Spawn debugger agent
     → [Debugger] Reproduce bug (basher/test-runner)
     → [Debugger] Isolate root cause (code-searcher)
     → [Debugger] Hypothesize fix
     → [Debugger] Return root cause + fix strategy
  → [Orchestrator] Implement fix (editor)
  → [Validator] Test fix + Review
```

## Model Routing Strategy (OpenCode Go Plan)

| Task Type | Model | Temperature | Rationale |
|-----------|-------|-------------|-----------|
| Orchestration (DEFAULT) | Kimi K2.6 | 0.1 | Best all-around: SWE-bench 80.2%, agent swarm (300 sub-agents, 4000 steps), 256K context. Ideal for coordinating sub-agents with strong reasoning and tool use. |
| Orchestration (MAX) | MiMo-V2.5-Pro | 0.1 | 1M context window for deep context gathering. 40-60% fewer tokens than competitors — critical for MAX mode. Agent-optimized with self-correcting loops. |
| Orchestration (PLAN) | DeepSeek V4 Pro | 0.2 | 1.6T MoE (49B active). Strong analytical reasoning for architectural analysis. Slightly higher temp for creative planning. |
| Orchestration (FAST) | DeepSeek V4 Flash | 0.2 | Fastest option (100+ tokens/sec). 284B MoE (13B active). Cost-effective for high-volume use. Good enough for quick tasks. |
| Deep reasoning | MiMo-V2.5-Pro | 0.1 | Best reasoning capability. 1M context for processing complex problem context. Agent-optimized with self-correction loops for deep analysis. |
| Code editing | Kimi K2.6 | 0 | SWE-bench Verified 80.2%. Agent swarm for multi-file coordination. Temperature 0 for deterministic output. 256K context for large edits. |
| Code review | MiniMax M2.7 | 0.1 | Stable analytical review. Agent-native design for complex review criteria. SWE-bench ~78% with strong instruction following. |
| Security review | DeepSeek V4 Pro | 0 | Strongest analytical reasoning for security vulnerability detection. Temperature 0 for maximum determinism. |
| Debugging | Kimi K2.6 | 0.1 | Agent swarm (300 sub-agents) helps systematic debugging across files. Strong long-context (256K) for tracing issues. |
| Refactoring | GLM-5.1 | 0 | Strong at structured code transformations. Autonomous engineering loops with self-correction. Temperature 0 for behavior preservation. |
| File discovery | DeepSeek V4 Flash | 0.1 | Fast file discovery. Cheap for high-volume use. |
| Code search | MiniMax M2.5 | 0 | Fast mechanical search. Cost-effective. Temperature 0 for precise results. |
| Web research | Qwen3.6 Plus | 0.3 | 1M context for processing research results. Hybrid thinking (toggleable reasoning). Higher temp for creative synthesis. |
| Docs reading | MiniMax M2.5 | 0.2 | Fast documentation reading. Cost-effective. |
| Bash execution | DeepSeek V4 Flash | 0 | Fastest command execution. Deterministic output parsing. |
| Interactive CLI | MiniMax M2.5 | 0.1 | Fast structured CLI testing workflows. |
| Browser testing | GLM-5.1 | 0.2 | Good at structured browser automation tasks. MCP integration. |
| General purpose | Kimi K2.6 | 0.1 | Versatile fallback. Agent swarm for multi-step workflows. |
| Context pruning | MiniMax M2.7 | 0.1 | Good summarization capability. Agent-native for context management. |

## Design Principles

### Role-Based Naming
- Agent filenames describe **what the agent does**, not **which model powers it**
- Model selection is in frontmatter only — change models without renaming files
- No model-specific suffixes (-gpt, -claude, -deepseek, -gemini, etc.)

### Model Assignment Guidelines
- **Thinkers/Planners** → Strongest reasoning models (MiMo-V2.5-Pro, DeepSeek V4 Pro, Kimi K2.6)
- **Code Editors** → Best coding + instruction following at temperature 0 (Kimi K2.6)
- **Reviewers** → Stable analytical models at low temperature (MiniMax M2.7, DeepSeek V4 Pro)
- **Fast/Light tasks** → Deterministic fast models (DeepSeek V4 Flash, MiniMax M2.5)
- **File Discovery** → Lightweight fast models (DeepSeek V4 Flash)
- **Research** → Models with good context and synthesis (Qwen3.6 Plus, MiniMax M2.5)

### Temperature Guidelines
- **Temperature 0** — Code editing, refactoring, bash execution, security review, code search. Maximum determinism and precision.
- **Temperature 0.1** — Orchestration, thinking, debugging, review, testing. Slight variability for nuanced analysis.
- **Temperature 0.2** — Planning, fast mode, docs reading, browser testing. More creativity for exploration/research.
- **Temperature 0.3** — Web research. Higher creativity for information synthesis and connecting ideas.

## Files

All agent files are in `~/.config/opencode/agents/` (global) or `.opencode/agents/` (per-project).

### Primary Agents (23 files total)
- `orchestrator.md` — Full orchestrator (DEFAULT mode) — Kimi K2.6
- `orchestrator-max.md` — MAX mode orchestrator — MiMo-V2.5-Pro
- `orchestrator-plan.md` — PLAN mode orchestrator — DeepSeek V4 Pro
- `orchestrator-fast.md` — FAST mode orchestrator — DeepSeek V4 Flash
- `planner.md` — Standalone planning agent — DeepSeek V4 Pro

### Subagents
- `file-picker.md` — Fuzzy file discovery — DeepSeek V4 Flash
- `code-searcher.md` — Mechanical code search (ripgrep) — MiniMax M2.5
- `researcher-web.md` — Web research — Qwen3.6 Plus
- `researcher-docs.md` — Documentation research — MiniMax M2.5
- `thinker.md` — Deep reasoning (no tools) — MiMo-V2.5-Pro
- `editor.md` — Code implementation — Qwen3.6 Plus
- `code-reviewer.md` — Code review (unified: standard/fast/multi-perspective) — MiniMax M2.7
- `basher.md` — Terminal command runner — DeepSeek V4 Flash
- `tmux-cli.md` — Interactive CLI testing — MiniMax M2.5
- `browser-use.md` — Browser automation — GLM-5.1
- `general-agent.md` — General purpose agent — Kimi K2.6
- `librarian.md` — Repository explorer — MiniMax M2.5
- `security-reviewer.md` — Security review — DeepSeek V4 Pro
- `debugger.md` — Debugging specialist — Kimi K2.6
- `refactorer.md` — Safe refactoring — GLM-5.1
- `test-runner.md` — Test execution — MiniMax M2.5
- `context-pruner.md` — Context management (hidden) — MiniMax M2.7
