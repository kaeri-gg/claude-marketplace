---
name: my-skill-name
description: One or two sentences describing what this skill does AND when Claude should use it. This is the ONLY part Claude reads when deciding whether to load the skill — make the trigger conditions explicit, e.g. "Use whenever writing Vue component tests" or "Use when the user asks to scaffold a new NestJS module".
---

# What this skill does

Explain the goal in one short paragraph.

# Instructions

Write the instructions as if briefing a competent developer who is new to the team.
Be concrete and prescriptive — Claude follows these verbatim.

## Preferred patterns

Show real code examples of what GOOD looks like:

```ts
// good example here
```

## Anti-patterns

Show what to AVOID and why:

```ts
// bad example here
```

# Tips for writing a good skill

- **The `description` frontmatter is the trigger.** If Claude doesn't invoke your skill,
  the description probably doesn't state clearly *when* to use it.
- **Name it short.** The plugin already namespaces it: a skill named `write-tests` in the
  `vuejs` plugin is invoked as `/vuejs:write-tests`. Don't repeat the plugin name.
- **Show, don't tell.** One good/bad code pair beats three paragraphs of prose.
- **Keep it focused.** One skill = one job. "Write tests" and "scaffold a component"
  are two skills, not one.
- **Extra files are allowed.** Put longer reference material next to SKILL.md
  (e.g. `references/style-guide.md`, `assets/logo.png`) and mention them in the
  instructions — Claude can read them on demand.
