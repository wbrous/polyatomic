# Mobile Support for Polyatomic Ion Memorizer

## TL;DR

> **Quick Summary**: Transform the Polyatomic Ion Memorizer from a desktop-focused single-page quiz app into a fully mobile-optimized, installable PWA that works offline — targeting phones (iPhone SE → Pro Max, Android) and tablets (iPad).
> 
> **Deliverables**:
> - Responsive layout working flawlessly across all 3 views on mobile viewports
> - PWA with offline support (manifest.json, sw.js, icons)
> - Mobile-native UX: haptic feedback, install prompt, pull-to-refresh prevention, touch-optimized inputs
> - Working submit button (currently dead code)
> - iOS-specific workarounds (focus persistence, custom install banner)
> 
> **Estimated Effort**: Medium
> **Parallel Execution**: YES — 4 waves
> **Critical Path**: T1/T2 (PWA files) → T3 (responsive layout) → T4 (mobile JS) → T5 (QA)

---

## Context

### Original Request
User requested "creation of mobile support" for the Polyatomic Ion Memorizer.

### Interview Summary
**Key Discussions**:
- **Scope**: Full mobile experience — responsive + PWA + mobile-native UX (not just responsive fixes)
- **Target devices**: Phones (iPhone SE → Pro Max, all Android) + tablets (iPad range). iOS Safari and Chrome Android.
- **Architecture**: Keep single-file `index.html`. Only add `manifest.json`, `sw.js`, and icon assets as new files.
- **Mobile UX features selected**: Haptic feedback, pull-to-refresh prevention, install-to-homescreen prompt, offline support, touch-optimized inputs
- **Mobile UX features rejected**: Swipe navigation between views
- **Tests**: No automated test framework. Agent-executed QA via Playwright on mobile viewports.

**Research Findings**:
- App is ~400 lines in a single `index.html` with inline `<style>` and `<script>`, using Tailwind CSS via CDN
- Already has viewport meta tag and some Tailwind breakpoints (`sm:`, `md:`)
- All quiz data is hardcoded in JS (`ionData` array) — no API calls, perfect for offline
- Three views: Selection Menu, Quiz Loop, Assessment

### Metis Review
**Identified Gaps** (all addressed):
- **Tailwind CDN is a JS runtime, not a static CSS file** — highest-risk item for offline support. Must validate caching works. Fallback: if CDN script fails offline, a minimal inline `<style>` provides basic usability.
- **`navigator.vibrate()` not supported on iOS** — haptic feedback will be Android-only. Accepted as platform limitation.
- **`beforeinstallprompt` not supported on iOS** — need two code paths: native prompt for Android Chrome, custom instructional banner for iOS Safari.
- **`quizInput.focus()` inside `setTimeout` fails on iOS Safari** — keyboard dismisses after each correct answer. Must restructure to chain focus from user gesture.
- **Submit button (`btn-submit`) is dead code** — exists in HTML with `hidden` class but no JS ever shows it. Must be made always-visible and wired to `handleSubmission()`.
- **`min-h-[500px]` container overflows iPhone SE** — viewport height after Safari chrome is ~460px. Must remove or make responsive.
- **Body `flex items-center justify-center` fights virtual keyboard** — card gets squeezed when keyboard opens. Must adjust to `items-start` on mobile.
- **Input font < 16px triggers iOS auto-zoom** — all inputs must be ≥ 16px.
- **PWA icons needed** — manifest requires 192×192 and 512×512 PNGs, plus `apple-touch-icon`.
- **HTTPS required for service workers** — noted in plan; GitHub Pages / Netlify / localhost all provide this.
- **iPhone notch / Dynamic Island** — need `viewport-fit=cover` + `safe-area-inset-*` padding for standalone PWA mode.
- **Landscape orientation** — lock to portrait via manifest `"orientation": "portrait"` for simplicity.

---

## Work Objectives

### Core Objective
Make the Polyatomic Ion Memorizer work beautifully on mobile phones and tablets, installable as a PWA with full offline support and native-feeling UX enhancements.

### Concrete Deliverables
- `manifest.json` — PWA web app manifest
- `sw.js` — Service worker with offline caching strategy
- `icon-192.png`, `icon-512.png`, `apple-touch-icon.png` — PWA icon assets
- Updated `index.html` — responsive layout, mobile CSS, PWA integration, mobile JS features

### Definition of Done
- [ ] App renders correctly on iPhone SE (375×667), iPhone 14 (390×844), iPad (768×1024) — no horizontal overflow, all views functional
- [ ] App installs as PWA and works fully offline (all 3 views, complete quiz flow)
- [ ] Submit button visible and functional on quiz view
- [ ] Touch targets ≥ 44×44px on all interactive elements
- [ ] Haptic feedback fires on Android for correct/incorrect answers
- [ ] Install prompt appears (native on Android Chrome, custom banner on iOS Safari)
- [ ] Pull-to-refresh prevented during quiz
- [ ] Virtual keyboard does not obscure quiz input
- [ ] Desktop layout unchanged

### Must Have
- Responsive layout across all 3 views for phone + tablet viewports
- PWA manifest with icons, service worker with offline caching
- Working submit button (currently dead code)
- Touch-optimized inputs and targets (≥ 44×44px, ≥ 16px font on inputs)
- Haptic feedback on correct/incorrect (Android; graceful no-op on iOS)
- Install prompt (native Android + custom iOS banner)
- Pull-to-refresh prevention during quiz
- iOS Safari focus() workaround for quiz input persistence
- Safe area handling for notched devices in standalone mode
- Portrait orientation lock via manifest

