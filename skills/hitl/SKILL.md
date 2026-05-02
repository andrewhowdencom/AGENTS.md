---
name: hitl
description: Guidelines for configuring the pi.hitl CEL-based permission sandbox via permissions.yaml files.
---

# pi.hitl Permission Configuration

pi.hitl is a CEL-based (Common Expression Language) permission sandbox for the pi coding agent. It intercepts every tool call the LLM attempts and evaluates it against YAML-defined rules. Rules can **allow** operations within a sandbox, **block** dangerous actions, or **confirm** sensitive operations with an interactive dialog.

This skill provides instructions for creating and editing `permissions.yaml` configuration files.

## Configuration File Locations

Config files are loaded from three locations and merged. Lower precedence configs are loaded first; higher precedence configs override and append to them.

| Location | Scope | Precedence |
|----------|-------|------------|
| `~/.agents/permissions.yaml` | Agent-wide defaults | Lowest |
| `~/.pi/agent/permissions.yaml` | Global (all projects) | Middle |
| `.pi/permissions.yaml` | Project-local | Highest |

**Merge semantics:**
- `rules` and `hidden_tools` are **concatenated** (project rules are appended after global rules).
- All other keys (`version`, `default_action`) are **overwritten** by the highest-precedence config that defines them.

## Top-Level Schema

```yaml
version: 1
default_action: block
rules:
  - ...
hidden_tools:
  - ...
```

| Key | Type | Required | Default | Description |
|-----|------|----------|---------|-------------|
| `version` | `number` | No | `1` | Config format version. Must be `1`. |
| `default_action` | `string` | No | `block` | Action applied when no rule matches. One of: `allow`, `block`, `confirm`. |
| `rules` | `Rule[]` | No | `[]` | Ordered list of rules evaluated top-to-bottom. First match wins. |
| `hidden_tools` | `string[]` | No | `[]` | Tool names that are silently blocked on every call. |

## Rule Structure

Each rule is an object with the following fields:

```yaml
- name: "Human-readable name"
  condition: 'CEL expression'
  action: allow | block | confirm
  message: "Optional message shown on block or confirm"
  rules:        # Optional child rules (mutually exclusive with action)
    - ...
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | `string` | Yes | Human-readable identifier for debugging and UI messages. |
| `condition` | `string` | Yes | CEL expression. Must evaluate to `true` for the rule to match. |
| `action` | `string` | Yes (leaf) | `allow`, `block`, or `confirm`. |
| `message` | `string` | No | Message shown when the rule blocks or confirms. |
| `rules` | `Rule[]` | No (parent) | Child rules. Mutually exclusive with `action`. Parent conditions are AND-ed with children. |

### Evaluation Order

Rules are evaluated **top-to-bottom**, **first match wins**. This is intentional and mirrors firewall/ACL semantics:
- It is predictable. You can read the YAML top-to-bottom and know which rule fires.
- No complex conflict resolution or priority system is needed.

### Parent and Leaf Rules

A rule with `rules` is a **parent** — it has no `action` and its `condition` is prepended (AND-ed) to every child's condition. A rule with `action` is a **leaf** and must not have `rules`.

At load time, nested rules are **flattened** into a single ordered list:
- Parent `condition` is AND-ed with each child's `condition`.
- Parent `name` is prefixed onto each child's name (e.g., `Bash > rm`).

**Example — nested rules:**

```yaml
rules:
  - name: "Bash"
    condition: 'tool == "bash"'
    rules:
      - name: "Allow safe commands"
        condition: 'command.matches("^(ls|find|grep|git\\s+status)\\b")'
        action: allow
      - name: "Block destructive commands"
        condition: 'command.contains("rm") || command.contains("sudo")'
        action: block
      - name: "Confirm other bash"
        condition: 'true'
        action: confirm
