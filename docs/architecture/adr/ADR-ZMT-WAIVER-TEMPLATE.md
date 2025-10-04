# ADR-ZMT-WAIVER-[YYYY-MM-DD]-[SHORT-DESCRIPTION]

**Type:** ZMT Waiver (Exception to Axioms)  
**Status:** [Draft | Under Review | Approved | Rejected | Expired]  
**Owner:** [Name, Team, Email]  
**Created:** [YYYY-MM-DD]  
**Expires:** [YYYY-MM-DD] (Max 30 days from creation)  
**Surfaces:** [phone | tablet | desktop | watch | glasses_microdisplay | tv | ar_vr]  
**Components:** [List affected routes, components, or features]  
**SoT Version:** [e.g., 1.0.0]  
**PR Links:** [List all related PRs]  
**CI Label:** `ZMT-WAIVER`  

---

## 1. Context and Reason

**Why is this exception needed?** (≤3 lines)

[Provide brief, clear explanation of the technical constraint, design trade-off, or temporary solution requiring this waiver. Examples:
- "OS-level control (native iOS picker) cannot be resized below 40px; platform limitation."
- "Power-user dashboard requires dense data table; 6px separation with hit-slop to maintain scan-ability."
- "Interim fix while new design system rolls out over next 3 sprints."]

**Business Impact:**  
[What happens if we do NOT grant this waiver? E.g., "Feature blocked; release delayed 4 weeks" or "Competitive parity lost; users defect to competitor."]

---

## 2. ZMT Rules to Except

List **specific** axioms being violated. Reference keys from `sot-zmt-defaults.json`.

| SoT Key                        | Required Value      | Proposed Value      | Variance (%) | Norm Violated                | Justification                          |
|--------------------------------|---------------------|---------------------|--------------|------------------------------|----------------------------------------|
| `surfaces.phone.tap_targets.min_size_px` | 44px               | 40px                | -9%          | WCAG 2.2 SC 2.5.8 (AAA)      | Native iOS date picker; OS limitation  |
| `surfaces.phone.separation.comfortable_px` | 8px                | 6px                 | -25%         | iOS HIG spacing guidelines   | Dense toolbar; hit-slop +2px applied   |
| `contrast_requirements.text_small_min` | 4.5:1              | 4.0:1               | -11%         | WCAG 2.1 SC 1.4.3 (AA)       | Brand color #6A7B8C; adding stroke     |

**Total Axioms Violated:** [Number]  
**Surfaces Affected:** [phone, tablet, etc.]

---

## 3. Scope

### Routes/Components
[List specific URLs, components, or screens where exception applies. Be narrow and precise.]

- `/settings/notifications` (toggle switches only)
- `<ButtonGroup>` component in `src/components/ButtonGroup.tsx` (lines 45-67)
- Admin dashboard data table (`/admin/users`)

**NOT in Scope:**  
[List what is explicitly excluded to prevent scope creep.]

- Marketing landing pages
- Checkout flow
- Mobile web (separate waiver required)

---

### User Cohorts
[Who is affected? Use percentages or absolute numbers.]

- **Total Users Affected:** ~5,000 users (~2% of active user base)
- **Segment:** Power users (analysts, admins) who opted into "compact view"
- **Regions:** Global (no geo-restriction)
- **Devices:** Phone and tablet only; desktop unaffected

---

### Duration
**Start Date:** [YYYY-MM-DD]  
**End Date:** [YYYY-MM-DD] (Max 30 days; renewable once)  
**Renewal Plan:** [If renewal needed, explain why and provide new expiry date]

---

### Enforcement Mode During Waiver
- [ ] **Audit Mode:** Report violations as warnings; do not fail CI
- [ ] **Progressive Mode:** Fail only on Blocker violations; warn on Alta
- [x] **Strict Mode:** Fail on Blocker and Alta (default; waiver grants exception for this PR only)

---

## 4. Risk and Impact Assessment

### Miss-Tap Rate (MTR)
**Current MTR (Component):** [e.g., 0.015 or "not yet measured"]  
**Projected MTR (With Waiver):** [e.g., 0.022 or "increase by ~1.5%"]  
**Threshold:** [e.g., 0.02 for phone]  
**Acceptable?** [Yes/No + explanation]

