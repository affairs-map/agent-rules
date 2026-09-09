# DEV_UI VALIDATION RULESET — Mockup vs Implementation Comparison

**Primary Mandate**: Web app implementation must match mockup EXACTLY (except data content).
Data content can vary (names, dates, numbers), but every other aspect must be identical.

Validation occurs in TWO TIERS (both must pass before APPROVAL):
- **Tier 0 (Route Health — Smoke Test)**: Every live route returns HTTP 200 and fires zero console errors. Blocking prerequisite; gate before visual inspection.
- **Tier 1+ (Visual/Structural Parity)**: Snapshot-based visual and structural comparison; evidence artifacts required; pixel/DOM/typography/spacing/interaction-state checks per §1–14.

---

## 0. ROUTE HEALTH — SMOKE TEST (TIER 0, BLOCKING PREREQUISITE)

Before any visual parity work on a page, the route must be known to load correctly.

**What this validates**:
- ✅ Route resolves to a live page (HTTP 200, no 404/500)
- ✅ Page hydrates with zero `console.error` or uncaught exceptions
- ✅ Page is not a stub/placeholder/pending-implementation state
- ❌ HTTP status not 200 = FAIL (cannot proceed to visual tier)
- ❌ Console errors present = FAIL (indicates incomplete/broken implementation)

**Coverage requirement (EXHAUSTIVE, not sampled)**:
- Every live route that exists in the compiled entity data must get a tier-0 smoke check.
- Routes derived from `orgs.json`, `people.json`, etc. — not from memory or assumption; use the actual compiled data to generate the route list.
- Sampling is not permitted: "checked 1 org, all orgs OK" is invalid. Every distinct entity instance that will be live gets tested.

**Validation approach** (cite: Linkinator crawl-all-routes pattern + Playwright console listeners):
1. Derive routes from compiled data: e.g. for orgs, iterate `app/data/orgs.json` to get `["/organizations", "/organizations/o_rbi", "/organizations/o_air_india", ...]`.
2. Crawl each route with Playwright or similar headless browser.
3. Assert: `page.goto(route)` returns status 200.
4. Assert: zero `page.on('console', msg => msg.type === 'error')` and zero `page.on('pageerror')` events during page load and initial interaction.
5. Capture results as a structured log (JSON or CSV): `route | status | errors_found | timestamp`.
6. **Any route that fails tier 0 must be flagged as FAIL in the final report** — cannot be masked or deferred.

**Tools**: Use Playwright's built-in `page.goto()`, `page.on('console')`, `page.on('pageerror')` listeners (no separate tool needed). Optionally cite Linkinator as prior art for the crawl-all-pattern, even if implemented with Playwright.

---

## 1. OVERALL LAYOUT & STRUCTURE

### 1.1 Page Layout
- ✅ Same page grid/flex structure as mockup
- ✅ Same number of sections (header, main, sidebar, footer if present)
- ✅ Same section order (top to bottom)
- ✅ Same column distribution (1-col, 2-col, 3-col layout)
- ❌ Different grid structure than mockup = FAIL

**Validation**: Screenshot → Compare section count and arrangement → Match or Mismatch

---

## 2. SPACING & PADDING

### 2.1 Margins
- ✅ Space between sections matches mockup (within 1px tolerance)
- ✅ Margins around page edges match mockup
- ✅ Gap between columns/rows matches mockup
- ❌ Margins significantly larger/smaller = FAIL

**Standard values to check**:
- Page edge padding: 16px, 20px, 24px, 32px (common mobile/desktop standards)
- Section margin: 16px, 24px, 32px (block separation)
- Element margin: 8px, 12px, 16px (item spacing)

**Validation**: Measure in browser DevTools → Compare with mockup measurements

### 2.2 Padding (Internal)
- ✅ Padding inside cards/containers matches mockup
- ✅ Padding inside buttons matches mockup
- ✅ Padding inside form fields matches mockup
- ❌ Padding off by more than 2px = FLAG for review

