# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A one-page marketing site for "Horizon Wealth Planning", delivered as a single self-contained `index.html`. CSS lives in one `<style>` tag and JS in one `<script>` tag. This is a hard constraint: no frameworks, no build tools, no extra files. The only external resource is Google Fonts (Fraunces, Instrument Sans). There are no third-party images: the hero art, icons and avatars are inline SVG and CSS.

`index.html` on `main` is version 2 of the site. Version 1 is not a file in the working tree; it is the `index.html` at commit `6675e85`, which the deploy workflow republishes unchanged.

## Commands

There is no build, lint or test tooling. To run the site, open the file in a browser:

```powershell
Start-Process "index.html"
```

## Deployment

`.github/workflows/pages.yml` runs on every push to `main` and publishes two pages to GitHub Pages:

- `/` is v1, read from git with `git show $V1_COMMIT:index.html`. Do not change `V1_COMMIT` unless the user asks to replace v1.
- `/v2/` is the current `index.html` with a Content-Security-Policy `<meta>` tag injected (see Security).

## Verifying changes in the Browser pane

- The pane renders the file as a `data:` snapshot, and navigating to the same URL again does not reload it, so old DOM state (for example a submitted form) persists. Append a new query string (`index.html?v=2`) to force a fresh load after edits.
- The pane's native width is about 320px. Use `resize_window` to 1280x800 for desktop checks and reset it afterwards.
- Reading `getComputedStyle` straight after toggling a class returns the pre-transition value (borders and colours transition over 0.25s); wait before asserting.
- While the pane is not displayed, animation frames are throttled: fade-ins and smooth scrolls can take seconds, so a screenshot may catch them half-way. Take it again before treating it as a bug.
- To test with the CSP active, run the Node script from the workflow's "Stage site files" step locally (`SRC=index.html OUT=<temp>/index.html`) and serve the output over `http://127.0.0.1`.

## Architecture

The CSS, HTML and JS are each divided by comment banners into matching sections (nav, hero, trust marquee, sliders, services rail, retirement calculator, reviews, enquiry form, footer, back-to-top). Keep that banner structure when adding code.

**CSS**
- All colours, spacing, radius, shadow, easing and nav height are custom properties on `:root`. Use the tokens rather than literal values.
- Mobile-first with exactly two breakpoints: `min-width: 768px` and `min-width: 1024px`.
- `.fade-in` only hides content under the `.js` class, which the script adds to `<html>`, so the page stays readable without JS. `data-delay="1"` to `"4"` staggers a reveal.
- `.js-only` hides things that cannot work without JS (the calculator, slider controls).
- A `prefers-reduced-motion` block disables transitions; the JS also checks it to skip count-up, fade-in and carousel auto-play.
- Icons are `<symbol>`s in the sprite at the top of `<body>`, used as `<svg class="icon"><use href="#i-name"/></svg>`. Add `icon-fill` for filled shapes.

**JS** (one IIFE)
- Sliders: `initSlider(root, { loop, autoplay })` drives both the services rail and the reviews carousel. The track is a native scroll-snap container, so touch and trackpad scrolling need no code; the JS adds arrows, dots, keyboard, mouse drag, auto-play and the progress bar. The number of visible cards is defined once, as `--per-view` in CSS (1.12, then 2 at 768px and 3 at 1024px); `perView()` reads it. There are `slides - perView + 1` positions, and dots are rebuilt when that changes.
- Calculator: the five range inputs live in the `calc` object and `updateCalc()` recomputes everything on `input`. Each input needs a matching `<id>-out` output element and a formatter in `calcFormat`.
- Enquiry form: validation is driven by the `validators` object, keyed by field `name`. Each key must have a matching `<name>-error` element in the HTML. Adding a field means adding the input, its error `<p>`, a validator, and an entry in the `data` object built on submit. `highlightTarget()` and `inputFor()` special-case the radio group and consent checkbox.
- Links with `data-interest="<option text>"` preselect the form's Area of Interest.
- Hero stats take their values from `data-target`, `data-prefix` and `data-suffix` attributes; the text in the markup is the no-JS fallback.

## Security

- The deployed page has a strict CSP: `default-src 'none'`, with the inline `<style>` and `<script>` allowed by SHA-256 hash. The workflow computes the hashes at deploy time and replaces the `<!-- CSP: ... -->` comment in `<head>`, so they cannot go stale. Keep that comment, and keep exactly one `<style>` and one attribute-less `<script>` tag or the deploy fails.
- Because of the CSP, these do not work on the deployed page: `style="..."` attributes, inline event handlers (`onclick=`), extra `<script>` tags, and images or requests to any other origin. Setting styles from JS (`el.style.setProperty`) is fine. A new external origin must be added to the policy in the workflow.
- The CSP also sets `require-trusted-types-for 'script'`, so `innerHTML`, `outerHTML` and `insertAdjacentHTML` throw. Build DOM with `createElement`, `textContent`, `replaceChildren` and `<template>` (see `showSuccess`).
- User input is only ever written with `textContent`. Form values go through `clean()` (strips control characters, trims, caps length) and are never logged to the console.
- The form has a honeypot field (`company`); a submission that fills it is dropped silently.
- All of this is client-side. If a backend is added, it must repeat the validation, and add rate limiting and CSRF protection.

## Content notes

The address, phone number, email address and social links (`#`) are placeholders. The firm, the reviews, the rating figures and the case study are fictional samples, and the page says so in the "About these reviews" note. Keep that note, never present the sample reviews as verified, and replace them only with real reviews the client has consent to publish.
