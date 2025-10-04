# Contributing to ZMT (Zero-Miss-Tap) Framework

**Version:** 1.0.0  
**Last Updated:** 2025-10-03  
**Status:** Active  

---

## Table of Contents

1. [Scope and Objectives](#scope-and-objectives)
2. [Pre-Flight Checklist](#pre-flight-checklist)
3. [Acceptance Criteria (CI Gates)](#acceptance-criteria-ci-gates)
4. [Severity Table](#severity-table)
5. [Common Errors and Solutions](#common-errors-and-solutions)
6. [Surface-Specific Critical Points](#surface-specific-critical-points)
7. [Exception and Waiver Process](#exception-and-waiver-process)
8. [Required PR Evidence](#required-pr-evidence)
9. [Metrics and Observability](#metrics-and-observability)
10. [Source of Truth (SoT) Versioning](#source-of-truth-sot-versioning)
11. [Normative References](#normative-references)
12. [UX Design Review Checklist](#ux-design-review-checklist)
13. [Collaboration Best Practices](#collaboration-best-practices)

---

## Scope and Objectives

The **Zero-Miss-Tap (ZMT)** framework enforces accessible, ergonomic touch targets across **7 surfaces**: phone, tablet, desktop, watch, glasses_microdisplay, tv, and ar_vr. This guide helps contributors maintain compliance with ZMT axioms while developing new features or fixing bugs.

### Primary Goals

1. **Preserve Axioms:** Maintain hard limits (44px touch targets, 8px separation, 4.5:1 contrast, etc.).
2. **Maintain Scores:** Keep Tap Safety Score (TSS) ≥90 (phone/tablet), ≥85 (desktop/watch/TV), ≥80 (glasses/AR/VR).
3. **Trace Exceptions:** All deviations from axioms require Architecture Decision Records (ADRs) with 30-day max expiry.

### Non-Goals

- **Not a Linter:** ZMT is not a style guide; it enforces functional, measurable accessibility/ergonomics standards.
- **Not Optional:** CI gates are **fail-fast**; PRs violating axioms without waivers will be blocked.
- **Not Retroactive:** Existing violations are grandfathered; new code must comply (unless ADR waiver granted).

---

## Pre-Flight Checklist

Before submitting a PR, verify **all 13 items** below. Each item maps to a ZMT axiom or enforcement rule.

### Touch Targets

- [ ] **1. Size:** All interactive elements (buttons, links, icons) are ≥44px on touch surfaces (phone/tablet/watch), ≥24px on pointer surfaces (desktop).
- [ ] **2. Separation:** Minimum 8px gap between adjacent interactive elements (6px compact, 4px ultra-compact with hit-slop).
- [ ] **3. Hit-Slop:** If using ultra-compact density (4px separation), invisible hit-slop padding of ≥4px is applied.

### Contrast and Visual Design

- [ ] **4. Text Contrast:** Body text (<18pt) has ≥4.5:1 contrast ratio; large text (≥18pt) has ≥3:1.
- [ ] **5. UI Component Contrast:** Buttons, icons, form controls have ≥3:1 contrast against background.
- [ ] **6. Focus Indicator:** Keyboard/controller focus has ≥3:1 contrast and is clearly visible (not color-only).

### Responsive Design

- [ ] **7. Reflow:** Content works at 320px width without horizontal scrolling (WCAG 2.1 SC 1.4.10).
- [ ] **8. Zoom:** UI remains functional and readable at 200% browser/OS zoom (WCAG 2.1 SC 1.4.4).
- [ ] **9. Line Length:** Text blocks ≤80 characters per line (60 chars for TV, 40 for watch).

### Gestures and Timing

- [ ] **10. Tap Debounce:** Tap/click actions debounced for 120-160ms to prevent double-activation.
- [ ] **11. Gesture Windows:** Double-tap detection within 250-300ms, long-press threshold 300-500ms.

### Accessibility

- [ ] **12. Keyboard Navigable:** All interactive elements reachable via keyboard/controller; no mouse/touch-only actions.
- [ ] **13. No Color-Only Info:** Critical information (errors, required fields, status) does not rely on color alone.

**How to Use:**
- Mark items `[x]` in PR description.
- If any item is `[ ]` (unchecked), provide justification or link to ADR waiver.

---

## Acceptance Criteria (CI Gates)

All PRs must pass the following gates for **each surface**. Failure triggers CI build failure.

| Surface                | TSS Min | MTR Max | Blocker Violations | Alta Violations | Runtime Overhead (p95) |
|------------------------|---------|---------|---------------------|-----------------|------------------------|
| **Phone**              | 90      | 0.02    | 0                   | 0               | ≤10ms                  |
| **Tablet**             | 90      | 0.02    | 0                   | 0               | ≤10ms                  |
| **Desktop**            | 85      | 0.01    | 0                   | 0               | ≤10ms                  |
| **Watch**              | 85      | 0.03    | 0                   | 0               | ≤10ms                  |
| **Glasses Microdisplay** | 80    | 0.02    | 0                   | 0               | ≤10ms                  |
| **TV**                 | 85      | 0.01    | 0                   | 0               | ≤10ms                  |
| **AR/VR**              | 80      | 0.02    | 0                   | 0               | ≤10ms                  |

**Legend:**
- **TSS (Tap Safety Score):** 0-100 metric; 100 = perfect compliance.
- **MTR (Miss-Tap Rate):** % of taps activating wrong or no target.
- **Blocker/Alta:** Severity levels (see [Severity Table](#severity-table)).
- **Runtime Overhead:** Time added by ZMT audit execution (p95 percentile).

**Enforcement:**
- CI runs `tapguard audit --surfaces=all --report=zmt-report.json`.
- Each surface checked individually; **all must pass**.
- PRs with `ZMT-WAIVER` label may bypass gates if ADR attached.

---

## Severity Table

Violations are classified into **4 severity levels**. CI gates fail on **Bloqueante** and **Alta** by default.

| Severity       | Definition                                                                 | Examples                                      | PR Status       | Action Required                          |
|----------------|---------------------------------------------------------------------------|-----------------------------------------------|-----------------|------------------------------------------|
| **Bloqueante** (Blocker) | Axiom violation with no compensatory controls | Button 36px on phone (< 44px), no focus indicator | ❌ **Blocked** | Fix immediately or create ADR waiver     |
| **Alta** (High)          | Axiom violation with compensatory controls present | Button 42px on phone, has hit-slop +4px       | ⚠️ **Requires ADR** | Create ADR waiver or fix                 |
| **Media** (Medium)       | Non-axiom violation; UX degradation           | Button 46px on phone (meets 44px, < 48px recommended) | ⚠️ **Warning** | Optional fix; does not block PR          |
| **Baja** (Low)           | Minor deviation; no user impact               | Separation 10px instead of 12px on TV         | ℹ️ **Info**     | Informational only; no action required   |

**Compensatory Controls:**
- **Hit-Slop:** Invisible padding expanding touch area.
- **Focus Indicator:** 2px+ outline with ≥3:1 contrast.
- **Tooltips/Labels:** Clarify target purpose.
- **Reduce Motion:** Respect `prefers-reduced-motion` media query.
- **Monitoring:** Track MTR for affected component; auto-rollback if threshold exceeded.

---

## Common Errors and Solutions

Below are **8 frequent violations** and their fixes. Use as quick reference during development.

### 1. Button Smaller Than 44px (Touch Surfaces)

**Error:**  
```
[Bloqueante] Button "Submit" is 40x40px on phone (min: 44x44px)
```

**Solution:**  
```css
/* Increase button size */
.button {
  min-width: 44px;
  min-height: 44px;
  padding: 12px 16px;
}
```

**Alternative (with hit-slop):**  
```css
/* Keep visual size 40px, expand touch area with pseudo-element */
.button {
  width: 40px;
  height: 40px;
  position: relative;
}
.button::after {
  content: '';
  position: absolute;
  inset: -4px; /* Adds 4px padding = 48x48px touch area */
}
```

---

### 2. Insufficient Separation Between Buttons

**Error:**  
```
[Alta] Buttons "Cancel" and "Confirm" are 4px apart (min: 8px comfortable, 6px compact)
```

**Solution:**  
```css
/* Increase gap in flexbox layout */
.button-group {
  display: flex;
  gap: 8px; /* or 6px for compact mode */
}
```

**Alternative (with hit-slop for ultra-compact):**  
```css
.button-group {
  gap: 4px; /* Visual spacing */
}
.button::after {
  inset: -2px; /* Each button gets 2px padding = 4px total between buttons */
}
```

---

### 3. Contrast Ratio Too Low

**Error:**  
```
[Alta] Icon "#share" has 2.8:1 contrast (min: 3:1 for UI components)
```

**Solution:**  
```css
/* Darken icon or lighten background */
.icon {
  color: #333; /* Was #777; now 4.5:1 on white background */
}

/* Or add stroke */
.icon {
  stroke: #000;
  stroke-width: 1px;
}
```

**Tool:** Use [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/) or browser DevTools.

---

### 4. No Focus Indicator

**Error:**  
```
[Bloqueante] Input field "email" has no visible focus indicator
```

**Solution:**  
```css
/* Add outline on focus */
input:focus {
  outline: 2px solid #0066cc; /* ≥3:1 contrast with background */
  outline-offset: 2px;
}

/* Or use box-shadow for rounded elements */
button:focus {
  box-shadow: 0 0 0 3px rgba(0, 102, 204, 0.5);
}
```

---

### 5. Horizontal Scroll at 320px Width

**Error:**  
```
[Media] Page requires horizontal scroll at 320px width (WCAG 2.1 SC 1.4.10)
```

**Solution:**  
```css
/* Use flexbox with wrap */
.container {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

/* Or stack elements vertically on narrow screens */
@media (max-width: 320px) {
  .grid {
    grid-template-columns: 1fr; /* Single column */
  }
}
```

---

### 6. Tap Debounce Too Short

**Error:**  
```
[Media] Button "Like" has debounce of 80ms (min: 120ms)
```

**Solution:**  
```javascript
// Increase debounce timeout
let lastTapTime = 0;
button.addEventListener('click', (e) => {
  const now = Date.now();
  if (now - lastTapTime < 120) { // Was 80ms
    e.preventDefault();
    return;
  }
  lastTapTime = now;
  // Handle click
});
```

**Libraries:** Use Lodash `_.debounce(fn, 120)` or similar.

---

### 7. Color-Only Error Indication

**Error:**  
```
[Alta] Required field "password" uses red border only (no icon or text)
```

**Solution:**  
```html
<!-- Add icon and aria-label -->
<label for="password">
  Password <span aria-label="required">*</span>
</label>
<input id="password" required aria-invalid="true" />

<!-- Add error message text -->
<span class="error" id="password-error">
  <svg aria-hidden="true"><!-- Error icon --></svg>
  This field is required
</span>
```

```css
/* Red border + icon, not just color */
input[aria-invalid="true"] {
  border: 2px solid #d32f2f;
  background-image: url('error-icon.svg');
}
```

---

### 8. Non-Keyboard-Accessible Dropdown

**Error:**  
```
[Bloqueante] Dropdown menu "Settings" not reachable via keyboard
```

**Solution:**  
```html
<!-- Use native <select> or ARIA -->
<button aria-haspopup="true" aria-expanded="false" id="settings-btn">
  Settings
</button>
<ul role="menu" aria-labelledby="settings-btn" hidden>
  <li role="menuitem" tabindex="0">Profile</li>
  <li role="menuitem" tabindex="0">Logout</li>
</ul>
```

```javascript
// Handle keyboard navigation
button.addEventListener('keydown', (e) => {
  if (e.key === 'Enter' || e.key === ' ') {
    toggleMenu();
  }
});
```

---

## Surface-Specific Critical Points

Each surface has unique constraints and common failure modes. Prioritize these checks per surface.

### Phone

**Primary Input:** Touch (finger, thumb)  
**Critical Zones:** Bottom 50% of screen (thumb reach); top corners difficult  
**Common Violations:**
- Buttons < 44px
- Separation < 8px
- Contrast < 4.5:1
- No focus indicator (affects keyboard users on iOS/Android with external keyboards)

**Tips:**
- Use `safe-area-inset-*` for notched screens.
- Test one-handed mode; place critical actions in bottom half.
- Avoid hamburger menus; use bottom tab bars.

---

### Tablet

**Primary Input:** Touch (finger, stylus), keyboard (external)  
**Critical Zones:** Edges for one-handed grip; center for two-handed  
**Common Violations:**
- Same as phone
- Landscape/portrait reflow issues
- Horizontal scroll at narrow widths

**Tips:**
- Test both orientations.
- Use adaptive layouts (sidebars collapse on narrow widths).
- Ensure stylus targets ≥24px (higher precision than finger).

---

### Desktop

**Primary Input:** Mouse, trackpad, keyboard  
**Critical Zones:** Top menu bar, sidebars, modal dialogs  
**Common Violations:**
- Buttons < 24px
- Contrast < 4.5:1
- No keyboard focus indicator
- Close buttons < 24px in modals

**Tips:**
- Use CSS `:focus-visible` for keyboard-only focus styles.
- Test with keyboard only (Tab, Enter, Esc).
- Ensure dropdowns/modals trap focus.

---

### Watch

**Primary Input:** Touch (finger), digital crown, side buttons  
**Critical Zones:** Full screen (all targets critical due to small size)  
**Common Violations:**
- Buttons < 40px
- Separation < 6px
- Text too small (<14px)
- Multi-step flows without back navigation

**Tips:**
- Minimize on-screen content; use scrolling or paging.
- Leverage digital crown for scroll/zoom.
- Avoid complex forms; use voice input or companion app.

---

### Glasses (Microdisplay)

**Primary Input:** Gaze + pinch, voice, head gestures  
**Critical Zones:** Central 30° FOV; peripheral UI difficult  
**Common Violations:**
- Targets < 0.5° angular
- Separation < 0.3° angular
- Contrast < 3:1 (glare from environment)
- No depth cues (flat UI hard to target)

**Tips:**
- Use angular sizing (degrees), not pixels.
- Add shadows/outlines for depth perception.
- Avoid clutter; max 3-5 elements per view.
- Respect user's gaze; don't hijack focus.

---

### TV (10-Foot UI)

**Primary Input:** Remote D-pad, voice, game controller  
**Critical Zones:** Center 60% of screen; edges hard to reach  
**Common Violations:**
- Buttons < 48px
- Separation < 12px
- No focus indicator (users lose track of selection)
- Small text (<24px unreadable at 10 feet)

**Tips:**
- Use large, high-contrast focus indicators (4px outline).
- Limit horizontal scrolling (difficult with D-pad).
- Test at 10-foot distance or simulate with 2x zoom.
- Provide voice shortcuts for common actions.

---

### AR/VR (Spatial)

**Primary Input:** Hand tracking, ray-cast, gaze, controllers  
**Critical Zones:** Arm's reach (0.5-1.5m); avoid floor, ceiling, behind user  
**Common Violations:**
- Targets < 1.2° angular
- Separation < 0.5° angular
- No depth cues (flat menus in 3D space)
- Hand jitter not accounted for (requires larger targets)

**Tips:**
- Use angular sizing (degrees); test at multiple distances.
- Add depth shadows, 3D borders, or hover effects.
- Avoid placing UI at extreme angles (>45° from forward).
- Provide snap-to-target assist for hand tracking.

---

## Exception and Waiver Process

### When to Request a Waiver

Request a waiver when:
1. **Technical Constraint:** Platform limitation prevents compliance (e.g., OS-level control size).
2. **Design Trade-off:** Meeting axiom would break critical UX (e.g., information density for power users).
3. **Temporary Solution:** Interim fix while permanent solution is developed (max 30 days).

**Do NOT request waiver for:**
- Laziness or tight deadlines (not valid reasons).
- Aesthetic preferences ("looks better smaller").
- Unaware of ZMT requirements (read this guide first).

---

### ADR Waiver Template

Use `docs/architecture/adr/ADR-ZMT-WAIVER-TEMPLATE.md` as starting point. Required sections:

1. **Context:** Why exception needed (≤3 sentences).
2. **Rules Excepted:** SoT keys, required vs. proposed values, variance %.
3. **Scope:** Routes, cohorts, duration (max 30 days).
4. **Risk:** Impact on MTR, TSS, accessibility, affected user %.
5. **Compensatory Controls:** Hit-slop, focus, tooltips, monitoring (checklist).
6. **Monitoring:** Metrics, thresholds, auto-rollback plan.
7. **Exit Criteria:** DoD with 3 tasks and deadline.
8. **Rollback Plan:** Feature toggles, steps, impact.
9. **Evidence:** Screenshots, zmt-report.json, user research.
10. **Approvals:** Signatures from UX Lead, Accessibility Owner, Eng Lead, PM/PO.

---

### Waiver Lifecycle

1. **Create ADR:** Use template, fill all sections, commit to `docs/architecture/adr/`.
2. **Attach to PR:** Link ADR in PR description, add `ZMT-WAIVER` label.
3. **Review:** UX, Accessibility, and Engineering leads review within 2 business days.
4. **Approve/Reject:** All 4 approvers must sign; majority not sufficient.
5. **Expiry:** Auto-expires at 30 days; renew with new ADR (one-time renewal allowed).
6. **Monitoring:** Check metrics weekly; rollback if MTR/TSS exceeds threshold.

**Rejection Reasons:**
- Insufficient compensatory controls.
- Risk too high (affects >10% users, MTR increase >1%, TSS drop >5 points).
- Scope too broad (waiver should be narrow and specific).
- Missing evidence or approvals.

---

## Required PR Evidence

All PRs touching interactive UI must include:

### 1. ZMT Audit Report

Attach `zmt-report.json` (CI generates automatically). If local testing:
```bash
tapguard audit --surfaces=phone,tablet,desktop --report=zmt-report.json
```

Include in PR description:
```markdown
## ZMT Audit Results
- **Phone TSS:** 92 (threshold: 90) ✅
- **Phone MTR:** 0.018 (max: 0.02) ✅
- **Violations:** 0 Blocker, 0 Alta, 2 Media, 1 Baja
```

---

### 2. Screenshots (Per Surface)

Capture before/after for each affected surface:
- **Phone:** Portrait and landscape (if applicable).
- **Tablet:** Landscape (primary orientation).
- **Desktop:** Full viewport at 1920x1080 and 1366x768.
- **Watch:** Annotate target sizes with rulers.
- **TV:** Simulated 10-foot view (or 2x zoom screenshot).

**Tools:**
- Browser DevTools device emulation.
- `scrcpy` for real Android device.
- Xcode Simulator for iOS.
- Figma/Sketch overlays for design validation.

---

### 3. ADR Link (If Waiver Requested)

```markdown
## ZMT Waiver
This PR requires exception to ZMT axioms. See ADR: [ADR-ZMT-WAIVER-2025-10-03-BUTTON-SIZE](docs/architecture/adr/ADR-ZMT-WAIVER-2025-10-03-BUTTON-SIZE.md)

**Expiry:** 2025-11-02 (30 days)
**Approvals:** ✅ UX Lead, ✅ Accessibility Owner, ✅ Eng Lead, ✅ PM
```

---

### 4. Pre-Flight Checklist (13 Items)

Copy from [Pre-Flight Checklist](#pre-flight-checklist) above and mark items `[x]` or `[ ]` with justification.

---

## Metrics and Observability

### ZMT Reports

CI generates `zmt-report.json` on every PR. View in GitHub Actions artifacts.

**Structure:**
```json
{
  "surfaces": {
    "phone": {
      "tap_safety_score": 92,
      "miss_tap_rate": 0.018,
      "violations": { "blocker": 0, "alta": 0, "media": 2, "baja": 1 },
      "runtime_overhead_p95_ms": 8
    }
  }
}
```

---

### Trend Analysis

Track TSS/MTR over time:
1. Export `zmt-report.json` from each PR.
2. Upload to analytics platform (optional; opt-in).
3. Create dashboards:
   - TSS by surface (line chart over time).
   - Violation counts by severity (stacked bar chart).
   - MTR by route/component (table).

**Tools:**
- Google Analytics (custom events).
- Amplitude, Mixpanel, or similar.
- Internal BI tools (Looker, Tableau).

---

### Opt-In Telemetry

Teams can enable anonymous telemetry:
```json
// .zmtrc
{
  "telemetry": {
    "enabled": true,
    "endpoint": "https://metrics.example.com/zmt",
    "events": ["zmt_audit_run", "zmt_violation", "zmt_waiver_created"]
  }
}
```

**Privacy:**
- No user-identifiable data.
- No code snippets or component names (only generic IDs).
- Aggregate metrics only.

---

## Source of Truth (SoT) Versioning

### Updating SoT

File: `docs/architecture/zmt/sot-zmt-defaults.json`

**Versioning Rules:**
- **Major (x.0.0):** Axiom change (e.g., 44px → 48px minimum).
- **Minor (1.x.0):** New surface, density mode, or enforcement rule.
- **Patch (1.0.x):** Bug fix, clarification, typo.

**Process:**
1. Create ADR proposing change (separate from waiver ADRs).
2. Review with ZMT Core Team (UX, Accessibility, Eng).
3. PR to update `sot-zmt-defaults.json`.
4. Update `version` and `updated` fields.
5. Merge requires 2+ approvals from Core Team.

**Breaking Changes:**
If update increases strictness (e.g., 44px → 48px):
1. **Announce:** 30-day notice to all teams.
2. **Audit:** Run `tapguard audit --sot-version=next` on all repos.
3. **Fix:** Teams remediate violations before deadline.
4. **Enforce:** Update CI to use new version.
5. **Monitor:** Track MTR/TSS for 14 days; rollback if regressions.

---

## Normative References

ZMT is based on these standards. When in doubt, refer to original docs.

1. **WCAG 2.1/2.2 Level AA**  
   [https://www.w3.org/WAI/WCAG21/quickref/](https://www.w3.org/WAI/WCAG21/quickref/)  
   Success Criteria: 1.4.3 (Contrast), 1.4.10 (Reflow), 2.4.7 (Focus Visible), 2.5.8 (Target Size AAA).

2. **Apple Human Interface Guidelines (HIG)**  
   [https://developer.apple.com/design/human-interface-guidelines/](https://developer.apple.com/design/human-interface-guidelines/)  
   Touch targets ≥44pt, spacing ≥8pt.

3. **Material Design 3 (Google)**  
   [https://m3.material.io/](https://m3.material.io/)  
   Touch targets ≥48dp, spacing ≥8dp.

4. **ISO 9241-9:2000 (Fitts' Law)**  
   Ergonomic requirements for office work with VDTs; basis for target size/distance trade-offs.

5. **Shneiderman et al., "Designing the User Interface" (6th ed., 2016)**  
   Line length (50-80 chars), font size, legibility best practices.

---

## UX Design Review Checklist

Use during design review (before implementation) to catch issues early.

### Layout and Spacing

- [ ] All buttons ≥44px (touch) or ≥24px (pointer)?
- [ ] Separation ≥8px between adjacent interactive elements?
- [ ] Critical actions in thumb-reachable zones (phone)?
- [ ] Content reflows at 320px width without horizontal scroll?

### Visual Design

- [ ] Text contrast ≥4.5:1 (small), ≥3:1 (large)?
- [ ] UI components contrast ≥3:1?
- [ ] Focus indicator ≥3:1 contrast, clearly visible?
- [ ] No color-only information (errors, required fields, status)?

### Interaction Design

- [ ] Tap debounce 120-160ms?
- [ ] Double-tap window 250-300ms?
- [ ] Long-press threshold 300-500ms?
- [ ] All actions keyboard/controller accessible?

### Responsive Design

- [ ] Tested at 320px, 375px, 768px, 1024px, 1920px widths?
- [ ] Landscape and portrait orientations (phone/tablet)?
- [ ] Zoom 200% functional?

### Surface-Specific

- [ ] **Phone:** Thumb zones, one-handed mode?
- [ ] **Tablet:** Landscape/portrait, stylus support?
- [ ] **Desktop:** Keyboard navigation, focus trapping in modals?
- [ ] **Watch:** Minimal content, digital crown support?
- [ ] **Glasses:** Angular sizing, central FOV, depth cues?
- [ ] **TV:** 10-foot distance, D-pad navigation, large text?
- [ ] **AR/VR:** Arm's reach, hand jitter, depth shadows?

---

## Collaboration Best Practices

### For Designers

1. **Early Validation:** Share mockups with ZMT team before high-fidelity.
2. **Annotate Sizes:** Include target sizes (px/degrees) in Figma/Sketch.
3. **Test Prototypes:** Use device emulators or real devices.
4. **Request Waivers Early:** If constraint known, start ADR process in design phase.

### For Engineers

1. **Incremental PRs:** Submit small, focused changes; easier to review.
2. **Test Locally:** Run `tapguard audit` before pushing.
3. **Fix Violations:** Don't ignore CI failures; fix or request waiver.
4. **Pair on Waivers:** Work with UX/Accessibility on compensatory controls.

### For Reviewers

1. **Check Pre-Flight:** Verify 13-item checklist marked `[x]`.
2. **Inspect Screenshots:** Validate target sizes visually.
3. **Review ADRs:** Ensure waivers have all 10 sections filled.
4. **Test on Device:** Emulators insufficient; use real phone/tablet/watch.

### For Accessibility Specialists

1. **Triage Violations:** Help teams prioritize fixes (Blocker → Alta → Media → Baja).
2. **Suggest Compensatory Controls:** Hit-slop, focus, ARIA, etc.
3. **Approve Waivers:** Only if risk acceptable and monitoring plan solid.

---

## Questions or Issues?

- **Slack:** `#zmt-support` (internal)
- **GitHub Discussions:** [sabasm/zmt/discussions](https://github.com/sabasm/zmt/discussions)
- **Email:** zmt-core-team@example.com

**Document Version:** 1.0.0  
**Last Updated:** 2025-10-03  
**Next Review:** 2026-01-03 (quarterly)

---

**END OF CONTRIBUTING-ZMT.md**