**Common padding standards**:
- Card padding: 16px, 20px, 24px
- Button padding: 8px-12px (height), 16px-24px (width)
- Input field padding: 10px-12px

**Validation**: Inspect element in DevTools → Compare padding values

### 2.3 Line Height & Text Spacing
- ✅ Space between text lines matches mockup
- ✅ Paragraph spacing matches mockup
- ✅ List item spacing matches mockup
- ❌ Line height significantly different = FAIL

**Standard line heights**:
- Heading: 1.2-1.4x font size
- Body text: 1.5-1.6x font size
- Compact text: 1.3-1.4x font size

**Validation**: Measure line-height in DevTools → Compare with mockup

---

## 3. TYPOGRAPHY

### 3.1 Font Family
- ✅ Same font family for headings as mockup
- ✅ Same font family for body text as mockup
- ✅ Same font family for labels/captions as mockup
- ❌ Different font family = FAIL

**Validation**: Inspect element → Check font-family property

### 3.2 Font Size
- ✅ H1 size: Matches mockup (±1px tolerance)
- ✅ H2 size: Matches mockup (±1px tolerance)
- ✅ H3 size: Matches mockup (±1px tolerance)
- ✅ Body size: Matches mockup (±1px tolerance)
- ✅ Small/caption size: Matches mockup (±1px tolerance)
- ❌ Font size off by >2px = FLAG

**Standard sizes**:
- H1: 28px-32px
- H2: 24px-28px
- H3: 20px-24px
- Body: 14px-16px
- Small: 12px-14px

**Validation**: Inspect element → Check font-size property → Compare with mockup

### 3.3 Font Weight
- ✅ Bold text is bold (weight 600-700)
- ✅ Regular text is regular (weight 400)
- ✅ Light text is light (weight 300-400)
- ✅ Headings have correct weight (typically 600-700)
- ❌ Wrong weight (regular when should be bold) = FAIL

**Validation**: Inspect element → Check font-weight property

### 3.4 Text Alignment
- ✅ Left-aligned text: Left
- ✅ Center-aligned text: Center
- ✅ Right-aligned text: Right
- ✅ Justified text: Justified (if present)
- ❌ Alignment different from mockup = FAIL

**Validation**: Inspect element → Check text-align property

### 3.5 Text Color
- ✅ Primary text color matches mockup
- ✅ Secondary/muted text color matches mockup
- ✅ Heading color matches mockup
- ✅ Link color matches mockup
- ✅ Error/warning color matches mockup
- ❌ Color hex differs from mockup = FLAG

**Validation**: Inspect element → Check color property → Compare hex values

### 3.6 Text Transformation
- ✅ Uppercase text: UPPERCASE
- ✅ Lowercase text: lowercase
- ✅ Capitalize text: Capitalize
- ✅ Normal text: Normal
- ❌ Wrong transformation = FAIL

**Validation**: Inspect element → Check text-transform property

---

## 4. VISUAL HIERARCHY

### 4.1 Element Prominence
- ✅ Most important element is visually largest/boldest in mockup and implementation
- ✅ Secondary elements are smaller/lighter in mockup and implementation
- ✅ Tertiary elements are smallest/lightest in mockup and implementation
- ❌ Visual hierarchy reversed or different = FAIL

**Validation**: Compare size, weight, color prominence in mockup vs implementation

### 4.2 Contrast
- ✅ Text contrast meets WCAG AA standard (4.5:1 for normal text, 3:1 for large text)
- ✅ Text legible on background (mockup and implementation)
- ❌ Poor contrast = FAIL

**Validation**: Use contrast checker tool on both mockup and implementation

### 4.3 Emphasis
- ✅ Bold text is bold
- ✅ Italic text is italic
- ✅ Underlined text is underlined
- ✅ Highlighted text is highlighted
- ❌ Emphasis missing or wrong = FAIL

**Validation**: Visually compare mockup and implementation

---

## 5. COLORS & COLOR PALETTE

