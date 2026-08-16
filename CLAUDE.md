# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

`Naturalclar/Naturalclar` is a GitHub **profile repository**: because the repo name matches the account name, `README.md` is rendered at the top of https://github.com/Naturalclar. There is no application code, package manager, test suite, or linter — every file here exists to be rendered by GitHub, so "verifying a change" means checking how GitHub displays it, not running a command.

The repository is small enough to hold in your head:

- `README.md` — the profile page body.
- `header.svg` — the animated banner embedded by the README.
- `profile/stats.svg` — the GitHub stats card. **Generated, not hand-edited** (see below).
- `.github/workflows/grs.yml` — regenerates `profile/stats.svg` and commits it.
- `.github/FUNDING.yml` — drives the "Sponsor" button (`github: [Naturalclar]`).

## header.svg

`header.svg` is not a normal vector drawing. It wraps HTML in `<foreignObject>` and styles it with an inline `<style>` block — the technique from https://github.com/sindresorhus/css-in-readme-like-wat, credited at the bottom of the README. The visible text ("React Native Developer") is an `<h1>`, and the animations (`gradientBackground`, `rotate`, `fadeIn`) are CSS keyframes, so edits are CSS/HTML edits rather than path edits.

Two constraints follow from how GitHub serves this:

- **Everything must stay inline.** GitHub proxies the SVG as an image, so external stylesheets, fonts, or scripts will not load. Keep CSS in the embedded `<style>` block and stick to the system-font stack already declared.
- **The README must embed it with a raw `<img>` tag**, as it does today (`<img src="header.svg" width="500" height="250">`). Markdown image syntax and inline `<svg>` markup are both sanitized differently by GitHub; the current form is the one known to animate.

`foreignObject` rendering varies by browser, so confirm changes by loading `header.svg` directly in a browser *and* by looking at the pushed README on GitHub — a local Markdown preview is not representative.

## The stats card

`profile/stats.svg` is a **build artifact that is committed to the repo**. `.github/workflows/grs.yml` runs `stats-organization/github-readme-stats-action`, writes the card to `profile/stats.svg`, and pushes the result back to the default branch — daily on a cron, or on demand via `workflow_dispatch`. The README embeds it by relative path, so the profile page serves a static file instead of calling an external service on every render.

Consequences worth knowing before touching it:

- **Never hand-edit `profile/stats.svg`.** The next workflow run overwrites it.
- **Card appearance is configured in the workflow**, not the README — the `options` input is a `github-readme-stats` query string (currently `show_icons=true&theme=radical`). Changing the look means editing `grs.yml`.
- **A card change only appears after the workflow runs.** After merging an `options` change, trigger "Update README cards" manually rather than waiting for the cron.
- **The commit step is intentionally tolerant of no-ops**: `git commit ... || exit 0` ends the step successfully when the regenerated card is byte-identical, so a run with no diff is not a failure.
- `card: stats` is one of several types the action supports (`top-langs`, `pin`, `wakatime`, `gist`); adding another card means another step with its own `path`, plus an embed in the README.

## Workflow

Content changes are committed and pushed with git; the only automation is the card job above. Anything that renders on the profile page (README, SVG, funding config) should be eyeballed on GitHub after pushing, since there is no local check that can catch a sanitization or proxy issue.
