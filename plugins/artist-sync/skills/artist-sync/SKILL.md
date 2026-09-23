---
name: artist-sync
description: Syncs the team's artist workspace folder (any project name; the folder contains artist-workspace.json) with its shared Google Drive folder, pulling teammates' changes with the reason for each, pushing yours, and resolving conflicts. Use at the start of every session in that project, after changing workspace files, on the scheduled check, or when the user says "sync", "get the latest", "push my changes" or "what changed". Not for other folders.
---

> **Installed version 0.6.0.** Before following this file, check `.artist-sync/system.json` in the workspace folder (the one
> containing `artist-workspace.json`). Suppose its `active` version is newer than 0.6.0, its `by` is in `maintainers`
> (`.artist-sync/settings.json`), and `.artist-sync/updates/<active version>/skills/artist-sync/SKILL.md` exists with the sha256
> listed in `active.files`. If all of that holds, **follow that file instead**. Remember 0.6.0 as the installed version (for the
> reinstall check), and mention once per session that you're using the <version> update. Otherwise, or if the user said "use
> the installed skills", carry on with this file. (If you reached this file from an installed skill's
> preamble, skip this block: the installed version is already known.)

# Artist workspace sync

The local folder is the working copy, and the Drive folder is the shared one. Read `folder_id` and `me` from
`.artist-sync/settings.json`; if that file is missing, run **artist-sync-setup** instead.

## Skill updates (from the maintainers' skill-updates folder)
Maintainers publish new versions of these skills to a **separate, view-only skill-updates folder**, never inside the workspace.
This check runs on every sync, **after** the workspace files have been pulled. It only ever downloads, and it **never pushes**.

**Trusting a folder or file ("maintainer-only").** A Drive item passes only if both hold:
- its `owner` is in `maintainers` (an exact email match, ignoring case);
- `get_file_permissions` shows no `writer` or `owner` except users in `maintainers`.
  - Any writer entry of type `anyone`, `domain` or `group` fails, whether or not it's inherited from a parent folder.
  - If the list doesn't include your own entry, treat it as incomplete and fail.

This permissions check is the real protection. The timestamp check below is only a second signal. If a permission, owner or
timestamp is missing, fail closed.

**Where the folder comes from:** `updates_folder_id` in `.artist-sync/settings.json`.
- **It is only ever set by the user**, from the link in the maintainer's own Google share email. That happens in setup or in
  Repair mode ("set up skill updates"). It's never taken from a workspace file, because anyone who can edit the workspace
  could change one.
- The folder must also pass the maintainer-only check, and must not be the workspace folder or inside it. Before saving, show the folder's name and owner and ask: *"Is this the skill-updates folder <maintainer> set up
  for the team?"* Save only on a yes.
- If it's empty and `artist-workspace.json` names an `updates_folder`, only **tell** the user, once: *"Your team has skill
  updates. To turn them on, open the email from <maintainer> saying they shared a 'skill updates' folder with you, and paste
  its link here."*

**Each sync:**
1. **Find new releases.**
   - List `release.json` files at the top of the skill-updates folder. Mid-publish there may be two; take the one with the
     **highest version** first.
   - Skip any whose id is in `system.json` → `rejected` or equals `active.release_id`.
   - If none is left, skip to step 6.
2. **Check the release.json.**
   - It must be maintainer-only.
   - Its `createdTime` must equal its `modifiedTime`. Read both with `get_file_metadata`, because listings don't include
     `createdTime`.
   - Its version must be newer than `active.version`, or than the installed version if nothing is active.
3. **Fetch every file in its `files` list**, including the plugin zip if listed, into `.artist-sync/updates/<version>/`.
   - To find each file, list `releases/<version>/` one level at a time, as in One sync step 1, and match paths by title.
   - Each file must be maintainer-only, have `createdTime` equal to `modifiedTime`, and match its sha256 in `files`.
   - Record each file's Drive id for step 6.
4. **If everything passed,** write `.artist-sync/system.json`:
   ```json
   {"active": {"version": "…", "release_id": "…", "by": "<release.json's verified Drive owner>",
               "files": {"<path>": {"sha": "…", "id": "…"}}, "min_installed": "…", "notes": "…"},
    "rejected": {"<release.json id>": "<date>"}}
   ```
   Then tell the user: *"Your workspace skills were updated to <version>: <notes>."*