### 5.1 Primary Colors
- ✅ Primary brand color matches mockup
- ✅ Primary button color matches mockup
- ✅ Primary text color matches mockup
- ❌ Color differs from mockup hex = FLAG

**Validation**: Color picker on mockup and implementation → Compare hex

### 5.2 Secondary Colors
- ✅ Secondary button color matches mockup
- ✅ Secondary text color matches mockup
- ✅ Accent color matches mockup (if present)
- ❌ Color differs = FLAG

**Validation**: Color picker tool

### 5.3 Neutral Colors
- ✅ Background color matches mockup
- ✅ Border color matches mockup
- ✅ Disabled text/button color matches mockup
- ❌ Color differs = FLAG

**Validation**: Color picker on both

### 5.4 Status Colors
- ✅ Error/danger red: Matches mockup
- ✅ Warning/caution yellow/orange: Matches mockup
- ✅ Success/positive green: Matches mockup
- ✅ Info/neutral blue: Matches mockup
- ❌ Color differs = FLAG

**Validation**: Color picker comparison

---

## 6. COMPONENTS & ELEMENTS

### 6.1 Buttons
- ✅ Button size matches mockup (height, width, padding)
- ✅ Button text matches mockup (size, weight, color)
- ✅ Button background color matches mockup
- ✅ Button border/outline matches mockup (if present)
- ✅ Button corners (border-radius) match mockup
- ✅ Button hover state matches mockup (if shown)
- ✅ Button disabled state matches mockup (if present)
- ❌ Any difference = FLAG/FAIL

**Validation**: Click inspect tool → Compare button dimensions, colors, states

### 6.2 Input Fields
- ✅ Input field height matches mockup
- ✅ Input field width matches mockup
- ✅ Input field border matches mockup
- ✅ Input field background color matches mockup
- ✅ Input field text color matches mockup
- ✅ Input field placeholder color matches mockup (if visible)
- ✅ Input field corner radius matches mockup
- ✅ Input field focus state matches mockup (if shown)
- ❌ Any difference = FLAG

**Validation**: Inspect input elements → Compare all properties

### 6.3 Cards/Containers
- ✅ Card dimensions match mockup
- ✅ Card padding matches mockup
- ✅ Card background color matches mockup
- ✅ Card border matches mockup (color, width, style)
- ✅ Card shadow matches mockup (if present)
- ✅ Card corner radius matches mockup
- ✅ Card spacing between items matches mockup
- ❌ Any difference = FLAG

**Validation**: Inspect card elements → Compare all properties

### 6.4 Icons
- ✅ Icon present if mockup shows icon
- ✅ Icon name/identifier matches mockup (same icon)
- ✅ Icon size matches mockup (within 2px)
- ✅ Icon color matches mockup
- ✅ Icon position matches mockup (left, right, center)
- ✅ Icon alignment matches mockup (vertical, horizontal)
- ❌ Icon missing or wrong = FAIL
- ❌ Icon size/color different = FLAG

**Validation**: Visually compare and inspect icon properties

### 6.5 Icon Sizing & Styling (Material Symbols / Font Icons)
- ✅ Custom icon sizes use `!important` when overriding base icon class defaults (e.g., `.material-symbols-outlined` sets font-size: 24px)
- ✅ Large/background icons (120px+) need explicit font-size rule separate from base icon class
- ✅ CSS computed styles match mockup (use DevTools > Inspect > Computed tab, not just HTML)
- ✅ Icon styling classes don't conflict with parent container constraints (overflow, clip-path)
- ❌ Icon size wrong in browser but correct in HTML = Check for CSS specificity issues
- ❌ Icon invisible/clipped = Check parent overflow-hidden, position constraints

**Root causes to verify**:
1. Base icon font-size (e.g., 24px) overrides custom size → Add `!important`
2. Parent `overflow: hidden` clips absolutely positioned icons → Remove overflow constraint
3. CSS class not in compiled stylesheet → Check if Tailwind/PostCSS content scanner includes file
4. Inline styles not rendering → Check if framework strips style attributes during hydration