### Tap Safety Score (TSS)
**Current TSS (Surface):** [e.g., 92 for phone]  
**Projected TSS (With Waiver):** [e.g., 87 (drop of 5 points due to 3 Alta violations)]  
**Threshold:** [e.g., 90 for phone]  
**Acceptable?** [Yes/No + explanation]

### Accessibility Impact
**Affected User Groups:**  
- [ ] Low vision (contrast, size)
- [ ] Motor impairment (target size, separation)
- [ ] Cognitive impairment (complexity, timing)
- [ ] Deaf/Hard of Hearing (N/A for ZMT; included for completeness)

**Severity:** [Low | Medium | High | Critical]  
**Mitigation:** [Describe compensatory controls below]

### User Impact (Estimated)
**Users Affected:** [Number or percentage]  
**Severity:** [Blocker | Usability Issue | Inconvenience | None]  
**Support Tickets Expected:** [e.g., <5/month based on similar changes]

---

## 5. Compensatory Controls

Check **ALL** that apply and provide implementation details.

- [ ] **1. Hit-Slop (Invisible Padding)**  
  **Implementation:** Add `::after` pseudo-element with `inset: -Xpx` to expand touch area.  
  **Code Snippet:**  
  ```css
  .button-compact::after {
    content: '';
    position: absolute;
    inset: -4px; /* Expands 40px button to 48px touch area */
  }
  ```

- [ ] **2. Focus Indicator (≥3:1 Contrast)**  
  **Implementation:** Add 2px outline on `:focus` or `:focus-visible`.  
  **Code Snippet:**  
  ```css
  .button:focus-visible {
    outline: 2px solid #0066cc;
    outline-offset: 2px;
  }
  ```

- [ ] **3. Reach-Zone Consideration**  
  **Implementation:** Place critical actions in bottom 50% of phone screen (thumb reach).  
  **Rationale:** Reduces stretch; lowers miss-tap probability.

- [ ] **4. Tooltips / ARIA Labels**  
  **Implementation:** Add `aria-label` and tooltip on hover/long-press.  
  **Code Snippet:**  
  ```html
  <button aria-label="Save changes (Cmd+S)" title="Save changes">
    <IconSave />
  </button>
  ```

- [ ] **5. Typography Enhancement**  
  **Implementation:** Increase font size, weight, or add stroke to improve contrast.  
  **Code Snippet:**  
  ```css
  .text-low-contrast {
    font-size: 16px; /* Was 14px */
    font-weight: 600; /* Was 400 */
    text-shadow: 0 0 1px rgba(0,0,0,0.3); /* Improves perceived contrast */
  }
  ```

- [ ] **6. Reduce Motion / Animation Off**  
  **Implementation:** Respect `prefers-reduced-motion` media query.  
  **Code Snippet:**  
  ```css
  @media (prefers-reduced-motion: reduce) {
    .button { transition: none; }
  }
  ```

- [ ] **7. Monitoring and Auto-Rollback**  
  **Implementation:** Track MTR via telemetry; rollback if exceeds threshold.  
  **Details:** See Section 6 below.

**Notes:**  
[Explain why these controls are sufficient to mitigate risk. E.g., "Hit-slop compensates for visual size reduction; focus indicator ensures keyboard accessibility."]

---

## 6. Monitoring and Auto-Rollback

### Metrics to Track
| Metric                  | Baseline (Before) | Threshold (Max)   | Measurement Frequency | Owner         | Alert Channel     |
|-------------------------|-------------------|-------------------|-----------------------|---------------|-------------------|
| **Miss-Tap Rate (MTR)** | 0.015             | 0.025 (+67%)      | Daily                 | @eng-lead     | #alerts-zmt       |
| **Tap Safety Score**    | 92                | 85 (drop ≤7 pts)  | Per PR                | @qa-lead      | CI report         |
| **Support Tickets**     | 2/month           | 10/month          | Weekly                | @support-lead | #support-escalate |
| **User Complaints**     | 0.5%              | 2%                | Daily                 | @product-lead | #product-feedback |

