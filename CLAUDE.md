# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

`Naturalclar/Naturalclar` is a GitHub **profile repository**: because the repo name matches the account name, `README.md` is rendered at the top of https://github.com/Naturalclar. There is no application code, package manager, test suite, or linter — every file here exists to be rendered by GitHub, so "verifying a change" means checking how GitHub displays it, not running a command.

The repository is small enough to hold in your head:

- `README.md` — the profile page body.
- `profile/*.svg` — the stats and top-languages cards. **Generated, not hand-edited** (see below).
- `typing.svg` / `typing-light.svg` — the typewriter intro line that opens the profile. Hand-authored (see below).
- `x-logo.svg` / `x-logo-light.svg` — the X icon for the profile link. Hand-authored, unlike everything in `profile/`; they live at the root to keep that distinction obvious. Same `<picture>` pairing as the cards — the unsuffixed file is the dark-mode (light-coloured) variant.
- `.github/workflows/grs.yml` — regenerates the cards in `profile/` and commits them.
- `.github/FUNDING.yml` — drives the "Sponsor" button (`github: [Naturalclar]`).

## typing.svg

The intro line at the top of the profile is a CSS typewriter effect: an inline `<style>` block, no script, embedded through `<img>` inside a `<picture>`.

**Everything must stay inline.** GitHub serves the SVG as an image, so external stylesheets, fonts, or scripts will not load. Keep CSS in the embedded `<style>` block and stick to the font stack already declared — this holds for every hand-authored SVG here, not just this one.

Because GitHub sanitizes Markdown image syntax and inline `<svg>` markup differently, the `<img>` embed is the form known to animate; a local Markdown preview is not representative, so check the pushed README on GitHub.

The mechanism is a `<clipPath>` rectangle that scales from `scaleX(0)` to `scaleX(1)` under `steps(41, end)`, so the clip edge lands on a character boundary rather than mid-glyph, plus a caret `<rect>` translating across the same distance under the same step count. The `reveal` and `caret` keyframes share one 6s timeline — type, hold, erase, hold — so the two stay in lockstep.

**Three numbers are coupled and must change together if the text does:**

| | Current | What it is |
| --- | --- | --- |
| `steps(N)` | 41 | character count of the string |
| `textLength` | 738 | pinned width of the `<text>` (41 chars x 18px advance at font-size 30) |
| `translateX` | 738px | caret travel, equal to `textLength` |

`textLength` is what makes the stepping exact: it forces each character's advance to `textLength / N` whatever monospace face the renderer resolves, so the reveal cannot drift out of sync with the glyphs. Changing the string without updating all three leaves the caret and the clip edge landing in the wrong places.

The two files differ only in `fill` — neon `#05dfd7` for dark, deeper `#0b9e90` for light, because the neon tone measures 1.67:1 on white and is unreadable there. Edits must be applied to **both** — `typing.svg` carries the dark-mode (light-coloured) text.

A `prefers-reduced-motion: reduce` block disables both animations and pins the line fully typed.

To check a specific moment of the animation, bake a negative `animation-delay` into a copy (`#reveal, .caret { animation-delay: -2.7s; }`) and open that — it seeks the cycle without needing to catch it live.

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

## The links row

The links under the cards are `<a>` elements inside one `<p>` — the X entry wraps the icon `<picture>` plus the handle text, the Dotfiles entry is plain text, separated by a `·`. Both constraints are load-bearing:

- **The `<p>` is what puts the row on its own line.** `<picture>`, `<a>` and `<img>` are all inline, and a blank line between raw HTML blocks does *not* introduce block-level separation — without a block parent the row flows up beside the cards. The typing line is wrapped for the same reason; only the two card `<picture>`s are left unwrapped, deliberately, so they sit side by side.
- **No `style` attributes.** GitHub strips them from README HTML, so the icon cannot be vertically centred with CSS. It is sized to match the body text (16px) so that baseline alignment reads correctly instead.

Adding another link means another `<a>` in that same `<p>`, after a `·`.

## Workflow

Content changes are committed and pushed with git; the only automation is the card job above. Anything that renders on the profile page (README, SVG, funding config) should be eyeballed on GitHub after pushing, since there is no local check that can catch a sanitization or proxy issue.