**Validation**: 
- Compare HTML source vs browser-rendered HTML (attributes may be stripped)
- Check computed styles in DevTools, not just element inspection
- Verify CSS rule exists in stylesheet (DevTools > Sources > CSS file)
- Test with explicit `!important` for size-critical icon overrides
- Ensure parent container allows positioned children (remove overflow-hidden)

### 6.5 Badges/Labels/Tags
- ✅ Badge size matches mockup
- ✅ Badge background color matches mockup
- ✅ Badge text color matches mockup
- ✅ Badge border/outline matches mockup (if present)
- ✅ Badge corner radius matches mockup
- ✅ Badge position matches mockup
- ❌ Any difference = FLAG

**Validation**: Inspect badge elements

### 6.6 Links
- ✅ Link color matches mockup
- ✅ Link underline present/absent as per mockup
- ✅ Link hover state matches mockup
- ✅ Link cursor (pointer) works
- ❌ Color or style different = FLAG

**Validation**: Inspect link elements, hover to check state

### 6.7 Forms/Sections
- ✅ Form layout matches mockup
- ✅ Form field order matches mockup
- ✅ Form field labels match mockup (text, position, size)
- ✅ Form validation messages match mockup (if present)
- ✅ Form spacing matches mockup
- ❌ Layout or order different = FAIL

**Validation**: Compare form structure visually

---

## 7. BORDERS & OUTLINES

### 7.1 Border Width
- ✅ Border width matches mockup (1px, 2px, etc.)
- ❌ Border width different = FLAG

**Validation**: Inspect element → Check border-width

### 7.2 Border Color
- ✅ Border color matches mockup hex
- ❌ Color different = FLAG

**Validation**: Inspect element → Check border-color

### 7.3 Border Style
- ✅ Solid border: Solid
- ✅ Dashed border: Dashed (if present)
- ✅ Dotted border: Dotted (if present)
- ❌ Style different = FLAG

**Validation**: Inspect element → Check border-style

### 7.4 Border Radius (Corners)
- ✅ Sharp corners (0px) if mockup shows sharp
- ✅ Rounded corners (4px, 8px, 12px, etc.) match mockup
- ✅ Full circle (50%) if mockup shows circle
- ❌ Radius different from mockup = FLAG (±1px tolerance acceptable)

**Validation**: Inspect element → Check border-radius

---

## 8. SHADOWS & DEPTH

### 8.1 Box Shadows
- ✅ Shadow presence: Present if mockup shows shadow
- ✅ Shadow absence: Absent if mockup shows no shadow
- ✅ Shadow color matches mockup (if present)
- ✅ Shadow blur matches mockup (if present)
- ✅ Shadow spread matches mockup (if present)
- ✅ Shadow offset matches mockup (if present)
- ❌ Shadow missing when should be present = FAIL
- ❌ Shadow present when should be absent = FAIL

**Validation**: Inspect element → Check box-shadow property

### 8.2 Opacity/Transparency
- ✅ Element opacity matches mockup (fully opaque, semi-transparent, etc.)
- ✅ Background opacity matches mockup
- ❌ Opacity different = FLAG

**Validation**: Inspect element → Check opacity property

---

## 9. RESPONSIVE BEHAVIOR

### 9.1 Mobile (320px - 480px)
- ✅ Layout changes match mockup (stacking, reordering)
- ✅ Font sizes reduce appropriately
- ✅ Spacing reduces appropriately
- ✅ Elements stack vertically if mockup shows stacking
- ❌ Layout broken or different from mockup = FAIL

**Validation**: Test at 320px, 375px, 480px viewports → Compare with mockup

### 9.2 Tablet (768px - 1024px)
- ✅ Layout matches mockup (2-column, 3-column, etc.)
- ✅ Font sizes match mockup
- ✅ Spacing matches mockup
- ❌ Layout or sizing different = FAIL

**Validation**: Test at 768px, 1024px viewports