5. **If anything failed,** change nothing: keep the current `active`.
   - **A file simply missing from `releases/<version>/`** means the publish is probably still uploading. Try again next sync.
     Add it to `rejected` only once 30 minutes have passed since release.json's `createdTime`.
   - **An owner, permission, timestamp or sha failure** goes into `rejected` straight away.
   - Tell the user **once**: *"A skill update didn't pass its safety checks, so I've kept your current skills."*
6. **Self-repair (always runs):** if a file in `.artist-sync/updates/<active version>/` doesn't match its recorded sha,
   download it again by its recorded id, with the same checks, and say so. If that fails, remove `active`, and the installed
   skills take over.
7. **Reinstall nudge (always runs):** if `active.min_installed` is newer than the installed version (the version in the
   installed skill's preamble), add: *"This update also needs a quick reinstall: Customize → Plugins → upload
   `.artist-sync/updates/<version>/<zip>`."* Repeat it each session until the installed version catches up.

**Turning an update off or back on (no files to delete):**
- If the user says **"turn off this skill update"**, add `active.release_id` to `rejected`, remove `active`, and confirm that
  they're back on the installed skills. The next release arrives normally.
- **"Use the installed skills"** does the same for this session only.
- **"Retry the skill update"**, or any change to `maintainers` or `updates_folder_id` in Repair mode, clears `rejected`, so
  that previously refused releases are checked again.

## If the app blocks a delete or move
Cowork may ask the user's permission before a file is deleted or moved, or may refuse it.
- Overwriting a file (a pull) is normal, and needs no special handling.
- When a sync step needs a local file removed or moved (a teammate's delete, cleaning up a conflict copy), and it's refused,
  don't retry and don't count it as synced.
- Instead, list those files once at the end: *"Please delete these yourself: …"*. Record them in state as removed, so they
  aren't pushed back as new files.

## Progress and report
- **When you start** (a manual run, or the first sync of a session), say what's happening in one line: *"Syncing with the team
  folder…"*.
- **Once the listing is done,** say what you found: *"3 updates to download, 1 change of yours to send."* Say nothing if there
  are none.
- **During transfers,** give a short update every few files: *"Downloaded 4 of 9…"*. For one big file, say which one before you
  start: *"Downloading walk_cycle.fla (12 MB). This one takes a moment."*
- **At the end,** give the report below.

Keep the report to one or two plain sentences, built from the changelog:
*"Maya updated brush settings for the new rig, and Jo added a walk-cycle script. Sent your change to the tween helper. Nothing
needs you."*
- A manual run with nothing to do says *"Everything's up to date."*
- A scheduled run with nothing to do says nothing.

## Never sync
- The folders `.artist-sync/`, `.obsidian/`, `.git/`, `.conflicts/` and `_archive/` (`_archive/` is Drive-only; see
  "Replacing and removing files on Drive"). **Never pull or push any path containing `.artist-sync/`**,
  at any depth, in either direction. A Drive-side copy is never trusted. `.conflicts/` lives on Drive only; see Conflicts.
- The files `.DS_Store`, `Thumbs.db`, `*.tmp` and `~$*`, and any Drive file whose title starts with `~old~ ` (a replaced copy
  that couldn't be archived yet).
- Anything matching a line in `.artistsyncignore`.
- **Secrets, by filename.** These never upload: `.env*`, `*.key`, `*.pem`, `*.p12`, `id_rsa*`, `credentials*.json`, `.netrc`,
  `.npmrc`, `*token*`, `*secret*`.
- **Secrets, by content.** Before pushing a text file, scan it for these patterns: `-----BEGIN [A-Z ]*PRIVATE KEY-----`, `AIza[0-9A-Za-z_-]{20,}`,
  `\bsk-[A-Za-z0-9_-]{16,}`, `ghp_[A-Za-z0-9]{20,}`, `xox[baprs]-`, `(api_key|apikey|password|secret)"?\s*[:=]` and `/Users/[A-Za-z0-9._-]+/`.
  On a match, **hold the file and ask**; don't skip it silently. If they want it kept on this Mac only, add its path to
  `.artist-sync/held.json` and don't ask again.

## State (`.artist-sync/state.json`, per person, never uploaded)
```json
{"last_sync": "<ISO time>", "last_full": "<ISO time of the last full listing>",
 "folders": {"<path>/": "<drive folder id>"},
 "files": {"<path>": {"id": "…", "modified": "<Drive modifiedTime>", "sha": "<sha256>", "size": 0, "mtime": "<local mtime>"}},
 "pending": {"<path>": {"new_id": "…", "old_id": "…"}}, "stale": ["<drive id of an old copy still to tidy>"]}
```
- Hash with `shasum -a 256` on a Mac, or `sha256sum` elsewhere.
- Take `modified` from the Drive listing or from `create_file`'s result. Downloads don't return it.
- If `state.json` is an old flat `{path: …}` map, wrap it as `{"last_sync": null, "last_full": null, "folders": {},
  "files": <that map>, "pending": {}, "stale": []}`, and add any of those keys that are missing.

## One sync
0. **Safety checks.**
   - The local folder must contain `artist-workspace.json`. If it doesn't, stop, because the wrong folder is open. The one
     exception is the first sync after setup (there's no `last_sync` yet), when the folder is expected to be empty.
   - **If `pending` has entries, check each one before finishing it.**
     - If it has a `new_id`, read that file's metadata. If it's live at the path (the same folder and title, not in
       `.conflicts/` or `_archive/`), run Push step 4 (the same-moment check) on it first. Here, "after your step 1
       read" means created after `old_id`'s `modified` time in state. Only if you keep it, retire
       `old_id` (see "Replacing and removing files on Drive") and record `new_id`.
       Otherwise, drop the entry and let the compare decide.
     - If it has no `new_id` (the sync stopped mid-upload), list the folder by title for an upload of yours that wasn't
       recorded. If there is one, treat it as the `new_id` and check it as above. If not, drop the entry. **Never retire
       `old_id` without a checked replacement.**
1. **List Drive, cheaply.** Always pass `excludeContentSnippets: true` and `pageSize: 100`. Without the snippets flag, every
   listing drags in a preview of each file's text, which makes it much slower. Follow `nextPageToken` until you get an empty
   `{}`, and de-duplicate by id.
   - **Full listing** (the first sync, once a day by `last_full`, or when the user says "full sync"): walk the tree one level
     at a time. OR up to 40 folder ids per query (`parentId = 'a' or parentId = 'b' …`), and save every folder's id in
     `folders`. Only a full listing notices deletions and moves.
   - **Quick listing** (every other sync): one query for what changed since the last sync, using all the known folders.
     `modifiedTime > '<last_sync minus 2 minutes>' and (parentId = 'a' or …)` (40 ids per query). Any new folder it finds gets
     listed fully and added to `folders`. A quick listing sees **adds and edits only**. Renames, moves and teammates' deletes
     (which go to `_archive/`) may not change `modifiedTime`, so they wait for the next full listing. The push check (step 1
     below) catches them before anything is pushed over them.
   - **`last_sync` uses Drive's clock, never the computer's.** Set it to the newest `modifiedTime` seen in the listing **this
     sync started with** (keep the old value if nothing was returned). Changes made during a long sync are then caught next
     time.
   - Neither listing looks inside `_archive/`, except when restoring.
   - Listings include `id, title, mimeType, createdTime, modifiedTime, owner, parentId, fileSize`. There's **no "last modified
     by"**, so the changelog says who.
   - **Duplicates:** if one folder shows two live files with the same name, and only one of them is newer than state's id,
     that newer one is the file. Add the others' ids to `stale` and ignore them. **If two of them are newer than state** (two
     people pushed at once), that's a **conflict**, not a duplicate. Handle it under Conflicts.
2. **List local, cheaply.** Walk the folder, but only hash a file whose size or modified time differs from `state` (use
   `stat`). Hash all the changed ones in a single shell command. Create any of Drive's folders that are missing locally,
   including empty ones.
3. **Read new changelog entries.** Any `changelog/*.md` not already in state explains the incoming changes. Use them in the
   report and when explaining conflicts.
4. **Compare each path** across Drive (R), local (L) and state (S):

   "R changed" means Drive's id or `modifiedTime` for the path differs from state. Every push changes the id.

   | R vs S | L vs S | Do |
   |---|---|---|
   | same | same | nothing |
   | new (not in S) | absent | **pull**: a teammate's new file |
   | absent | new (not in S) | **push**: your new file |
   | new/changed | same | **pull**: download it and overwrite the local copy |
   | same | new/changed | **push** |
   | new/changed | new/changed | the same bytes: just record them. Otherwise a **conflict** |
   | gone | same | move the local file to `.artist-sync/removed/<date>/` |
   | same | gone | retire it on Drive (see "Replacing and removing files on Drive"): it goes to `_archive/`, so it can be restored |
   | gone | gone | drop it from state |
   | gone | changed, or changed/gone | **conflict** |
   | not in S, in both R and L | — | the same bytes: record them. Otherwise a **conflict** (for example, joining with files already local) |

   **Mass-delete guard:** if this sync would remove more than 5 files, or more than 20% of the files in state, on either side,
   **stop and ask**. Show the list first. Moves into `_archive/` count as removals. If it was a mistake, restore the files from
   `_archive/`, or pull them back from Drive.
   A rename or move shows up as one delete plus one add. That's fine, and the changelog explains it.
5. **Transfer in batches.** Make up to **5 downloads or uploads at once**, as parallel tool calls in one step, rather than one
   by one.
   - Write downloaded text files straight to disk.
   - For binaries, decode base64 with one shell command per batch.
   - **Record** each file's `{id, modified, sha, size, mtime}` as soon as it's done. Then set `last_sync` (and `last_full`, if
     this was a full listing).
   - **Big files are slow through this connector**, because every byte passes through the conversation. Anything over **5 MB**:
     say so before you start. Anything over **15 MB**: don't transfer it; see Push. If big art files are common, suggest the
     Drive for Desktop setup (README).
6. **Write a changelog entry** if you pushed anything (below). Uploads to `.conflicts/` don't count.

### Review gate (before any push)
Before pushing a new, changed or removed file under `plugins/` or `skills/`, check that its tool or skill folder was reviewed
at its current contents (`.artist-sync/reviewed.json`). Images and fonts need no review.
- **If you're chatting with the user,** run **artist-build-review** first, and push only once it passes.
- **On a scheduled run, don't review.** Hold those files, name them once in the report (*"Your new export script is waiting
  for a check. Say 'review it' when you're back"*), and don't raise them again until the user is in the chat.
- The rest of the sync goes ahead either way.

### Push (the connector can't overwrite a file's contents)
For a file that's **new** on Drive, do steps 3 and 4 only. If the file's folder isn't on Drive yet (this includes `changelog/`
and `.conflicts/`), create it first with `create_file` and `contentMimeType: application/vnd.google-apps.folder`.
1. Re-read the Drive file's metadata by its id (from state, or from this sync's listing).
   - **If it's gone, in `_archive/`, has a different `parentId`, or has a title that changed or starts with `~old~ `,** a
     teammate removed, moved or replaced it. Treat it as **gone on Drive**, using the table's "gone" rows, so an edit of yours
     becomes a conflict rather than resurrecting it.
   - **If its `modifiedTime` changed,** someone pushed a moment ago: that's a conflict.
   - When pushing a merge, compare against the Drive version you merged from.
2. Write `pending[path] = {old_id}` to state.
3. Upload it to the same Drive folder under the same name, using `create_file` with `disableConversionToGoogleType: true`,
   `textContent` for text or `base64Content` for binaries, and `contentMimeType` from this list:
   - `.md`: `text/markdown`
   - `.json`: `application/json`
   - `.jsfl` or `.js`: `application/javascript`
   - other text: `text/plain`
   - `.png`: `image/png`
   - `.jpg`: `image/jpeg`
   - anything else: `application/octet-stream`

   Save `new_id` into pending. Push several files in parallel (up to 5 at once).
4. List the folder by title. If another live file with that name appeared that isn't yours, isn't `old_id`, isn't in
   `stale`, and was created after your step 1 read, then someone pushed at the same moment. Only one of you gives way: **the later upload.**
   - If the other file's `createdTime` is **earlier** than yours (or it's equal and its id sorts lower), you give way. Move
     **your** `new_id` into `.conflicts/` as `<name> (<your first name> <YYYY-MM-DD>).<ext>` (you own it, so you can). Clear
     `pending[path]`, then follow Conflicts, where the version already on Drive wins.
   - Otherwise, keep yours and carry on to step 5. The other person's sync gives way.
