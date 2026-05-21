# Skill: UI/UX Polisher

## Purpose
Elevate the visual quality, usability, and accessibility of any frontend (React, Next.js, plain HTML) without changing business logic or restructuring architecture.

## When to Use
- App works but looks unpolished or inconsistent
- Pre-demo or pre-launch UI refinement
- Accessibility audit before public release
- Responsive design fixes for mobile/tablet
- Design token / spacing inconsistency cleanup

---

## Rules
- Only modify styling, layout, copy, and interaction feedback — not business logic.
- Do not restructure component hierarchy unless strictly required for a layout fix.
- Preserve all existing class names used in tests or by external consumers.
- Every change must improve the experience — do not change for change's sake.
- Flag accessibility issues even if not asked; they are not optional.

---

## Step-by-Step Workflow

### Step 1 — Visual Audit
```
1. Review every page/screen systematically: typography, spacing, color.
2. Check font sizes: body ≥ 16px, headings have clear hierarchy.
3. Check spacing: consistent 4px/8px grid, no magic numbers.
4. Check color contrast: text on background ≥ 4.5:1 (AA standard).
5. Check that interactive elements have visible hover/focus states.
```

### Step 2 — Consistency Audit
```
1. List all button variants in use — are they from the same design system?
2. Check form fields: same border radius, padding, error state style everywhere.
3. Verify icon sizes are consistent (16px / 20px / 24px).
4. Check loading states: spinner, skeleton, or shimmer used consistently.
5. Verify empty states (no data) are handled with meaningful UI.
```

### Step 3 — Responsive Check
```
1. Check layouts at: 375px (mobile), 768px (tablet), 1280px (desktop).
2. Identify text that overflows or truncates badly on small screens.
3. Check touch target sizes ≥ 44×44px on mobile.
4. Verify modals and drawers work on small screens.
5. Check tables — do they scroll horizontally or collapse on mobile?
```

### Step 4 — Accessibility (a11y)
```
1. All images have meaningful alt text (decorative images: alt="").
2. Form inputs have associated <label> elements.
3. Interactive elements are keyboard-navigable (Tab, Enter, Space, Escape).
4. Focus trap implemented in modals and drawers.
5. Color is not the only indicator of state (error, success, disabled).
6. ARIA roles used only where native HTML semantics are insufficient.
```

### Step 5 — Interaction Polish
```
1. Add loading indicators on async operations >300ms.
2. Confirm success/error toasts are present on form submissions.
3. Add subtle transitions (150–250ms) on hover, modal open/close.
4. Check that error messages are human-readable, not raw API errors.
5. Confirm destructive actions have a confirmation step.
```

---

## Commands to Run

```bash
# Accessibility audit
npx axe-cli http://localhost:3000 --tags wcag2a,wcag2aa

# Lighthouse (run from Chrome DevTools or CLI)
npx lighthouse http://localhost:3000 --only-categories=accessibility,best-practices

# Check for missing alt text
grep -rn "<img" src/ | grep -v "alt="

# Tailwind unused classes (if using Tailwind)
npx tailwindcss --content './src/**/*.{js,ts,jsx,tsx}' --output /dev/null
```

---

## Validation Checklist
- [ ] No text below 14px in the UI
- [ ] All interactive states visible (hover, focus, disabled, loading)
- [ ] Color contrast passes AA standard
- [ ] Forms have visible labels, not just placeholder text
- [ ] Mobile layout looks good at 375px
- [ ] Keyboard navigation works end-to-end
- [ ] Loading and error states are handled on all async flows
- [ ] No layout overflow or horizontal scroll on mobile
- [ ] Consistent spacing (4px/8px base grid)
- [ ] Empty states have meaningful copy and a call-to-action

---

## Final Response Format

```
## UI/UX Polish Summary

**Pages Reviewed:** [list]

### Fixed
- [component/page] — [what was changed and why]

### Accessibility Issues Resolved
- [issue] — [file:line] — [fix applied]

### Remaining Recommendations
- [recommendation] — [priority: High/Medium/Low]

**Screenshots / Before-After Notes:**
[describe notable visual changes if screenshots unavailable]
```