### Must NOT Have (Guardrails)
- DO NOT change quiz logic (answer validation, scoring, question selection, ion data)
- DO NOT add a build step, npm, bundler, or any tooling
- DO NOT split app into multi-file JS/CSS architecture (only add manifest.json, sw.js, icons)
- DO NOT add localStorage/IndexedDB persistence or progress saving
- DO NOT add dark mode or `prefers-color-scheme` support
- DO NOT conduct a comprehensive accessibility audit (we're optimizing mobile UX, not adding ARIA/screen-reader support)
- DO NOT add splash screen images for iOS PWA (rabbit hole)
- DO NOT add service worker update notification UI ("New version available")
- DO NOT add audio feedback as iOS haptics fallback
- DO NOT add swipe gestures or new animations/transitions beyond existing `fade-in`
- DO NOT break existing desktop layout — all responsive changes must be mobile-first or use breakpoint prefixes
- DO NOT refactor the imperative DOM JS style into classes, modules, or state management

---

## Verification Strategy

> **ZERO HUMAN INTERVENTION** — ALL verification is agent-executed. No exceptions.

### Test Decision
- **Infrastructure exists**: NO
- **Automated tests**: None
- **Framework**: None

### QA Policy
Every task includes agent-executed QA scenarios using Playwright on mobile viewports.
Evidence saved to `.sisyphus/evidence/task-{N}-{scenario-slug}.{ext}`.

- **All tasks**: Playwright with mobile viewport sizes — iPhone SE (375×667), iPhone 14 (390×844), iPad (768×1024)
- **PWA tasks**: Playwright offline mode (`context.setOffline(true)`) after service worker install
- **Touch targets**: Verified programmatically via `element.getBoundingClientRect()` — assert `width >= 44 && height >= 44`
- **Overflow check**: Verified via `document.documentElement.scrollWidth <= document.documentElement.clientWidth`

---

## Execution Strategy

### Parallel Execution Waves

```
Wave 1 (Start Immediately — new files, fully parallel):
├── Task 1: PWA Manifest + Icon Assets [quick]
└── Task 2: Service Worker [unspecified-high]

Wave 2 (After Wave 1 — index.html structure):
└── Task 3: Responsive Layout + PWA Head Tags [visual-engineering]

Wave 3 (After Wave 2 — index.html JS):
└── Task 4: Mobile JavaScript Features [deep]

Wave 4 (After Wave 3 — verification):
└── Task 5: Full Mobile QA + Offline Testing [unspecified-high + playwright]

Wave FINAL (After ALL tasks — independent review, 4 parallel):
├── Task F1: Plan compliance audit (oracle)
├── Task F2: Code quality review (unspecified-high)
├── Task F3: Real manual QA (unspecified-high + playwright)
└── Task F4: Scope fidelity check (deep)

Critical Path: T1 → T3 → T4 → T5 → F1-F4
Parallel Speedup: Wave 1 runs 2 tasks concurrently; Final wave runs 4 concurrently
Max Concurrent: 4 (Final wave)
```

### Dependency Matrix

| Task | Depends On | Blocks | Wave |
|------|-----------|--------|------|
| T1   | —         | T3     | 1    |
| T2   | —         | T4     | 1    |
| T3   | T1        | T4     | 2    |
| T4   | T2, T3    | T5     | 3    |
| T5   | T4        | F1-F4  | 4    |
| F1-F4| T5        | —      | FINAL|

### Agent Dispatch Summary

| Wave | Tasks | Dispatch |
|------|-------|----------|
| 1    | 2     | T1 → `quick`, T2 → `unspecified-high` |
| 2    | 1     | T3 → `visual-engineering` |
| 3    | 1     | T4 → `deep` |
| 4    | 1     | T5 → `unspecified-high` + `playwright` |
| FINAL| 4     | F1 → `oracle`, F2 → `unspecified-high`, F3 → `unspecified-high` + `playwright`, F4 → `deep` |

---

## TODOs


- [ ] 1. PWA Manifest + Icon Assets

  **What to do**:
  - Create `manifest.json` with:
    - `name`: "Polyatomic Ion Memorizer"
    - `short_name`: "Ion Quiz"
    - `start_url`: "/index.html"
    - `display`: "standalone"
    - `orientation`: "portrait"
    - `background_color`: "#f1f5f9" (matches `bg-slate-100`)
    - `theme_color`: "#2563eb" (matches `bg-blue-600` header)
    - `icons` array referencing the 3 icon sizes
  - Generate 3 simple PNG icon files:
    - `icon-192.png` (192×192) — solid blue (#2563eb) square with white "⚛" or "PI" text
    - `icon-512.png` (512×512) — same design scaled up
    - `apple-touch-icon.png` (180×180) — same design for iOS
  - Icons can be generated via any method: canvas-based script, ImageMagick, or a simple online generator. Keep them minimal — solid color + text/symbol. No complex artwork.

  **Must NOT do**:
  - Do not create splash screen images
  - Do not add `screenshots` or `shortcuts` to manifest
  - Do not add `push` or `notifications` related fields

  **Recommended Agent Profile**:
  - **Category**: `quick`
    - Reason: Creating 2 config/asset files with well-defined content — no complex logic
  - **Skills**: []
    - No special skills needed — file creation only
  - **Skills Evaluated but Omitted**:
    - `playwright`: Not needed — QA for icons/manifest is in Task 5

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 1 (with Task 2)
  - **Blocks**: Task 3 (manifest must exist before linking it in `<head>`)
  - **Blocked By**: None (can start immediately)

  **References**:

  **Pattern References**:
  - `index.html:36-39` — Header uses `bg-blue-600` (the `#2563eb` color for icons and `theme_color`)
  - `index.html:31` — Body uses `bg-slate-100` (the `#f1f5f9` color for `background_color`)
  - `index.html:6` — Current `<title>` text for manifest `name` field

  **External References**:
  - MDN Web App Manifest: https://developer.mozilla.org/en-US/docs/Web/Manifest
  - Icon requirements: minimum 192×192 and 512×512 for Chrome install prompt, 180×180 for `apple-touch-icon`

  **WHY Each Reference Matters**:
  - The color values ensure manifest colors match the existing app theme exactly
  - The title provides the exact app name string to use in the manifest

  **Acceptance Criteria**:

  - [ ] `manifest.json` exists at project root and is valid JSON
  - [ ] `manifest.json` contains all required fields: `name`, `short_name`, `start_url`, `display`, `orientation`, `background_color`, `theme_color`, `icons`
  - [ ] `icon-192.png` exists and is a valid PNG, dimensions 192×192
  - [ ] `icon-512.png` exists and is a valid PNG, dimensions 512×512
  - [ ] `apple-touch-icon.png` exists and is a valid PNG, dimensions 180×180

  **QA Scenarios:**

  ```
  Scenario: Manifest file is valid and complete
    Tool: Bash
    Preconditions: Task 1 completed, files exist at project root
    Steps:
      1. Run: python3 -c "import json; m=json.load(open('manifest.json')); assert m['name']=='Polyatomic Ion Memorizer'; assert m['display']=='standalone'; assert m['orientation']=='portrait'; assert len(m['icons'])>=2; print('VALID')"
      2. Run: python3 -c "import json; m=json.load(open('manifest.json')); assert m['theme_color']=='#2563eb'; assert m['background_color']=='#f1f5f9'; print('COLORS MATCH')"
    Expected Result: Both commands print their success messages without errors
    Failure Indicators: KeyError, AssertionError, or json.JSONDecodeError
    Evidence: .sisyphus/evidence/task-1-manifest-valid.txt

  Scenario: Icon files are valid PNGs with correct dimensions
    Tool: Bash
    Preconditions: Icon files generated
    Steps:
      1. Run: file icon-192.png icon-512.png apple-touch-icon.png (verify PNG format)
      2. Run: python3 -c "from PIL import Image; i=Image.open('icon-192.png'); assert i.size==(192,192); print('192 OK')" OR sips -g pixelHeight -g pixelWidth icon-192.png (macOS)
      3. Repeat dimension check for icon-512.png (512×512) and apple-touch-icon.png (180×180)
    Expected Result: All files are PNG format with correct dimensions
    Failure Indicators: "not a PNG" from file command, dimension assertion failure
    Evidence: .sisyphus/evidence/task-1-icons-valid.txt
  ```

  **Commit**: YES
  - Message: `feat(pwa): add web app manifest and icon assets`
  - Files: `manifest.json`, `icon-192.png`, `icon-512.png`, `apple-touch-icon.png`
  - Pre-commit: `python3 -c "import json; json.load(open('manifest.json'))"`

---

- [ ] 2. Service Worker for Offline Caching

  **What to do**:
  - Create `sw.js` at project root with a cache-first offline strategy:
    - Define a cache name constant (e.g., `polyatomic-v1`)
    - On `install` event: pre-cache the critical assets:
      - `/index.html`
      - `/manifest.json`
      - `/icon-192.png`, `/icon-512.png`, `/apple-touch-icon.png`
      - `https://cdn.tailwindcss.com` — **CRITICAL**: This is a JS runtime that generates CSS. It MUST be cached for the app to have ANY styling offline.
    - On `fetch` event: use cache-first strategy — return cached response if available, fall back to network, cache new successful responses
    - On `activate` event: clean up old caches (delete caches that don't match current cache name)
  - **CRITICAL RISK — Tailwind CDN Offline**: The Tailwind CDN (`https://cdn.tailwindcss.com`) is NOT a static CSS file. It's a JavaScript engine that JIT-compiles Tailwind classes at runtime. When cached by the service worker and served offline, the script must still function correctly (it scans the DOM and generates `<style>` tags). This is the #1 risk to offline support. The QA in Task 5 will specifically validate this. If it fails, the fallback approach is to add minimal inline CSS in `<style>` as insurance (would be handled as a fix task).

  **Must NOT do**:
  - Do not implement versioned cache busting with UI prompts
  - Do not add background sync or push notification handling
  - Do not add a "New version available" update UI
  - Do not cache external analytics or tracking scripts (there are none, but don't add any)

  **Recommended Agent Profile**:
  - **Category**: `unspecified-high`
    - Reason: Service worker caching strategy requires careful thought — cache-first logic, CDN handling, activation cleanup. More than a trivial file creation.
  - **Skills**: []
    - No special skills needed — pure JS file creation
  - **Skills Evaluated but Omitted**:
    - `playwright`: Not needed here — offline testing is in Task 5

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 1 (with Task 1)
  - **Blocks**: Task 4 (service worker must exist before JS registers it)
  - **Blocked By**: None (can start immediately)

  **References**:

  **Pattern References**:
  - `index.html:7` — `<script src="https://cdn.tailwindcss.com"></script>` — the exact CDN URL to cache
  - `index.html:1-400` — the entire app is this one HTML file — `/index.html` is the only page to cache

  **External References**:
  - MDN Service Worker API: https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API
  - Cache API: https://developer.mozilla.org/en-US/docs/Web/API/Cache
  - Service worker lifecycle: install → waiting → activate → fetch

  **WHY Each Reference Matters**:
  - Line 7 gives the exact Tailwind CDN URL that must be in the pre-cache list
  - The single-file architecture means we only need to cache one HTML file + CDN + icons

  **Acceptance Criteria**:

  - [ ] `sw.js` exists at project root with no syntax errors (`node -c sw.js` passes)
  - [ ] `sw.js` defines an `install` event listener that pre-caches: `/index.html`, `/manifest.json`, icon files, and `https://cdn.tailwindcss.com`
  - [ ] `sw.js` defines a `fetch` event listener with cache-first strategy
  - [ ] `sw.js` defines an `activate` event listener that cleans old caches

  **QA Scenarios:**

  ```
  Scenario: Service worker has no syntax errors and correct structure
    Tool: Bash
    Preconditions: sw.js created at project root
    Steps:
      1. Run: node -c sw.js
      2. Run: grep -c "addEventListener.*install" sw.js (expect 1)
      3. Run: grep -c "addEventListener.*fetch" sw.js (expect 1)
      4. Run: grep -c "addEventListener.*activate" sw.js (expect 1)
      5. Run: grep "cdn.tailwindcss.com" sw.js (expect match — CDN URL in pre-cache list)
    Expected Result: No syntax errors, all 3 event listeners present, Tailwind CDN URL in pre-cache
    Failure Indicators: SyntaxError from node -c, missing event listeners, missing CDN URL
    Evidence: .sisyphus/evidence/task-2-sw-structure.txt

  Scenario: Service worker file does not contain forbidden patterns
    Tool: Bash
    Preconditions: sw.js created
    Steps:
      1. Run: grep -i "push" sw.js (expect no matches — no push notification handling)
      2. Run: grep -i "background.*sync" sw.js (expect no matches)
      3. Run: grep -i "notification" sw.js (expect no matches)
    Expected Result: No forbidden patterns found
    Failure Indicators: Any matches for push/sync/notification
    Evidence: .sisyphus/evidence/task-2-sw-no-forbidden.txt
  ```

  **Commit**: YES
  - Message: `feat(pwa): add service worker for offline caching`
  - Files: `sw.js`
  - Pre-commit: `node -c sw.js`

---

- [ ] 3. Responsive Layout + PWA Head Tags

  **What to do**:

  **A. `<head>` section additions** (insert after existing `<meta>` tags, before `<title>`):
  - Add `<link rel="manifest" href="manifest.json">`
  - Add `<link rel="apple-touch-icon" href="apple-touch-icon.png">`
  - Add `<meta name="apple-mobile-web-app-capable" content="yes">`
  - Add `<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">`
  - Add `<meta name="theme-color" content="#2563eb">`
  - Modify existing viewport meta to: `width=device-width, initial-scale=1.0, viewport-fit=cover`

  **B. `<style>` block additions** (append to existing `<style>`):
  - Add safe area padding for standalone PWA mode:
    ```css
    @supports (padding: env(safe-area-inset-top)) {
      body { padding-top: env(safe-area-inset-top); padding-bottom: env(safe-area-inset-bottom); }
    }
    ```
  - Add pull-to-refresh prevention:
    ```css
    html, body { overscroll-behavior-y: none; }
    ```
  - Add mobile touch optimization:
    ```css
    * { -webkit-tap-highlight-color: transparent; }
    input { font-size: 16px !important; } /* Prevents iOS auto-zoom on focus */
    ```
  - Add keyboard-aware quiz layout (see HTML changes below for the container this targets)

  **C. HTML body modifications** across all 3 views:

  **Container (line 31)**:
  - Change body from `flex items-center justify-center` to `items-start sm:items-center justify-center` — prevents vertical centering squeeze when mobile keyboard opens
  - Keep `p-4` on body but add `sm:p-4` as-is (already fine)

  **App container (line 33)**:
  - Remove `min-h-[500px]` (overflows on iPhone SE). Replace with `min-h-0 sm:min-h-[500px]`
  - Keep `max-w-3xl` — fine for tablets, naturally full-width on phones

  **Header (line 36-39)**:
  - Reduce title from `text-3xl` to `text-xl sm:text-3xl`
  - Reduce padding from `p-6` to `p-4 sm:p-6`

  **Selection Menu view (lines 44-69)**:
  - Change ion grid from `grid-cols-2 sm:grid-cols-3 md:grid-cols-4` to `grid-cols-2 sm:grid-cols-3 md:grid-cols-4` (already OK)
  - Change `max-h-[40vh]` to `max-h-[45vh] sm:max-h-[40vh]` — give more room on short screens
  - Make Select All / Clear buttons stack on very small screens: wrap in `flex flex-wrap gap-2` instead of `space-x-2`
  - Ensure each checkbox label has minimum touch target: add `min-h-[44px]` to the `<div>` inside each label
  - Make ion checkbox labels slightly larger on mobile: `px-3 py-3 sm:px-4 sm:py-3`

  **Quiz view (lines 72-96)**:
  - **CRITICAL: Make submit button visible and functional**. Remove `hidden` from `btn-submit`'s class. It's currently dead code — the button exists but is never shown. Make it always visible below the input.
  - Reduce question text from `text-5xl` to `text-3xl sm:text-5xl`
  - Reduce input text from `text-3xl` to `text-xl sm:text-3xl`
  - Ensure input font-size stays ≥ 16px (Tailwind `text-xl` = 20px, so this is fine)
  - Reduce feedback `text-lg` — keep as-is, it's fine on mobile
  - Reduce `mt-20` on submit button to `mt-8 sm:mt-20` — 80px margin pushes it offscreen on mobile
  - Add `pb-4` to the quiz container to ensure submit button isn't clipped
  - Change stop button padding from `px-4 py-2` to `px-3 py-2 sm:px-4 sm:py-2`

  **Assessment view (lines 99-137)**:
  - Assessment grid already uses `md:grid-cols-3` which stacks on mobile — good
  - Reduce title from `text-3xl` to `text-2xl sm:text-3xl`
  - Make assessment list items more touch-friendly: ensure minimum height `min-h-[44px]`
  - Reduce column padding from `p-5` to `p-3 sm:p-5` for breathing room on small screens

  **Must NOT do**:
  - Do not modify any JavaScript in this task (JS changes are in Task 4)
  - Do not change quiz logic, scoring, or ion data
  - Do not add dark mode styles
  - Do not add ARIA attributes or screen reader support
  - Do not change the existing desktop layout behavior (all changes via mobile-first or breakpoints)

  **Recommended Agent Profile**:
  - **Category**: `visual-engineering`
    - Reason: This is primarily CSS/HTML layout work requiring visual awareness of spacing, sizing, and responsive breakpoints
  - **Skills**: [`playwright`]
    - `playwright`: Needed to visually verify responsive layout at multiple mobile viewport sizes
  - **Skills Evaluated but Omitted**:
    - `frontend-ui-ux`: Overlaps with visual-engineering category; not needed for responsive fixes on an existing design

  **Parallelization**:
  - **Can Run In Parallel**: NO
  - **Parallel Group**: Wave 2 (solo)
  - **Blocks**: Task 4 (JS needs to reference the submit button which this task makes visible)
  - **Blocked By**: Task 1 (manifest.json must exist to link in `<head>`)

  **References**:

  **Pattern References** (existing code to modify):
  - `index.html:5` — Existing viewport meta tag to modify (add `viewport-fit=cover`)
  - `index.html:7` — Tailwind CDN script tag (DO NOT modify — reference only)
  - `index.html:8-29` — Existing `<style>` block to append mobile CSS to
  - `index.html:31` — `<body>` tag with current classes to modify
  - `index.html:33` — App container `#app` with `min-h-[500px]` to fix
  - `index.html:36-39` — Header with text sizes and padding to reduce
  - `index.html:44-69` — Selection menu view with ion grid and buttons
  - `index.html:50-53` — Select All / Clear buttons that need flex-wrap
  - `index.html:56` — Ion grid with `max-h-[40vh]` to adjust
  - `index.html:72-96` — Quiz view with oversized text and hidden submit button
  - `index.html:82` — Question text `text-5xl` to reduce on mobile
  - `index.html:85` — Input with `text-3xl` to reduce on mobile
  - `index.html:92` — Submit button with `hidden` class — **REMOVE `hidden`**
  - `index.html:99-137` — Assessment view to optimize for mobile

  **External References**:
  - Apple PWA meta tags: https://developer.apple.com/library/archive/documentation/AppleApplications/Reference/SafariHTMLRef/Articles/MetaTags.html
  - Safe area insets: https://developer.mozilla.org/en-US/docs/Web/CSS/env
  - `viewport-fit=cover`: https://developer.mozilla.org/en-US/docs/Web/HTML/Viewport_meta_tag

  **WHY Each Reference Matters**:
  - Lines 31-33 are the most critical — body centering and container min-height are the root causes of mobile layout issues
  - Line 92 is the submit button dead code that must be activated
  - Lines 50-53 are the cramped buttons that need flex-wrap on mobile
  - The Apple meta tags are required for iOS to treat this as a capable web app

  **Acceptance Criteria**:

  - [ ] `<head>` contains `<link rel="manifest">`, `<link rel="apple-touch-icon">`, `<meta name="theme-color">`, `<meta name="apple-mobile-web-app-capable">`
  - [ ] Viewport meta includes `viewport-fit=cover`
  - [ ] `<style>` block includes `overscroll-behavior-y: none` and safe area padding
  - [ ] Body tag uses `items-start sm:items-center` (not just `items-center`)
  - [ ] App container no longer has `min-h-[500px]` unconditionally
  - [ ] Submit button (`btn-submit`) does NOT have `hidden` class
  - [ ] Submit button has reduced margin: `mt-8 sm:mt-20` (not `mt-20`)
  - [ ] Header uses `text-xl sm:text-3xl` (not `text-3xl`)
  - [ ] Question text uses `text-3xl sm:text-5xl` (not `text-5xl`)
  - [ ] Input text uses `text-xl sm:text-3xl` (not `text-3xl`)
  - [ ] All interactive elements have minimum 44×44px touch target area
  - [ ] No horizontal overflow on iPhone SE viewport (375×667)

  **QA Scenarios:**

  ```
  Scenario: App renders without horizontal overflow on iPhone SE
    Tool: Playwright (playwright skill)
    Preconditions: index.html served via local HTTP server (python3 -m http.server 8080)
    Steps:
      1. Launch Playwright with viewport { width: 375, height: 667 }
      2. Navigate to http://localhost:8080
      3. Wait for page to fully load (Tailwind CSS compiled)
      4. Execute: document.documentElement.scrollWidth <= document.documentElement.clientWidth
      5. Screenshot the selection menu view
      6. Verify header text is visible and not truncated
      7. Verify ion grid is scrollable (scroll to bottom, verify last ion visible)
    Expected Result: scrollWidth <= clientWidth (no horizontal overflow), all content visible
    Failure Indicators: scrollWidth > clientWidth, content clipped or overlapping
    Evidence: .sisyphus/evidence/task-3-iphone-se-menu.png

  Scenario: Submit button is visible on quiz view (mobile)
    Tool: Playwright (playwright skill)
    Preconditions: App loaded, ions selected
    Steps:
      1. Set viewport to { width: 375, height: 667 }
      2. Navigate to http://localhost:8080
      3. Click "Start Quiz" button
      4. Verify element #btn-submit is visible: await page.locator('#btn-submit').isVisible()
      5. Verify #btn-submit bounding box: width >= 44, height >= 44
      6. Screenshot quiz view showing visible submit button
    Expected Result: Submit button is visible, not hidden, and has adequate touch target size
    Failure Indicators: isVisible() returns false, element has 'hidden' class, dimensions < 44px
    Evidence: .sisyphus/evidence/task-3-submit-visible.png

  Scenario: App renders correctly on iPad viewport
    Tool: Playwright (playwright skill)
    Preconditions: App loaded
    Steps:
      1. Set viewport to { width: 768, height: 1024 }
      2. Navigate to http://localhost:8080
      3. Verify ion grid shows 3-4 columns (grid children per row)
      4. Click "Start Quiz", verify quiz layout is centered and spacious
      5. Click "Stop & Assess", verify assessment shows 3-column grid
      6. Screenshot all 3 views
    Expected Result: Tablet layout uses wider grids and spacing, 3-col assessment
    Failure Indicators: Single-column layout on tablet, cramped spacing
    Evidence: .sisyphus/evidence/task-3-ipad-views.png

  Scenario: Touch targets meet minimum size requirements
    Tool: Playwright (playwright skill)
    Preconditions: App loaded on mobile viewport
    Steps:
      1. Set viewport to { width: 375, height: 667 }
      2. Navigate to http://localhost:8080
      3. For each ion checkbox label: get bounding rect, assert height >= 44
      4. For #btn-start: get bounding rect, assert height >= 44 and width >= 200
      5. Click "Start Quiz", for #btn-submit: assert height >= 44
      6. For #btn-stop: assert height >= 44
    Expected Result: All interactive elements >= 44px in their tap dimension
    Failure Indicators: Any element has height or width < 44px
    Evidence: .sisyphus/evidence/task-3-touch-targets.txt
  ```

  **Commit**: YES
  - Message: `feat(mobile): responsive layout and PWA meta tags`
  - Files: `index.html`
  - Pre-commit: Open in browser, verify no JS errors in console

---

- [ ] 4. Mobile JavaScript Features

  **What to do**:

  Add all mobile-specific JavaScript to the existing `<script>` block in `index.html`. Append new code AFTER the existing `init()` call (line 396). Do not modify existing functions unless explicitly stated below.

  **A. Service Worker Registration** (append after `init()`):
  - Register `sw.js` in a `load` event listener:
    ```js
    if ('serviceWorker' in navigator) {
      window.addEventListener('load', () => {
        navigator.serviceWorker.register('/sw.js')
          .then(reg => console.log('SW registered:', reg.scope))
          .catch(err => console.warn('SW registration failed:', err));
      });
    }
    ```
  - Use feature detection (`if ('serviceWorker' in navigator)`) — graceful no-op if unsupported

  **B. Install Prompt** (two code paths):
  - **Android Chrome** — intercept `beforeinstallprompt` event:
    - Store the event in a variable (e.g., `deferredInstallPrompt`)
    - Show a non-intrusive install banner at the TOP of the app (inside `#app`, before the header)
    - Banner design: slim bar with text "Install this app for offline use" + "Install" button + "×" dismiss
    - On "Install" click: call `deferredInstallPrompt.prompt()`, await `userChoice`, hide banner
    - On dismiss: hide banner, don't show again this session (use a JS variable, NOT localStorage)
    - Style with Tailwind classes to match the app's blue theme
  - **iOS Safari** — detect iOS + not-standalone:
    - Detection: `(/iPad|iPhone|iPod/.test(navigator.userAgent) && !window.navigator.standalone)`
    - Show a custom banner with instructions: "Tap the Share button, then 'Add to Home Screen' to install"
    - Include a small share icon visual (⤴ symbol or similar)
    - Same dismiss behavior as Android banner
  - Banner must be created via JS (not in static HTML) — only appears when relevant

  **C. Haptic Feedback** (modify existing `handleSubmission` function):
  - After the `isCorrect` check (line 312):
    - On correct: `if ('vibrate' in navigator) navigator.vibrate(50);` (short pulse)
    - On incorrect: `if ('vibrate' in navigator) navigator.vibrate([100, 50, 100]);` (double pulse)
  - Feature detection ensures graceful no-op on iOS (which doesn't support Vibration API)
  - **These are the ONLY modifications to the existing `handleSubmission` function** — do not change answer validation, scoring, or display logic

  **D. iOS Focus Workaround** (modify existing `nextQuestion` function):
  - **Problem**: `setTimeout(nextQuestion, 700)` on line 326 calls `nextQuestion()` which calls `quizInput.focus()` on line 299. On iOS Safari, `focus()` called outside a user gesture's event loop is BLOCKED. The keyboard dismisses after every correct answer, forcing users to re-tap the input.
  - **Solution**: Instead of using `setTimeout` to auto-advance, restructure the correct-answer flow:
    - After showing "Correct!" feedback, keep the input focused (don't blur it)
    - Use `setTimeout(nextQuestion, 700)` as before BUT in `nextQuestion()`, instead of calling `quizInput.focus()` directly, set `quizInput.value = ''` (clear it) without calling focus — the input retains focus from the previous interaction
    - Remove the explicit `quizInput.focus()` call from `nextQuestion()` and instead call it ONLY from `startQuiz()` (initial focus) and from the incorrect-answer path (where the user just interacted)
    - In `nextQuestion()`, replace `quizInput.focus()` with a conditional: only call focus if the input is not already the active element (`if (document.activeElement !== quizInput) quizInput.focus()`)
  - **This modifies `nextQuestion()` (line 282) and the correct-answer `setTimeout` call (line 326)**

  **E. Submit Button Wiring** (modify existing code):
  - Task 3 made `btn-submit` visible. Now wire it:
  - In `attachEventListeners()` (line 229), add: `document.getElementById('btn-submit').addEventListener('click', handleSubmission);`
  - That's it — the button already exists in HTML, Task 3 removed `hidden`, now just add the click handler

  **Must NOT do**:
  - Do not change answer validation logic or scoring algorithm
  - Do not change how `stats` object works
  - Do not change question selection / randomization logic
  - Do not add localStorage, IndexedDB, or any persistent storage
  - Do not change the ion data array
  - Do not refactor existing code into classes or modules
  - Do not add error boundaries or try-catch wrappers around existing code
  - Do not add console.log statements beyond the SW registration (which logs to console intentionally)

  **Recommended Agent Profile**:
  - **Category**: `deep`
    - Reason: Multiple interconnected JS features (SW registration, install prompt with 2 code paths, haptics, iOS focus workaround) requiring careful integration with existing imperative code. The iOS focus workaround is particularly tricky.
  - **Skills**: []
    - No special skills needed — pure JS modifications to an existing inline script
  - **Skills Evaluated but Omitted**:
    - `playwright`: Testing is deferred to Task 5 to avoid testing incomplete state

  **Parallelization**:
  - **Can Run In Parallel**: NO
  - **Parallel Group**: Wave 3 (solo)
  - **Blocks**: Task 5 (QA depends on all features being implemented)
  - **Blocked By**: Task 2 (sw.js must exist for registration), Task 3 (submit button must be visible for wiring)

  **References**:

  **Pattern References** (existing code to modify):
  - `index.html:229-254` — `attachEventListeners()` function — add submit button click handler here
  - `index.html:282-300` — `nextQuestion()` function — modify focus logic for iOS workaround
  - `index.html:302-345` — `handleSubmission()` function — add vibrate calls after correct/incorrect checks
  - `index.html:312-327` — Correct answer path — add `navigator.vibrate(50)` after line 312
  - `index.html:328-344` — Incorrect answer path — add `navigator.vibrate([100, 50, 100])` after line 328
  - `index.html:326` — `setTimeout(nextQuestion, 700)` — the call that causes iOS focus issues
  - `index.html:396` — `init()` call — append all new code AFTER this line
  - `index.html:92-94` — Submit button HTML (made visible by Task 3) — reference for element ID `btn-submit`

  **External References**:
  - `beforeinstallprompt`: https://developer.mozilla.org/en-US/docs/Web/API/Window/beforeinstallprompt_event
  - `navigator.vibrate()`: https://developer.mozilla.org/en-US/docs/Web/API/Navigator/vibrate
  - Service Worker registration: https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerContainer/register
  - iOS Safari focus restrictions: programmatic focus() only works in the same call stack as a user gesture

  **WHY Each Reference Matters**:
  - Lines 229-254 show the existing event listener pattern — follow the same `btn.addEventListener` style
  - Lines 282-300 and 326 are the iOS focus problem — must understand the exact call chain to fix correctly
  - Lines 312 and 328 are the exact insertion points for haptic calls
  - Line 396 is the boundary — all new standalone code goes after this line

  **Acceptance Criteria**:

  - [ ] Service worker registers successfully (verify via browser DevTools > Application > Service Workers)
  - [ ] Install banner appears on Android Chrome (or when `beforeinstallprompt` fires)
  - [ ] iOS install banner appears on iOS Safari (non-standalone) with share instructions
  - [ ] Install banners have a dismiss button that hides them
  - [ ] `navigator.vibrate(50)` called on correct answer (feature-detected)
  - [ ] `navigator.vibrate([100, 50, 100])` called on incorrect answer (feature-detected)
  - [ ] Quiz input retains focus between questions (iOS focus workaround)
  - [ ] Submit button (`#btn-submit`) triggers `handleSubmission()` on click
  - [ ] No existing quiz logic changed (scoring, validation, question selection)
  - [ ] No JS errors in console on page load or during quiz flow

  **QA Scenarios:**

  ```
  Scenario: Service worker registers and caches assets
    Tool: Playwright (playwright skill)
    Preconditions: App served via HTTP server (python3 -m http.server 8080)
    Steps:
      1. Navigate to http://localhost:8080 with viewport { width: 390, height: 844 }
      2. Wait 2 seconds for SW to install
      3. Execute in page: const reg = await navigator.serviceWorker.getRegistration(); return reg !== undefined;
      4. Verify result is true
    Expected Result: Service worker is registered
    Failure Indicators: getRegistration() returns undefined, console errors about SW
    Evidence: .sisyphus/evidence/task-4-sw-registered.png

  Scenario: Submit button triggers answer submission on click
    Tool: Playwright (playwright skill)
    Preconditions: App loaded, quiz started
    Steps:
      1. Set viewport to { width: 375, height: 667 }
      2. Navigate to http://localhost:8080
      3. Click "Start Quiz" (with at least 1 ion selected)
      4. Read the question text from #quiz-question
      5. Type a deliberately wrong answer into #quiz-input: await page.fill('#quiz-input', 'WRONG')
      6. Click #btn-submit
      7. Verify #quiz-feedback becomes visible (opacity > 0) with red/rose text ("Incorrect")
    Expected Result: Clicking submit button triggers the same behavior as pressing Enter
    Failure Indicators: No feedback appears, button click does nothing, JS error in console
    Evidence: .sisyphus/evidence/task-4-submit-click.png

  Scenario: Haptic feedback calls vibrate on correct/incorrect (Android)
    Tool: Playwright (playwright skill)
    Preconditions: App loaded, quiz started
    Steps:
      1. Set viewport to { width: 390, height: 844 }
      2. Navigate to http://localhost:8080
      3. Inject vibration spy: await page.evaluate(() => { window._vibrateLog = []; const orig = navigator.vibrate; navigator.vibrate = function(p) { window._vibrateLog.push(p); return orig ? orig.call(navigator, p) : true; }; })
      4. Start quiz, type correct answer (lookup from ionData), press Enter
      5. Execute: return window._vibrateLog (expect [50])
      6. Wait for next question, type wrong answer, press Enter
      7. Execute: return window._vibrateLog (expect [50, [100, 50, 100]])
    Expected Result: vibrate(50) called on correct, vibrate([100,50,100]) called on incorrect
    Failure Indicators: _vibrateLog is empty or has wrong values
    Evidence: .sisyphus/evidence/task-4-haptic-feedback.txt

  Scenario: iOS install banner appears with share instructions
    Tool: Playwright (playwright skill)
    Preconditions: App loaded
    Steps:
      1. Set viewport to { width: 375, height: 667 }
      2. Set user agent to iOS Safari: 'Mozilla/5.0 (iPhone; CPU iPhone OS 17_0 like Mac OS X) AppleWebKit/605.1.15'
      3. Navigate to http://localhost:8080
      4. Wait 1 second for banner to appear
      5. Verify banner element exists and is visible
      6. Verify banner contains text about "Share" or "Add to Home Screen"
      7. Click dismiss button on banner
      8. Verify banner is hidden
    Expected Result: iOS install banner appears with instructions, dismissable
    Failure Indicators: No banner appears, banner doesn't mention Share, dismiss doesn't work
    Evidence: .sisyphus/evidence/task-4-ios-install-banner.png

  Scenario: Quiz flow works correctly after JS changes (regression)
    Tool: Playwright (playwright skill)
    Preconditions: App loaded
    Steps:
      1. Set viewport to { width: 375, height: 667 }
      2. Navigate to http://localhost:8080
      3. Click "Start Quiz" (all ions selected)
      4. Answer 3 questions correctly (look up answers from ionData in page context)
      5. Answer 1 question incorrectly, then correctly (retry flow)
      6. Click "Stop & Assess"
      7. Verify assessment view shows results with correct/incorrect counts
      8. Verify at least one item shows percentage
    Expected Result: Full quiz flow works identically to before, with haptic calls added
    Failure Indicators: Questions don't advance, scoring is wrong, assessment is empty
    Evidence: .sisyphus/evidence/task-4-quiz-regression.png
  ```

  **Commit**: YES
  - Message: `feat(mobile): add mobile JS features (haptics, install, focus)`
  - Files: `index.html`
  - Pre-commit: Open in browser, complete a quiz flow, verify no console errors

---

- [ ] 5. Full Mobile QA + Offline Verification

  **What to do**:
  - Serve the complete app via a local HTTP server
  - Run comprehensive Playwright tests across 3 mobile viewports: iPhone SE (375×667), iPhone 14 (390×844), iPad (768×1024)
  - Test ALL 3 views (Selection Menu, Quiz, Assessment) on each viewport
  - Test complete offline flow: load app → wait for SW → go offline → reload → verify styling → complete full quiz
  - Test install prompt on Android user-agent and iOS user-agent
  - Verify no regressions on desktop viewport (1280×800)
  - Capture screenshot evidence for every scenario
  - **CRITICAL OFFLINE TEST**: After going offline, verify the app has FULL Tailwind styling (not unstyled HTML). This validates that the Tailwind CDN JS runtime works when served from service worker cache. If this fails, file a fix task.

  **Must NOT do**:
  - Do not modify any source files in this task — this is QA only
  - Do not add a test framework or test runner
  - Do not skip the offline test — it's the most important scenario

  **Recommended Agent Profile**:
  - **Category**: `unspecified-high`
    - Reason: Comprehensive testing across multiple viewports and scenarios requires methodical execution
  - **Skills**: [`playwright`]
    - `playwright`: Core tool for all QA scenarios — viewport simulation, offline mode, element assertions, screenshots
  - **Skills Evaluated but Omitted**:
    - `dev-browser`: Playwright skill is more appropriate for automated testing than interactive browsing

  **Parallelization**:
  - **Can Run In Parallel**: NO
  - **Parallel Group**: Wave 4 (solo)
  - **Blocks**: Final Verification Wave (F1-F4)
  - **Blocked By**: Task 4 (all implementation must be complete)

  **References**:

  **Pattern References**:
  - `index.html` — The complete app file with all responsive changes, PWA meta tags, and mobile JS
  - `manifest.json` — PWA manifest to verify is linked and functional
  - `sw.js` — Service worker to verify registers and caches correctly
  - `icon-192.png`, `icon-512.png`, `apple-touch-icon.png` — Icon files to verify exist

  **API/Type References**:
  - `index.html:144-166` — `ionData` array — use this to look up correct answers for quiz testing (e.g., "Sulfate" → "SO42-")

  **External References**:
  - Playwright `context.setOffline(true)`: https://playwright.dev/docs/api/class-browsercontext#browser-context-set-offline
  - Playwright `page.setViewportSize()`: https://playwright.dev/docs/api/class-page#page-set-viewport-size

  **WHY Each Reference Matters**:
  - The `ionData` array provides exact correct answers needed to test the quiz flow programmatically
  - The icon files, manifest, and sw.js are all deliverables that need existence verification

  **Acceptance Criteria**:

  - [ ] All 3 views render correctly on iPhone SE, iPhone 14, and iPad viewports (screenshots captured)
  - [ ] No horizontal overflow on any mobile viewport
  - [ ] Full quiz flow works on mobile (select → quiz → answer correct + incorrect → assessment)
  - [ ] App works fully offline (load, go offline, reload, complete quiz, assessment) WITH Tailwind styling
  - [ ] Install banners appear correctly (Android native, iOS custom)
  - [ ] Submit button works on all viewports
  - [ ] Desktop layout unchanged at 1280×800
  - [ ] Evidence screenshots saved for every scenario

  **QA Scenarios:**

  ```
  Scenario: Full offline quiz flow with Tailwind styling
    Tool: Playwright (playwright skill)
    Preconditions: App served via HTTP server
    Steps:
      1. Launch browser context
      2. Navigate to http://localhost:8080 with viewport { width: 390, height: 844 }
      3. Wait 3 seconds for service worker to install and cache assets
      4. Verify SW is active: page.evaluate(() => navigator.serviceWorker.controller !== null)
      5. Set context offline: context.setOffline(true)
      6. Reload page: page.reload()
      7. CRITICAL CHECK: Verify Tailwind is working — page.evaluate(() => { const body = getComputedStyle(document.body); return body.backgroundColor !== '' && body.backgroundColor !== 'rgba(0, 0, 0, 0)'; }) — body should have bg-slate-100 color (#f1f5f9 / rgb(241, 245, 249))
      8. Verify header has blue background: getComputedStyle(header).backgroundColor should be #2563eb / rgb(37, 99, 235)
      9. Screenshot: offline state with full styling
      10. Start quiz (click Start Quiz button)
      11. Type correct answer for displayed ion (look up from ionData)
      12. Verify "Correct!" feedback appears
      13. After auto-advance, type incorrect answer
      14. Verify "Incorrect" feedback with correct formula shown
      15. Click "Stop & Assess"
      16. Verify assessment view renders with results
      17. Screenshot: offline assessment view
    Expected Result: App is fully functional and STYLED offline — Tailwind CSS renders correctly from cache
    Failure Indicators: Unstyled HTML (no colors, no layout), JS errors, quiz flow breaks
    Evidence: .sisyphus/evidence/task-5-offline-styled.png, .sisyphus/evidence/task-5-offline-assessment.png

  Scenario: All 3 views on iPhone SE (smallest target)
    Tool: Playwright (playwright skill)
    Preconditions: App served, online
    Steps:
      1. Set viewport to { width: 375, height: 667 }
      2. Navigate to http://localhost:8080
      3. Screenshot Selection Menu — verify grid is 2 columns, buttons visible, no overflow
      4. Verify scrollWidth <= clientWidth
      5. Click Start Quiz
      6. Screenshot Quiz View — verify question text fits, input visible, submit button visible
      7. Answer 2 questions, click Stop & Assess
      8. Screenshot Assessment View — verify columns stack vertically, results readable
    Expected Result: All 3 views render cleanly at 375px width with no overflow or clipping
    Failure Indicators: Horizontal scroll, text clipped, buttons off-screen, submit hidden
    Evidence: .sisyphus/evidence/task-5-se-menu.png, task-5-se-quiz.png, task-5-se-assessment.png

  Scenario: Desktop layout unchanged (regression)
    Tool: Playwright (playwright skill)
    Preconditions: App served, online
    Steps:
      1. Set viewport to { width: 1280, height: 800 }
      2. Navigate to http://localhost:8080
      3. Verify app container is centered (not left-aligned)
      4. Verify app container has max-width constraint (not full-width)
      5. Verify header uses larger text (text-3xl at this breakpoint)
      6. Verify ion grid shows 4 columns (md:grid-cols-4)
      7. Screenshot desktop view
      8. Start quiz, verify question uses text-5xl at desktop
      9. Stop quiz, verify assessment uses 3-column grid
    Expected Result: Desktop layout identical to original — centered card, large text, multi-column grids
    Failure Indicators: Layout is mobile-style on desktop, text too small, single-column grids
    Evidence: .sisyphus/evidence/task-5-desktop-regression.png

  Scenario: Install prompt appears on Android user-agent
    Tool: Playwright (playwright skill)
    Preconditions: App served
    Steps:
      1. Set viewport to { width: 390, height: 844 }
      2. Inject beforeinstallprompt mock: page.evaluate(() => { window.dispatchEvent(new Event('beforeinstallprompt')); })
      3. Wait 1 second
      4. Look for install banner element in DOM (text containing "Install" or "offline")
      5. If banner found, verify dismiss button exists and works
      6. Screenshot install banner
    Expected Result: Install banner appears when beforeinstallprompt fires
    Failure Indicators: No banner appears, banner has no dismiss, banner doesn't respond to event
    Evidence: .sisyphus/evidence/task-5-android-install.png
  ```

  **Commit**: YES
  - Message: `test(mobile): full mobile QA verification`
  - Files: `.sisyphus/evidence/*`
  - Pre-commit: Evidence files exist

---

## Final Verification Wave

> 4 review agents run in PARALLEL. ALL must APPROVE. Rejection → fix → re-run.

- [ ] F1. **Plan Compliance Audit** — `oracle`
  Read the plan end-to-end. For each "Must Have": verify implementation exists (read file, run command). For each "Must NOT Have": search codebase for forbidden patterns — reject with file:line if found. Check evidence files exist in `.sisyphus/evidence/`. Compare deliverables against plan.
  Output: `Must Have [N/N] | Must NOT Have [N/N] | Tasks [N/N] | VERDICT: APPROVE/REJECT`

- [ ] F2. **Code Quality Review** — `unspecified-high`
  Review all changed files for: `as any`/`@ts-ignore`, empty catches, `console.log` in prod, commented-out code, unused imports. Check AI slop: excessive comments, over-abstraction, generic names (data/result/item/temp). Verify Tailwind classes are valid. Verify inline JS follows existing code style (imperative, DOM manipulation, no unnecessary abstractions).
  Output: `Files [N clean/N issues] | Style [consistent/inconsistent] | VERDICT`

- [ ] F3. **Real Manual QA** — `unspecified-high` (+ `playwright` skill)
  Start from clean state. Serve `index.html` via local HTTP server. Using Playwright with mobile viewports (iPhone SE, iPhone 14, iPad):
  1. Execute EVERY QA scenario from EVERY task — follow exact steps, capture evidence
  2. Test cross-task integration: full quiz flow (select ions → quiz with correct/incorrect → assessment) on mobile
  3. Test offline: load app, go offline, reload, complete full quiz flow
  4. Test install prompt appearance (Android Chrome user-agent)
  5. Test edge cases: empty selection, single ion, all ions, rapid answer submission
  Save to `.sisyphus/evidence/final-qa/`.
  Output: `Scenarios [N/N pass] | Integration [N/N] | Edge Cases [N tested] | VERDICT`

- [ ] F4. **Scope Fidelity Check** — `deep`
  For each task: read "What to do", read actual diff (`git log`/`git diff`). Verify 1:1 — everything in spec was built (no missing), nothing beyond spec was built (no creep). Specifically check:
  - Quiz logic unchanged (answer validation, scoring, question selection, ion data)
  - No build step added
  - No localStorage/IndexedDB
  - No dark mode
  - Desktop layout unchanged (test at 1280×800)
  - Only new files are manifest.json, sw.js, and icon PNGs
  Output: `Tasks [N/N compliant] | Guardrails [N/N clean] | Unaccounted [CLEAN/N files] | VERDICT`

---

## Commit Strategy

| After Task | Commit Message | Files | Pre-commit Check |
|------------|---------------|-------|-----------------|
| T1 | `feat(pwa): add web app manifest and icon assets` | `manifest.json`, `icon-192.png`, `icon-512.png`, `apple-touch-icon.png` | Files exist and manifest.json is valid JSON |
| T2 | `feat(pwa): add service worker for offline caching` | `sw.js` | File exists, no syntax errors |
| T3 | `feat(mobile): responsive layout and PWA meta tags` | `index.html` | App loads without errors in browser |
| T4 | `feat(mobile): add mobile JS features (haptics, install, focus)` | `index.html` | App loads without errors in browser |
| T5 | `test(mobile): full mobile QA verification` | `.sisyphus/evidence/*` | Evidence files exist |

---

## Success Criteria

### Verification Commands
```bash
# Serve locally and verify
python3 -m http.server 8080  # Serves index.html over HTTP

# Verify manifest.json is valid JSON
python3 -c "import json; json.load(open('manifest.json'))"

# Verify sw.js has no syntax errors
node -c sw.js

# Verify icon files exist
ls -la icon-192.png icon-512.png apple-touch-icon.png
```

### Final Checklist
- [ ] All "Must Have" features present and working
- [ ] All "Must NOT Have" guardrails respected
- [ ] No horizontal overflow on any mobile viewport
- [ ] Full offline quiz flow works after service worker installs
- [ ] Desktop layout unchanged (verify at 1280×800)
- [ ] All evidence screenshots captured in `.sisyphus/evidence/`
