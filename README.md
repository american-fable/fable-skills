# Fable Skills for Claude Code

Custom [Claude Code](https://claude.com/claude-code) skills for [Fable](https://fable.is).

## Available Skills

### take-task

Fetches assigned tasks from Fable and executes them in your codebase.

- `/take-task` — take and complete one task
- `/take-task --loop` — continuously poll for and complete tasks

## Installation

Copy `skills/take-task/SKILL.md` to either:

- `~/.claude/skills/take-task/SKILL.md` (available in all projects)
- `<your-project>/.claude/skills/take-task/SKILL.md` (single project only)

## Configuration

Set these environment variables:

| Variable | Required | Description |
|---|---|---|
| `FABLE_JWT` | Yes | Your Fable authentication token |

## License

MIT