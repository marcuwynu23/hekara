# AGENTIC_GUIDE.md - Using SKILLS.md & AGENTS.md Across AI Coding Tools

> **Project:** Hekara - Distributed Infrastructure and Network Path Observability Platform
> **Purpose:** Guide for configuring AI coding tools to leverage `SKILLS.md` and `AGENTS.md`
> **Standard:** AGENTS.md (de facto standard, Agentic AI Foundation / Linux Foundation)
> **Compatibility:** Claude Code, OpenCode, Kiro CLI, Cursor, Gemini CLI, GitHub Copilot, and more

---

## 1. Overview: How AI Tools Read Project Instructions

### The Standard File: AGENTS.md

```text
AGENTS.md is the README for your AI coding agents —
a dedicated, predictable place to provide context and instructions.
```

AGENTS.md is read natively by 15+ tools as of 2026:

| Category | Tools |
|----------|-------|
| **Primary readers** | Claude Code, OpenCode, Cursor, Gemini CLI, Codex, Amp, Windsurf, Zed AI |
| **Also reads AGENTS.md** | GitHub Copilot, Aider, Devin, Sourcegraph Amp, Google Jules, Continue, Roo Code, Factory Droids, Amazon Q |
| **Uses custom config** | Kiro CLI (`.kiro/agents/*.json`) |
| **Legacy files still supported** | `CLAUDE.md`, `.cursorrules`, `.windsurfrules`, `.github/copilot-instructions.md` |

### Key Principle

```text
The closest AGENTS.md to the edited file wins.
Explicit user chat prompts override everything.
```

This means:
- Root `AGENTS.md` applies project-wide
- Per-directory `AGENTS.md` can override for sub-packages
- User instructions in chat take highest priority

### How SKILLS.md & AGENTS.md Work Together

```text
SKILLS.md
  └── Defines: What skills are needed (catalog, competencies, assessment)
  └── Audience: Contributors, maintainers, hiring managers
  └── Used by: Humans evaluating skill gaps, CI pipeline checks, onboarding guides

AGENTS.md
  └── Defines: How AI agents should behave (rules, conventions, commands)
  └── Audience: AI coding agents (Claude, OpenCode, Kiro, etc.)
  └── Used by: AI agents at session start for context

AGENTIC_GUIDE.md (this file)
  └── Defines: How to configure tools to read SKILLS.md & AGENTS.md
  └── Audience: Developers setting up their AI coding environment
  └── Used by: Tool configuration setup
```

---

## 2. File Placement & Structure

### Recommended Layout at Project Root

```text
hekara/
├── AGENTS.md              # Agent instructions (operational rules)
├── SKILLS.md              # Skills catalog (competency definitions)
├── AGENTIC_GUIDE.md       # This file (tool configuration guide)
├── PLAN.md                # Architecture & design decisions
├── README.md              # Project overview
├── opencode.json          # OpenCode configuration
├── .claude/               # Claude Code configuration
│   ├── CLAUDE.md          # Claude Code primary instructions
│   └── rules/             # Scoped rule files
│       ├── go.md          # Go-specific rules
│       └── testing.md     # Testing-specific rules
├── .kiro/                 # Kiro CLI configuration
│   └── agents/
│       └── hekara-agent.json  # Custom Kiro agent
├── .gemini/               # Gemini CLI configuration
│   └── settings.json
├── .cursorrules             # Cursor rules (legacy)
├── .github/
│   └── copilot-instructions.md  # GitHub Copilot instructions
└── agent/ backend/ cli/   # Sub-packages (may have their own AGENTS.md)
```

### Monorepo Pattern (If Hekara Expands)

```text
hekara/
├── AGENTS.md                  # Root: shared rules
├── SKILLS.md                  # Root: shared skills catalog
├── agent/
│   └── AGENTS.md              # Agent-specific: focuses on agent development
├── backend/
│   └── AGENTS.md              # Backend-specific: control plane rules
├── cli/
│   └── AGENTS.md              # CLI-specific: command structure rules
└── opencode.json              # Glob patterns loading per-package AGENTS.md
```

