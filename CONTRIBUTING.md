# Contributing a skill

A skill is a folder with a `SKILL.md` file that teaches Claude how to do a task the team's way — coding conventions, test style, scaffolding patterns, review checklists.

## Repo layout

```
.claude-plugin/
  marketplace.json          # marketplace catalog — lists the plugins
plugins/
  godot/
    .claude-plugin/
      plugin.json           # plugin manifest (name, version)
    skills/
      godot-itch-deploy/
        SKILL.md            # ← the skill itself
        templates/          # optional supporting files the skill reads
  javascript/               # (example of a future plugin)
    .claude-plugin/
      plugin.json
    skills/
docs/
  skill-template/SKILL.md   # copy this to start a new skill
```

## Adding a new skill — step by step

1. **Pick the plugin.** One plugin per stack: Godot skill → `plugins/godot/skills/`. If your stack has no plugin yet (JavaScript, React, ...), see [Adding a whole new plugin](#adding-a-whole-new-plugin) below.

2. **Create the folder and copy the template:**

   ```bash
   mkdir plugins/godot/skills/<skill-name>
   cp docs/skill-template/SKILL.md plugins/godot/skills/<skill-name>/SKILL.md
   ```

3. **Name it short — the plugin provides the context.** The slash menu lists skills under their short name with the owning plugin next to it: `/itch-deploy  (godot) …`. Try not to repeat the plugin in the name — a skill named `godot-deploy` in the `godot` plugin shows up as the redundant `/godot-deploy (godot)`.

   Good names: `write-tests`, `create-module`, `scaffold-component`, `itch-deploy`
   Bad names: `godot-deploy`, `tests`, `helper`

4. **Write the skill.** The template explains what goes where. The two things that matter most:
   - The frontmatter `description` must state **when** Claude should use the skill — it's the only thing Claude reads when deciding whether to load it.
   - Concrete good/bad code examples beat prose.

5. **Bump the plugin version** in `plugins/<plugin>/.claude-plugin/plugin.json` (e.g. `1.0.0` → `1.1.0`). ⚠️ **Not optional — do this for EVERY change**, including edits to existing skills. Claude Code caches installed plugins by version (`~/.claude/plugins/cache/<marketplace>/<plugin>/<version>/`); if the version doesn't change, `plugin update` sees "already up to date" and your change silently never reaches anyone.

6. **Test it locally** before publishing:

   ```
   /plugin marketplace add /path/to/this/repo     # add your local checkout as a marketplace
   /plugin install godot@developer-claude-marketplace
   ```

   Start a session in a real project and check that (a) the skill appears, and (b) Claude actually triggers it when you describe the task without naming the skill. If it doesn't trigger, rewrite the `description`.

7. **Commit and push.** Once pushed, pick up the change with `/plugin marketplace update` + `claude plugin update <plugin>@developer-claude-marketplace`.

## Adding a whole new plugin

Needed when a stack has no plugin yet (e.g. `javascript`, `react`). Copy the shape of the `godot` plugin:

**Naming the plugin:** name it after what one developer installs. Framework-specific names (`react`, `vuejs`) and generic language names (`javascript`, `python`) can coexist — pick the specific name when the skills are framework-bound, the generic one when they aren't. A generic language plugin can always be split into framework-specific plugins later (e.g. `javascript` → `react`) once there are enough skills to justify it. Don't add language prefixes like `js-react` — the name only needs to be unambiguous, and the slash menu already shows which plugin each skill comes from.

1. `mkdir -p plugins/<name>/.claude-plugin plugins/<name>/skills`
2. Add `plugins/<name>/.claude-plugin/plugin.json` (copy from `plugins/godot/`, change name/description, version `1.0.0`).
3. Register it in `.claude-plugin/marketplace.json` under `plugins`.

## What else a plugin can ship

Besides skills, a plugin can bundle:

- **MCP servers** — `.mcp.json` in the plugin root (Jira, GitLab, internal APIs)
- **Hooks** — `hooks/hooks.json` (e.g. run something at session start)
- **Agents** — custom subagent definitions
- **Dependencies** — other plugins, including from the official marketplace, via `dependencies` in `plugin.json`

Start with skills; add the rest when a real need shows up.

## Skill quality checklist

- [ ] `description` says **when** to use it, not just what it does
- [ ] Short name, no plugin prefix
- [ ] Contains concrete code examples (good and bad)
- [ ] One skill = one job
- [ ] Plugin version bumped
- [ ] Tested locally in a real project