### 9.3 Desktop (1280px+)
- ✅ Layout matches mockup
- ✅ Font sizes match mockup
- ✅ Spacing matches mockup
- ❌ Any difference = FAIL

**Validation**: Test at 1280px, 1440px, 1920px viewports

---

## 10. DOM STRUCTURE

### 10.1 Element Hierarchy
- ✅ Parent-child relationships match mockup
- ✅ Element nesting matches mockup
- ✅ Section structure matches mockup
- ❌ Wrong hierarchy = FAIL (can cause styling/accessibility issues)

**Validation**: Compare DOM tree structure

### 10.2 Element Count
- ✅ Number of buttons matches mockup
- ✅ Number of cards matches mockup (for layout, not data content)
- ✅ Number of sections matches mockup
- ❌ Missing elements = FAIL
- ❌ Extra elements = FLAG (only if not data-related)

**Validation**: Count elements in mockup and implementation

### 10.3 Element Classes/IDs
- ✅ Elements have semantic class names (not just 'div1', 'div2')
- ✅ IDs match mockup (if present)
- ✅ Consistent naming conventions

**Validation**: Inspect element → Check classes and IDs

---

## 11. INTERACTION STATES

### 11.1 Hover State
- ✅ Button hover color matches mockup (if shown)
- ✅ Link hover state matches mockup
- ✅ Card hover effect matches mockup (if shown)
- ✅ Cursor changes on interactive elements
- ❌ Hover state missing = FLAG

**Validation**: Hover over interactive elements → Compare with mockup

### 11.2 Active/Pressed State
- ✅ Active button color matches mockup
- ✅ Pressed state visual feedback matches mockup
- ❌ State missing or different = FLAG

**Validation**: Click element → Compare state

### 11.3 Focus State
- ✅ Focused element has visible focus indicator (accessibility requirement)
- ✅ Focus indicator color/style matches mockup (if shown)
- ❌ No focus indicator = FAIL (accessibility issue)

**Validation**: Tab through elements → Check focus indicator

### 11.4 Disabled State
- ✅ Disabled button color matches mockup
- ✅ Disabled button opacity/styling matches mockup
- ✅ Disabled input field styling matches mockup
- ❌ Disabled state missing or different = FLAG

**Validation**: Disable element (if applicable) → Compare with mockup

---

## 12. IMAGES & ASSETS

### 12.1 Image Presence
- ✅ Image present if mockup shows image
- ✅ Image absent if mockup shows no image
- ❌ Image missing when should be present = FAIL
- ❌ Image present when should be absent = FAIL

**Validation**: Visually compare mockup and implementation

### 12.2 Image Size
- ✅ Image dimensions match mockup (within content-aware scaling)
- ✅ Image aspect ratio matches mockup
- ❌ Image significantly larger/smaller = FLAG

**Validation**: Inspect image element → Check dimensions

### 12.3 Image Position
- ✅ Image position matches mockup (left, center, right)
- ✅ Image alignment matches mockup
- ❌ Position different = FLAG

**Validation**: Visually compare positioning

### 12.4 Image Styling
- ✅ Image border matches mockup (if present)
- ✅ Image corner radius matches mockup (if present)
- ✅ Image shadow matches mockup (if present)
- ❌ Styling different = FLAG

**Validation**: Inspect image element

---

## 13. ACCESSIBILITY

### 13.1 Alt Text
- ✅ All images have descriptive alt text
- ✅ Icons have aria-labels (if not decorative)
- ❌ Missing alt text = FAIL

**Validation**: Inspect image alt attributes

### 13.2 Color Contrast
- ✅ Text contrast >= 4.5:1 (normal text)
- ✅ Text contrast >= 3:1 (large text)
- ❌ Low contrast = FAIL

**Validation**: Use contrast checker

### 13.3 Font Size
- ✅ Minimum font size >= 12px (preferably 14px+)
- ❌ Font too small = FLAG

**Validation**: Inspect element