### Auto-Rollback Conditions
**Trigger:** Any metric exceeds threshold for **2 consecutive measurements** (fail-safe against false positives).

**Rollback Steps:**
1. **Feature Toggle Off:** Disable `ENABLE_COMPACT_VIEW` flag.
2. **Notify Stakeholders:** Post to #zmt-waiver-incidents within 15 minutes.
3. **Triage:** Engineering lead investigates root cause within 4 hours.
4. **Fix or Revert:** Either fix compensatory controls or revert PR entirely.

**Responsible Party:** @eng-lead (primary), @on-call-sre (backup)

**Testing:** Rollback plan tested in staging on [YYYY-MM-DD]. [Link to test report]

---

## 7. Exit Criteria (Definition of Done)

Waiver expires when **all 3 tasks** completed or deadline reached (whichever first).

| Task                                                                 | Owner         | Deadline   | Status        |
|----------------------------------------------------------------------|---------------|------------|---------------|
| **1. Implement permanent fix** (e.g., new design system with 44px buttons) | @design-lead  | 2025-11-01 | [ ] Not Started |
| **2. Migrate all instances** to compliant implementation             | @eng-lead     | 2025-11-10 | [ ] Not Started |
| **3. Remove feature toggle** and waiver-specific code                | @eng-lead     | 2025-11-15 | [ ] Not Started |

**DoD Verification:**  
- [ ] All affected components pass ZMT audit with 0 Blocker, 0 Alta violations.
- [ ] TSS ≥ threshold for all surfaces.
- [ ] MTR ≤ baseline.
- [ ] Waiver ADR marked as "Closed" with link to final PR.

**If Deadline Missed:**  
[Describe what happens. E.g., "Auto-rollback triggered; feature disabled until DoD met."]

---

## 8. Rollback Plan

### Pre-Rollback Checklist
- [ ] Feature toggle exists and tested in staging.
- [ ] Database migrations are reversible.
- [ ] CDN cache purge ready (if UI assets cached).
- [ ] User communication plan (email, in-app banner, etc.).

### Rollback Steps
1. **Disable Feature Toggle:** Set `ENABLE_COMPACT_VIEW=false` in config.
2. **Deploy Rollback PR:** Revert commits [list SHAs].
3. **Clear CDN Cache:** Purge `/assets/button-group.*` (if applicable).
4. **Verify:** Check MTR and TSS return to baseline within 1 hour.
5. **Notify Users:** Email affected cohort (power users) with explanation.

**Rollback Time (RTO):** 15 minutes (toggle flip)  
**Data Loss:** None (toggle preserves user preferences)

### Post-Rollback Impact
**User Experience:**  
- Power users lose "compact view" option; revert to default (comfortable mode).
- No data loss; preferences saved for future re-enable.

**Business Impact:**  
- Potential churn from power users (~5% of affected cohort = ~250 users).
- Support tickets spike expected (~20-30 in first 48 hours).

---

## 9. Evidence and Attachments

### ZMT Audit Report
**File:** [Link to `zmt-report.json` in PR or attach here]

**Summary:**
```json
{
  "surfaces": {
    "phone": {
      "tap_safety_score": 87,
      "miss_tap_rate": 0.022,
      "violations": {
        "blocker": 0,
        "alta": 3,
        "media": 5,
        "baja": 2
      }
    }
  }
}
```

**Interpretation:**  
- TSS 87 vs. threshold 90 (3-point drop acceptable with compensatory controls).
- MTR 0.022 vs. threshold 0.02 (2.2% vs. 2%; within margin of error).
- 3 Alta violations covered by hit-slop and focus indicators.

---

### Screenshots

**Before (Non-Compliant):**  
![Button Group Before](link-or-attach-image-here.png)  
- Button size: 40x40px (should be 44x44px)
- Separation: 6px (should be 8px)

**After (With Waiver + Compensatory Controls):**  
![Button Group After](link-or-attach-image-here.png)  
- Button size: 40x40px (visual), 48x48px (hit-slop)
- Separation: 6px (visual), 10px (effective with hit-slop)
- Focus indicator: 2px blue outline (4.8:1 contrast)

