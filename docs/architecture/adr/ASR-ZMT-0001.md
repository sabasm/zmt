# ASR-ZMT-0001: ZMT (Zero-Miss-Tap) Framework — Architecture Specification Requirements

**Status:** Active  
**Version:** 1.0.0  
**Date:** 2025-10-03  
**Owner:** ZMT Core Team  
**Applies To:** All surfaces (phone, tablet, desktop, watch, glasses_microdisplay, tv, ar_vr)  

---

## 1. Executive Summary

The **Zero-Miss-Tap (ZMT)** framework — also marketed as **TapGuard 0Mistaps** — is an open-source system to enforce accessible, ergonomic touch targets across all device types. It prevents tap/touch errors ("miss-taps") that degrade user experience, user satisfaction, and accessibility. This Architecture Specification Requirements (ASR) document establishes hard limits (axioms), measurement criteria, and enforcement mechanisms for the framework.

ZMT is built on three pillars:
1. **Scientific Research:** Ergonomics, motor-skill variance, Fitts' Law, and accessibility standards.
2. **Industry Standards:** WCAG 2.1/2.2 Level AA, Apple Human Interface Guidelines, Material Design 3.
3. **Empirical Evidence:** Field telemetry and controlled experiments showing correlation between target size, separation, and miss-tap rates.

---

## 2. Core Axioms (Hard Limits)

These are **non-negotiable defaults**. Exceptions require ADR approval, compensatory controls, and time-bound waivers.

### 2.1 Touch Target Minimum Sizes

| Surface                     | Minimum (px) | Recommended (px) | Notes                                                                 |
|-----------------------------|--------------|------------------|-----------------------------------------------------------------------|
| **Phone (touch)**           | 44           | 48               | Based on WCAG 2.2 Target Size (Level AAA 2.5.8), iOS HIG             |
| **Tablet (touch)**          | 44           | 48               | Same as phone                                                         |
| **Desktop (mouse)**         | 24           | 32               | Smaller due to pointer precision; icons, buttons, close controls      |
| **Watch**                   | 40           | 44               | Limited screen space; touch targets must be large relative to display |
| **Glasses Microdisplay**    | 0.5° angular | 0.8° angular     | Gaze + pinch/tap hybrid input; physical pixels irrelevant             |
| **TV (10-foot UI)**         | 48           | 56               | D-pad navigation; must be visible and reachable via remote            |
| **AR/VR (spatial)**         | 1.2° angular | 1.5° angular     | Ray-cast or hand tracking; angular size ensures consistent reachability|

**Rationale:**  
- **44px mobile**: WCAG 2.2 Level AAA (2.5.8), iOS HIG, Android Material Design all converge on 44-48px for accessible touch targets.
- **40px watch**: Constrained screen real estate requires compromise, but 40px is minimum for reliable touch on small displays.
- **Angular sizing (AR/VR, glasses)**: Physical pixels vary with distance; angular size ensures consistent visual and motor difficulty.

---

### 2.2 Separation Between Targets

Minimum gap between interactive elements to prevent accidental activation:

| Surface                     | Comfortable (px) | Compact (px) | Ultra-Compact (px) | Notes                                      |
|-----------------------------|------------------|--------------|--------------------|-------------------------------------------|
| **Phone / Tablet**          | ≥8               | ≥6           | ≥4 (hit-slop +4px) | Dense UI requires hit-slop compensation   |
| **Desktop**                 | ≥8               | ≥6           | ≥4 (hit-slop +4px) | Toolbars, menus, data tables              |
| **Watch**                   | ≥6               | ≥4           | Not recommended    | Screen too small for ultra-compact        |
| **Glasses**                 | ≥0.3° angular    | ≥0.2° angular| Not recommended    | Gaze precision is limited                 |
| **TV**                      | ≥12              | ≥8           | Not applicable     | D-pad navigation; must be clearly distinct|
| **AR/VR**                   | ≥0.5° angular    | ≥0.3° angular| Not recommended    | Hand tracking jitter requires more space  |

**Rationale:**  
- **8px default**: Prevents "fat finger" errors; supported by iOS spacing guidelines (≥8pt between controls).
- **Hit-slop**: Invisible padding around target that expands touch area; offsets visual spacing reduction in dense UIs.
- **Angular separation**: Analogous to physical separation; accounts for gaze/hand tracking precision limits.

---

### 2.3 Contrast Ratios (WCAG 2.1 Level AA)