### 13.4 Interactive Elements
- ✅ Buttons/links are keyboard accessible (tab order)
- ✅ Touch target size >= 44px x 44px (mobile)
- ❌ Not accessible = FAIL

**Validation**: Keyboard navigation test

---

## 14. CSS SPECIFICITY & STYLING DEBUGGING

### 14.1 CSS Cascade & Specificity Issues
- ✅ Element classes render in final HTML with correct styling applied
- ✅ Inline styles are preserved and applied (not stripped by framework)
- ✅ CSS rules exist in compiled stylesheet (not just in source)
- ✅ Parent class defaults don't override child-specific styles (use `!important` if needed)
- ❌ Styles correct in source HTML but wrong in browser = Specificity conflict
- ❌ CSS rules don't appear in stylesheet = Content scanner not finding file or Tailwind not generating

**Common issues**:
1. **Base class font-size override**: Material Symbols icons default to 24px. Custom large icons need `font-size: 120px !important;`
2. **Parent container constraints**: `overflow: hidden` clips absolutely positioned children → Remove unless needed
3. **Framework hydration stripping**: Inline styles may be stripped during React hydration → Use CSS classes instead
4. **Tailwind content scanner**: Arbitrary values like `text-[120px]` need explicit Tailwind config or manual CSS class definition
5. **CSS specificity**: Utility classes lower specificity than component classes → Use `!important` for deliberate overrides

**Validation**:
- DevTools Inspect > Computed tab (shows final computed styles)
- DevTools Sources > CSS file (verify rule exists)
- Compare HTML source vs rendered HTML (may differ after hydration)
- Test on production build, not dev server (dev has different CSS generation)

### 14.2 Styled Element Verification Checklist
For any styled element (icon, button, card, etc.):

1. ☐ Does it appear in mockup? (Yes = must implement exactly)
2. ☐ Is the style in source code? (className or style prop)
3. ☐ Does the CSS rule exist in compiled stylesheet?
4. ☐ Do computed styles match mockup? (DevTools Computed tab)
5. ☐ Are parent constraints preventing display? (overflow, clip-path, z-index)
6. ☐ Is framework stripping the attribute? (test rendered HTML vs source)

If styled element wrong in browser:
- ✅ First: Check computed styles (not element HTML)
- ✅ Second: Verify CSS rule exists in .css file
- ✅ Third: Check parent container constraints
- ✅ Fourth: Look for CSS specificity conflicts (use `!important` if needed)
- ✅ Fifth: Test on production build (dev may differ)

---

## 15. VALIDATION CHECKLIST — TIER 1 (VISUAL/STRUCTURAL PARITY)

**PREREQUISITE**: Route must have passed Tier 0 smoke test (§0). Do not attempt Tier 1 on a failed route.

**Coverage enumeration (ANTI-SAMPLING):**
Enumerate ALL of the following before starting Tier 1:
- **Template coverage**: List every distinct page template type (e.g. "organizations hub", "organizations detail", "people hub", "people detail", etc. — currently ~14 types across hubs + entity details). For each template, mark whether a mockup exists. If mockup exists, one representative entity instance of that template must be checked against the mockup (equivalence partitioning: one case per class, but no class skipped).
- **Route coverage**: List all live routes that will be deployed (e.g. `/organizations`, `/organizations/o_rbi`, `/organizations/o_air_india`, ... derived from compiled entity data, not assumed). All routes already passed Tier 0 smoke test (as a prerequisite). Now, mark which routes have mockup-backed templates vs. template-only inference (no mockup sample exists). This enumeration must be in the final report; hiding it is not permitted.

**Per-component checklist** (for each component/section matching a mockup):