```

Flattened result:
1. `[allow] Bash > Allow safe commands: (tool == "bash") && (command.matches("..."))`
2. `[block] Bash > Block destructive commands: (tool == "bash") && (command.contains("rm") || command.contains("sudo"))`
3. `[confirm] Bash > Confirm other bash: (tool == "bash") && (true)`

## CEL Variables

These variables are available in every `condition` expression:

| Variable | Type | Description |
|----------|------|-------------|
| `tool` | `string` | Tool name: `read`, `write`, `edit`, `bash`, or any custom tool name. |
| `args` | `map` | Full tool arguments object. Access nested fields with dot notation: `args.path`, `args.timeout`. |
| `cwd` | `string` | Absolute current working directory of the session. |
| `path` | `string` | Resolved absolute path for file-based tools (`read`, `write`, `edit`). Empty string (`""`) for `bash` and tools without a `path` argument. |
| `command` | `string` | Bash command string. Available only when `tool == "bash"`. Empty string for all other tools. |

### Path Resolution

All relative paths are resolved to **absolute paths** before CEL evaluation:
- `./src/file.ts` → resolves to `/home/user/project/src/file.ts`
- `../other/file.ts` → resolves to `/home/user/other/file.ts` (will not match `cwd`)

Path resolution uses `node:path.resolve(cwd, args.path)`. The `cwd` variable itself is also resolved to an absolute path.

## CEL Functions

In addition to standard CEL operators (`==`, `!=`, `&&`, `\|\|`, `!`, `<`, `>`, etc.), pi.hitl provides these custom functions:

| Function | Signature | Description | Example |
|----------|-----------|-------------|---------|
| `path.startsWith(prefix)` | `(string, string) → bool` | String prefix check. | `path.startsWith(cwd)` |
| `path.contains(substr)` | `(string, string) → bool` | Substring check. | `command.contains("sudo")` |
| `str.matches(pattern)` | `(string, string) → bool` | Regex match. Pattern is a JavaScript `RegExp` string. | `command.matches("rm\\s+-rf")` |

**YAML escaping note:** In YAML double-quoted strings, `\` must be escaped as `\\`. The CEL regex engine receives a single `\`.

## Common Configuration Patterns

### 1. Default Sandbox (allow in project, block outside, confirm bash)

```yaml
version: 1
rules:
  - name: "Confirm bash commands"
    condition: 'tool == "bash"'
    action: confirm
    message: "Shell commands require manual approval"

  - name: "Allow within project"
    condition: 'path.startsWith(cwd)'
    action: allow

  - name: "Block outside project"
    condition: 'true'
    action: block
    message: "Operations outside the project directory are blocked"
```

### 2. Read-Only Sandbox

```yaml
version: 1
default_action: block
rules:
  - name: "Allow reads"
    condition: 'tool == "read"'
    action: allow
```

### 3. Allow Harmless Tracking Tools

```yaml
rules:
  - name: "Allow harmless task management"
    condition: 'tool.startsWith("Task") && tool != "TaskExecute"'
    action: allow
```

### 4. Confirm Destructive, Block Dangerous

```yaml
version: 1
default_action: allow
rules:
  - name: "Confirm rm"
    condition: 'tool == "bash" && command.contains("rm")'
    action: confirm
  - name: "Block sudo"
    condition: 'tool == "bash" && command.contains("sudo")'
    action: block
    message: "sudo is not allowed"
```

### 5. MCP-CLI Through Bash

Since mcp-cli runs through the `bash` tool, `path` is `""`. Use `command.contains()` or `command.matches()` instead.

```yaml
rules:
  - name: "Allow mcp-cli discovery"
    condition: 'tool == "bash" && command.matches("mcp-cli\\s+(info|grep|list)")'
    action: allow

  - name: "Allow mcp-cli filesystem reads"
    condition: 'tool == "bash" && command.contains("mcp-cli call filesystem read_file")'
    action: allow

  - name: "Confirm mcp-cli filesystem writes"
    condition: 'tool == "bash" && command.contains("mcp-cli call filesystem") && (command.contains("write_file") || command.contains("edit_file"))'
    action: confirm
```

### 6. Hidden Tools (Silently Block)

```yaml
hidden_tools:
  - "write"
  - "TaskExecute"
```

Hidden tools are silently blocked (the LLM sees "tool not available" rather than "blocked by policy"), reducing retry loops.

## Default Action Behavior

When no rule matches, `default_action` is applied:

| `default_action` | Interactive Mode | Non-Interactive Mode |
|------------------|------------------|----------------------|
| `allow` | Tool executes without dialog. | Tool executes without dialog. |
| `block` | Immediate block with reason. | Immediate block with reason. |
| `confirm` | Confirmation dialog appears. | Defaults to **block** (safety-first). |

## Commands

The `/permissions` command is available in every pi session:

| Command | Description |
|---------|-------------|
| `/permissions` or `/permissions status` | Show current rules, hidden tools, status, and default action. |
| `/permissions reload` | Reload config from disk. Re-evaluates all config locations and re-flattens nested rules. |
| `/permissions on` | Enable permission checks. |
| `/permissions off` | Disable permission checks. All tool calls are allowed. |

The `on`/`off` state is persisted across session reload, resume, and fork.

## Best Practices

1. **Order matters.** Put the most specific rules first, catch-all rules last. A `condition: 'true'` catch-all at the end is a common pattern.
2. **Use nested rules for readability.** Group related rules under a parent to avoid repeating shared conditions.
3. **Prefer `allow` over `confirm` for harmless tools.** Tracking tools like `TaskList`, `TaskGet`, `TaskOutput` are safe to auto-allow.
4. **Use `hidden_tools` for tools that should never exist.** This prevents LLM retry loops better than block rules.
5. **Always include a `name` and `message`.** These appear in the UI and make debugging easier.
6. **Validate CEL syntax.** Invalid expressions are logged as warnings and the rule is skipped at load time.
7. **Test with `/permissions status`.** After editing the YAML, use `/permissions reload` and then `/permissions` to verify the loaded rules.
8. **Be careful with `default_action: allow`.** Any tool not explicitly matched will execute without confirmation. Use explicit `block` rules for dangerous operations.
