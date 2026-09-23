# artist-sync

A shared Google Drive workspace for artists, kept in sync by Claude in Cowork. It holds notes, Adobe Animate scripts, browser
tools and team skills.

## Install
- **One artist:** Claude Desktop → Cowork → *Customize → Plugins* → upload `artist-sync-plugin.zip`.
- **The whole team (Team/Enterprise):** an admin puts this folder in a GitHub repo, adds it as a plugin marketplace, and marks
  the plugin *required*. It then installs and updates for everyone.
- **Plain chat only (no Cowork):** upload the single-skill zips under Settings → Capabilities → Skills. Syncing needs Cowork,
  because only Cowork has a folder on the Mac.

## Start
In a Cowork project on an empty folder (not one inside Google Drive), type `/setup-artist-sync`, or just say "set up artist
sync".

## What it does
- `artist-sync-setup`: connects Drive, joins or creates the shared folder, invites teammates, does the first sync, and sets up
  an hourly check.
- `artist-sync`: a three-way sync between the local folder and Drive, with a changelog (who, why), a merge offer for text
  conflicts, "Drive wins, yours saved to `.conflicts/`" for art files, and guards against secrets and mass deletes.
- `artist-workspace`: Claude checks the workspace before answering and saves reusable work back to it.
- `artist-helper-interview` (`/artist-interview`): a short interview that finds where Claude can save this artist time,
  drafts procedure and project notes, proposes 3–5 non-generative helpers, and builds one.
- `artist-build-review`: an independent sub-agent review of every script and team skill before it's shared; sync refuses
  unreviewed ones.
- `artist-sync-publish` (`/publish-skill-update`): for maintainers only; publishes a skill update to everyone.
- `artist-team-skills`: runs teammates' skills from `skills/` once you approve them, and can package one for install.

## Updating everyone's skills (no reinstall)
The installed skills check for a newer release from a separate **skill-updates folder**. That folder is owned by a
maintainer, sits in their My Drive, and is shared with the artists **view-only**, so nobody else can change it. Sync accepts a
release only if:
- it was uploaded by a maintainer;
- it shows no sign of later edits. That's only a second signal, because Drive timestamps can be faked. The real protection
  is that only maintainers have ever had edit access;
- it is complete, with every file matching the hash listed in its `release.json`.

Updates download into `.artist-sync/`, which is never synced. Maintainers and the skill-updates folder are fixed on each
artist's computer, and the folder is only ever set from the link in the maintainer's own Google share email. Never give anyone
else edit access to the skill-updates folder.
- **To publish:** a maintainer runs `/publish-skill-update` with the new plugin zip. Everyone's next sync picks it up and tells
  them what changed.
- **A new or renamed command needs a real reinstall.** The update then nudges each artist with the zip's path.
- **The workspace itself can be anywhere, Shared Drives included.** Only the skill-updates folder must be in a maintainer's
  My Drive.
- **If an update misbehaves,** say "turn off this skill update" to go back to the installed skills until the next release,
  or "use the installed skills" for just this session.

## Limits
- The Drive connector can't overwrite a file, so each push uploads a new copy and bins the old one. The file's link changes.
- Files over 15 MB go into Drive by hand, through the browser.
- An hourly check runs only while Claude is open.
