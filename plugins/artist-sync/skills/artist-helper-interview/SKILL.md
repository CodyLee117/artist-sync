---
name: artist-helper-interview
description: Runs a short, friendly interview with an artist to find where Claude can save them time (automations, research, file organisation, image editing and processing), then researches and proposes a few concrete helpers and offers to build one. Use when the user runs /artist-interview, asks "how can you help me", "what can you automate for me", "find ways to save me time", or after setup finishes. Never proposes generative art.
---

> **Installed version 0.7.0.** Before following this file, check `.artist-sync/system.json` in the private folder (the project folder
> whose `.artist-sync/settings.json` exists; in the connector setup, that's the workspace folder itself). Suppose its `active` version is newer than 0.7.0, its `by` is in `maintainers`
> (`.artist-sync/settings.json`), and `.artist-sync/updates/<active version>/skills/artist-helper-interview/SKILL.md` exists with the sha256
> listed in `active.files`. If all of that holds, **follow that file instead**. Remember 0.7.0 as the installed version (for the
> reinstall check), and mention once per session that you're using the <version> update. Otherwise, or if the user said "use
> the installed skills", carry on with this file. (If you reached this file from an installed skill's
> preamble, skip this block: the installed version is already known.)

# Find where Claude can help this artist

Talk like a helpful studio assistant, not a consultant. Ask **one question per message**, keep them short, and give examples so
nobody has to invent an answer from scratch. The whole interview is **8 questions at most**, about 10 minutes. The artist can
say "skip" or "that's enough" at any point, and you work with what you have. The build question and the profile question
(below) don't count toward the 8, and you still ask them after "that's enough".

## The hard line: no generative art
- **Never propose making or altering artwork with image-generation models.** That covers generating images, "AI fill",
  style transfer, and AI or model-based upscaling. For multiple sizes, downscale from the largest export. Plain resampling
  upscaling (bicubic or nearest-neighbour) needs the artist's explicit OK.
- Editing **is** in scope when it's deterministic and the artist directs it: resize, crop, trim, pad, rename, convert
  formats, sprite sheets, contact sheets, colour or level adjustments the artist specifies, palette extraction, batch
  exports, and cleanup scripts inside their own tools (Animate JSFL, Photoshop actions/scripts).
- If the artist asks for something generative, say plainly that it's outside what this setup does, and offer the nearest
  non-generative help.

## Before asking anything
Read `knowledge/tools.md`, `skills/`, `knowledge/procedures/`, `knowledge/projects/` and any existing
`knowledge/people/<first name>.md`, so you don't propose what the team already has. **Check that every path `tools.md` lists
actually exists**; if one is missing, say so and don't count on it. If there's a profile, say *"Last time you mentioned X. Has anything changed?"* and ask only about what's new.

## The interview (pick the questions that matter; ask question 7 within your first three, whatever order you use)
1. *"What kind of art do you mostly make, and what for?"* (character animation for games, UI, backgrounds, marketing…)
2. *"Which programs are open on a normal day?"* (Animate, Photoshop, Illustrator, After Effects, Blender, Figma, a browser…)
3. *"Walk me through a typical task from start to finish. What happens first, and what comes last?"*
4. *"Which part feels most repetitive, or like clicking the same things again and again?"* (exporting, renaming, resizing,
   setting up files…)
5. *"Where do you lose time looking for things?"* (references, old files, the right version, feedback…)
6. *"When you research something, like a technique, a reference or how a tool works, what does that look like?"*
7. *"Is there anything you'd **never** want me to touch?"* Respect the answer, and write it down.
8. *"If I could take one boring job off your plate this week, what would it be?"*

Reflect back briefly between questions (*"So exporting every frame at three sizes eats your Fridays. Got it."*). Don't lecture.

## Turn answers into opportunities
Sort what you heard into these categories, and only keep ideas grounded in something they said:
- **Automations:**
  - Animate JSFL commands (batch export, symbol renaming, layer cleanup, timeline checks);
  - Photoshop or Illustrator scripts and actions;
  - folder watchers and batch file jobs.
- **Research:** reference boards gathered as links with notes, technique and tool how-tos, summaries of docs or tutorials,
  checking specs (platform image sizes, export settings).
- **Organisation:** naming conventions, and scripts that apply them; project folder templates; an index of assets or
  versions; turning feedback into checklists.
- **Image processing and editing** (non-generative, as above): resize, crop, convert, trim, pad, sprite and contact sheets,
  palettes, batch colour adjustments the artist specifies.
- **Admin:** status notes, handoff notes, meeting or feedback summaries saved to `knowledge/`.

For each idea, **research it before proposing it**. Check it's doable with what they actually have: their program and version,
whether scripting is available, whether **the artist's computer** can run Python or ImageMagick. Use web search or docs if available, and
look in the team's `plugins/` and `skills/` for something to reuse.

## Capture what you learned (the knowledge base)
The interview is also documentation. Before proposing, offer drafts, and save only the ones they approve.

**Before anything goes to the shared folder** (notes, profiles, summaries, changelog lines), remove client names, unreleased
work, and remarks about colleagues or clients, unless the artist says it's fine. The changelog is shared and never edited, so
its "why" names the task, never the complaint. Pain points in a shared profile describe the work, never people.
- **Procedures:** their "walk me through a task" answer becomes `knowledge/procedures/<task>.md`, with numbered steps,
  programs, settings and naming rules. It is **the team's way**, so check whether a procedure already exists and offer to
  update it instead.
- **Projects:** each project they mention gets a stub at `knowledge/projects/<name>.md` (what it is, their role, where the
  files live relative to the workspace, open questions). Leave out client-confidential details unless they say it's fine.
- Keep drafts short, and show them first: *"Here's the export procedure as I understood it. Anything wrong?"*
- Add new notes to `knowledge/README.md`, the index.

## Propose
Show **3 to 5 ideas**, most valuable first. For each one, one short block:
- **What:** one line in their words.
- **Saves:** a rough, honest time estimate.
- **How:** a script, a team skill, or a note or template, plus where it would live.
- **Effort to set up:** tiny, small or medium.
- **What I'd need from you:** a sample file, a naming rule, and so on.

Then ask: *"Want me to build one now?"*

## Build (if they say yes)
- **A script:** write it into the right `plugins/` folder. Test it on a copy of a sample file, never the only copy. If there's
  no sample yet, test it on dummy files and say so. Write its `knowledge/tools.md` entry (with how to run it), then run
  **artist-build-review**: an independent check, with fixes, and a plain-words summary for them. Then show them how to run
  it.
- **A repeatable workflow:** draft a team skill with **artist-team-skills** (they approve it, and teammates can use it too).
- **Reference or research:** save it to `knowledge/`.
- Sync afterwards (**artist-sync**). The changelog's "why" is the problem it solves for them.

## Remember the artist
Ask: *"Can I save a short profile, so I remember your tools and preferences next time? The team can see it in the shared
workspace, or I can keep it on this computer only."*
- **Shared:** `knowledge/people/<first name>.md`.
- **This computer only:** `.artist-sync/me.md`.

Keep it to: their role, their tools, their pain points, their "never touch" list, what was built, and ideas for later.
Nothing personal beyond work.