**Annotations:**  
[Use red boxes, arrows, or rulers to highlight target sizes and separations.]

---

### User Research / A/B Test Results
**Study:** [Link to report or attach PDF]

**Summary:**
- **Sample Size:** 1,000 users (500 control, 500 waiver variant)
- **Duration:** 7 days
- **MTR (Control):** 0.015
- **MTR (Waiver):** 0.021 (+40% relative increase, but below 0.025 threshold)
- **User Satisfaction:** 4.2/5 (control) vs. 4.1/5 (waiver); not statistically significant (p=0.14)

**Conclusion:** Waiver acceptable; no significant UX degradation detected.

---

### Additional Links
- **Figma Mockups:** [Link]
- **Prototype:** [Link to interactive demo]
- **Related ADRs:** [Links to other ADRs if applicable]
- **WCAG Exceptions Precedent:** [If similar exceptions exist in industry, cite here]

---

## 10. Required Approvals

All **4 approvers** must sign. Majority approval not sufficient.

| Role                     | Name              | Email                  | Approved? | Date       | Signature / Comment                          |
|--------------------------|-------------------|------------------------|-----------|------------|----------------------------------------------|
| **UX Lead**              | [Name]            | [email@example.com]    | [ ]       | [YYYY-MM-DD] | [Sign here or add comment]                 |
| **Accessibility Owner**  | [Name]            | [email@example.com]    | [ ]       | [YYYY-MM-DD] | [Sign here or add comment]                 |
| **Engineering Lead**     | [Name]            | [email@example.com]    | [ ]       | [YYYY-MM-DD] | [Sign here or add comment]                 |
| **PM / Product Owner**   | [Name]            | [email@example.com]    | [ ]       | [YYYY-MM-DD] | [Sign here or add comment]                 |

**Approval Criteria:**
- All 10 sections completed.
- Compensatory controls sufficient for risk level.
- Monitoring plan credible; auto-rollback tested.
- Exit criteria clear and achievable.
- Business case compelling (not just "nice to have").

**Rejection Allowed:** Any approver can reject with written reason. PR blocked until concerns addressed.

---

## Change Log

| Version | Date       | Author    | Changes                                   |
|---------|------------|-----------|-------------------------------------------|
| 1.0     | [YYYY-MM-DD] | [Name]  | Initial waiver request                    |
| 1.1     | [YYYY-MM-DD] | [Name]  | Added hit-slop implementation details     |
| 1.2     | [YYYY-MM-DD] | [Name]  | Updated MTR threshold after A/B test      |

---

## Post-Expiry Review

**Completed:** [YYYY-MM-DD]  
**Outcome:** [Extended | Closed | Reverted]  
**Final MTR:** [Value]  
**Final TSS:** [Value]  
**Lessons Learned:** [2-3 bullet points on what went well, what didn't, how to improve process]

---

**Template Version:** 1.0.0  
**Last Updated:** 2025-10-03  
**Maintained By:** ZMT Core Team  

---

## How to Use This Template

1. **Copy File:** `cp ADR-ZMT-WAIVER-TEMPLATE.md ADR-ZMT-WAIVER-2025-10-15-COMPACT-BUTTONS.md`
2. **Fill All Sections:** Replace `[placeholders]` with actual values. Do not leave blanks.
3. **Commit to Repo:** `git add docs/architecture/adr/ADR-ZMT-WAIVER-*.md`
4. **Link in PR:** Add link to PR description and apply `ZMT-WAIVER` label.
5. **Request Reviews:** Tag all 4 approvers in PR comments.
6. **Iterate:** Address feedback; update ADR version in Change Log.
7. **Merge:** Once all approvals obtained, merge PR (CI may warn but won't fail with waiver label).
8. **Monitor:** Check metrics per Section 6; rollback if thresholds exceeded.
9. **Close on Expiry:** Mark ADR as "Expired" or "Closed" when DoD met or 30 days elapsed.

**Questions?** Slack: `#zmt-support` | Email: `zmt-core-team@example.com`

---

**END OF ADR-ZMT-WAIVER-TEMPLATE**