---

## 3. OpenCode Configuration

### How OpenCode Reads AGENTS.md

OpenCode follows this discovery order:

```text
1. Project root AGENTS.md         ← Primary project-level
2. ~/.config/opencode/AGENTS.md   ← Global personal rules
3. opencode.json instructions     ← Additional files/globs
4. Fallback: CLAUDE.md            ← If no AGENTS.md found
```

When both exist, **AGENTS.md takes precedence** over CLAUDE.md.

### Configuration: `opencode.json`

Create `opencode.json` at the Hekara project root:

```json
{
  "instructions": [
    "AGENTS.md",
    "SKILLS.md",
    "agent/AGENTS.md",
    "backend/AGENTS.md",
    "cli/AGENTS.md",
    "https://raw.githubusercontent.com/anomalyco/hekara/main/PLAN.md"
  ],
  "model": "claude-opus-4",
  "temperature": 0
}
```

**Explanation of each field:**

| Entry | Purpose |
|-------|---------|
| `"AGENTS.md"` | Core agent instructions — operational rules, workflow, standards |
| `"SKILLS.md"` | Skills catalog — competency requirements for contributors |
| `"agent/AGENTS.md"` | Agent-specific development rules |
| `"backend/AGENTS.md"` | Backend/control plane development rules |
| `"cli/AGENTS.md"` | CLI development rules |
| `"https://..."` | Remote URL for shared rules (fetch at session start) |

### Using Glob Patterns

```json
{
  "instructions": [
    "AGENTS.md",
    "SKILLS.md",
    "packages/*/AGENTS.md"
  ]
}
```

### Auto-Generate with `/init`

```text
In OpenCode, run:

/init

This scans the repository and generates AGENTS.md by inspecting:
- go.mod (identifies Go project)
- test configurations
- Linter/formatter configs
- CI files
- Existing CLAUDE.md or AGENTS.md
```

### Global AGENTS.md (Personal Preferences)

```bash
# Create global preferences
mkdir -p ~/.config/opencode

# Add ~/.config/opencode/AGENTS.md
# Content: personal preferences across all projects
# - Preferred code style
# - Commit message format
# - Tools to never use
```

### Verification

```text
In OpenCode session:
1. Start a new session in the hekara project directory
2. Ask: "What are the project's coding standards?"
3. It should reference AGENTS.md and SKILLS.md content
4. If it doesn't, check opencode.json instructions field
```

---

## 4. Claude Code Configuration

### How Claude Code Reads CLAUDE.md & AGENTS.md

```text
Load order:
1. ./CLAUDE.md                    ← Project root (primary)
2. ./CLAUDE.local.md              ← Personal overrides (gitignored)
3. .claude/rules/*.md             ← Scoped rule files (paths: frontmatter)
4. Walk up directory tree loading every CLAUDE.md found
5. Fallback: AGENTS.md            ← If no CLAUDE.md exists
```

When both exist, **AGENTS.md takes precedence** over CLAUDE.md in OpenCode, but Claude Code primarily uses CLAUDE.md. Best practice: keep both in sync or use AGENTS.md as the single source.

### Configuration: `.claude/CLAUDE.md`

Create `.claude/CLAUDE.md` that references SKILLS.md:

