---
name: agent-validation
description: Guidelines for ensuring architectural changes are validated against core use cases.
---

# Agent Validation

All designs must be evaluated against practical, real-world constraints to ensure Dux remains focused on delivering functional "agents."

## Core Requirement
When formulating an `implementation_plan.md` or making a significant architectural design decision, you **MUST** include a checklist explicitly validating the impact of those changes against the following 5 core example agents:

1. **Customer Feedback Agent**: Processing, categorizing, and summarizing customer feedback.
2. **Customer Support Agent**: Navigating knowledge bases and responding directly to support inquiries.
3. **Migration Agent**: Automating the migration of a codebase across library versions containing breaking changes.
4. **Guild Management Agent**: Handling internal company tasks (marketing, talk sourcing, meeting recaps).
5. **Product Marketing Agent**: Drafting release announcements and generating case studies.

## Enforcement
- Ensure that the impact of your proposed designs on *all 5 agents* is documented.
- If a capability is unlocked or improved that benefits these agents, you **MUST** update the respective tutorial document in `docs/tutorials/` within the same pull request/effort. Currently, the tutorials detail "Current Limitations/Known Gaps"; when these gaps are closed, the documentation must reflect the newfound capabilities.
