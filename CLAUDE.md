# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**MBDX KMU Scout** is a German-language, single-page B2B sales intelligence tool. It guides sales reps through a 6-step structured research workflow on small-to-medium enterprises (KMUs) before first contact, then generates a formatted briefing with export options.

## Architecture

This is a **zero-dependency, single-file application**: everything lives in `index.html` (~1,200 lines). There is no build step, no package manager, no framework, and no separate JS or CSS files.

- **CSS** (~590 lines): embedded in `<style>`. Uses CSS custom properties for theming (light/dark). Layout via flexbox/grid. Breakpoint at `600px` for mobile.
- **JavaScript** (~440 lines): inline `<script>`. Vanilla ES6+. No modules.
- **Persistence**: only the theme preference is saved (`localStorage`). Form data is not persisted across page reloads.

### 6-Step Wizard

The core UX is a sequential 6-panel wizard. All panels are rendered in the DOM simultaneously; `showStep(n)` toggles visibility via `display: none/block` on `.step-panel` elements. The current step index is tracked in the `current` variable (0–5).

| Step | ID | Topic |
|------|----|-------|
| 1 | Basisdaten | Company basics (name, industry, size, contact, location) |
| 2 | Selbstbild | Company self-image from website |
| 3 | Innenleben | Internal culture from Kununu/Indeed reviews |
| 4 | Bewegung | Growth signals (job postings, LinkedIn) |
| 5 | Außenwahrnehmung | External perception (press, customer reviews) |
| 6 | Briefing | Auto-generated summary + action plan |

### Key JavaScript Functions

- `showStep(n)` — navigates between panels, triggers `buildPrompts()` and `renderBriefing()` as needed
- `buildPrompts()` — reads Step 1 inputs (company name, contact, location) and injects them into the AI prompt templates shown in Steps 2–5
- `renderBriefing()` — collects all form values and builds the HTML briefing in Step 6; called each time Step 6 is shown
- `copyPrompt(btn)` — clipboard copy with visual feedback for AI prompt boxes
- `saveBriefing()` — downloads Step 6 content as a `.txt` file named `MBDX_Scout_[FirmName].txt`
- `toggleTheme()` — switches `data-theme` attribute on `<html>`, persists to localStorage
- `getVal(id)` — safe helper to get trimmed input/textarea values

### Color System

Steps use distinct accent colors that propagate to borders, badges, and focus rings:
- **Green** (`--green`): Steps 1, 3, 6
- **Purple** (`--purple`): Steps 2, 5
- **Orange** (`--orange`): Step 4

Both light and dark themes redefine all CSS variables under `[data-theme="dark"]`.

## Development Workflow

No build or install step required. Open `index.html` directly in a browser, or serve it with any static file server:

```bash
# Simple local server options
python3 -m http.server 8080
npx serve .
```

There are no tests, no linter, and no CI configuration.

## Editing Conventions

- All UI text is in **German**. Keep new strings in German.
- When adding a new step input, also update `buildPrompts()` if Step 1 data feeds into it, and `renderBriefing()` if it should appear in the Step 6 summary.
- Print styles (`@media print`) hide all UI chrome and show only Step 6 — update these if Step 6 structure changes.
- The tab bar (`#tabs`) and progress bar are driven by `showStep()`; any new step requires updating both the tab rendering logic and the step count.
