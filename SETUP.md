# Setup Guide — OpenCode Configuration

## Overview

This guide describes how to configure the entire OpenCode environment, including the multi-agent system. OpenCode uses a combination of JSON configuration files and role-based agent definitions.

## 1. Global Configuration Setup

OpenCode is configured via `opencode.json`. You can place this file in two locations:
- **Global:** `~/.config/opencode/opencode.json` (or `$XDG_CONFIG_HOME/opencode/opencode.json`)
- **Project-Specific:** `.opencode/opencode.json` in your project root.

### Basic Configuration Structure
The configuration controls orchestrator behavior, agent permissions, model selection, and system-wide settings.

```json
{
  "$schema": "https://opencode.ai/config.json",
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
    }
  }
}
```

## 2. Multi-Agent System Setup

OpenCode utilizes a role-based agent system. All agents use **role-based filenames** — model selection belongs in frontmatter, not filenames.

### Installing Agent Definitions
Agent definition files (Markdown files with YAML frontmatter) must be placed in specific directories to be recognized.

#### Global Installation (All Projects)
```bash
# Create the agents directory
mkdir -p ~/.config/opencode/agents

# Copy all agent files
cp -r path/to/agents/* ~/.config/opencode/agents/
```

#### Project-Specific Installation
Place `.opencode/agents/` in your project root. Project agents are loaded in addition to global ones and can override global agents with the same filename.
```bash
mkdir -p .opencode/agents
cp -r path/to/agents/* .opencode/agents/
```

### Project Structure Example
```bash
my-project/
├── .opencode/
│   ├── opencode.json         # Project-specific config
│   └── agents/
│       ├── orchestrator.md    # Project-specific orchestrator
│       └── custom-helper.md    # Custom project-only agent
├── src/
└── package.json
```

## 3. Advanced Configuration

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
          "editor": "allow"
        }
      }
    }
  }
}
```

### Model Overrides
Balance cost vs. quality by overriding models for specific agents:
```json
{
  "agent": {
    "thinker": { "model": "opencode-go/mimo-v2.5-pro", "temperature": 0.1 },
    "editor": { "model": "opencode-go/qwen3.6-plus", "temperature": 0 }
  }
}
```

### Multi-Mode Configuration
Configure multiple primary agents to cycle through with Tab:
```json
{
  "agent": {
    "orchestrator": { "mode": "primary", "model": "opencode-go/kimi-k2.6" },
    "orchestrator-max": { "mode": "primary", "model": "opencode-go/mimo-v2.5-pro" },
    "orchestrator-fast": { "mode": "primary", "model": "opencode-go/deepseek-v4-flash" },
    "orchestrator-plan": { "mode": "primary", "model": "opencode-go/deepseek-v4-pro" }
  }
}
```

## 4. Custom Prompt Files
For large prompts, reference external files in `opencode.json`:
```json
{
  "agent": {
    "orchestrator": { "prompt": "{file:./prompts/orchestrator.txt}" }
  }
}
```

## 5. Verifying Installation

1. Start OpenCode in a project.
2. Type `@` to see available subagents.
3. Press Tab to cycle through primary agents.
4. Test with: `@file-picker find the main entry point file in this project`.

## 6. Available Models (OpenCode Go Plan)

| Model ID | Best For | Key Strength |
|----------|----------|--------------|
| `opencode-go/glm-5.1` | Refactoring, browser automation | Autonomous engineering loops, 200K context |
| `moonshot/kimi-k2.5` | Lightweight tasks | Agent swarm (100 sub-agents) |
| `opencode-go/kimi-k2.6` | Orchestration, editing, debugging, general | SWE-bench Verified 80.2%, 256K context |
| `xiaomi/mimo-v2.5` | Lightweight agentic tasks | 1M context, good efficiency |
| `opencode-go/mimo-v2.5-pro` | Deep reasoning, MAX mode, thinking | 1.02T MoE, 1M context |
| `opencode-go/minimax-m2.5` | Code search, test running, docs, CLI | SWE-bench 80.2%, cost-effective |
| `opencode-go/minimax-m2.7` | Code review, context pruning | Agent-native, deeper reasoning |
| `qwen/qwen3.5-plus` | Lightweight research | Baseline performance |
| `opencode-go/qwen3.6-plus` | Web research | 1M context, hybrid thinking |
| `opencode-go/deepseek-v4-pro` | Planning, security review, PLAN mode | Strongest analytical reasoning |
| `opencode-go/deepseek-v4-flash` | FAST mode, file picking, bash | Fastest, cheapest |

## 7. Cost-Optimized Configuration
```json
{
  "agent": {
    "orchestrator": { "model": "opencode-go/deepseek-v4-flash", "temperature": 0.2 },
    "file-picker": { "model": "opencode-go/deepseek-v4-flash", "temperature": 0.1 },
    "basher": { "model": "opencode-go/deepseek-v4-flash", "temperature": 0 },
    "code-reviewer": { "model": "opencode-go/minimax-m2.5", "temperature": 0.1 },
    "editor": { "model": "moonshot/kimi-k2.5", "temperature": 0 }
  }
}
```

## Troubleshooting

**Q: Agents don't appear in @ menu**
- Check directories: `~/.config/opencode/agents/` or `.opencode/agents/`.
- Ensure valid YAML frontmatter.
- Restart OpenCode.

**Q: "permission" errors**
- Use `"allow"`, `"deny"`, or `"ask"`.
- Task format: `"task": { "agent-name": "allow" }`.

**Q: Model not found / unsupported model error**
- Only use models on the OpenCode Go plan.
- Valid prefixes: `opencode-go`.
- NOT available: Anthropic, OpenAI, Google, Gemma, `ollama-cloud/`, `deepseek/`, `minimax/`, `moonshot/`, `xiaomi/`, `zhipu/`, `qwen/`.
