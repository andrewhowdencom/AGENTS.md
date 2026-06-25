# Agent Skills Library

This repository contains a collection of **Agent Skills** designed to guide AI software engineers (agents) through a structured, documentation-driven development process.

## Using These Skills

You can use the `skr` CLI to install these skills into your own projects.

### 1. Install from Registry (Recommended)
These skills are automatically published to GitHub Container Registry.

```bash
# General syntax
skr install ghcr.io/andrewhowdencom/<skill>

# Examples:
skr install ghcr.io/andrewhowdencom/git:latest
skr install ghcr.io/andrewhowdencom/go:latest
skr install ghcr.io/andrewhowdencom/ci:latest
```

### 2. Install from Source
If you are modifying these skills or working offline:

```bash
# 1. Build the skill locally
cd skills/go
skr build . --tag go:local

# 2. Install into your project
cd /path/to/your/project
skr install go:local
```

## Available Skills

| Skill | Description | Registry Ref |
| :--- | :--- | :--- |
| **[agent-validation](./skills/agent-validation/SKILL.md)** | Validate designs against use cases | `ghcr.io/andrewhowdencom/agent-validation` |
| **[architecture](./skills/architecture/SKILL.md)** | System design patterns | `ghcr.io/andrewhowdencom/architecture` |
| **[ci](./skills/ci/SKILL.md)** | CI/CD pipelines | `ghcr.io/andrewhowdencom/ci` |
| **[cli](./skills/cli/SKILL.md)** | CLI development | `ghcr.io/andrewhowdencom/cli` |
| **[configuration](./skills/configuration/SKILL.md)** | Config management | `ghcr.io/andrewhowdencom/configuration` |
| **[dependency-injection](./skills/dependency-injection/SKILL.md)** | DI patterns | `ghcr.io/andrewhowdencom/dependency-injection` |
| **[documentation](./skills/documentation/SKILL.md)** | Documentation standards | `ghcr.io/andrewhowdencom/documentation` |
| **[engram](./skills/engram/SKILL.md)** | Memory store interaction via MCP | `ghcr.io/andrewhowdencom/engram` |
| **[extensibility](./skills/extensibility/SKILL.md)** | Dual-use CLI/library architecture | `ghcr.io/andrewhowdencom/extensibility` |
| **[git](./skills/git/SKILL.md)** | Git workflow & standards | `ghcr.io/andrewhowdencom/git` |
| **[go](./skills/go/SKILL.md)** | Go language standards | `ghcr.io/andrewhowdencom/go` |
| **[hitl](./skills/hitl/SKILL.md)** | CEL permission sandbox config | `ghcr.io/andrewhowdencom/hitl` |
| **[instrumentation](./skills/instrumentation/SKILL.md)** | Metrics & Tracing | `ghcr.io/andrewhowdencom/instrumentation` |
| **[mcp-cli](./skills/mcp-cli/SKILL.md)** | MCP server CLI interface | `ghcr.io/andrewhowdencom/mcp-cli` |
| **[python](./skills/python/SKILL.md)** | Python development standards | `ghcr.io/andrewhowdencom/python` |
| **[rpc](./skills/rpc/SKILL.md)** | RPC/Proto standards | `ghcr.io/andrewhowdencom/rpc` |
| **[taskfile](./skills/taskfile/SKILL.md)** | Taskfile patterns | `ghcr.io/andrewhowdencom/taskfile` |
| **[tools](./skills/tools/SKILL.md)** | Tooling configuration | `ghcr.io/andrewhowdencom/tools` |
| **[ui](./skills/ui/SKILL.md)** | Dux UI design principles | `ghcr.io/andrewhowdencom/ui` |
| **[worktree](./skills/worktree/SKILL.md)** | Git worktree task isolation | `ghcr.io/andrewhowdencom/worktree` |
| **[writing-skills](./skills/writing-skills/SKILL.md)** | Create, review, and improve agent skills | `ghcr.io/andrewhowdencom/writing-skills` |

## Philosophy: Inlined Expertise

All skills in this repository follow the **Inlined Expertise** philosophy. The `SKILL.md` file contains the most critical, actionable "Standard Operating Procedures" (SOPs) directly, reducing the need for agents to fetch external references for common tasks.