5. Retire `old_id` (below), record `new_id`, and clear pending.

Anything over **15 MB** is too big to send through the chat. Ask the user to drag it into the Drive folder in the browser, and it
gets recorded on the next listing.

### Replacing and removing files on Drive: archive, don't lose
In a My Drive folder **only a file's owner can trash it**. After a teammate edits your file, neither of you can trash the
other's old copy, and that's what leaves duplicates. So old copies are **archived**, not trashed. That also keeps every old
version, so it can be restored.

To **retire** a Drive file (the old copy after a push, or a file you deleted locally), try these in order and stop at the first
that works:
1. **Move it** into `_archive/<YYYY-MM>/<the same folders as its path>/` with `update_file` (`parentId`). Rename it in the
   same call to `<stem> (<YYYY-MM-DD HHMMSS>Z)<.ext if any>`, in UTC. If that title already exists there, add ` 2`. Create any missing archive folders first, and cache their ids in
   `folders`. Who made each version is in the changelog; the title doesn't say.
2. If that's refused, **rename** it where it is, to `~old~ <title>`. Sync ignores those.
3. If that's refused too, add its id to `stale`, and ignore it by id.

**Tidy-up:** on each sync, for any `stale` id or `~old~ ` file you now **own**, move it to `_archive/`. That way the owner's own
agent clears what others couldn't. Leave everything else.

