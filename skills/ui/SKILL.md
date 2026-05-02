---
name: ui
description: Guidelines for ensuring feature parity and core principles when developing Dux User Interfaces.
---

# UI Design Skill

When designing, modifying, or creating user interfaces for the Dux application (e.g. Web Apps, Terminals, IDE extensions), you must adhere to a core set of standardized pillars. These pillars guarantee feature parity between our clients and ensure that the complex background behavior of our agentic system remains observable.

## Core Pillars

### Pillar 1: Stream-Oriented Architecture
Dux is built around iterative text and structured-data streams. Interfaces must eagerly consume and render stream chunks (`[]llm.Part`) as they arrive.
- Never wait for the entire payload payload or stream to complete before rendering.
- Markdown rendering should occur per-chunk on the wire dynamically.

### Pillar 2: Agent Lifecycle Transparency
Advanced agents perform hidden logic. A Dux UI is responsible for visualizing this background execution natively as disjoint elements from the raw Assistant Response.
- **Thinking / Reasoning**: You must extract and intercept reasoning tokens (`llm.ReasoningPart`) and place them in a visually distinct bounding box/element near the top of the interaction. Do not merge them into the assistant's standard text message.
- **Tool Intentions and Results**: Track explicit tool executions (`llm.ToolRequestPart` and `llm.ToolResultPart`). Render the names of the tools and their payload/responses so the user knows what background services were interrogated. **Important**: Because the LLM continues reasoning in a connected internal loop after a tool finishes, you must cleanly interrupt the visual stream so that the assistant's *final* answer appears natively *after/below* the visualized tool call, rather than merging back into the preamble above it.
- **Errors**: Render errors vividly inline (rather than crashing).
- **Telemetry**: Provide performance/cost visibility by accumulating and rendering telemetry statistics (`llm.TelemetryPart`). Show token usage (Input, Output, Reasoning) and overall query duration sequentially.

### Pillar 3: Secure by Default (Human-In-The-Loop)
An unpredictable AI must be gated appropriately before mutating system state.
- **Explicit Approval Gating**: Any UI must implement logic to handle Human-in-the-Loop intercepts. Whenever the backend yields execution control requesting authorization, the UI must interrupt standard chat flow and ask the user to explicitly "Approve" or "Deny" the proposed action sequence.

### Cross-platform Feature Parity
When building a new UI or assessing an existing one, check it against the definitions above. Regardless of visual style, all Dux UIs must expose the above functional phenomena. If functionality like Telemetry Tracking or explicit Background Tool Visualization is missing, treat it as a critical parity gap and address it.
