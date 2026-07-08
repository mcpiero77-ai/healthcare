# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

This is a single-page healthcare clinic marketing site ("Willow Creek Family Clinic"). The entire site — markup, styles, and behavior — lives in one file: `index.html`. There is no build step, no package manager, and no dependencies beyond a Google Fonts `<link>`.

## Development

There is no build/lint/test tooling in this repo. To work on the site:
- Open `index.html` directly in a browser (double-click, or `Start-Process index.html` on Windows) — no server required.
- After any change, manually re-verify in the browser: sticky nav + smooth scroll to each section, mobile hamburger menu at narrow widths, scroll fade-in animations, and both forms — the appointment form (`#appointmentForm`) and the checklist lead-magnet form (`#checklistForm`) — each with an empty submit, invalid email, and valid submit, including the console-logged payload on success.

## Architecture

Everything is inline inside `index.html`, organized top-to-bottom in three blocks:

- **`<style>` in `<head>`**: CSS custom properties on `:root` (`--color-*`, `--font-*`, `--radius`, `--nav-height`, etc.) drive the whole visual system — change the palette/spacing there rather than hardcoding values in rules. Layout is mobile-first with `@media (max-width: ...)` breakpoints per section. Scroll-triggered reveals use a `.fade-in` / `.fade-in.visible` class pair rather than animation libraries.
- **`<body>`**: semantic sections in a fixed order, each with an `id` used for nav-anchor scrolling: `header.navbar` → `#hero` → `#services` → `#checklist` (lead magnet) → `#testimonials` → `#contact` (appointment form) → `footer`. Section boundaries are marked with `<!-- ===== SECTION ===== -->` comments — keep that convention when adding sections.
- **`<script>` at the end of `<body>`**: five independent, self-contained behaviors — footer year fill, mobile nav toggle, an `IntersectionObserver` that adds `.visible` to `.fade-in` elements (with a fallback that reveals everything if `IntersectionObserver` is unsupported), the appointment form handler, and the checklist (lead magnet) form handler. `showFieldError()` is the one shared helper between the two form handlers.

### Form validation pattern

Both the appointment form (`#appointmentForm`) and the checklist lead-magnet form (`#checklistForm`) are validated entirely client-side via a `fields` object in the script, keyed by field name, each with `{ input, error, validate() }`. `validate()` returns `''` for a valid value or an error string. On submit: every field is validated, `aria-invalid` and the paired `.field-error` span are set per-field (via the shared `showFieldError()` helper), and the first invalid field is focused. There is no real backend — on success each handler builds a `formData` object, `console.log`s it, and shows its inline `.form-status` success banner (`#formStatus` / `#checklistStatus`). The comment block above each `console.log` marks where a real `fetch(...)` POST should be added later; wire it in there rather than restructuring the handler.

When adding a new form field, follow the existing pattern: an `input`/`textarea` with a matching `<span class="field-error" id="...Error" role="alert">`, plus an entry in the relevant `fields` object with its own `validate()`. When adding a whole new form, mirror `#checklistForm`: its own `fields` object, its own `.form-status` element, reusing `showFieldError()`.
