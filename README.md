# dotskills

My portable collection of [Agent Skills](https://agentskills.io) that I take from project to project.

## Usage

Clone this repo and point your agent to it:

```bash
git clone git@github.com:Michaelliv/dotskills.git ~/.dotskills
```

## Structure

```
skills/
├── <skill-name>/
│   └── SKILL.md
└── ...
```

## Adding Skills

Each skill is a folder with at minimum a `SKILL.md` file:

```yaml
---
name: skill-name
description: What this skill does and when to use it.
---

Instructions for the agent...
```

See the [Agent Skills specification](https://agentskills.io/specification) for the full format.
