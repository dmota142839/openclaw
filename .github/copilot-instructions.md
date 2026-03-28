# .github/copilot-instructions.md

## Purpose

This file defines workspace-specific conventions, memory, and agent workflow rules for all AI agents in this workspace. It is based on the AGENTS.md and quality-framework anti-patterns. Link to existing documentation instead of duplicating content.

---

## Core Principles

- **Read AGENTS.md first**: All agents must follow the session, memory, and safety rules in [workspace-main/AGENTS.md](workspace-main/AGENTS.md).
- **Link, don't embed**: Reference docs like `COMFYUI.md`, `specialist-agents/`, and `skills/` instead of copying content.
- **Avoid anti-patterns**: See [quality-framework.md](skills/prompt-optimizer/references/quality-framework.md#common-anti-patterns) for pitfalls to avoid in prompts and instructions.

---

## Agent Workflow

1. **Session Start**
   - Read `AGENTS.md` for workspace rules
   - Load today's and yesterday's `memory/YYYY-MM-DD.md` for context
   - If in main session, load `MEMORY.md`
2. **Task Execution**
   - Use specialist agents as described in `AGENTS.md`
   - For image generation, always route to ComfyUI pipeline (see `COMFYUI.md`)
   - For engineering, design, or product tasks, delegate to the correct specialist and load their file
3. **Memory Updates**
   - Log significant events in `memory/YYYY-MM-DD.md`
   - Curate long-term learnings in `MEMORY.md`
   - Update this file if new conventions or anti-patterns are discovered

---

## Common Anti-Patterns (link)

- [quality-framework.md#common-anti-patterns](skills/prompt-optimizer/references/quality-framework.md#common-anti-patterns)

---

## Example Prompts

- "Summarize today's memory and update MEMORY.md with key learnings."
- "Route this image generation request to the ComfyUI pipeline."
- "Delegate this bug fix to the engineering specialist and follow their workflow."

---

## Next Steps

- Consider creating `/create-skill`, `/create-agent`, or `/create-instruction` customizations for recurring workflows.
- Review and update this file as the workspace evolves.
