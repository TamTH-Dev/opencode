# Setup Guide — Multi-Agent System for OpenCode (Role-Based)

## Overview

This guide shows you how to configure the multi-agent system in OpenCode. All agents use **role-based filenames** — model selection belongs in frontmatter, not filenames. This makes it easy to swap models without renaming files.

## Quick Install

```bash
# Create the agents directory (global)
mkdir -p ~/.config/opencode/agents

# Copy all agent files
cp -r path/to/agents/* ~/.config/opencode/agents/

# Or for a single project
mkdir -p .opencode/agents
cp -r path/to/agents/* .opencode/agents/
```

## Configuration via opencode.json

### Global Config (~/.opencode.json or $XDG_CONFIG_HOME/opencode/opencode.json)

```json
{
  "$schema": "https://opencode.ai/config.json",
  "agent": {
    "orchestrator": {
      "mode": "primary",
      "model": "opencode-go/kimi-k2.6",
      "temperature": 0.1,
      "color": "primary",
      "permission": {
        "edit": "allow",
        "bash": "allow",
        "read": "allow",
        "glob": "allow",
        "grep": "allow",
        "list": "allow",
        "webfetch": "allow",
        "websearch": "allow",
        "skill": "allow",
        "task": {
          "*": "allow"
        }
      }
    },
    "orchestrator-plan": {
      "mode": "primary",
      "model": "opencode-go/deepseek-v4-pro",
      "temperature": 0.2,
      "color": "warning",
      "permission": {
        "edit": "deny",
        "bash": "deny",
        "read": "allow",
        "glob": "allow",
        "grep": "allow",
        "list": "allow",
        "webfetch": "allow",
        "websearch": "allow",
        "task": {
          "*": "allow"
        }
      }
    },
    "orchestrator-fast": {
      "mode": "primary",
      "model": "opencode-go/deepseek-v4-flash",
      "temperature": 0.2,
      "color": "success",
      "permission": {
        "edit": "allow",
        "bash": "allow",
        "read": "allow",
        "glob": "allow",
        "grep": "allow",
        "list": "allow",
        "webfetch": "allow",
        "websearch": "allow",
        "task": {
          "file-picker": "allow",
          "code-searcher": "allow",
          "researcher-web": "allow",
          "researcher-docs": "allow",
          "basher": "allow"
        }
      }
    }
  }
}
```

### Restricting Task Access (Security)

To restrict which subagents the orchestrator can spawn:

```json
{
  "agent": {
    "orchestrator": {
      "permission": {
        "task": {
          "*": "deny",
          "file-picker": "allow",
          "code-searcher": "allow",
          "editor": "allow",
          "code-reviewer": "allow",
          "basher": "allow",
          "thinker": "allow",
          "researcher-web": "allow",
          "researcher-docs": "allow"
        }
      }
    }
  }
}
```

### Model Overrides (OpenCode Go Plan Compatible)

Override models for specific agents to balance cost vs. quality. All models below are available on the OpenCode Go plan:

```json
{
  "agent": {
    "thinker": {
      "model": "opencode-go/mimo-v2.5-pro",
      "temperature": 0.1
    },
    "editor": {
      "model": "opencode-go/qwen3.6-plus",
      "temperature": 0
    },
    "code-reviewer": {
      "model": "opencode-go/minimax-m2.7",
      "temperature": 0.1
    },
    "file-picker": {
      "model": "opencode-go/deepseek-v4-flash",
      "temperature": 0.1
    },
    "basher": {
      "model": "opencode-go/deepseek-v4-flash",
      "temperature": 0
    }
  }
}
```

## Multi-Mode Configuration

Configure all four modes as primary agents that you can cycle through with Tab:

```json
{
  "agent": {
    "orchestrator": {
      "mode": "primary",
      "model": "opencode-go/kimi-k2.6",
      "temperature": 0.1,
      "permission": {
        "edit": "allow",
        "bash": "allow",
        "task": { "*": "allow" }
      }
    },
    "orchestrator-max": {
      "mode": "primary",
      "model": "opencode-go/mimo-v2.5-pro",
      "temperature": 0.1,
      "permission": {
        "edit": "allow",
        "bash": "allow",
        "task": { "*": "allow" }
      }
    },
    "orchestrator-fast": {
      "mode": "primary",
      "model": "opencode-go/deepseek-v4-flash",
      "temperature": 0.2,
      "permission": {
        "edit": "allow",
        "bash": "allow",
        "task": { "*": "allow" }
      }
    },
    "orchestrator-plan": {
      "mode": "primary",
      "model": "opencode-go/deepseek-v4-pro",
      "temperature": 0.2,
      "permission": {
        "edit": "deny",
        "bash": "deny",
        "task": { "*": "allow" }
      }
    }
  }
}
```

