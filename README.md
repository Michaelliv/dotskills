# dotskills

My portable collection of [Agent Skills](https://agentskills.io) that I take from project to project.

## Skills

| Skill | Description |
|-------|-------------|
| **handoff** | Generate a prompt for a fresh conversation to continue the current implementation. Use when hitting context limits, ending a session, or handing off work. |
| **thinktank** | Simulate an expert panel discussion on a topic. Suggests 3 relevant experts and has them debate approaches, trade-offs, and arrive at recommendations. |

## Install

```bash
# Install all skills
npx skills add Michaelliv/dotskills

# Install a specific skill
npx skills add Michaelliv/dotskills --skill thinktank

# Install globally (available across all projects)
npx skills add Michaelliv/dotskills -g
```

Or clone directly:

```bash
git clone git@github.com:Michaelliv/dotskills.git ~/.dotskills
```

## Adding Skills

Each skill is a folder under `skills/` with a `SKILL.md` file:

```
skills/
├── handoff/
│   └── SKILL.md
└── thinktank/
    └── SKILL.md
```

See the [Agent Skills specification](https://agentskills.io/specification) for the full format.
