# AI Agent Skills

Repository containing reusable skills for AI agents, following the [skills.sh](https://skills.sh) specification.

## Available Skills

### conventional-commits

Generate and validate commit messages following the Conventional Commits specification.

**Location**: `conventional-commits/`

**Use when**: Creating git commits, writing commit messages, reviewing commit history, generating changelogs, or when working with semantic versioning.

**Features**:

- Complete Conventional Commits specification reference
- Quick reference tables for commit types and structure
- Common mistakes and best practices
- Examples for all commit types
- Breaking change handling
- Semantic versioning correlation

## Structure

Each skill follows the standard skills.sh structure:

```
skill-name/
└── SKILL.md          # Required: metadata + instructions
```

## Installation

### Install conventional-commits skill

```bash
npx skills add https://github.com/EnderPuentes/ai-agent-skills --skill conventional-commits
```

For more information about using skills in your agent, follow the [skills.sh integration guide](https://skills.sh).

## Contributing

When adding new skills, ensure they follow:

- Skills.sh specification
- Conventional Commits for commit messages
- Clear, concise documentation
- Practical examples
