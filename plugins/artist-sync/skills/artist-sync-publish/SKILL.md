---
name: artist-sync-publish
description: For workspace maintainers only. Publishes a new version of the artist-sync skills to the team's skill-updates folder so every artist's agent switches to it on its next sync, with no reinstall. Use when the maintainer runs /publish-skill-update, or says "publish the new skills" or "push a skill update to the team".
---

> **Installed version 0.7.0.** Before following this file, check `.artist-sync/system.json` in the private folder (the project folder
> whose `.artist-sync/settings.json` exists; in the connector setup, that's the workspace folder itself). Suppose its `active` version is newer than 0.7.0, its `by` is in `maintainers`
> (`.artist-sync/settings.json`), and `.artist-sync/updates/<active version>/skills/artist-sync-publish/SKILL.md` exists with the sha256
> listed in `active.files`. If all of that holds, **follow that file instead**. Remember 0.7.0 as the installed version (for the
> reinstall check), and mention once per session that you're using the <version> update. Otherwise, or if the user said "use
> the installed skills", carry on with this file. (If you reached this file from an installed skill's
> preamble, skip this block: the installed version is already known.)

# Publish a skill update to the team (maintainers only)

Updates live in a **separate skill-updates folder**, not in the workspace. It's owned by a maintainer, sits in their My Drive,
and is shared with the artists as **view-only**, so nobody else can change what's in it.

1. **Check the user is a maintainer.** `me` must be in `maintainers` in `.artist-sync/settings.json` (an exact email match,
   ignoring case). If they aren't, stop.
2. **Find or make the skill-updates folder** (`updates_folder_id` in settings).
   - **If there isn't one yet**, ask first, then create a folder named "<workspace name>: skill updates" in the maintainer's My
     Drive root, with a `releases` folder inside it.
     - Share it **as viewer (`reader`) only** with each artist. Offer the writers from `get_file_permissions` on the workspace
       folder, and confirm the list.
     - Save its id as `updates_folder_id`, and also as `"updates_folder": "<id>"` in the workspace's `artist-workspace.json`.
       Then run **artist-sync** to push that file (it isn't under `skills/`, so there's no review gate). The workspace copy is
       only a **hint**: each artist turns updates on by pasting the link from Google's share email, which comes from you.
     - Tell the maintainer: *"Each artist will get a Google email saying you shared the folder. They paste that link when their
       agent asks, or say 'set up skill updates'."*
     - **Never give anyone else edit access to this folder.** A past editor could have changed files in ways the checks can't
       see.
   - **If there is one**, check that it still passes artist-sync's maintainer-only check (including no writers of type
     `anyone`, `domain` or `group`).
3. **Get the new version.** Ask them to put the new plugin zip in `.artist-sync/publish/`, which is never synced, and unzip it
   there.
   - Read the version from `.claude-plugin/plugin.json`.
   - **Every** SKILL.md's preamble must use that version, in every place it appears ("Installed version", "newer than",
     "Remember").
   - It must be newer than the current `release.json`, or than the installed version if nothing has been published yet.
4. **Review it** with **artist-build-review**, covering the whole `skills/` folder, since it's instructions for every artist's
   agent. Show the maintainer which skills changed compared with the current release (or with the installed skills, if there isn't
   one yet), one line each.
5. **Ask two things:**
   - *"One line for the team on what's new?"* That becomes the `notes`.
   - *"Does this add or rename a command, or change anything in the plugin besides the skills and version numbers?"* If yes,
     set `min_installed` to this version and upload the zip. If no, keep the previous `min_installed` (on a first publish,
     use the installed version).
6. **Upload a complete release, creating files only and never replacing them.** Make `releases/<version>/` and upload every
   file of every skill into it: `skills/<name>/SKILL.md` plus any templates. Add the zip too, if needed. Create the subfolders as
   you go, and use `disableConversionToGoogleType: true`.
7. **Then, last of all**, create a new `release.json` at the top of the skill-updates folder:
   ```json
   {"version": "<v>", "notes": "<line>", "min_installed": "<v or earlier>", "plugin_zip": "releases/<v>/<zip name>" or null,
    "published": "YYYY-MM-DD", "by": "<email>",
    "files": {"skills/<name>/SKILL.md": "<sha256>", "…": "…", "<zip name, if any>": "<sha256>"}}
   ```
   Once it exists, trash the previous `release.json`. Leave old `releases/<v>/` folders in place for rollback.
8. **Changelog.** Write a workspace changelog entry named `<YYYY-MM-DDTHHMMZ>-<first name>-skills-<v with dots as dashes>.md`,
   with `why: skills updated to <v>: <notes> (reviewed)` and `files: release <v>`, plus any workspace files pushed in the same
   run (such as `artist-workspace.json`).
9. **Tell the maintainer:** *"Published <v>. Teammates switch to it on their next sync"* (plus *"and get a reinstall nudge"* if
   `min_installed` changed).

**Rolling back:** publish the old files again with a **higher** version number. First change `plugin.json` and every version
mention in every SKILL.md preamble to that new number, because step 3 checks them. Agents never move to a lower version.
