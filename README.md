# UClaw Agent Skills

Agent skills for working with UClaw.

## Installation

Install all public skills from this repository:

```bash
bunx skills add uclaw-dev/agent-skills
```

Install only the UClaw SDK skill:

```bash
bunx skills add uclaw-dev/agent-skills --skill uclaw-sdk
```

## Skills

- `uclaw-sdk` - Install, configure, and write code with `@uclaw/sdk`.

## Repository Structure

```text
skills/
└── uclaw-sdk/
    └── SKILL.md
```

Each skill lives in its own directory under `skills/` and must include a `SKILL.md` file with YAML frontmatter containing `name` and `description`.

## License

MIT
