# Factorial Skills Collection

A curated collection of AI coding assistant skills used for development work at Factorial. These skills provide specialized guidance, workflows, and best practices that enhance AI-assisted development.

## What are Skills?

Skills are structured instruction sets that teach AI coding assistants (like Claude Code or OpenCode) how to handle specific tasks, tools, or workflows. When a skill is invoked, the assistant follows its guidance to provide consistent, safe, and effective help.

## Available Skills

| Skill | Description |
|-------|-------------|
| **jujutsu** | Use `jj` (Jujutsu) instead of `git` for version control. Provides a simpler mental model and powerful history editing with Git compatibility. |
| **phabalicious** | DevOps and deployment workflows using Phabalicious (phab) CLI. Covers environment management, data synchronization, backups, and safe deployment practices. |
| **security-audit** | Systematic methodology for security reviews with severity assessment, dual documentation (client + internal), and acceptance-focused ticket creation. |

## Installation

Install skills from this collection using the `skills` CLI:

```bash
# Install to your project (interactive - choose skills and agents)
npx skills add factorial-io/skills

# Install globally (user-level, available across all projects)
npx skills add factorial-io/skills --global

# Install a specific skill
npx skills add factorial-io/skills --skill phabalicious

# Install to a specific agent (opencode, claude-code, cline, codex, cursor, etc.)
npx skills add factorial-io/skills --agent claude-code

# List available skills without installing
npx skills add factorial-io/skills --list

# Install all skills to all agents without prompts
npx skills add factorial-io/skills --all
```

### Keeping Skills Updated

```bash
# Check for available updates
npx skills check

# Update all installed skills
npx skills update
```

## Usage

### With OpenCode

Skills are automatically available when working in repositories that include this collection. OpenCode will invoke relevant skills based on context:

```
# Mentioning "phab" or deployment topics triggers the phabalicious skill
> How do I copy production data to my local environment?
```

### With Claude Code

Use the `Skill` tool to load skills:

```
# The assistant will invoke skills when relevant topics arise
> I need to deploy to staging
```

## Skill Structure

Each skill lives in its own directory with a `SKILL.md` file:

```
skills/
  phabalicious/
    SKILL.md
  another-skill/
    SKILL.md
```

### SKILL.md Format

Skills use frontmatter for metadata:

```markdown
---
name: skill-name
description: Brief description of when to use this skill
---

# Skill Title

## Overview
What the skill does and its core principles.

## When to Use
Triggers and scenarios that should invoke this skill.

## Commands/Workflows
Detailed guidance, command references, and workflows.
```

## Contributing

### Adding a New Skill

1. Create a directory with the skill name: `mkdir my-skill`
2. Add a `SKILL.md` file with proper frontmatter
3. Include:
   - Clear trigger conditions (when should the AI use this skill?)
   - Safety guidelines (especially for destructive operations)
   - Command references with examples
   - Common workflows and best practices

### Skill Design Principles

- **Safety first**: Destructive operations should require confirmation
- **Context-aware**: Skills should ask clarifying questions when intent is ambiguous
- **Practical**: Include real command examples and common workflows
- **Focused**: Each skill should cover a specific tool or domain

## License

Internal use at Factorial.
