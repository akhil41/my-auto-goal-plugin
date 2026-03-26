---
name: my-auto-goal
description: Start an autonomous goal-achievement loop for research, code, app, content, or web-seo work
trigger: command
allowed-tools: Bash, Read, Edit, Write, Glob, Grep, WebSearch, WebFetch, AskUserQuestion, Agent
---

# /my-auto-goal — Autonomous Goal Loop

You are now running the my-auto-goal autonomous loop. Follow these steps IN ORDER. Do NOT skip any step.

## Step 1: Read the full skill instructions

Read the SKILL.md file for complete instructions:

```
Read the file at the path: <plugin-skills-dir>/my-auto-goal/SKILL.md
```

Where `<plugin-skills-dir>` is the `.claude/skills` directory inside this plugin's install path. Use Glob to find it:

```
Glob({ pattern: "**/.claude/skills/my-auto-goal/SKILL.md", path: "~/.claude/plugins/cache/my-auto-goal-plugin" })
```

## Step 2: Read the domain-specific loop guide

After reading SKILL.md and determining the domain, read the loop guide:

```
Glob({ pattern: "**/.claude/skills/my-auto-goal/loops/<domain_type>.md", path: "~/.claude/plugins/cache/my-auto-goal-plugin" })
```

## Step 3: Follow SKILL.md phases exactly

After reading SKILL.md and the loop guide, execute phases in strict order:
1. Phase 1: Goal Clarification — use AskUserQuestion
2. Phase 2: Plan Brief & Approval — present brief, get approval
3. Phase 3: Session Setup — create session directory and files
4. Phase 4: The Loop — iterate with keep/discard decisions

**IMPORTANT:** Do NOT invoke `Skill(my-auto-goal-plugin:my-auto-goal)` — that would create an infinite loop. The skill content is in SKILL.md. Read it with the Read tool.
