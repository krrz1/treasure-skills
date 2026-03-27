# Treasure Skills

[中文版](README_CN.md)

A personal collection of practical skills for Claude Code (and compatible AI coding agents).

Each skill is a self-contained procedure that helps your agent better understand what you actually want.

## Skills

| Skill | Description |
|-------|-------------|
| [clarity-check](clarity-check/) | **Clarity Check** - Evaluates ambiguous technical requests before acting. When a prompt like "test this" or "fix the CI" could lead to meaningfully different work, this skill scores interpretation reliability and decides whether to proceed, state its assumption, or ask a clarifying question. |

## Structure

```
treasure-skills/
├── <skill-name>/
│   └── SKILL.md        # Skill definition (YAML frontmatter + instructions)
└── ...
```

Each skill lives in its own directory with a `SKILL.md` file containing:

- **Frontmatter** — `name` and `description` (used for triggering)
- **Body** — The full procedure the agent follows when the skill is invoked

## Usage

Clone or symlink this repo into your Claude Code skills directory, or reference individual skills from your project's configuration.