## Per-Project Configuration

Place `.opencode/agents/` in your project root. OpenCode loads per-project agents in addition to global ones. Project agents can override global agents with the same filename.

```bash
my-project/
├── .opencode/
│   └── agents/
│       ├── orchestrator.md          # Project-specific orchestrator
│       ├── file-picker.md           # Project-specific file picker
│       └── custom-helper.md         # Custom project-only agent
├── src/
└── package.json
```

## Custom Agent Prompt Files

For large prompts, reference external files:

```json
{
  "agent": {
    "orchestrator": {
      "prompt": "{file:./prompts/orchestrator.txt}"
    }
  }
}
```

## Verifying Installation

After installation, test that agents are loaded:

1. Start OpenCode in a project
2. Type `@` to see available subagents — you should see all role-based agents listed
3. Press Tab to cycle through primary agents — you should see orchestrator, orchestrator-plan, etc.
4. Test with: `@file-picker find the main entry point file in this project`

## Available Models (OpenCode Go Plan) with Provider Prefixes

Only these models are available for assignment in `provider/model-name` format:

| Model ID | Best For | Key Strength |
|----------|----------|--------------|
| `opencode-go/glm-5.1` | Refactoring, browser automation | Autonomous engineering loops, 200K context |
| `moonshot/kimi-k2.5` | Lightweight tasks | Agent swarm (100 sub-agents) |
| `opencode-go/kimi-k2.6` | Orchestration, editing, debugging, general | SWE-bench Verified 80.2%, 1T MoE (32B active), 256K context |
| `xiaomi/mimo-v2.5` | Lightweight agentic tasks | 1M context, good efficiency |
| `opencode-go/mimo-v2.5-pro` | Deep reasoning, MAX mode, thinking | 1.02T MoE (42B active), 1M context, token-efficient |
| `opencode-go/minimax-m2.5` | Code search, test running, docs, CLI, librarian | SWE-bench 80.2%, extremely cost-effective, fast |
| `opencode-go/minimax-m2.7` | Code review, context pruning | Agent-native, deeper reasoning, SWE-bench ~78% |
| `qwen/qwen3.5-plus` | Lightweight research | Baseline performance |
| `opencode-go/qwen3.6-plus` | Web research | 1M context, hybrid thinking, cost-effective |
| `opencode-go/deepseek-v4-pro` | Planning, security review, PLAN mode | 1.6T MoE (49B active), strongest analytical reasoning |
| `opencode-go/deepseek-v4-flash` | FAST mode, file picking, bash, fast operations | 284B MoE (13B active), fastest (100+ tokens/sec), cheapest |

## Cost-Optimized Configuration

For a budget-conscious setup:

```json
{
  "agent": {
    "orchestrator": {
      "model": "opencode-go/deepseek-v4-flash",
      "temperature": 0.2
    },
    "orchestrator-max": {
      "model": "opencode-go/minimax-m2.5",
      "temperature": 0.1
    },
    "file-picker": {
      "model": "opencode-go/deepseek-v4-flash",
      "temperature": 0.1
    },
    "basher": {
      "model": "opencode-go/deepseek-v4-flash",
      "temperature": 0
    },
    "code-reviewer": {
      "model": "opencode-go/minimax-m2.5",
      "temperature": 0.1
    },
    "editor": {
      "model": "moonshot/kimi-k2.5",
      "temperature": 0
    }
  }
}
```

## Troubleshooting

**Q: Agents don't appear in @ menu**
- Check that files are in the correct directory (`~/.config/opencode/agents/` or `.opencode/agents/`)
- Ensure each file has valid YAML frontmatter (between `---` delimiters)
- Restart OpenCode after adding files

**Q: "permission" errors**
- Ensure `permission` values use the correct format: `"allow"`, `"deny"`, or `"ask"`
- For task permissions: `"task": { "agent-name": "allow" }`

**Q: Agent can't access required tools**
- Check the agent's `permission` block — tools not listed as `"allow"` are denied by default

**Q: Subagent returns "no tools available"**
- Some agents (thinker, code-reviewer, security-reviewer) intentionally have no tools — they provide text-only feedback

**Q: Model not found / unsupported model error**
- Only use models available on the OpenCode Go plan (see table above)
- Available model prefixes: `deepseek/`, `minimax/`, `moonshot/`, `xiaomi/`, `zhipu/`, `qwen/`
- NOT available: Anthropic Claude, OpenAI GPT, Google Gemini, Gemma models, `ollama-cloud/` prefix
