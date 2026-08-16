# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

`Naturalclar/Naturalclar` is a GitHub **profile repository**: because the repo name matches the account name, `README.md` is rendered at the top of https://github.com/Naturalclar. There is no application code, package manager, build step, test suite, or linter — every file here exists to be rendered by GitHub, so "verifying a change" means checking how GitHub displays it, not running a command.

The entire repository is three files:

- `README.md` — the profile page body.
- `header.svg` — the animated banner embedded by the README.
- `.github/FUNDING.yml` — drives the "Sponsor" button (`github: [Naturalclar]`).

## header.svg

`header.svg` is not a normal vector drawing. It wraps HTML in `<foreignObject>` and styles it with an inline `<style>` block — the technique from https://github.com/sindresorhus/css-in-readme-like-wat, credited at the bottom of the README. The visible text ("React Native Developer") is an `<h1>`, and the animations (`gradientBackground`, `rotate`, `fadeIn`) are CSS keyframes, so edits are CSS/HTML edits rather than path edits.

Two constraints follow from how GitHub serves this:

- **Everything must stay inline.** GitHub proxies the SVG as an image, so external stylesheets, fonts, or scripts will not load. Keep CSS in the embedded `<style>` block and stick to the system-font stack already declared.
- **The README must embed it with a raw `<img>` tag**, as it does today (`<img src="header.svg" width="500" height="250">`). Markdown image syntax and inline `<svg>` markup are both sanitized differently by GitHub; the current form is the one known to animate.

`foreignObject` rendering varies by browser, so confirm changes by loading `header.svg` directly in a browser *and* by looking at the pushed README on GitHub — a local Markdown preview is not representative.

## README.md

The stats card is an external service call to `github-readme-stats.vercel.app` with `theme=radical`; it renders only once GitHub fetches it, and changes to query parameters can be cached by GitHub's image proxy for a while before taking effect.

## Workflow

Content-only changes, committed and pushed with git. Anything that renders on the profile page (README, SVG, funding config) should be eyeballed on GitHub after pushing, since there is no local check that can catch a sanitization or proxy issue.
