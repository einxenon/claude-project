# claude-project

A one-page marketing site for **Horizon Wealth Planning**, a fictional financial planning firm offering retirement, investment and insurance planning. The whole site is a single self-contained `index.html` with no frameworks and no build step.

**Live site:** https://einxenon.github.io/claude-project/

## What is on the page

- **Navigation** with links to the Home, Testimonials and Contact sections
- **Hero** with the headline "Plan Today. Prosper Tomorrow." and three statistics that count up when they scroll into view
- **Testimonials carousel** showing one card on small screens and three from 1024px, with dots, auto-play, arrow-key and swipe support
- **Enquiry form** with client-side validation and a success panel. There is no backend: a submitted enquiry is only logged to the browser console.
- **Footer** with quick links, services, a newsletter sign-up and social links
- **Back-to-top** button

The page is mobile-first with breakpoints at 768px and 1024px. It respects `prefers-reduced-motion`, and stays readable with JavaScript turned off.

## Run it locally

Open `index.html` in a browser. On Windows:

```powershell
Start-Process "index.html"
```

There is nothing to install, build, lint or test.

## Project structure

| Path | Purpose |
| --- | --- |
| `index.html` | The whole site: markup, one `<style>` tag and one `<script>` tag |
| `.github/workflows/pages.yml` | Deploys the site to GitHub Pages |
| `.claude/commands/publish.md` | Claude Code `/publish` command that scans, pushes and deploys this repo |
| `CLAUDE.md` | Guidance for Claude Code when working in this repo |

Keeping everything in one file is a hard constraint. The only external resources are Google Fonts (Playfair Display and Inter), one Unsplash hero image and `i.pravatar.cc` avatars.

## Deployment

Every push to `main` runs the **Deploy to GitHub Pages** workflow, which can also be started manually from the Actions tab. It publishes only `index.html`; the README, `CLAUDE.md` and the `.claude` folder are not part of the deployed site.

## Placeholder content

The address, phone number, email address and social links are placeholders, and the testimonials are fictional.
