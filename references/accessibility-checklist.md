# Accessibility Checklist

WCAG 2.1 AA quick-reference for `/design`, `/review`, `/qa`, and `/audit`.

## Keyboard Navigation

- [ ] All interactive elements reachable via Tab key
- [ ] Visible focus indicator on all focusable elements
- [ ] Logical tab order matches visual layout
- [ ] No keyboard traps (user can always Tab away)
- [ ] Escape closes modals, dropdowns, and overlays
- [ ] Custom components support expected keyboard patterns (arrow keys for menus, Space/Enter for buttons)

## Screen Reader Support

- [ ] Semantic HTML elements used (nav, main, article, aside, button, not div-for-everything)
- [ ] All images have meaningful alt text (or alt="" for decorative)
- [ ] Form inputs have associated labels (label[for] or aria-labelledby)
- [ ] Error messages associated with form fields (aria-describedby)
- [ ] Dynamic content changes announced (aria-live regions)
- [ ] Page has proper heading hierarchy (h1 → h2 → h3, no skipped levels)

## Visual Design

- [ ] Text contrast ratio ≥ 4.5:1 (WCAG AA)
- [ ] Large text contrast ratio ≥ 3:1
- [ ] UI component contrast ratio ≥ 3:1 against adjacent colors
- [ ] Color is not the only means of conveying information
- [ ] Text resizable to 200% without loss of content or functionality

## Focus Management

- [ ] Focus moves to modal/dialog when opened
- [ ] Focus returns to trigger element when modal/dialog closes
- [ ] Focus trapped inside open modals (Tab doesn't escape to background)
- [ ] Focus managed on route changes in SPAs (announce new page content)

## Responsive & Interaction

- [ ] Content usable at 320px viewport width (no horizontal scroll)
- [ ] Touch targets ≥ 44x44 CSS pixels on mobile
- [ ] No content or functionality requires specific device orientation
- [ ] Test at standard breakpoints: 320px, 768px, 1024px, 1440px

## Testing

- Run automated tools: axe-core, Lighthouse accessibility audit
- Manual keyboard-only navigation test
- Screen reader spot-check (VoiceOver, NVDA, or similar)
- High contrast mode verification
