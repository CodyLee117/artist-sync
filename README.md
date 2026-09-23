# artist-sync marketplace

A Claude **Cowork** plugin that gives a team of artists a shared Google Drive workspace. Claude keeps it in sync, and it holds
notes, Adobe Animate scripts, browser tools and team skills. Details: [plugins/artist-sync/README.md](plugins/artist-sync/README.md).

## Install (each artist, once)
1. Claude Desktop → **Cowork** → *Customize → Plugins* → **Add marketplace** → paste this repository's URL.
2. Install **artist-sync** from it.
3. In a Cowork project, type `/setup-artist-sync`.

If you installed artist-sync earlier from a zip file, **uninstall that copy first** (*Customize → Plugins → artist-sync →
Uninstall*), so the two don't clash.

## Updates
New versions are published here. Cowork checks the marketplace for updates, or you can click **Update** on the marketplace to
pull the latest. Each release bumps `version` in `plugins/artist-sync/.claude-plugin/plugin.json`.