```markdown
# Hekara - Claude Code Instructions

## Project
Hekara is a distributed infrastructure and network-path observability platform.
Primary language: Go. Target domain: distributed systems, network observability, Kubernetes.
License: APACHE 2.0.

## Key Files
- `AGENTS.md` - Operational rules for AI agents
- `SKILLS.md` - Skills catalog and competency requirements
- `PLAN.md` - Architecture and design decisions (62 sections)

## Commands
- `make test` - Run all unit tests
- `make lint` - Run golangci-lint
- `make vet` - Run go vet
- `make build` - Build all binaries
- `make diagrams` - Validate mermaid diagrams
- `make coverage` - Show test coverage report
- `go test ./...` - Run Go tests
- `golangci-lint run` - Run linter

## Conventions
- Go: camelCase functions, PascalCase types, UPPER_SNAKE_CASE constants with HEKARA_ prefix
- Error handling: propagate all errors, never ignore
- No silent failures: all errors must be logged or returned
- 100% test coverage on new code
- Mermaid diagrams validated via `make diagrams`

## References
For detailed skill requirements, see: `SKILLS.md`
For agent operational rules, see: `AGENTS.md`
For architecture decisions, see: `PLAN.md`
```

### Configuration: `.claude/rules/` (Scoped Rules)

Create scoped rule files that load only when relevant files are being edited:

**`.claude/rules/go.md`:**
```markdown
---
paths:
  - "agent/**/*.go"
  - "backend/**/*.go"
  - "cli/**/*.go"
---

# Go Development Rules

## Language
- Go 1.23.x
- Use generics (1.18+) where appropriate
- Goroutines and channels for concurrency
- Context propagation for cancellation/timeouts

## Code Style
- `gofmt -w .` for formatting
- `go vet ./...` for vetting
- Named exports, no default exports in Go
- Structs for configuration, interfaces for abstraction

## Error Handling
```go
if err != nil {
    return Result{}, fmt.Errorf("operation failed: %w", err)
}
```

## Testing
- Table-driven tests
- 100% coverage on new code
- `assert.Contains(t, ...)` from testify
```

**`.claude/rules/testing.md`:**
```markdown
---
paths:
  - "**/*_test.go"
  - "internal/test/**/*"
---

# Testing Rules

## Test Types
- Unit tests: individual functions, edge cases (`go test`)
- Integration tests: end-to-end scenarios (`go test -tags=integration`)
- Property-based testing: randomized input validation
- Chaos testing: failure injection for resilience

## Quality Gates
- 100% coverage on new code
- `golangci-lint` zero new issues
- `govulncheck ./...` no vulnerabilities
- Mermaid diagram syntax validation (`make diagrams`)
```

### Import Syntax (CLAUDE.md)

Claude Code supports `@path/to/file` imports (5 levels deep):

```markdown
# CLAUDE.md

## Setup
See @SKILLS.md for competency requirements.
See @AGENTS.md for operational rules.
See @PLAN.md for architecture decisions.

## Commands
Run `make test`, `make lint`, `make build` as documented in @AGENTS.md.
```

### Size Recommendation

Keep CLAUDE.md under 200 lines. Move detailed content to referenced files (SKILLS.md, AGENTS.md). Use `@import` syntax to link to them.

### Verification

```text
In Claude Code session:
1. Start a new session in the hekara project directory
2. Ask: "What Go conventions should I follow?"
3. It should reference rules from .claude/rules/go.md or CLAUDE.md
4. If it doesn't, check .claude/CLAUDE.md and .claude/rules/ directory
```

---

## 5. Kiro CLI Configuration

### How Kiro CLI Works

Kiro CLI uses **custom agents** defined as JSON configuration files, not AGENTS.md directly.

```text
Configuration file: ~/.kiro/agents/my-agent.json
Resources: file:// paths to project files including SKILLS.md
Prompt: Instructions for the agent behavior
Model: Which AI model to use
```

### Configuration: `.kiro/agents/hekara-agent.json`

Create a custom Kiro agent for Hekara development:

```json
{
  "name": "hekara-agent",
  "description": "Hekara - Distributed Infrastructure and Network Path Observability Platform",
  "tools": ["read", "write", "bash", "edit", "grep", "glob"],
  "allowedTools": ["read", "write", "bash", "edit", "grep", "glob"],
  "resources": [
    "file://README.md",
    "file://PLAN.md",
    "file://AGENTS.md",
    "file://SKILLS.md",
    "file://AGENTIC_GUIDE.md",
    "file://agent/**/*.go",
    "file://backend/**/*.go",
    "file://cli/**/*.go",
    "file://.kiro/steering/**/*.md"
  ],
  "prompt": "You are a Hekara development assistant. Hekara is a distributed infrastructure and network-path observability platform written in Go.\n\nFollow these principles:\n- Read AGENTS.md for operational rules\n- Read SKILLS.md for required competencies\n- Follow PLAN.md architecture decisions\n- Always validate with make test, make lint, make diagrams\n- Never assume ICMP failure means network failure\n- Never invent topology - mark unknown relationships as UNKNOWN\n- 100% test coverage on new code\n- No silent failures - propagate all errors",
  "model": "claude-sonnet-4"
}
```

### Creating via Kiro CLI Command

```text
In Kiro CLI chat:

> /agent generate

✔ Enter agent name:  · hekara-agent
✔ Enter agent description:  · Hekara platform development assistant
✔ Agent scope · Local (current workspace)
Select MCP servers: (Space to toggle, Enter to confirm)
✔ markdown-downloader (node)
✔ code-analysis (uv)

✓ Agent 'hekara-agent' has been created and saved successfully!
```

### Switching Agents

```text
In Kiro CLI chat:

> /agent swap

 Choose one of the following agents
❯ hekara-agent
  kiro_default
  backend-specialist

[hekara-agent] >
```

Or via CLI:

```bash
kiro-cli --agent hekara-agent
```

### Verification

```text
1. Start Kiro CLI in the hekara project directory
2. Run: /agent swap hekara-agent
3. Ask: "What are the project's coding standards?"
4. It should reference AGENTS.md and SKILLS.md content
5. Check that the agent loaded resources from file:// paths
```

---

## 6. Cursor Configuration

### How Cursor Reads Instructions

Cursor reads:
- `.cursorrules` (legacy format, now also reads AGENTS.md)
- `AGENTS.md` (de facto standard)

### Configuration: `.cursorrules`

Create `.cursorrules` at the project root:

```text
Hekara is a distributed infrastructure and network-path observability platform written in Go.

## Project Structure
- agent/ - Lightweight agents for infrastructure discovery and measurement
- backend/ - Control plane: orchestration, topology engine, probe scheduling, correlation
- cli/ - Command-line interface for operations

## Key Files
- AGENTS.md - Agent operational rules
- SKILLS.md - Skills catalog and competency requirements
- PLAN.md - Architecture and design decisions

## Commands
- make test - Run all unit tests
- make lint - Run golangci-lint
- make build - Build all binaries
- make diagrams - Validate mermaid diagrams
- go test ./... - Run Go tests
- golangci-lint run - Run linter

## Conventions
- Go language with camelCase functions, PascalCase types
- 100% test coverage on new code
- Never ignore errors, propagate all
- No silent failures
- Validate mermaid diagrams before committing

## References
- AGENTS.md - Operational rules
- SKILLS.md - Competency requirements
```

### Verification

```text
1. Open Cursor in the hekara project directory
2. Ask: "What are the project conventions?"
3. It should reference .cursorrules and AGENTS.md content
```

---

## 7. GitHub Copilot Configuration

### How GitHub Copilot Reads Instructions

GitHub Copilot reads:
- `.github/copilot-instructions.md` (primary)
- `AGENTS.md` (also reads as fallback)

### Configuration: `.github/copilot-instructions.md`

Create `.github/copilot-instructions.md`:

```markdown
# Hekara Copilot Instructions

## Project Overview
Hekara is a distributed infrastructure and network-path observability platform.
Language: Go. Domain: distributed systems, network observability.

## Key Files
- `AGENTS.md` - Operational rules for AI agents
- `SKILLS.md` - Skills catalog and competency requirements
- `PLAN.md` - Architecture and design decisions (62 sections)
- `README.md` - Project overview and features

## Development Commands
- `make test` - Run all unit tests (100% coverage on new code)
- `make lint` - Run golangci-lint (zero new issues)
- `make build` - Build all binaries
- `make diagrams` - Validate mermaid diagrams
- `govulncheck ./...` - Security vulnerability scan

## Go Conventions
- camelCase functions, PascalCase types
- UPPER_SNAKE_CASE constants with HEKARA_ prefix
- Error handling: propagate all errors with context
- No silent failures
- Context propagation for cancellation/timeouts

## Project-Specific Rules
- Control plane ≠ measurement plane health
- ICMP failure ≠ network failure
- Never invent topology (mark UNKNOWN or INFERRED)
- VPN detection requires multiple evidence sources
- Mesh testing becomes expensive (N agents → N*(N-1) relationships)

## References
For detailed rules: `AGENTS.md`
For skills requirements: `SKILLS.md`
For architecture: `PLAN.md`
```

### Verification

```text
1. Open a file in GitHub Codespaces or VS Code with Copilot
2. Ask Copilot: "What are the testing requirements?"
3. It should reference .github/copilot-instructions.md content
```

---

## 8. Gemini CLI Configuration

### How Gemini CLI Reads Instructions

Gemini CLI reads AGENTS.md when configured:

```json
// .gemini/settings.json
{
  "context": {
    "fileName": "AGENTS.md"
  }
}
```

### Configuration: `.gemini/settings.json`

Create `.gemini/settings.json`:

```json
{
  "context": {
    "fileName": "AGENTS.md"
  },
  "model": "gemini-2.0-flash"
}
```

### Verification

```text
1. Start Gemini CLI in the hekara project directory
2. Ask: "What are the project conventions?"
3. It should reference AGENTS.md content
4. If it doesn't, check .gemini/settings.json configuration
```

---

## 9. Cross-Tool Compatibility Summary

### File Compatibility Matrix

| Tool | Primary File | Reads AGENTS.md | Reads SKILLS.md | Reads CLAUDE.md | Config File |
|------|-------------|-----------------|-----------------|-----------------|-------------|
| **OpenCode** | AGENTS.md | ✅ Native | ⚠️ Via opencode.json instructions | ✅ Fallback | `opencode.json` |
| **Claude Code** | CLAUDE.md | ✅ Fallback | ⚠️ Via @import in CLAUDE.md | ✅ Native | `.claude/CLAUDE.md` |
| **Kiro CLI** | JSON config | ⚠️ Via resources | ⚠️ Via resources | ❌ | `.kiro/agents/*.json` |
| **Cursor** | `.cursorrules` / AGENTS.md | ✅ | ⚠️ Via `.cursorrules` | ❌ | `.cursorrules` |
| **GitHub Copilot** | `.github/copilot-instructions.md` | ✅ | ⚠️ Via copilot-instructions.md | ❌ | `.github/copilot-instructions.md` |
| **Gemini CLI** | AGENTS.md | ✅ Configured | ⚠️ Via context | ❌ | `.gemini/settings.json` |
| **Codex / Amp** | AGENTS.md | ✅ | ⚠️ Via references | ❌ | — |
| **Windsurf / Zed** | AGENTS.md | ✅ | ⚠️ Via references | ❌ | — |

### What Each Symbol Means

- ✅ **Native** - Tool reads the file directly without configuration
- ⚠️ **Via reference** - Tool can read the file if explicitly referenced in configuration
- ❌ **Not supported** - Tool does not support this file format

### Best Practice: Single Source of Truth

```text
Keep AGENTS.md as the primary operational instructions.
Keep SKILLS.md as the skills catalog.
Use AGENTIC_GUIDE.md (this file) to explain how to configure each tool.
Use CLAUDE.md, .cursorrules, etc. as lightweight references that @import or reference AGENTS.md and SKILLS.md.
```

