---
name: design
description: Translates a spec into a navigable HTML prototype, exports all screens as Figma-ready frames, and imports them into a target Figma file. Use after /spec when the project has a UI. Covers three sequential phases: prototype → figma-export → figma-import.
---

# Design

## Overview

Turn a written spec into a visual, interactive prototype — then import every screen into Figma. This skill bridges the gap between specification and design tooling without requiring a designer or manual screenshot work.

The three phases are **strictly sequential**. Do not start Phase 2 until the human approves Phase 1, and do not start Phase 3 until Phase 2 is complete.

```
SPEC ──→ PHASE 1: HTML Prototype ──→ PHASE 2: Figma Export HTML ──→ PHASE 3: Figma Import
              │                              │                             │
              ▼                              ▼                             ▼
           Human                         Human                          Human
           reviews                       reviews                        provides
           screens                       frames                         Figma URL
```

---

## Phase 1 — HTML Prototype

### Goal

Produce a single self-contained HTML file with all app screens, navigable via a sidebar panel. Each screen is a `390×844 px` phone frame rendered at full fidelity using only HTML + CSS (no external dependencies, no build step).

### Inputs required

Before writing a single line of HTML, collect from the spec or ask the human:

- App name and color palette (primary, secondary, background, surface, text)
- Complete screen list with navigation relationships
- UI patterns: tab bar? top nav? bottom sheets? modals?
- Typography and border-radius conventions
- Any existing design reference (screenshots, Figma links, competitor apps)

### Structure of the prototype file

```
spezy-prototype.html  (single file, fully self-contained)
├── <style>            CSS variables, component classes, all screen CSS
├── .dev-nav           Fixed sidebar listing every screen — clicking shows it
├── .phone-frame       390×844 container; clips overflow
│   ├── .screen        Each screen (position:absolute; hidden by default)
│   └── .tab-bar       Bottom tab bar (shown on main tab screens only)
└── <script>           Navigation engine + any interactive micro-behaviours
```

**Dev nav panel rules:**

- Fixed position, left side, dark background with blur
- Grouped by flow: Auth, Onboarding, Main tabs, Feature flows, Dialogs
- Clicking a button calls `navTo('screen-name')` and highlights the active entry
- On mobile viewports (`< 900px`) the panel hides; body padding resets to 20px

**Screen rules:**

- Every screen is `position: absolute; inset: 0; opacity: 0; pointer-events: none`
- Active screen gets `opacity: 1; pointer-events: auto; z-index: 10`
- Transitions: `opacity 0.28s ease, transform 0.28s ease`
- Every screen has a `<div class="status-bar">` showing `9:41 ●●●● 100%`
- Main tab screens show the shared `.tab-bar`; full-screen flows (detail, cooking) hide it

**Design system rules:**

- All colours, radii, shadows, font sizes live in `:root { --var: value }` CSS variables
- Never hardcode hex values outside `:root`
- Component classes (`.card`, `.chip`, `.btn-primary`, etc.) are defined once and reused
- Mobile-first; fixed phone frame simulates iOS dimensions

### Navigation engine (JavaScript)

```javascript
const screens = { 'screen-name': 'screen-name', ... };
let history = ['first-screen'];

function show(name) {
  document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
  document.getElementById(screens[name])?.classList.add('active');
  document.querySelectorAll('.dev-nav button')
    .forEach(b => b.classList.toggle('active', b.dataset.screen === name));
  // update tab bar, status bar colour, etc.
}

function navTo(name) { history.push(name); show(name); }
function back() { if (history.length > 1) { history.pop(); show(history.at(-1)); } }
```

Attach `data-nav="screen-name"` to buttons/links; attach `data-back` to back buttons. The script handles both via `querySelectorAll`.

### Interactive micro-behaviours

Implement where meaningful:
- Toggle switches (weaning mode, settings)
- Chip selection (add/remove ingredient chips)
- Tab switching inside a detail screen (Ingredients / Steps / Nutrition)
- Star rating tap
- Loading state → result state (setTimeout or click trigger)

Do NOT implement: API calls, persistence, authentication, real timers.

### Deliverable

A single HTML file at `<project>/prototypes/<app-name>-prototype.html`.

Open it in the browser. Walk through every flow listed in the dev nav. Only proceed to Phase 2 after the human has reviewed and approved.

---

## Phase 2 — Figma Export HTML (5 Steps)

### Goal

Produce a second HTML file where **every screen and every dialog is a visible, static `390×844` frame** laid out in a scrollable grid. This file is consumed by the Figma MCP to import all frames in one operation.

