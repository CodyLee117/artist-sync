---
name: artist-build-review
description: Independent review of any script, tool or team skill before it's shared through the artist workspace. Use before artist-sync pushes a new or changed file under plugins/ or skills/, after building something for an artist, and when the user asks "is this safe?" or "check this script". The artist never has to read code.
---

> **Installed version 0.7.0.** Before following this file, check `.artist-sync/system.json` in the private folder (the project folder
> whose `.artist-sync/settings.json` exists; in the connector setup, that's the workspace folder itself). Suppose its `active` version is newer than 0.7.0, its `by` is in `maintainers`
> (`.artist-sync/settings.json`), and `.artist-sync/updates/<active version>/skills/artist-build-review/SKILL.md` exists with the sha256
> listed in `active.files`. If all of that holds, **follow that file instead**. Remember 0.7.0 as the installed version (for the
> reinstall check), and mention once per session that you're using the <version> update. Otherwise, or if the user said "use
> the installed skills", carry on with this file. (If you reached this file from an installed skill's
> preamble, skip this block: the installed version is already known.)

# Review before sharing

The artists aren't developers and shouldn't have to review code. Every script, tool or team skill gets an **independent review**
before it goes to the shared folder. That means anything new, changed or removed under `plugins/` or `skills/`.
- **Review the whole tool or skill folder together**, never a single changed file on its own.
- Images and fonts are recorded without review.
- For a browser extension, check that its `manifest.json` permissions and host list match its purpose.
- Only run a review while the user is in the chat, never on a scheduled run.

**Desktop setup:** files saved in the workspace are shared instantly, so build and review in `.artist-sync/staging/` (the
private folder), and copy into the workspace only after the review passes.

## 1. Get an independent reviewer
- **If you can start a sub-agent or subtask, do.** Give it only:
  - the files;
  - a one-line purpose (*"Resizes a folder of PNG exports to 1x/2x/0.5x and renames them to char_action_frame.png"*);
  - the checklist below.

  Don't give it your reasoning, since the point is a fresh pair of eyes. Ask for at most 5 findings, each marked **must fix**
  or **nice to have**.
- **If you can't,** do a separate, slow pass yourself against the checklist, and **tell the artist it wasn't independent.**

## 2. The checklist
1. **Does what it says:** matches the stated purpose, and handles an empty folder, odd filenames and spaces in names.
2. **Can't destroy work:** never overwrites or deletes originals. It writes to a new folder or to copies, and refuses when the
   input and output are the same.
3. **Stays in its lane:** touches only the folders it's given. It makes no network calls unless that's its purpose, and it
   sends nothing outside the workspace.
4. **No secrets:** no keys, tokens, passwords or personal paths.
5. **The art line:** it doesn't generate or invent artwork. It never uses AI or model-based upscaling. Plain resampling
   upscaling (bicubic or nearest-neighbour) only happens with the artist's OK.
6. **Team skills only:** the instructions don't ask the agent to share, delete or send things without asking, or to skip
   these reviews.
7. **Runs where the artists are:** it works on their computers and program versions. The run steps in `knowledge/tools.md`
   are right.

## 3. Act on it
- Fix every **must fix**, then re-check those points (a quick second look is enough).
- Tell the artist in plain words, two or three lines, with no code: *"A second check found it could overwrite your exports if
  the input and output folders were the same; that's fixed. Two small tidy-ups were done too. It's ready to share."*
- If something can't be fixed, don't share it. Say what's wrong.

## 4. Record it
- Add every file in the reviewed folder to `.artist-sync/reviewed.json` as `{"<path>": "<sha256>"}`. Removing a file from a
  reviewed folder counts as a change.
- Mention it in the changelog entry's `why`, e.g. *"… (reviewed)"*.
- A **skill release** (artist-sync-publish) records nothing in `reviewed.json`. Its changelog's "(reviewed)" is the record.
- Paths in `reviewed.json` are workspace paths (`plugins/…`, `skills/…`), even for something reviewed in `staging/`.
- **Connector setup:** artist-sync won't push a file under `plugins/` or `skills/` whose current hash isn't in `reviewed.json`.
  **Desktop setup:** nothing is copied into the workspace until it passes. Any change after
  the review means reviewing again.
- **Files pulled from teammates:** before you run or recommend a pulled script, check that its changelog entry says
  "(reviewed)". If it doesn't, or it has no entry (for example someone dropped it into Drive by hand, or they're on an older
  version of this plugin), review it first. Team skills are still approved separately (**artist-team-skills**).