This minimizes maintenance overhead while maximizing cross-tool compatibility.

---

## 10. Hekara-Specific Setup Checklist

### Step 1: Verify Core Files Exist

```bash
# At project root (C:\Users\Mark Wayne\Desktop\Projects\hekara\)
ls -la AGENTS.md SKILLS.md AGENTIC_GUIDE.md PLAN.md README.md
```

Expected output:
```
AGENTS.md
SKILLS.md
AGENTIC_GUIDE.md
PLAN.md
README.md
```

### Step 2: Create Tool Configuration Files

```bash
# OpenCode
cat > opencode.json << 'EOF'
{
  "instructions": [
    "AGENTS.md",
    "SKILLS.md"
  ]
}
EOF

# Claude Code
mkdir -p .claude/rules
# Create .claude/CLAUDE.md (reference AGENTS.md and SKILLS.md)
# Create .claude/rules/go.md
# Create .claude/rules/testing.md

# Kiro CLI
mkdir -p .kiro/agents
# Create .kiro/agents/hekara-agent.json

# Gemini CLI
mkdir -p .gemini
# Create .gemini/settings.json

# GitHub Copilot
mkdir -p .github
# Create .github/copilot-instructions.md

# Cursor
# Create .cursorrules
```

### Step 3: Verify Each Tool Reads the Files

```text
OpenCode:
  1. Start session in hekara directory
  2. Ask: "What are the project rules?"
  3. Verify it references AGENTS.md content
  4. Ask: "What skills are needed?"
  5. Verify it references SKILLS.md content

Claude Code:
  1. Start session in hekara directory
  2. Ask: "What Go conventions should I follow?"
  3. Verify it references CLAUDE.md or .claude/rules/go.md

Kiro CLI:
  1. Start session, swap to hekara-agent
  2. Ask: "What are the coding standards?"
  3. Verify it references AGENTS.md content

Cursor:
  1. Open project in Cursor
  2. Ask: "What are the testing requirements?"
  3. Verify it references .cursorrules content

GitHub Copilot:
  1. Open file in Copilot-enabled editor
  2. Ask: "What are the quality gates?"
  3. Verify it references copilot-instructions.md

Gemini CLI:
  1. Start session in hekara directory
  2. Ask: "What are the project conventions?"
  3. Verify it references AGENTS.md content
```

### Step 4: Keep Files in Sync

When AGENTS.md or SKILLS.md change:

```text
1. Update AGENTS.md (primary operational rules)
2. Update SKILLS.md (if competencies change)
3. Update AGENTIC_GUIDE.md (if tool configurations change)
4. Update CLAUDE.md @import references if needed
5. Update .cursorrules if needed
6. Update .github/copilot-instructions.md if needed
7. Update .kiro/agents/hekara-agent.json if needed
8. Run: make diagrams  (validate mermaid diagrams)
9. Run: make test     (ensure nothing broke)
10. Commit all changes together
```

---

## 11. Quick Reference: Copy-Paste Configs

### OpenCode — `opencode.json`

```json
{
  "instructions": [
    "AGENTS.md",
    "SKILLS.md"
  ]
}
```

### Claude Code — `.claude/CLAUDE.md`

```markdown
# Hekara - Claude Code Instructions

See @AGENTS.md for operational rules.
See @SKILLS.md for competency requirements.
See @PLAN.md for architecture decisions.

## Commands
- make test - Run all unit tests
- make lint - Run golangci-lint
- make build - Build all binaries
- make diagrams - Validate mermaid diagrams

## Conventions
- Go language, 100% test coverage on new code
- Never ignore errors, propagate all
- Never invent topology (mark UNKNOWN or INFERRED)
```

### Kiro CLI — `.kiro/agents/hekara-agent.json`

