# Implementation summary

Base commit: `ee46738`. Original-state findings are in `PORTFOLIO_AUDIT.md`.
Changes are **uncommitted and unpushed** in the session workspace (pushing is refused: the Claude GitHub App is not installed on the repo).

## Files changed

| File | Change |
|---|---|
| `index.html` | Markup, CSS and JS edits described below. |
| `api/chat.js` | Moved from `chat.js` (git rename) so Vercel actually deploys it as `/api/chat`; prompt and input handling updated. |
| `PORTFOLIO_AUDIT.md`, `IMPLEMENTATION_SUMMARY.md` | New. |

Not touched: `admin.html`, `smart-exit-reminder.html`, `taxi-lost-item-detection.html`, `favicon.svg`, `photo.jpg`, all copy and case-study content.

## What was implemented

### Fixes
- **Chat endpoint now exists.** `chat.js` at the repo root was never served as a function. It is now `api/chat.js`. Whether the live AI answers still depends on `ANTHROPIC_API_KEY` being set in Vercel (see Remaining).
- **Hero image path fixed** (`assets/photo.jpg` 404 → `photo.jpg`); removed the `onerror` fallback; added `width`/`height`, `decoding="async"`, `fetchpriority="high"` and descriptive alt text.
- LinkedIn handle in the chat prompt corrected to `mouryadusi`; country count corrected to 9 (the list has nine); dissertation (AirSpaceGNN) added to the chat's facts using only figures already on the page, including its stated limitations.
- `api/chat.js` now caps each history turn at 2,000 characters and enforces user-first, alternating turns, which the Messages API requires.

### Mobile (≤700px)
- First screen leads with name, role line, summary and calls to action; the portrait follows. Clock bar compacted.
- Section padding cut roughly in half; decorative chapter curves hidden; project exhibits tightened.
- Selected Work, Credentials and Experiments become swipeable snap rows instead of tall stacks. The rows are keyboard-focusable named regions.
- Mobile menu fits a 390×844 screen without scrolling, includes a theme switch (the header one is hidden on phones), and hides floating buttons while open.
- Chat panel is bottom-anchored with `dvh` sizing, safe-area insets and a 16px input (prevents iOS focus zoom). Floating buttons respect safe areas.
- Touch devices get 44px minimum heights on the hamburger, close button, buttons and filters.

### Navigation and structure
- Skip link, `<main>` landmark, named navigation regions, screen-reader name "Mourya Dusi" on the `h1`.
- Current-section indicator in desktop and mobile nav using a single `IntersectionObserver` (no new scroll listener).
- Mobile menu: dialog semantics, `aria-expanded`, focus moved in/restored, Tab trapped, page scroll locked, Escape closes.
- Footer: Email / LinkedIn / GitHub links and a working back-to-top.
- Dark mode was already the default and still is; the toggle and `localStorage` key are unchanged.

### Chat assistant
- Dialog semantics, labelled input, `role="log"` live region, 44px-class controls, visible focus, focus moves to the input on open and back to the launcher on close, Escape closes.
- "New chat" reset; double-send guard; 20-second timeout.
- Honest wording: removed "trained on Mourya's profile"; when the live backend is unreachable the answer comes from the built-in guide and a one-time note says so (previously silent).
- Built-in guide gained a dissertation entry; suggested questions updated.

### Performance
- Three.js (decorative hero background) is loaded lazily after first paint and not at all on phones or with reduced motion.
- Dropped two unused font families (DM Sans, Anton) from the Google Fonts request.
- D3 call guarded so a failed CDN load no longer throws.

### Accessibility and contrast
- `--text-subtle` raised in both themes (dark ≈3.3–3.8:1 → ≥5.4:1; light 2.2:1 → ≥4.8:1).
- Fixed remaining contrast failures in the light theme and in two decorative mock-UI elements; named the previously unnamed personalisation button.

## Tests performed and results

All against a local static server, headless Chromium (Playwright), **simulated viewports only**.

| Check | Result |
|---|---|
| Horizontal overflow at 1440/1280/1024/768/430/390/375/360 | none at any width |
| Clipped-content scan (text extending past right edge; detector verified with a negative control) | none at any width |
| Interaction suite (menu, chat, reset, keyboard, skip link, landmarks, current-section, back-to-top, swipe rows): 24 checks | all pass |
| Own contrast/name/alt/label/duplicate-id/heading-order audit, dark and light, 1280 and 390 | 0 failures |
| `api/chat.js` with mocked upstream: 405 / 503 / 400 / 200, history sanitising, key never in request body or response | pass |
| `node --check api/chat.js` | pass |

Page height on a phone, measured with the CDN libraries blocked in the sandbox:

| Width | Before | After |
|---|---|---|
| 430 | 24,268 px (26.0 screens) | 17,414 px (18.7) |
| 390 | 24,703 px (29.3) | 17,803 px (21.1) |
| 375 | 25,014 px (37.5) | 17,911 px (26.9) |
| 360 | 25,264 px (34.1) | 18,119 px (24.5) |

That is about 28–29% less scrolling on phones. Desktop heights are essentially unchanged.

## Not tested / limitations (please read)

- **No physical devices.** Everything above is emulation. Foldable and dual-screen configurations were not tested separately; layouts are fluid and use `dvh`/safe-area units, but that is unverified on hardware.
- **Third-party scripts were blocked in the sandbox** (GSAP, ScrollTrigger, Lenis, Three.js, D3, Google Fonts). I never saw the page with them loaded, so pinned scroll sequences in the project overlay, smooth scrolling, the hero particle field, the globe, and the real fonts are untested here. The back-to-top path through Lenis is untested; the native fallback was.
- **Live site not inspected** (not reachable from this session).
- **No Lighthouse / Core Web Vitals run**, so no performance scores are claimed — only the structural changes listed.
- **No automated axe run** (package unreachable); the accessibility results come from my own checks and do not replace a screen-reader pass.
- Light theme was checked for contrast but not reviewed visually.
- Not done: the project-overlay's pinned "scroll controls the sequence" mechanic, the six-font cleanup beyond two families, splitting the 19 style / 21 script blocks, further trimming 18 sub-44px controls per viewport (many are inline links and decorative controls), and any rewrite of case-study copy.

## Remaining issues and configuration

1. **Set `ANTHROPIC_API_KEY`** in Vercel (Project Settings → Environment Variables) and redeploy; until then the assistant uses the built-in guide and says so.
2. **`/resume.pdf` is not in the repo.** Both résumé buttons currently fall back to "Request by email" (existing behaviour, preserved). Add the file at the repo root to restore the download.
3. **Phone number**: it is published in the contact section, while the chat prompt tells the assistant never to share a phone number. Your call which is right; I changed neither.
4. `favicon.ico` is referenced but missing (`favicon.svg` works).
5. Push access: install the Claude GitHub App on `mouryadusi/mouryadusipf`, or copy the changes yourself. No deployment was performed.