| Element Type                | Minimum Contrast Ratio | Notes                                                                 |
|-----------------------------|------------------------|-----------------------------------------------------------------------|
| **Text (< 18pt)**           | 4.5:1                  | Body text, labels, descriptions                                       |
| **Large Text (≥ 18pt)**     | 3:1                    | Headlines, titles                                                     |
| **UI Components / Icons**   | 3:1                    | Buttons, form controls, icons (against background)                    |
| **Focus Indicators**        | 3:1                    | Keyboard focus visible; must not rely on color alone                  |

**Rationale:**  
- WCAG 2.1 Success Criteria 1.4.3 (Contrast Minimum), 1.4.6 (Contrast Enhanced), 1.4.11 (Non-text Contrast).
- Ensures readability for users with low vision, color blindness, or glare-affected environments.

---

### 2.4 Reflow and Responsive Design

| Requirement                | Value            | Notes                                                                 |
|----------------------------|------------------|-----------------------------------------------------------------------|
| **Minimum Width Support**  | 320px            | WCAG 2.1 SC 1.4.10 Reflow; must not require horizontal scrolling     |
| **Maximum Line Length**    | 80 characters    | Legibility best practice; prevents eye fatigue (AAA optional)         |
| **Zoom Support**           | 200%             | WCAG 2.1 SC 1.4.4; must remain functional and readable                |

**Rationale:**  
- **320px**: Smallest iPhone screen width (iPhone SE); ensures mobile accessibility.
- **Line length**: Studies show 50-75 characters optimal; 80 is acceptable upper bound.
- **Zoom**: Required for users with low vision; browser/OS-level zoom must not break layout.

---

### 2.5 Gesture Timing Windows

Prevents accidental activations and conflicts between gesture types:

| Gesture Type               | Timing Window (ms) | Notes                                                                 |
|----------------------------|--------------------|-----------------------------------------------------------------------|
| **Tap Debounce**           | 120-160            | Ignore subsequent taps within window; prevents double-activation      |
| **Double-Tap Detection**   | 250-300            | Second tap must occur within window to count as double-tap            |
| **Long-Press Threshold**   | 300-500            | Press-and-hold for context menu, reorder, etc.                        |
| **Swipe Velocity Minimum** | 0.3 px/ms          | Distinguishes intentional swipe from accidental drag                  |

