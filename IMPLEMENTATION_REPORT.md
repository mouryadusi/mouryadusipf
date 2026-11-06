# Implementation report: Mourya Dusi portfolio, final pass

Starting point: the uploaded source files plus `portfolio-updated.zip` (MD monogram, theme transition, MSc-completed wording).
GitHub (`mouryadusi/mouryadusipf`) was not reachable from my sandbox, so everything was done on the uploaded files. Nothing was pushed or deployed.

## Completed

| # | Item | What was done |
|---|---|---|
| 1 | Project gallery | All six exhibit thumbnails replaced with individually composed layouts built only from real figures. destination and AirSpaceGNN now use your real site screenshots (browser chrome cropped off, assets in `assets/projects/`) layered under the real feature labels and figures: 92% numeral + chips (Performance ML); three offset metric strips (Chatbot); stacked feature tiles (destination); Wi-Fi/GPS/BLE converging into a reminder, with path-draw (Smart Exit); concentric sensor rings (Lost-Item); struck-through 0.9994 vs 0.6898 (AirSpaceGNN). Each uses a mask/clip-path reveal on scroll. |
| 2 | Contextual cursor | Reuses the existing ring and its `cursor-view` states. Labels are now contextual: "Read case" / "Visit" on project visuals, "Open" on Selected Work cards, "Visit" on experiments, "Email" / "Copy" on contact actions. Fine-pointer devices only. Keyboard and native click are unchanged. |
| 3 | Animated numbers | Real odometer digit columns (scroll-triggered once, staggered), applied to 92%, 1,000+, 99.9%, −35%, 0.6898, 81,270, 2.2% and the Selected Work metrics. Values are unchanged and exposed via `aria-label`. Subtle hover lift. |
| 4 | Reading time | Computed from each project's actual description plus case-study text at 210 wpm, rounded up, minimum 1 (words: Performance ML 121, Chatbot 101, destination 55, Smart Exit 16, Lost-Item 17, AirSpaceGNN 200). All six come to 1 min. Shown on project cards, Selected Work cards and inside each case study. |
| 5 | Case studies | Bodies restyled as numbered chapters with staggered entrance. The empty "Model Comparison" heading in case 01 was removed (it had no content or data). |
| 6 | Navigation | Eight items became five (Work, Experience & Skills, About, Building & Experiments, Contact), in the desktop and mobile menus. In-section chip sub-navigation keeps every merged section reachable. The About link now scrolls to the section instead of opening the duplicate overlay. |
| 7 | Experience & Skills | Experience, What I Do, Skills and Education/credentials are one group (labels 02), with joined spacing and a sub-nav. |
| 8 | About | The duplicate full-page About overlay and the "a few fragments" scrapbook (which repeated location, degree, employer and countries) were removed. |
| 9 | Building & Experiments | One group (labels 04) with sub-nav. |
| 10 | Contact | Added a short availability line, an "Email me" button and a "Copy email" button with a live-region confirmation. All existing contact rows are kept. |
| 11 | Architecture diagrams | Three SVG diagrams of real pipelines only: Performance ML, University Chatbot, AirSpaceGNN dissertation. They draw in when the case study opens, use theme variables, and are readable at 360px. |
| 12 | Hover interactions | Layered pointer parallax and tilt per thumbnail, per-project hover transforms, title shift, arrow movement, magnetic pull on primary actions. |
| 13 | Mobile | On touch devices the hover effects are replaced by scroll-linked depth offsets (one passive scroll listener, visible items only). Existing swipe rows are kept. |
| 14 | Footer | The monogram's box is untouched. The footer is taller so "Dubai, UAE" and the other footer text no longer sit under the monogram (the old signature had the same collision). |
| 15 | chat.js | Moved to `api/chat.js` so Vercel serves `/api/chat`. Stale LinkedIn handle corrected. `ANTHROPIC_API_KEY` must still be set in Vercel. |
| 16 | Phone number | Your published number stays on the page. The assistant prompt and the on-page assistant now share it when asked, instead of refusing. |
| 17 | Theme / reduced motion | See tests. All new motion has a `prefers-reduced-motion` path (no clip reveal, immediate digits, drawn diagrams). |

Other changes: country count in Quick Facts set from "10 visited" to "9 visited" to match the nine countries listed on the page and in the chat facts. If a tenth was visited, add it to the list and revert.

## Changed files
`index.html` (markup, CSS block `#redesign-final-css`, script `#redesign-final-js`), `api/chat.js` (moved from `chat.js`, prompt edits). Unchanged: `admin.html`, project pages, `favicon.svg`, `photo.jpg`.

## Actual test results
Headless Chromium (Playwright), local file, viewports 1440, 768, 390, 360, 2560 and 1440 with reduced motion.
- 5 nav links (desktop) and 5 mobile menu links at every width.
- 3 diagrams present; the AirSpaceGNN diagram opened and showed all 6 nodes (desktop and 390).
- Odometers: displayed digits match the stated value for every number at 1440, 768, 2560 and reduced motion. At 390/360 one Selected Work metric sits in an unswiped horizontal row, so it stays at 0 until swiped (my test did not swipe).
- No horizontal overflow at any width. No page errors at any width.
- No footer text overlaps the monogram at any width.
- Cursor over a project visual showed the "Read case" state; pointer parallax variable updated.
- Dark to light to dark theme toggle worked; monogram stroke colour changed and the draw did not replay.
- Screenshots reviewed: all six thumbnails, footer, light-theme contact section, the AirSpaceGNN case study with diagram.

## Remaining limitations
- Real screenshots are used for destination and AirSpaceGNN only. The other four projects (Performance ML, Chatbot, Smart Exit, Lost-Item) have no screenshots, so they remain typographic/data compositions. The AirSpaceGNN crop shows only the header and headline: the full capture showed a "deployed backend is unavailable" message in the demo card, so I left it out. Check that the live demo backend on Render is awake.
- Every reading time is 1 min because each project's text is under 210 words of reading; that is the real result.
- CDN scripts (GSAP, Lenis, Three.js, D3, Google Fonts) were not loaded in my sandbox. The page was never seen with them.
- The project-detail overlay (`#project-detail`) was not changed and has no reading time.
- Experience & Skills, About and Building & Experiments were merged by relabelling, joined spacing and sub-navigation, not rewritten from scratch.
- The case-study fields Role, Research and Learnings appear only where your existing text covers them. Nothing was invented.
- Not tested: physical devices, foldables, Safari/Firefox (the circular theme reveal uses View Transitions, and other browsers get a colour cross-fade), Lighthouse, screen readers, the live site.
- Light theme was spot-checked (contact section) but not reviewed section by section.
- Set `ANTHROPIC_API_KEY` in Vercel and add `resume.pdf` for the résumé buttons.