```json
{
  "name": "hekara-agent",
  "description": "Hekara platform development assistant",
  "tools": ["read", "write", "bash", "edit", "grep", "glob"],
  "resources": [
    "file://AGENTS.md",
    "file://SKILLS.md",
    "file://PLAN.md"
  ],
  "prompt": "You are a Hekara development assistant. Read AGENTS.md and SKILLS.md for rules and skills.",
  "model": "claude-sonnet-4"
}
```

### Gemini CLI — `.gemini/settings.json`

```json
{
  "context": {
    "fileName": "AGENTS.md"
  }
}
```

### GitHub Copilot — `.github/copilot-instructions.md`

```markdown
# Hekara Copilot Instructions

See AGENTS.md for operational rules and SKILLS.md for competency requirements.

## Commands
- make test - Run all unit tests
- make lint - Run golangci-lint
- make build - Build all binaries
```

### Aider — `.aider.conf.yml`

```yaml
read: AGENTS.md
```

---

## 12. Common Pitfalls

### Pitfall 1: Assuming All Tools Read AGENTS.md Natively

**Problem:** Kiro CLI uses JSON configuration, not AGENTS.md directly.

**Solution:** Create `.kiro/agents/hekara-agent.json` with `resources` referencing `file://AGENTS.md`.

### Pitfall 2: Putting SKILLS.md Content in AGENTS.md

**Problem:** SKILLS.md is a skills catalog for humans and CI pipelines. AGENTS.md is for AI agents. Mixing them bloats context windows.

**Solution:** Keep them separate. Reference SKILLS.md from AGENTS.md when needed. Use `@import` in CLAUDE.md to link them.

### Pitfall 3: Forgetting to Update Tool Configs

**Problem:** AGENTS.md changes but CLAUDE.md, .cursorrules, etc. stay stale.

**Solution:** Update all config files together when AGENTS.md changes. Use the checklist in Section 10.

### Pitfall 4: Over-documenting in AGENTS.md

**Problem:** 300+ line AGENTS.md reduces adherence. Every line costs context tokens.

**Solution:** Keep AGENTS.md under 200 lines. Reference SKILLS.md and PLAN.md for detailed content.

### Pitfall 5: Ignoring Tool-Specific Features

**Problem:** Not using Claude Code's `@import` or Kiro CLI's `resources` field.

**Solution:** Configure each tool according to its native format (see Section 11).

### Pitfall 6: Not Validating Mermaid Diagrams in CI

**Problem:** Broken mermaid diagrams cause errors when AI tools try to parse them.

**Solution:** Add `make diagrams` to CI pipeline. Run `npx @mermaid-js/lint` on all markdown files containing mermaid blocks.

---

## 13. Token Efficiency Guide

### Minimize Context Usage

| Strategy | Benefit |
|----------|---------|
| Keep AGENTS.md under 200 lines | Reduces tokens consumed per session |
| Use `@import` to reference SKILLS.md | Loads only when needed |
| Use `.claude/rules/` with `paths:` frontmatter | Loads rules only for matching files |
| Keep SKILLS.md separate | Human-readable reference, not loaded into every agent context |
| Use `make diagrams` validation | Prevents broken diagrams from consuming context |
| Reference PLAN.md instead of embedding | Deep architecture knowledge on-demand |

### What to Include in AGENTS.md

- Build/test/lint commands
- Coding conventions
- Critical rules (e.g., "Never invent topology")
- Reference pointers to SKILLS.md and PLAN.md

### What NOT to Include in AGENTS.md

- Full API documentation (agents can read source code)
- Linter configuration details (agents read .eslintrc, biome.json)
- Frequently changing data (API keys, version numbers)
- Things agents already know (language syntax, general best practices)

---

*This guide is maintained as code alongside the project. Updates follow the AGENTS.md review process.*

*For questions about tool configuration, open a discussion in [GitHub Discussions](https://github.com/marcuwynu23/hekara/discussions) or consult 
AGENTIC_GUIDE.md.*
