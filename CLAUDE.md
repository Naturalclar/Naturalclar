# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

`Naturalclar/Naturalclar` is a GitHub **profile repository**: because the repo name matches the account name, `README.md` is rendered at the top of https://github.com/Naturalclar. There is no application code, package manager, test suite, or linter — every file here exists to be rendered by GitHub, so "verifying a change" means checking how GitHub displays it, not running a command.

The repository is small enough to hold in your head:

- `README.md` — the profile page body.
- `header.svg` — the animated banner embedded by the README.
- `profile/*.svg` — the stats and top-languages cards. **Generated, not hand-edited** (see below).
- `.github/workflows/grs.yml` — regenerates the cards in `profile/` and commits them.
- `.github/FUNDING.yml` — drives the "Sponsor" button (`github: [Naturalclar]`).

## header.svg

`header.svg` is not a normal vector drawing. It wraps HTML in `<foreignObject>` and styles it with an inline `<style>` block — the technique from https://github.com/sindresorhus/css-in-readme-like-wat, credited at the bottom of the README. The visible text ("React Native Developer") is an `<h1>`, and the animations (`gradientBackground`, `rotate`, `fadeIn`) are CSS keyframes, so edits are CSS/HTML edits rather than path edits.

Two constraints follow from how GitHub serves this:

- **Everything must stay inline.** GitHub proxies the SVG as an image, so external stylesheets, fonts, or scripts will not load. Keep CSS in the embedded `<style>` block and stick to the system-font stack already declared.
- **The README must embed it with a raw `<img>` tag**, as it does today (`<img src="header.svg" width="500" height="250">`). Markdown image syntax and inline `<svg>` markup are both sanitized differently by GitHub; the current form is the one known to animate.

`foreignObject` rendering varies by browser, so confirm changes by loading `header.svg` directly in a browser *and* by looking at the pushed README on GitHub — a local Markdown preview is not representative.

## The cards

Everything under `profile/` is a **build artifact that is committed to the repo**. `.github/workflows/grs.yml` runs `stats-organization/github-readme-stats-action` once per card, writes each SVG into `profile/`, and pushes the result back to the default branch — daily on a cron, or on demand via `workflow_dispatch`. The README embeds them by relative path, so the profile page serves static files instead of calling an external service on every render.

**Each card exists twice, once per colour scheme**, and the README chooses between them with `<picture>` + `prefers-color-scheme`:

| Card | Dark (`theme=radical`) | Light (default theme) |
| --- | --- | --- |
| stats | `profile/stats.svg` | `profile/stats-light.svg` |
| top languages | `profile/top-langs.svg` | `profile/top-langs-light.svg` |

The `<source>` carries the dark variant and the `<img>` the light one, so light mode is also the fallback when `prefers-color-scheme` is unsupported. **Both `<picture>` blocks sit in one HTML block with no blank line between them** — that is what makes them render side by side. A blank line splits them into separate paragraphs and stacks them.

Consequences worth knowing before touching any of this:

- **Never hand-edit anything in `profile/`.** The next workflow run overwrites it.
- **Card appearance is configured in the workflow**, not the README — the `options` input is a `github-readme-stats` query string. Changing the look means editing `grs.yml`, and a themed change usually means editing the dark *and* light step.
- **A card change only appears after the workflow runs.** After merging an `options` change, trigger "Update README cards" manually rather than waiting for the cron.
- **A newly added card's file does not exist until that run finishes**, so a README embed merged ahead of it renders broken in the meantime. Dispatch the workflow immediately after merging.
- **The commit step is intentionally tolerant of no-ops**: `git commit ... || exit 0` ends the step successfully when the regenerated cards are byte-identical, so a run with no diff is not a failure. Its `git add profile/*.svg` already covers any new card, so adding one needs no change there.
- `stats` and `top-langs` are two of the types the action supports (`pin`, `wakatime`, `gist` are the others); adding one means a step per colour scheme plus an embed in the README.

## Workflow

Content changes are committed and pushed with git; the only automation is the card job above. Anything that renders on the profile page (README, SVG, funding config) should be eyeballed on GitHub after pushing, since there is no local check that can catch a sanitization or proxy issue.
