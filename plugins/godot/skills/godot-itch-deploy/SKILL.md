---
name: godot-itch-deploy
description: Generate a GitHub Actions workflow that builds a Godot web export and deploys it to GitHub Pages on every push to main, then publishes to itch.io (via butler) on version tags. Use when the user wants to set up CI/CD to build and push a Godot game to itch.io and/or GitHub Pages, or asks for a deploy workflow for an HTML5 Godot export.
---

# Godot → GitHub Pages + itch.io deploy

Creates `.github/workflows/deploy.yml`: a single workflow that builds a Godot
**web/HTML5** export and ships it two ways from one `build` job:

- **GitHub Pages** — deployed on every push to `main` (and manual runs).
- **itch.io** — published via [`Ayowel/butler-to-itch`] **only on `v*` tags**, so
  Pages updates continuously but itch.io only gets tagged releases.

The `release/` output never needs to be committed — it stays gitignored.

This skill is derived from a known-good workflow that builds and publishes
"White Phosphorus: An Explosive Escape" correctly. The template lives at
`templates/deploy.yml` (in this skill directory) with `{{PLACEHOLDERS}}`.

## Steps

1. **Confirm it's a Godot web project.** Look for `project.godot` and
   `export_presets.cfg` at the repo root. If `export_presets.cfg` is missing,
   tell the user they must first add a **Web** export preset in the Godot editor
   (Project → Export → Add → Web) — the workflow can't export without one.

2. **Gather the placeholder values** — read them from the repo, don't ask if you
   can find them:
   - `EXPORT_PRESET` — the `name="..."` of the **Web** preset in
     `export_presets.cfg` (`platform="Web"`). This must match exactly.
   - `GODOT_VERSION` / `GODOT_STATUS` — e.g. `4.6` / `stable`. Check
     `project.godot` (`config/features`) for the engine version; default
     `GODOT_STATUS` to `stable`. Confirm with the user if unsure.
   - `GAME_TITLE` — the project name from `project.godot`
     (`application/config/name`).
   - `ITCH_IO_GAME_SLUG` — the game slug in the itch.io URL
     `https://<user>.itch.io/<slug>`. **Ask the user** — it's not in the repo.
   - `ITCH_IO_CHANNEL` — butler channel name. Default `html5` for web builds.

3. **Check the Web preset for a hardcoded `<base href>`.** Look at
   `html/head_include` in the Web preset. A value like
   `<base href="/<repo>/release/" />` is GitHub-Pages-specific and **breaks the
   itch.io build** — the same export is shipped to both targets, but on itch.io
   the game is served from `html-classic.itch.zone`, so an absolute base href
   forces every relative file (`index.js`, `index.wasm`, `index.pck`, audio
   worklets) to 404. The symptom is a **black canvas after pressing Run/Play**.
   Set `html/head_include=""` (or remove the absolute base tag). It isn't needed
   for Pages either — Godot's web export uses relative paths and the game is
   served from the `release/` directory, so paths resolve correctly without it.

4. **Write the workflow.** Read `templates/deploy.yml`, substitute every
   `{{PLACEHOLDER}}`, and write the result to `.github/workflows/deploy.yml`.
   Do not leave any `{{...}}` in the output. Preserve the comments.

5. **Ensure `release/` is gitignored.** Add `release/` to `.gitignore` if absent.
   If `release/` was previously committed, untrack it
   (`git rm -r --cached release/`) — the committed copy is usually a stale,
   different-Godot-version export that can shadow the CI build.

6. **Report the one-time manual setup** the user must do in the GitHub repo (you
   cannot do these — they require repo settings access):
   - **Settings → Pages → Source: "GitHub Actions"**.
   - **Settings → Secrets and variables → Actions**, add two repository secrets:
     - `ITCH_IO_API_KEY` — a butler API key from
       https://itch.io/user/settings/api-keys
     - `ITCH_IO_USERNAME` — the itch.io account name
   - The itch.io game page must already exist at `<user>.itch.io/<slug>`.
   - Publishing to itch.io is triggered by pushing a tag, e.g.
     `git tag v1.0.0 && git push origin v1.0.0`.

## How it works (so you can adapt it)

- **One `build` job, three downstream paths.** `build` exports once and uploads
  a `web-export` artifact. `deploy` (Pages) and `itch-publish` both `needs:
  build`; `itch-publish` is gated by `if: github.ref_type == 'tag'`.
- **Godot is cached** (`actions/cache`) keyed on version+status, so the ~hundreds
  of MB editor + export templates only download on a cache miss.
- **Import runs twice** headlessly — Godot's first headless `--import` pass often
  errors partway; the second produces a clean `.godot/` cache before export.
- **Pages layout** keeps the game at `…github.io/<repo>/release/` with a root
  redirect, matching a pre-existing deploy URL. Drop the redirect / move files to
  `_site/` root if you want the game served at `…github.io/<repo>/` directly.

## Troubleshooting

- **`WebAssembly.instantiate(): Import #0 "env": module is not an object or
  function` (blank page on Pages).** Almost always a **stale-cache version
  mismatch**: Godot's web files have fixed names (`index.js`, `index.wasm`, …)
  with no version hash, so a browser/CDN serves an old cached `index.js` glue
  against a freshly-built `index.wasm` of a different Godot version. Confirm by
  hard-refreshing (Ctrl+Shift+R) or loading in incognito. Verify the served
  `index.js`/`index.wasm` byte sizes match the CI build's `ls -lh release/`
  output. This recurs for visitors on every Godot version bump because the
  filenames never change.

- **Black screen after pressing Run/Play on itch.io.** The Web preset has a
  hardcoded absolute `<base href>` in `html/head_include` (see step 3). Remove
  it, re-export via a **new** version tag (re-pushing the same tag is
  unreliable), and hard-refresh the itch page.

- **itch.io embed settings to verify** (itch dashboard → Edit → Embed options):
  set viewport dimensions to the project's resolution; ensure "This file will be
  played in the browser" is checked. Leave **SharedArrayBuffer support OFF** for
  a no-threads export (`variant/thread_support=false`) — it doesn't need the
  cross-origin isolation headers that toggle adds.

## Adapting for non-web platforms

This workflow exports HTML5 only. For desktop builds (Windows/Linux/macOS),
add export presets for those platforms, export each to its own folder, and add
butler channels (e.g. `windows`, `linux`) — `butler-to-itch` accepts multiple
`files` lines mapping `<channel> <path>`.

[`Ayowel/butler-to-itch`]: https://github.com/Ayowel/butler-to-itch
