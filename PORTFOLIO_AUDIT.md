# Portfolio Audit — original state

Audited commit: `ee46738` ("Update index.html"), 30 Oct 2026.
This document describes the site **before** any change in this pass.

## 0. Scope and limits of this audit

What was actually done:

- Read the repository: `index.html`, `chat.js`, `admin.html`, the two project pages, assets.
- Rendered the original `index.html` in headless Chromium (Playwright) at 1440, 1280, 1024, 768, 430, 390, 375 and 360 px and measured overflow, page height, console errors, failed requests and touch-target sizes.

What could **not** be done, and why:

- **Live site not inspected.** `https://mouryadusi.vercel.app` is not on this session's network allowlist, and browser control is not available on this plan. Everything below about production behaviour is inferred from the repo. Anything about the deployed site that differs from the repo (for example a `resume.pdf` uploaded through some other route) is unknown.
- **CDN scripts were blocked in the test sandbox** (GSAP, ScrollTrigger, Lenis, Three.js, D3, Google Fonts all returned `ERR_TUNNEL_CONNECTION_FAILED`). Measurements therefore reflect the page *without* those libraries. Page heights and animation behaviour in production will differ somewhat. The `d3 is not defined` error seen in the baseline is a sandbox artefact, not a production bug.
- **No physical devices.** All viewport results are simulated.
- **Pushing to GitHub is refused** from this session (the Claude GitHub App is not installed on the repository). Changes live in the session workspace only.

## 1. Architecture

| Item | Finding |
|---|---|
| Framework | None. Static HTML, no build step, no `package.json`, no `vercel.json`. |
| Pages | `index.html` (4,055 lines, 258 KB), `smart-exit-reminder.html`, `taxi-lost-item-detection.html`, `admin.html`. |
| Routes | Single-page anchor navigation plus an in-page project-detail overlay. |
| Styling | 19 `<style>` blocks, ~160 inline `style=""` attributes, ~90 `!important`. Design tokens exist (`:root` variables) but are bypassed in many inline styles. |
| Scripts | 21 `<script>` blocks. 10 inline `onclick=` attributes. |
| Third-party JS | GSAP + ScrollTrigger, Lenis, Three.js r128, D3 v7, all render-blocking in `<head>` or loaded from CDNs. |
| Fonts | Six families requested in one Google Fonts call (Inter, DM Sans, Playfair Display, Cormorant Garamond, Bodoni Moda, Anton). |
| Backend | `chat.js` at the **repository root**, written as a Vercel serverless function calling the Anthropic Messages API. |
| Env vars | `ANTHROPIC_API_KEY` (server side only; not exposed to the client). |

### Defects found

1. **The chat backend is never deployed.** Vercel only treats files in `/api` as functions. `chat.js` at the root is served as a static file, so `POST /api/chat` cannot reach it. The front end swallows the failure and silently uses the local keyword assistant, so the site "works" but the LLM path is dead code regardless of whether the key is set.
2. **Hero image path is wrong.** `<img src="assets/photo.jpg">` 404s (there is no `assets/` folder; the file is `photo.jpg`). It only renders because of an `onerror` fallback, which costs a failed request on every load and prevents the browser from prioritising the LCP image.
3. **`/favicon.ico` is referenced but absent.** (`favicon.svg` exists.)
4. **`/resume.pdf` is not in the repository.** The script at the end of `index.html` already degrades both résumé buttons to a "request by email" link when the file is missing. That graceful handling is good and is preserved.
5. **Stale LinkedIn handle in the chat prompt.** `chat.js` says `linkedin.com/in/mouryadusi-dxb`; the site's contact section links `linkedin.com/in/mouryadusi`, which is the correct one.
6. **Chat system prompt is out of date with the site.** It does not mention the AirSpaceGNN case study that the page presents as a project.

## 2. Visual design assessment

Strengths worth keeping: a warm, editorial dark palette; Bodoni Moda / Playfair display type; a restrained accent system; a signature wordmark; chapter-colour accents. Dark mode is **already the default** (`data-theme="dark"` on `<html>`, `localStorage.theme || 'dark'`).

Weaknesses:

- Too many ideas competing: custom cursor with labels, WebGL node field, chapter curves, scene rail, loader with three languages, pinned scroll sequences in project detail, a globe, an accessibility "personalisation" panel with ~25 filters. Together they dilute the editorial minimalism the site is aiming for.
- Six font families is more than the hierarchy needs and costs load time.
- `--text-subtle` is `#7c6f5a` on `#17130f` in dark and `#aaa` on `#fafaf8` in light. The light value is about 2.3:1 contrast and fails WCAG AA for the small labels it is used on.
- Inline styles on the Selected Work cards duplicate values instead of using tokens, so hover/focus states cannot be refined centrally.

## 3. UX and usability

- First screen on mobile does not say who the person is. At 390 px the first viewport is the clock bar and the portrait; the name and role are below the fold, and the "Hey — I'm" line visibly collides with the identity line.
- The Dubai/local clock bar takes a full row at the top of every phone viewport for low-value information.
- "Selected Work" (3 cards) and "Projects" (6 exhibit articles) present overlapping content in two long, separate sections.
- Section order is Work → Experience → What I Do → Skills → About → Building → Experiments → Contact: eight nav items, which is long for a mobile menu.
- Pinned, scroll-controlled sequences in the project overlay ("Scroll controls the sequence") are a form of scroll-jacking and hard to use with assistive tech.

