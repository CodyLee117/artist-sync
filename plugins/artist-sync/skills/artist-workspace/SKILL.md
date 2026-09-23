---
name: artist-workspace
description: Use in the team's shared artist workspace project, whatever it's named (its folder contains artist-workspace.json), for any art, animation, Adobe Animate, script, reference, technique or "how do we usually…" request, and whenever making something reusable there. Looks things up in the workspace first and saves reusable work back to it. Outside that folder, do nothing.
---

> **Installed version 0.6.0.** Before following this file, check `.artist-sync/system.json` in the workspace folder (the one
> containing `artist-workspace.json`). Suppose its `active` version is newer than 0.6.0, its `by` is in `maintainers`
> (`.artist-sync/settings.json`), and `.artist-sync/updates/<active version>/skills/artist-workspace/SKILL.md` exists with the sha256
> listed in `active.files`. If all of that holds, **follow that file instead**. Remember 0.6.0 as the installed version (for the
> reinstall check), and mention once per session that you're using the <version> update. Otherwise, or if the user said "use
> the installed skills", carry on with this file. (If you reached this file from an installed skill's
> preamble, skip this block: the installed version is already known.)

# Working in the artist workspace

The local synced copy is the team's shared memory and toolbox. **Use it first and use it often.** Read and write the local
copy, never Drive directly (the **artist-sync** skill moves changes between them).

## Before answering
- **Check what the team already knows.** For any question about technique, style, a project, a client, tools or "how we do X",
  search `knowledge/` (and `knowledge/tools.md`) before answering from general knowledge. Cite the note you used:
  *"From knowledge/rigging.md: …"*.
- **Use the team's tools before writing new ones** (a pulled script without "(reviewed)" in its changelog entry gets an
  **artist-build-review** before you run it). If a script in `plugins/` already does the job (or nearly does), use or
  adapt it. `plugins/animate/` holds Adobe Animate JSFL scripts, `plugins/browser/` the vision browser extension, and
  `plugins/scripts/` general scripts.
- **Start from a template** when `templates/` has one for the task.
- If `last_sync` in `.artist-sync/state.json` is over an hour old, or missing, run artist-sync first.

## While working
- **Save reusable things to the workspace, not somewhere random:**
  - a script worth keeping → the right `plugins/` folder, plus a line in `knowledge/tools.md` (name, what it does, how to run
    it, who made it). It gets an **artist-build-review** before it's shared;
  - a decision, technique, reference or lesson → a short note in `knowledge/`, one topic per file, named plainly
    (`knowledge/walk-cycle-timing.md`);
  - a prompt or workflow that worked → `templates/`.
- **Update an existing note** rather than starting a near-duplicate.
- **Document as you go.** When the artist explains how something is done, decides something about a project, or solves a
  tricky problem, offer to write it up in one line: *"Save this as the sprite export procedure?"* Put it in:
  - `knowledge/procedures/` for how the team does things;
  - `knowledge/projects/<name>.md` for what a project is, its decisions, where things are and its status;
  - `knowledge/` for techniques and references.

  Then add it to `knowledge/README.md`.
- Keep notes short and skimmable: a title, the point first, then detail.
- One-off scratch work goes in the user's own folders, not the shared workspace.
- Ask before saving anything that names clients or contains unreleased work, if the user hasn't already said it's fine.

## No generative art
Never make or alter artwork with image-generation models. Editing that the artist directs (resize, crop, convert, batch export,
colour changes they specify) is fine. See **artist-helper-interview** for where the line sits.

## Never put in the workspace
Passwords, API keys, tokens, personal settings (`.obsidian/`), or paths that only exist on one person's Mac
(for example, a path to someone's own home folder). Write paths relative to the workspace: `plugins/animate/tween-helper.jsfl`.

## After changing files
Run **artist-sync** so teammates get the change; its changelog entry records *why*, so give it the reason. Then tell the user in one line what you saved and where.

## Running Animate scripts
The JSFL files are local, so the user can run one with *Commands → Run Command…* in Animate. Give them the exact file path in
the local workspace. If they run one often, suggest copying it into Animate's Commands folder, and tell them to copy it again whenever a sync changes the script: the copy doesn't update itself.