The file is built in exactly **5 steps**, pausing after each for human review before continuing.

### Rules for the export file

```
<head>
  <script src="https://mcp.figma.com/mcp/html-to-design/capture.js" async></script>
</head>
<body>  <!-- gray background, flex-wrap grid, gap: 48px; padding: 48px 40px -->
  <div class="group-break"><span>Group Name</span></div>  <!-- separator -->
  <div class="frame-wrapper">           <!-- flex column, align-items: center -->
    <div class="frame" id="frame-xxx"> <!-- 390×844, overflow:hidden, position:relative -->
      <!-- static screen HTML — NO opacity:0, NO position:absolute on screens -->
    </div>
    <div class="frame-label">01 · Screen Name</div>
  </div>
</body>
```

**Group rules:**
- One `<div class="group-break">` per logical group (Auth, Onboarding, Home, Chef AI, …)
- A group contains ONLY the screens and dialogs belonging to that flow/feature
- Dialogs (bottom sheets, modals, toasts) are separate frames within the same group as their parent screen, labeled `NNb · Dialog · Name`

**Frame rules:**
- Each `.frame` is exactly `390×844 px` with `overflow: hidden`
- All screen content is static (no JS interactions, no `opacity:0`)
- Status bar always visible with correct colour (dark on dark screens, light on light)
- Tab bar rendered in active state for the relevant tab
- Scrollable content shows the initial viewport (top of the screen)
- The Figma capture script (`capture.js`) must be in `<head>`

### Step breakdown

Break the work into 5 groups. Define the groups based on the app's structure — a typical split:

| Step | Content |
|------|---------|
| 1 | CSS foundation + Auth screens |
| 2 | Main tab screens |
| 3 | Primary feature flow (e.g., AI generation flow) |
| 4 | Secondary feature flow (e.g., cooking flow, detail screens) |
| 5 | Remaining screens, all dialogs/toasts, title/count cleanup |

**After each step:** open the file in the browser, show the human, wait for approval before the next step.

### Deliverable

A single HTML file at `<project>/prototypes/<app-name>-for-figma.html`.

Open it and verify every frame is visible and correctly rendered. Only proceed to Phase 3 after human approval.

---

## Phase 3 — Figma Import

### Goal

Import every frame from the Figma Export HTML into a target Figma file using the Figma MCP.

### Inputs required

Ask the human for the Figma file URL:

```
Per procedere con l'import, inviami il link al file Figma in cui vuoi
importare le schermate. Formato atteso:
https://www.figma.com/design/<fileKey>/<fileName>?node-id=...
```

### Import procedure

1. **Start a local server** serving the prototypes directory:
   ```bash
   cd <project>/prototypes && python3 -m http.server 8889 &
   ```

2. **Authenticate the Figma MCP** if not already connected:
   ```
   mcp__figma__authenticate → user opens URL → tools become available
   ```

3. **Generate a capture ID** targeting the existing Figma file:
   ```
   mcp__figma__generate_figma_design(outputMode: "existingFile", fileKey: "<key>")
   ```

4. **Open the export file** with the capture parameters:
   ```bash
   open "http://localhost:8889/<app-name>-for-figma.html\
   #figmacapture=<captureId>\
   &figmaendpoint=https%3A%2F%2Fmcp.figma.com%2Fmcp%2Fcapture%2F<captureId>%2Fsubmit\
   &figmadelay=1000"
   ```

5. **Poll** every 5 seconds with `mcp__figma__generate_figma_design(captureId: "...")` until status is `completed`. Poll up to 10 times.

6. **Open Figma** at the returned node URL. Confirm with the human that all frames are visible.

### After import

- Leave the capture script in the for-figma HTML file (enables future re-imports via the Figma toolbar)
- The Figma file is now the source of truth for UI — subsequent `/plan` tasks reference screen node URLs from this file

---

## Verification Checklist

Before marking this skill complete, confirm:

- [ ] Prototype: every screen in the spec is implemented and navigable
- [ ] Prototype: all interactive states (toggles, chips, tabs) work
- [ ] Prototype: no hardcoded hex values outside `:root`
- [ ] Figma export: every screen AND every dialog has its own frame
- [ ] Figma export: groups match logical flows (one group per feature area)
- [ ] Figma export: all frames render correctly in the browser at 390×844
- [ ] Figma import: all frames visible in the target Figma file
- [ ] Figma import: frame names and groups match the HTML labels