```
Component: [Name]
Section: [Location in page]
Mockup sample used: [e.g. "organizations/o_reserve_bank_of_india" or "NONE — template inference"]

Layout:
  ☐ Grid/flex structure matches
  ☐ Element positioning matches
  ☐ Section order matches

Spacing:
  ☐ Margins match (±1px)
  ☐ Padding match (±1px)
  ☐ Line height matches

Typography:
  ☐ Font family matches
  ☐ Font size matches (±1px)
  ☐ Font weight matches
  ☐ Color matches

Components:
  ☐ All elements present
  ☐ Element styling matches
  ☐ Icon present/correct
  ☐ Colors match

Interactions:
  ☐ Hover state correct
  ☐ Active state correct
  ☐ Focus indicator present
  ☐ Disabled state correct

Responsive:
  ☐ Mobile layout matches
  ☐ Tablet layout matches
  ☐ Desktop layout matches

Overall:
  ☐ Visual hierarchy maintained
  ☐ No visual regressions
  ☐ DOM structure correct
  ☐ Accessibility compliant

Status: ✅ PASS or ❌ FAIL
Issues Found: [List any]
```

---

## 16. VALIDATION OUTPUT FORMAT (TIER 1 — REQUIRED ARTIFACTS)

Report must include the following fields. **Reports missing any field are automatically FAIL, regardless of prose claims.**

```
DEV_UI Validation Report — [Page Type or Entity Instance]
Generated: [timestamp]
Checked by: [agent ID/name]

TIER 0 PREREQUISITES
  Tier 0 status: ✅ PASS (route health smoke test passed)
  
TIER 1 SCOPE & COVERAGE
  Template types audited: [list ~14 types, mark which have mockup vs. template-only]
  Routes checked: [n total, derived from compiled data]
  Routes failed smoke test (Tier 0): [list or "NONE"]
  Template representatives checked: [e.g. "organizations/o_rbi (mockup: YES), people/p_john_doe (mockup: NO — template inference)"]

VISUAL COMPARISON ARTIFACTS (REQUIRED — do not skip)
  Structural snapshot diff:
    - Baseline: [path to mockup HTML/YAML aria snapshot]
    - Implementation: [path to live page capture, YAML aria snapshot format]
    - Diff file: [path to structured diff showing element tree differences]
    - Diff result: [% elements matching, or "PASS"/"FAIL"]
  
  Visual (pixel) diff:
    - Baseline screenshot: [path to mockup PNG screenshot]
    - Implementation screenshot: [path to live page PNG screenshot]
    - Diff image: [path to highlighted-differences PNG]
    - Diff metric: [pixel-diff %, threshold used, e.g. "2.3% diff (threshold 5%)"]
    - Diff result: [PASS / FLAG / FAIL]

RUNTIME VALIDATION
  Console errors captured: [path to console log, or "NONE"]
  Pages with errors: [list routes that fired console.error, or "NONE"]
  
DETAILED FINDINGS (from §1–14 checklist)
  ✅ Section layout: Matches mockup
  ✅ Typography: All sizes and weights correct
  ⚠️ Spacing: Card padding 1px off (11px vs 12px) — Minor, acceptable
  ✅ Colors: All hex values match
  ✅ Icons: Correct icon, correct color, correct size
  ❌ Border radius: 6px implemented vs 8px in mockup — Needs fix
  [... continue per §1–14 ...]

ISSUES FOUND
  1. Card padding: 11px (should be 12px) — Fix: Adjust CSS padding
  2. [Any other issues]
  
SUMMARY
  Tier 0 status: ✅ PASS
  Tier 1 status: ✅ PASS or ⚠️ FLAG or ❌ FAIL
  Artifact completeness: ✅ All required fields present and linked
  
RECOMMENDATION
  [PASS — all tiers passed, artifacts verified, ready for QA_VISUAL re-spot-check]
  OR [FLAG — minor acceptable mismatches, see issues; QA_VISUAL review recommended]
  OR [FAIL — critical issues found; developer fixes required before QA review]
  
NOTES FOR QA_VISUAL (the independent verifying agent)
  - Mockup coverage: [which templates are mockup-backed vs. inference]
  - Artifact paths: [all diffs and screenshots attached and ready for independent review]
  - Manual re-checks suggested: [any areas where automated diff may be unreliable]
```

