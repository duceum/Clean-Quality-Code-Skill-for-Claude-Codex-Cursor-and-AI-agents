# Clean Code Skill for AI Coding Agents

Universal set of rules for AI coding agents that enforces production-quality coding standards across any language and project.

Works with: **Claude Code** | **OpenAI Codex** | **Cursor** | **GitHub Copilot** | **Windsurf** | **Aider** | **any AI agent that accepts system prompts or instruction files**

## What It Does

Loads a set of rules that guide AI coding agents to:

- **Plan before coding** — understand the task, plan file structure, write tests first, then implement
- **Write concise code** — use language idioms, eliminate boilerplate, remove dead code
- **Organize files by logic type** — start flat, split when different types of logic mix in one file
- **Design for testability** — pure functions, dependency injection, separate I/O from logic
- **Handle async correctly** — parallel vs sequential, partial failure handling
- **Build robust API clients** — timeouts always, retry only idempotent + transient errors, exponential backoff
- **Log meaningfully** — structured format, at boundaries only, never log secrets

## Quick Install (one command)

**Claude Code (global):**
```bash
curl -fsSL https://raw.githubusercontent.com/duceum/Clean-Quality-Code-Skill-for-Claude-Codex-Cursor-and-AI-agents/main/quality-code.md -o ~/.claude/commands/quality-code.md --create-dirs
```

**Cursor (project):**
```bash
curl -fsSL https://raw.githubusercontent.com/duceum/Clean-Quality-Code-Skill-for-Claude-Codex-Cursor-and-AI-agents/main/quality-code.md -o .cursorrules
```

**GitHub Copilot (project):**
```bash
mkdir -p .github && curl -fsSL https://raw.githubusercontent.com/duceum/Clean-Quality-Code-Skill-for-Claude-Codex-Cursor-and-AI-agents/main/quality-code.md -o .github/copilot-instructions.md
```

**Codex CLI (global):**
```bash
curl -fsSL https://raw.githubusercontent.com/duceum/Clean-Quality-Code-Skill-for-Claude-Codex-Cursor-and-AI-agents/main/quality-code.md -o ~/.codex/instructions.md --create-dirs
```

## Detailed Install

### Claude Code

```bash
# Global (all projects)
mkdir -p ~/.claude/commands
cp quality-code.md ~/.claude/commands/quality-code.md

# Per-project
mkdir -p .claude/commands
cp quality-code.md .claude/commands/quality-code.md
```

Then type `/quality-code` in any session.

### OpenAI Codex CLI

```bash
# Copy as system instructions
cp quality-code.md ~/.codex/instructions.md
```

Or pass directly:
```bash
codex --system-prompt "$(cat quality-code.md)" "your task here"
```

### Cursor

Add to project root as `.cursorrules`:
```bash
cp quality-code.md .cursorrules
```

Or add to `~/.cursor/rules/` for global use.

### GitHub Copilot

Add to project root as `.github/copilot-instructions.md`:
```bash
mkdir -p .github
cp quality-code.md .github/copilot-instructions.md
```

### Windsurf

Add to project root as `.windsurfrules`:
```bash
cp quality-code.md .windsurfrules
```

### Aider

```bash
# As a conventions file
cp quality-code.md .aider.conventions.md
```

Or pass via `--read`:
```bash
aider --read quality-code.md
```

### Any Other Agent

If the agent supports system prompts, custom instructions, or project-level config files — paste or reference the contents of `quality-code.md`. The rules are agent-agnostic and work with any LLM.

## What's Inside

| Section | Key Rules |
|---|---|
| **Method** | Plan → Test → Code → Document (6 steps) |
| **Write Short Code** | Brevity without sacrificing clarity, language idioms |
| **Architecture** | Start flat, split when logic types mix, cap ~500 lines |
| **Testability** | Pure functions first, dependency injection, I/O separation |
| **File Hygiene** | "Does this belong here?" before every addition |
| **Simplicity** | No premature abstractions, no over-engineering |
| **Async/Await** | Parallel when independent, handle partial failures |
| **API Clients** | Timeouts, idempotent retries, exponential backoff |
| **Logging** | 3 levels, structured, boundaries only, no secrets |
| **Security** | No hardcoded secrets, parameterized queries, sanitize input |

## Language Support

The rules are language-agnostic with specific defaults for:
- Python
- TypeScript / JavaScript
- Go

For any other language: existing project conventions take priority.

## Contributing

Found a rule that's missing or doesn't work well in practice? Open an issue or PR.

## License

MIT
