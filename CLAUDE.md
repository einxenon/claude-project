# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A one-page marketing site for "Horizon Wealth Planning", delivered as a single self-contained `index.html`. CSS lives in one `<style>` tag and JS in one `<script>` tag. This is a hard constraint: no frameworks, no build tools, no extra files. External resources are limited to Google Fonts (Playfair Display, Inter), one Unsplash hero image, and `i.pravatar.cc` avatars.

## Commands

There is no build, lint or test tooling. To run the site, open the file in a browser:

```powershell
Start-Process "index.html"
```

The folder is not a git repository.

## Verifying changes in the Browser pane

- The pane renders the file as a `data:` snapshot, and navigating to the same URL again does not reload it, so old DOM state (for example a submitted form) persists. Append a new query string (`index.html?v=2`) to force a fresh load after edits.
- The pane's native width is about 320px. Use `resize_window` to 1280x800 for desktop checks and reset it afterwards.
- Reading `getComputedStyle` straight after toggling a class returns the pre-transition value (borders and colours transition over 0.25s); wait before asserting.

## Architecture

The CSS, HTML and JS are each divided by comment banners into matching sections (nav, hero, testimonials carousel, enquiry form, footer, back-to-top). Keep that banner structure when adding code.

**CSS**
- All colours, spacing, radius, shadow and nav height are custom properties on `:root`. Use the tokens rather than literal values.
- Mobile-first with exactly two breakpoints: `min-width: 768px` and `min-width: 1024px`.
- `.fade-in` only hides content under the `.js` class, which the script adds to `<html>`, so the page stays readable without JS.
- A `prefers-reduced-motion` block disables transitions; the JS also checks it to skip count-up, fade-in and carousel auto-play.

**JS** (one IIFE)
- Carousel: the number of visible cards is defined twice and must stay in sync: `--per-view` in CSS (1, then 3 at 1024px) and `perView()` in JS, which reads the same `(min-width: 1024px)` media query. Dots are rebuilt when that query changes; there are `slides - perView + 1` positions.
- Enquiry form: validation is driven by the `validators` object, keyed by field `name`. Each key must have a matching `<name>-error` element in the HTML. Adding a field means adding the input, its error `<p>`, a validator, and an entry in the `data` object built on submit. `highlightTarget()` and `inputFor()` special-case the radio group and consent checkbox.
- The success panel inserts the user's name with `textContent`; keep user input out of `innerHTML`.
- Hero stats take their values from `data-target`, `data-prefix` and `data-suffix` attributes; the text in the markup is the no-JS fallback.

## Content notes

The address, phone number, email address and social links (`#`) are placeholders. Testimonials are fictional.
