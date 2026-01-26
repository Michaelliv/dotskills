---
name: handoff
description: Generate a prompt for a fresh conversation to continue the current implementation. Use when hitting context limits, ending a session, or needing to hand off work to another agent/conversation.
metadata:
  author: michaelliv
  version: "1.0"
---

# Handoff Prompt Generator

Generate a structured prompt that captures the current work state so it can be continued in a fresh conversation.

## When to Use

- Context window is getting full
- User says "handoff", "continue later", "generate context"
- Handing off to another agent or team member
- End of session wrap-up

## Workflow

1. **Identify the focus**:
   - If user specifies a subject, focus on that
   - Otherwise, identify the last significant task being worked on

2. **Analyze recent work**:
   - Review recent changes and implementations
   - Identify files that were modified or are central to the task
   - Understand what was implemented and what remains

3. **Generate the handoff prompt** using the format below

## Output Format

```markdown
## Task: [Subject/Focus]

I'm working on [clear description of the task].

### Current Status
[What has been implemented so far]

### Files Involved
[List key files with @ mentions - these will be loaded fully in the new session]
- @src/path/to/file.tsx
- @src/path/to/other.ts

### Recent Implementation
[Brief description of what was just done]

### Next Steps
[What needs to be done next, any blockers or issues]

### Technical Context
[Important technical decisions, patterns, or approaches being used]
```

## Guidelines

- Use `@path/to/file` syntax for file mentions - this loads files into the new session
- Be specific about what was just implemented vs what remains
- Include any gotchas or things that didn't work
- Mention design decisions so they don't get revisited
- Keep it concise but complete enough to resume without re-exploration