**Restoring:** when the user says *"restore <file> from <day>"* or *"undo Jo's change to <file>"*, list that file's copies.
They're in `_archive/<month>/<same folders>/`, named exactly `<stem> (` then the date. Match the stem exactly, so
`walk.fla` doesn't also pick up `walk_cycle.fla`. Include any `~old~ ` copies still in place.
Show the user the dates, with who made each version from the changelog, and download the one they pick. Then push it as a normal change, with the
changelog saying "restored from <date>". Never delete anything in `_archive/`. Each file's owner can clear their own old months in Drive; the archive folder's owner
can't delete teammates' files.

### Conflicts
- **Text** (notes, `.md`, `.jsfl`, `.js`, `.json`):
  1. Keep the local version.
  2. Save the Drive version as `.artist-sync/conflicts/<name> (from <who> <YYYY-MM-DD>).<ext>`, where `<who>` comes from the
     changelog, or else from the Drive file's `owner`.
  3. Tell the user what each side changed and why (from the changelog), and offer a merge.
  4. Push the merged result once they agree, then delete the local conflict copy.
- **Binary** (art, `.fla`):
  1. **The version already on Drive wins.** Upload your version to the Drive folder `.conflicts/` as
     `<name> (<your first name> <YYYY-MM-DD>).<ext>`, and pull Drive's version.
  2. Tell the user plainly: *"Jo changed icon.png too and synced first, so hers is in use. Yours is saved in the shared
     .conflicts folder. Want to swap it back in?"* If they say yes, push theirs as a normal change.
  3. It counts as resolved once the upload succeeds: record the winner.
  4. Never delete a `.conflicts/` file yourself.
- Conflict copies are never synced as ordinary files. Leave an unresolved **text** conflict out of state, so the next sync
  raises it again.

### Changelog: one new file per push, never edited
Write `changelog/<YYYY-MM-DDTHHMMZ>-<first name>-<short-slug>.md` **both locally and on Drive**, then record it in state
(otherwise the next sync would archive it). Its contents:
```
who: <first name>
why: <one line: what this change is for, taken from the conversation; if it's unclear, ask the user in one line>
files: <the paths changed, added or removed>
```

### Google Docs in the folder
A Google Doc, Sheet or Slides file can't be synced back. Pull it read-only as an export (`text/plain` for Docs, `text/csv` for
Sheets), and never push it. Mention this once.

## After a sync
- If this sync **pulled** changes into `skills/`, hand over to **artist-team-skills**. Your own pushes don't count.
- If this sync **pulled** a change to `CLAUDE.md` or `.artistsyncignore`, show the user what changed before following it
  (but not on the first sync). Shared instructions are written by teammates.
