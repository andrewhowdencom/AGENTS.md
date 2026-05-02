---
name: extensibility
description: Guidelines for dual-use architecture (CLI and Go Library), dependency separation, and extensible registry patterns.
---

# Extensibility & Dual-Use Architecture

`dux` is designed for two primary uses:
1. **As a binary / CLI tool**: Configured predominantly via YAML to dictatorial built-in behaviors. 
2. **As a Go library**: Consumed by external Go programs to build domain-specific agent orchestrators using the core primitives.

To support both use-cases cleanly, please adhere to the following rules when designing new systems, adapters, or capabilities:

## 1. Zero Dependency on the App-Layer in Core (`pkg/`)

Code within `pkg/` is considered the **Library boundary**. It must NEVER import from:
*   `cmd/`
*   `internal/` (e.g. `internal/config`) 
*   Heavy application-level frameworks like Viper or Cobra

*Why?* Go strictly prevents external modules from importing `internal/` directories. If `pkg/` depends on an `internal/` structure, the entire package becomes un-importable and fundamentally breaks the "as a Go library" use-case.

## 2. Primitives & Options Over Config Structs

When initializing core abstractions (Providers, Enrichers, Adapters), their constructors and factories must accept:
*   Fundamental Go types (e.g., `string`, `int`, `time.Duration`).
*   Custom `*Option` structs defined *within* the same package.
*   Interfaces holding necessary behavior definition.

Let the application layer (`cmd/` or `internal/`) handle mapping from Viper/YAML specs to these foundational structures before passing them to the core framework constructors.

## 3. Pure Variadic Options (No Registries)

We avoid global, mutable registries (e.g., `var registry = map[string]Constructor`). If an object requires multiple optional configurations, utilize the **Variadic Options Pattern** (e.g. `adapter.WithEnricher(...)`).

**Crucially:** the `pkg/` library does NOT need to map CLI string configurations (like YAML values `"time"`) to structs (`&timeEnricher{}`). The application layer (`internal/cli/` or `internal/config/`) handles all switch statements and deserialization of YAML values into core package constructors. `pkg/` files should simply export functions like `NewTime()` and rely on the calling layer to map user configuration identically.

## 4. Documentation by Examples (Specific Agents)

Whenever a new architectural enhancement, feature, or built-in agent capability is introduced, you MUST update the accompanying user documentation to provide concrete setup examples.

**Rule**: Provide structural examples featuring specific Prototype Agents (e.g., a "Q&A" agent) rather than generic placeholders. This ensures that documentation inherently acts as an integration test of practical use-cases. If a feature supports dynamic customization, provide an example showing how that feature configures a Q&A agent in `agents.yaml`.