**Artifact paths must be absolute, accessible to the next agent** (e.g. file paths in the repo or scratchpad, not in-memory references). **Prose prose claims ("looks good") replace artifacts will cause automatic FAIL during QA_VISUAL review and judge evaluation.**

---

## 17. SUMMARY: MOCKUP CONFORMANCE RULES

**Data can differ**: Names, dates, numbers, images based on content integration

**Everything else must match**:
- ✅ Layout structure (grid, flex, sections)
- ✅ Typography (fonts, sizes, weights, colors)
- ✅ Spacing (margins, padding, line-height)
- ✅ Colors (all hex values)
- ✅ Components (buttons, inputs, cards, badges, icons)
- ✅ Icons (presence, style, color, size)
- ✅ Borders (width, color, style, radius)
- ✅ Shadows (if present, must match)
- ✅ Interaction states (hover, active, focus, disabled)
- ✅ Responsive behavior (mobile, tablet, desktop)
- ✅ DOM structure (hierarchy, nesting)
- ✅ Accessibility (contrast, alt text, focus indicators)

**Tolerance levels**:
- Spacing: ±1px acceptable
- Font size: ±1px acceptable
- Colors: Exact hex match required
- Dimensions: ±1px acceptable
- Radius: ±1px acceptable

**Validation approach**:
1. Load mockup and implementation side-by-side
2. Use browser DevTools (Inspect Element)
3. Use color picker for exact color matching
4. Measure spacing/sizes with DevTools ruler
5. Compare DOM structure
6. Test all interaction states
7. Test responsive at 3+ breakpoints
8. Test keyboard accessibility
9. Document all findings
10. Report PASS/FLAG/FAIL with specific issues

---

## 18. CONVENTIONS ADOPTED — ESTABLISHED QA STANDARDS

This ruleset adopts terminology and workflow patterns from established software QA practice. Future edits should reference and extend these conventions rather than inventing new home-grown language.

- **Smoke testing** (ISTQB glossary): Tier 0 route-health checks (HTTP 200, zero console errors) are shallow "does it even run" smoke tests applied to the full breadth of routes before deeper visual testing proceeds.

- **Structural snapshot testing** (Jest snapshot pattern, Playwright `toMatchAriaSnapshot`): Tier 1 structural comparison uses accessibility-tree snapshots (YAML format) to compare DOM structure independently of pixel rendering — avoids false negatives from font rendering, anti-aliasing, etc.

- **Visual regression testing with thresholds** (BackstopJS, Percy, Playwright `toHaveScreenshot`): Tier 1 visual comparison produces pixel-diff images + numeric diff % with a configurable threshold (e.g. Playwright default: max 5% diff); reports include the artifact (diff image, not prose summary).

- **Equivalence partitioning** (ISTQB glossary): Tier 1 coverage requirement — every distinct page template type is one equivalence class; at least one representative entity per class must be checked against its mockup sample (if mockup exists). No class is skipped; sampling within a class (checking 1 org and claiming "all orgs pass" without checking others) is not permitted for the class-level rule, though Tier 0 smoke-tests all instances.

- **Four-eyes principle / maker-checker** (audit/governance best practice): The QA_VISUAL agent (verifying) must be structurally separate from DEV_UI (building). QA_VISUAL re-derives its own route list from compiled data and re-runs its own captures/diffs rather than accepting DEV_UI's artifacts as proof.

- **Evidence artifacts over prose sign-off** (ISO/IEC/IEEE 29119-3, successor to IEEE 829): Report sign-off requires named artifact files (diff image, snapshot YAML, console log) linked in the output format (§16), not just prose summary ("looks good"). Prose claims unaccompanied by artifacts are not evidence.

**Citation for future edits**: When tightening or extending this ruleset, reference these established standards by name (smoke test, snapshot testing, equivalence partitioning, four-eyes, IEEE 29119-3) rather than re-describing the concept or inventing a new term.

---

*This ruleset ensures web app implementation is pixel-perfect match to mockup (except data content), backed by evidence and independent verification.*