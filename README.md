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

**Note**: This skill is built from the official documentation at https://www.conventionalcommits.org/en/v1.0.0/

### pagespeed-insights

Audit web pages for performance optimization following PageSpeed Insights guidelines. Acts as a performance auditor that identifies bad practices and guides developers to achieve excellent PageSpeed scores.

**Location**: `pagespeed-insights/`

**Use when**: Analyzing page performance, optimizing web applications, reviewing performance metrics, implementing Core Web Vitals improvements, or when the user mentions page speed, performance optimization, Lighthouse scores, or Core Web Vitals.

**Features**:

- Complete PageSpeed Insights guidelines reference
- Core Web Vitals thresholds and best practices
- Common performance issues and solutions
- Lab and field data interpretation
- Performance optimization checklist
- Accessibility and SEO best practices
- Code examples for common optimizations

**Note**: This skill is built from the official documentation at https://developers.google.com/speed/docs/insights/v5/about?hl=es-419

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

### Install pagespeed-insights skill

```bash
npx skills add https://github.com/EnderPuentes/ai-agent-skills --skill pagespeed-insights
```

For more information about using skills in your agent, follow the [skills.sh integration guide](https://skills.sh).

## Contributing

When adding new skills, ensure they follow:

- Skills.sh specification
- Conventional Commits for commit messages
- Clear, concise documentation
- Practical examples