**Rationale:**  
- Aligns with iOS/Android platform defaults.
- Prevents gesture ambiguity and accidental activations.
- Accounts for motor skill variance (tremors, Parkinson's, age-related slowing).

---

### 2.6 Accessibility Axioms

These are derived from WCAG 2.1/2.2 Level AA and must be upheld in all surfaces:

1. **Focus Visible:** Keyboard/controller focus indicator must have ≥3:1 contrast ratio with adjacent colors (WCAG 2.4.7, 2.4.11).
2. **No Color-Only Information:** Critical information (status, errors, required fields) must not rely on color alone (WCAG 1.4.1).
3. **Keyboard Navigable:** All interactive elements must be reachable via keyboard/controller; no mouse/touch-only actions (WCAG 2.1.1).
4. **Touch Target Size:** Minimum 44x44px for touch (WCAG 2.5.8 Level AAA), 24x24px for pointer (ZMT axiom).
5. **No Motion-Only Actions:** Critical actions must not require motion/gestures only (WCAG 2.5.4).
6. **Zoom Resilient:** UI must remain functional at 200% zoom (WCAG 1.4.4).
7. **Timeout Warnings:** Provide warning and extension option before session expiry (WCAG 2.2.1).

**Enforcement:** All surfaces must pass these checks in CI; exceptions require ADR and compensatory controls.

---

## 3. Non-Functional Requirements

### 3.1 Performance Budget

| Metric                        | Threshold       | Notes                                                                 |
|-------------------------------|-----------------|-----------------------------------------------------------------------|
| **Runtime Overhead (p95)**    | ≤10ms           | ZMT audit execution time; must not degrade user experience            |
| **Bundle Size Increase**      | ≤25KB gzipped   | Client-side ZMT validator (if used)                                   |
| **CI Build Time Increase**    | ≤2 minutes      | ZMT audit must complete within 2 minutes for standard PRs             |

**Rationale:**  
- ZMT is a quality gate, not a runtime dependency; overhead must be minimal.
- CI pipelines must remain fast to encourage frequent integration.

---

### 3.2 Miss-Tap Rate (MTR)

**Definition:** Percentage of taps that result in incorrect activation (adjacent target, no target, or system navigation error).

**Target Values:**

| Surface                     | MTR Threshold   | Notes                                                                 |
|-----------------------------|-----------------|-----------------------------------------------------------------------|
| **Phone / Tablet**          | ≤2%             | Acceptable error rate for touch interfaces; industry benchmark        |
| **Desktop**                 | ≤1%             | Mouse precision allows lower error rate                               |
| **Watch**                   | ≤3%             | Small screen; slightly higher tolerance acceptable                    |
| **TV**                      | ≤1%             | D-pad precision; errors are frustrating at 10-foot distance           |
| **AR/VR / Glasses**         | ≤2%             | Emerging input methods; tolerance for early-stage tech                |

**Measurement:**  
- Telemetry: Track `touch_start` → `touch_end` → `activation_target` mismatches.
- Controlled Studies: A/B test with known correct answers.

**Enforcement:**  
- PRs must not increase MTR beyond threshold.
- Regressions trigger rollback or ADR-required waiver.

---

### 3.3 Tap Safety Score (TSS)

**Definition:** Composite metric (0-100) measuring compliance with ZMT axioms:

\[
\text{TSS} = 100 - \left( \sum_{i=1}^{n} w_i \times v_i \right)
\]

Where:
- \( v_i \): Violation count for axiom \( i \)
- \( w_i \): Weight (severity) for axiom \( i \)

**Weight Table:**

| Axiom Category                | Weight (per violation) | Examples                                                              |
|-------------------------------|------------------------|-----------------------------------------------------------------------|
| **Touch Target Size**         | 5 points               | Button < 44px on phone                                                |
| **Separation**                | 3 points               | Buttons < 8px apart                                                   |
| **Contrast**                  | 4 points               | Text < 4.5:1 contrast                                                 |
| **Focus Indicator**           | 5 points               | No visible focus on keyboard navigation                               |
| **Reflow**                    | 2 points               | Horizontal scroll required at 320px                                   |
| **Gesture Timing**            | 2 points               | Tap debounce < 120ms                                                  |
| **Accessibility (Other)**     | 4 points               | Color-only errors, missing alt text, etc.                             |

**Target Values:**

| Surface                     | TSS Threshold   | Notes                                                                 |
|-----------------------------|-----------------|-----------------------------------------------------------------------|
| **Phone / Tablet**          | ≥90             | Highest standard for touch interfaces                                 |
| **Desktop**                 | ≥85             | Mouse precision allows some flexibility                               |
| **Watch**                   | ≥85             | Constrained by small screen                                           |
| **TV**                      | ≥85             | 10-foot UI; some flexibility acceptable                               |
| **AR/VR / Glasses**         | ≥80             | Emerging; some flexibility for innovation                             |

**Enforcement:**  
- CI fails PR if TSS drops below threshold.
- Waivers require ADR, compensatory controls, and 30-day expiry.

---

## 4. Measurement and Enforcement

### 4.1 CI Integration (GitHub Actions)

**Workflow:** `.github/workflows/zmt-ci.yml`

**Steps:**
1. **Install:** `npm install -g tapguard-cli` (placeholder; real tool TBD).
2. **Audit:** `tapguard audit --surfaces=all --report=zmt-report.json`.
3. **Gates:**
   - Tap Safety Score ≥ threshold (surface-specific).
   - Miss-Tap Rate ≤ threshold (surface-specific).
   - Zero violations with severity ≥ "Alta" (High).
   - Runtime overhead p95 ≤ 10ms.
4. **Fail-Fast:** Exit 1 if any gate fails; upload `zmt-report.json` as artifact.

**Exception Handling:**  
- PRs can opt-in to "audit mode" (warnings only) via label `ZMT-AUDIT-ONLY`.
- Waivers must include `ZMT-WAIVER` label and link to ADR in PR description.

---

### 4.2 Telemetry (Optional Opt-In)

**Events:**  
- `zmt_audit_run`: CI execution time, violations, TSS, MTR.
- `zmt_violation`: Axiom violated, surface, component, severity.
- `zmt_waiver_created`: ADR ID, surface, duration, reason.

**Privacy:**  
- No user-identifiable data.
- Aggregate metrics only (e.g., "10% of PRs have touch-target violations").
- Opt-in per repository via `.zmtrc` config file.

**Storage:**  
- JSON artifacts uploaded to GitHub Actions.
- Optional export to analytics platform (e.g., Google Analytics, Amplitude) for trend analysis.

---

## 5. Surface-Specific Critical Points

### 5.1 Phone

- **Primary Input:** Touch (finger, thumb).
- **Critical Zones:** Thumb reach zones (bottom 50% of screen; side edges difficult).
- **Violations:** Buttons < 44px, separation < 8px, contrast < 4.5:1, no focus indicator.
- **TSS Threshold:** ≥90.

### 5.2 Tablet

- **Primary Input:** Touch (finger, stylus).
- **Critical Zones:** Edges for one-handed grip; center for two-handed.
- **Violations:** Same as phone; also check for landscape/portrait reflow.
- **TSS Threshold:** ≥90.

### 5.3 Desktop

- **Primary Input:** Mouse, trackpad, keyboard.
- **Critical Zones:** Top menu bar, sidebars, modal dialogs.
- **Violations:** Buttons < 24px, contrast < 4.5:1, no keyboard focus.
- **TSS Threshold:** ≥85.

### 5.4 Watch

- **Primary Input:** Touch (finger), digital crown, side buttons.
- **Critical Zones:** Full screen (small area; all targets critical).
- **Violations:** Buttons < 40px, separation < 6px, text too small.
- **TSS Threshold:** ≥85.

### 5.5 Glasses (Microdisplay)

- **Primary Input:** Gaze + pinch, voice, head gestures.
- **Critical Zones:** Central 30° FOV; peripheral UI difficult to target.
- **Violations:** Targets < 0.5° angular, separation < 0.3° angular, contrast < 3:1.
- **TSS Threshold:** ≥80.

### 5.6 TV (10-Foot UI)

- **Primary Input:** Remote D-pad, voice, game controller.
- **Critical Zones:** Center 60% of screen; edges difficult to reach.
- **Violations:** Buttons < 48px, separation < 12px, no focus indicator.
- **TSS Threshold:** ≥85.

### 5.7 AR/VR (Spatial)

- **Primary Input:** Hand tracking, ray-cast, gaze, controllers.
- **Critical Zones:** Arm's reach (0.5-1.5m); avoid extremes (floor, ceiling, behind).
- **Violations:** Targets < 1.2° angular, separation < 0.5° angular, no depth cues.
- **TSS Threshold:** ≥80.

---

## 6. Exception and Waiver Process

### 6.1 ADR Requirement

All exceptions to ZMT axioms must:
1. Create an Architecture Decision Record (ADR) using template `ADR-ZMT-WAIVER-TEMPLATE.md`.
2. Include:
   - **Context:** Why exception is needed (≤3 sentences).
   - **Rules Excepted:** Specific SoT keys, required vs. proposed values, variance.
   - **Scope:** Routes, cohorts, duration (max 30 days).
   - **Risk:** Estimated impact on MTR, TSS, accessibility, user %.
   - **Compensatory Controls:** Hit-slop, focus indicators, tooltips, monitoring.
   - **Monitoring:** Metrics, thresholds, auto-rollback plan.
   - **Exit Criteria:** Definition of Done (DoD) with deadline.
   - **Approvals:** UX Lead, Accessibility Owner, Eng Lead, PM/PO signatures.
3. Attach ADR to PR with label `ZMT-WAIVER`.
4. Re-review at 30 days; extend or close.

### 6.2 Waiver Expiry

- **Max Duration:** 30 days (renewable once).
- **Auto-Rollback:** If MTR or TSS exceeds threshold during waiver period, rollback immediately.
- **Review Cadence:** Weekly check-in with stakeholders.

### 6.3 Severity Table

| Severity    | Definition                                                                 | Examples                                      | PR Status       |
|-------------|---------------------------------------------------------------------------|-----------------------------------------------|-----------------|
| **Bloqueante** (Blocker) | Axiom violation; no compensatory controls | Button 36px on phone (< 44px), no focus       | ❌ Blocked       |
| **Alta** (High)          | Axiom violation; compensatory controls present | Button 42px on phone, has hit-slop +4px       | ⚠️ Requires ADR |
| **Media** (Medium)       | Non-axiom violation; UX degradation           | Button 46px on phone (meets 44px, < 48px rec) | ⚠️ Warning       |
| **Baja** (Low)           | Minor deviation; no user impact               | Separation 10px instead of 12px on TV         | ℹ️ Info         |

---

## 7. Versioning and Change Control

### 7.1 Source of Truth (SoT)

**File:** `docs/architecture/zmt/sot-zmt-defaults.json`  
**Format:** JSON schema with semantic versioning.  
**Versioning Rules:**
- **Major (x.0.0):** Axiom change (e.g., 44px → 48px minimum).
- **Minor (1.x.0):** New surface, density mode, or enforcement rule.
- **Patch (1.0.x):** Bug fix, clarification, typo.

**Update Process:**
1. Propose change in ADR (separate from waivers).
2. Review with ZMT Core Team (UX, Accessibility, Eng).
3. PR to update `sot-zmt-defaults.json`.
4. Update `version` and `updated` fields.
5. Merge requires 2+ approvals.

### 7.2 Breaking Changes

If an SoT update **increases** strictness (e.g., 44px → 48px), follow this process:
1. **Announce:** 30-day notice to all teams.
2. **Audit:** Run `tapguard audit --surfaces=all --sot-version=next` on all repos.
3. **Fix:** Teams remediate violations before deadline.
4. **Enforce:** Update CI workflow to use new SoT version.
5. **Monitor:** Track MTR, TSS for 14 days post-rollout; rollback if regressions detected.

---

## 8. Normative References

1. **WCAG 2.1:** Web Content Accessibility Guidelines (W3C, Level AA).
2. **WCAG 2.2:** Target Size (Minimum) Success Criterion 2.5.8 (Level AAA, 44x44px).
3. **Apple Human Interface Guidelines (HIG):** Touch targets ≥44pt, spacing ≥8pt.
4. **Material Design 3 (Google):** Touch targets ≥48dp, spacing ≥8dp.
5. **ISO 9241-9:2000:** Ergonomic requirements for office work with VDTs (Fitts' Law).
6. **Shneiderman et al., "Designing the User Interface" (6th ed., 2016):** Target size, spacing, legibility.
7. **WHO International Classification of Functioning, Disability and Health (ICF):** Motor skill variance, tremors.

---

## 9. Glossary

- **Axiom:** Hard limit that cannot be violated without ADR waiver.
- **Hit-Slop:** Invisible padding around touch target that expands tap area.
- **Miss-Tap Rate (MTR):** % of taps activating wrong target or no target.
- **Tap Safety Score (TSS):** 0-100 metric; 100 = perfect compliance.
- **Angular Size:** Visual angle subtended by object (degrees); used for AR/VR/glasses.
- **Thumb Zone:** Area of screen reachable by thumb in one-handed grip (phone/tablet).
- **Reflow:** Content adapts to narrow viewport without horizontal scrolling.
- **Density Mode:** UI compactness level (comfortable, compact, ultra-compact).
- **Compensatory Control:** Alternative mechanism to reduce risk (e.g., hit-slop, tooltips).

---

## 10. Appendix A: Example Violations and Fixes

| Violation                                  | Surface | Fix                                                                 |
|--------------------------------------------|---------|---------------------------------------------------------------------|
| Button 40px on phone                       | Phone   | Increase to 44px or add hit-slop +4px                               |
| Links < 6px apart in menu                  | Desktop | Increase spacing to 8px or add hit-slop                             |
| Icon contrast 2.5:1                        | All     | Increase to 3:1 (darken/lighten icon or background)                 |
| No focus indicator on form field           | Desktop | Add 2px outline with 3:1 contrast                                   |
| Horizontal scroll at 320px width           | Phone   | Use flexbox/grid with wrap; stack elements vertically              |
| Tap debounce 80ms                          | Phone   | Increase to 120ms minimum                                           |
| Double-tap detection 350ms                 | Phone   | Reduce to 300ms max                                                 |
| AR target 0.8° angular                     | AR/VR   | Increase to 1.2° angular or add depth cues (shadow, outline)        |

---

## 11. Appendix B: CI Enforcement Example

```yaml
# .github/workflows/zmt-ci.yml
name: ZMT Audit
on: [pull_request, push]
jobs:
  zmt-audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: 20
      - run: npm install -g tapguard-cli
      - run: tapguard audit --surfaces=all --report=zmt-report.json
      - name: Enforce Gates
        run: |
          TSS=$(jq '.tap_safety_score' zmt-report.json)
          MTR=$(jq '.miss_tap_rate' zmt-report.json)
          if (( $(echo "$TSS < 90" | bc -l) )); then exit 1; fi
          if (( $(echo "$MTR > 0.02" | bc -l) )); then exit 1; fi
      - uses: actions/upload-artifact@v3
        with:
          name: zmt-report
          path: zmt-report.json
```

---

## 12. Document History

| Version | Date       | Author         | Changes                                                                 |
|---------|------------|----------------|-------------------------------------------------------------------------|
| 1.0.0   | 2025-10-03 | ZMT Core Team  | Initial release; all axioms, enforcement, waiver process                |

---

**END OF ASR-ZMT-0001**