## 4. Mobile and responsive weaknesses (measured)

Baseline, original page, CDN libraries blocked, full-page document height divided by viewport height:

| Viewport | Document height | Screens of scrolling | Horizontal overflow |
|---|---|---|---|
| 1440×900 | 20,053 px | 22.3 | none |
| 1024×768 | 19,457 px | 25.3 | none |
| 768×1024 | 20,780 px | 20.3 | none |
| 430×932 | 24,268 px | 26.0 | none |
| 390×844 | 24,703 px | 29.3 | none |
| 375×667 | 25,014 px | 37.5 | none |
| 360×740 | 25,264 px | 34.1 | none |

- Phones scroll **26–37 screens**; the page is *taller* on a phone than on a desktop. This confirms the stated weakness.
- Section padding is a flat `100px` top/bottom, mostly unchanged on phones.
- 14–22 interactive elements per viewport are smaller than 44×44 px.
- Chat panel uses `100vw`/`vh` units and a 14px input, so iOS Safari zooms on focus and the dynamic toolbar can clip the panel.
- No horizontal overflow at any tested width (good).

## 5. Performance

- Four large third-party libraries on the critical path. Three.js r128 (~600 KB) is used only for a decorative background that is already disabled on small touch devices, yet it is still downloaded there.
- Scripts are in `<head>` without `defer`.
- One failed image request on every load (`assets/photo.jpg`).
- Hero portrait `photo.jpg` is 205 KB with no `width`/`height`, no `fetchpriority`, no `decoding`.
- Fonts: six families, many weights.
- Five scroll listeners (plus Lenis/ScrollTrigger) and six IntersectionObservers.

## 6. Accessibility

Good already: `lang`, a single `<h1>`, 13 `<h2>`, visible `:focus-visible` outline, `prefers-reduced-motion` handled in 14 places, an accessibility personalisation panel, Escape closes overlays.

Gaps:

- No skip link and no `<main>` landmark.
- `<nav>` has no accessible name; hamburger has no `aria-expanded` or `aria-controls`; mobile menu close button is a bare `×`.
- Mobile menu does not trap or restore focus and is not marked as a dialog.
- **Chat widget:** launcher is an emoji `<button>` with only `title`; panel has no role, no label, no focus management; message list is not a live region; input has no label; no way to reset the conversation; no error state (errors silently fall back to the local assistant).
- Project-detail overlay is `aria-hidden="true"` and used as a modal without focus management.
- Contrast failure on `--text-subtle` in light theme (see §2).
- `cursor: none` on `body` removes the native pointer on desktop; acceptable as a design choice because a replacement cursor is drawn, but it fails if the script fails.

## 7. AI / chat evaluation

Front end (`index.html`): bubble + panel injected by script, four suggested questions, `fetch('/api/chat')` with the last 16 turns, fallback to a local keyword matcher (`getAIReply`), artificial 400–700 ms minimum delay, plain `textContent` rendering.

Back end (`chat.js`): well structured. Input length cap (2,000 chars), history trimmed to 8, only `user`/`assistant` roles accepted, key read from `process.env`, 503 when unconfigured, generic error messages (no leakage). The system prompt is good: intent-first, proportional answers, explicit "never invent" rules, a verified-facts block, and refusal to reveal instructions.

Problems:

- Endpoint not deployed (see §1.1).
- No streaming (so none to preserve).
- The opening message claims "an AI trained on Mourya's profile". It is a general model given a fact sheet, not trained on anything.
- Panel header says "Ask me anything about my work" (first person) while the prompt instructs the model to speak *about* Mourya in the third person to visitors.
- Prompt omits AirSpaceGNN / the dissertation; contains a stale LinkedIn handle.
- History entries are not individually length-capped on the server.
- The prompt forbids sharing a phone number while the contact section of the site publishes one. That is a content decision for the owner, not something to change silently.

## 8. Prioritised improvements

**P0 — correctness**
1. Move `chat.js` to `api/chat.js` so the endpoint exists.
2. Fix the hero image path; add dimensions and priority hints.
3. Correct the LinkedIn handle and add AirSpaceGNN to the chat knowledge.

**P1 — mobile and first impression**
4. Put identity first on phones; compact the clock bar.
5. Cut vertical padding and decorative elements on phones.
6. Make the Selected Work cards a compact horizontal scroller on phones.
7. Bottom-safe, `dvh`-based chat panel with 16px input.

**P2 — accessibility**
8. Skip link, `<main>`, named navs, correct menu semantics and focus handling.
9. Chat dialog semantics, live region, label, reset, error state.
10. Contrast fix for `--text-subtle`.

**P3 — performance**
11. Defer non-critical scripts; do not load Three.js on phones or with reduced motion.
12. Trim the font request to families the page actually uses.
13. Guard D3 so a blocked CDN cannot throw.

**Not in scope for this pass (and why)**
- Splitting the monolith into components or adding a framework. The brief asks to preserve architecture; a rewrite of 4,000 lines of hand-tuned markup without a build pipeline would risk regressions I cannot fully test here.
- Rewriting case-study copy. Only facts already on the site can be used and none were invented.

## 9. Design and engineering decisions

- Keep the static-site architecture; make changes as a clearly delimited block of overrides plus targeted edits, so the original rules stay readable and reversible.
- Keep dark mode as the default; keep the existing toggle and `localStorage` key.
- Keep the existing résumé fallback.
- Do not add dependencies, services or credentials.
