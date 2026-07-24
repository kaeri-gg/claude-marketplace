# developer-claude-marketplace

Personal Claude Code marketplace. One plugin per stack — install only what you work with.

| Plugin       | Audience                          |
|--------------|-----------------------------------|
| `godot`      | Godot game developers             |
| `javascript` | _planned_                         |
| `react`      | _planned_                         |


## 1. Installation

### Add the marketplace

```
/plugin marketplace add /path/to/claude-marketplace
```

(Point it at your local checkout, or at the Git URL once this repo is pushed to a remote.)

### Install the plugin(s) for your stack

```
/plugin install godot@developer-claude-marketplace       # Godot game development
```

Install as many as apply to you, then apply them to the current session:

```
/reload-plugins
```

(Without this, the skills only show up in your *next* session.)

---

## 2. Using skills

Installed skills appear in the slash-command menu under their short name — type `/` and start typing:

```
/godot-itch-deploy   (godot) Generate a GitHub Actions workflow that builds a Godot web export…
```

The owning plugin is shown next to each entry, so when several plugins ship a skill with the same
name, pick the right row from the menu.

You usually don't have to invoke them at all. Claude reads each skill's description and applies it
automatically when it's relevant — e.g. asking "set up CI to publish my Godot game to itch.io"
triggers the `godot-itch-deploy` skill on its own. The slash command is just a way to force it.

To see what's installed: `/plugin` → browse installed plugins and their skills.

---

## 3. Available skills

### godot

| Skill               | What it does                                                                                                                              |
|---------------------|------------------------------------------------------------------------------------------------------------------------------------------|
| `godot-itch-deploy` | Generates a GitHub Actions workflow that builds a Godot web/HTML5 export, deploys to GitHub Pages on push to `main`, and publishes to itch.io (via butler) on `v*` tags |

---

## 4. Updates

To pick up newly published skills:

```
/plugin marketplace update
```

Then update the installed plugin(s):

```
claude plugin update godot@developer-claude-marketplace
```

---

## 5. Contributing

Want to add or improve a skill — or add a whole new stack plugin (JavaScript, React, ...)?
See [CONTRIBUTING.md](CONTRIBUTING.md) — a skill is one Markdown file.
