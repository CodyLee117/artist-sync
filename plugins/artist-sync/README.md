# artist-sync

A shared Google Drive workspace for artists, kept in sync by Claude in Cowork. It holds notes, Adobe Animate scripts, browser
tools and team skills.

## Install
- **One artist:** Claude Desktop → Cowork → *Customize → Plugins* → upload `artist-sync-plugin.zip`.
- **The whole team (Team/Enterprise):** an admin puts this folder in a GitHub repo, adds it as a plugin marketplace, and marks
  the plugin *required*. It then installs and updates for everyone.
- **Plain chat only (no Cowork):** upload the single-skill zips under Settings → Capabilities → Skills. Syncing needs Cowork,
  because only Cowork has a folder on the Mac.

## Two setups (setup asks which one)
- **Desktop (recommended):** the workspace lives in **Google Drive for Desktop**, and Google's app moves the files. It's fast,
  handles big art files, and edits keep Drive's version history. Claude explains what changed, handles conflicts, and runs the
  reviews and secret checks. The Cowork project gets two folders: the shared one, in Google Drive in Finder, and a private one
  outside it for settings and anything waiting for a review.
- **Connector:** no extra app. Claude copies the files through the Google Drive connector. It's slower with big files, and old
  copies go to `_archive/` because the connector can't overwrite a file.

## Start
In a Cowork project, type `/setup-artist-sync` (or just say "set up artist sync"). It walks you through the folders for your
setup.

## What it does
- `artist-sync-setup`: connects Drive, joins or creates the shared folder, invites teammates, does the first sync, and sets up
  an hourly check.
- `artist-sync`: a three-way sync between the local folder and Drive, with a changelog (who, why), a merge offer for text
  conflicts, "Drive wins, yours saved to `.conflicts/`" for art files, and guards against secrets and mass deletes.
- `artist-workspace`: Claude checks the workspace before answering and saves reusable work back to it.
- `artist-helper-interview` (`/artist-interview`): a short interview that finds where Claude can save this artist time,
  drafts procedure and project notes, proposes 3–5 non-generative helpers, and builds one.
- `artist-build-review`: an independent sub-agent review of every script and team skill before it's shared; unreviewed work is never shared: in the connector setup, sync refuses
  unreviewed ones.
- `artist-sync-config` (`/artist-sync-config`): shows and changes your settings, including switching between the desktop and
  connector setups.
- `artist-team-skills`: runs teammates' skills from `skills/` once you approve them, and can package one for install.

## Updates
New versions come through the plugin marketplace: click **Update** on the marketplace in Cowork (Cowork also checks by itself).
One plugin holds every skill and command, so new skills arrive the same way.

## Limits
- The Drive connector can't overwrite a file, so each push uploads a new copy and bins the old one. The file's link changes.
- Files over 15 MB go into Drive by hand, through the browser.
- An hourly check runs only while Claude is open.
