# Contribution 1: Cannot select recurrence option with keyboard

**Contribution Number:** 1
**Student:** Sapnil Basnet
**Issue:** https://github.com/SwitchbackTech/compass/issues/1326
**Status:** Phase I Complete

---

## Why I Chose This Issue

The issue is that the recurrence option dropdown in Compass Calendar cannot be 
selected using keyboard navigation — users who rely on keyboards instead of a 
mouse are blocked from setting recurring events. This matters because keyboard 
accessibility is a fundamental usability requirement, and Compass specifically 
markets itself as a "keyboard-first" app, making this bug a direct contradiction 
of the project's core identity.

I chose this issue because the scope is tightly contained to a single UI 
interaction with a clear definition of done (keyboard navigation works), and the 
React/TypeScript stack matches my experience. The project has excellent setup 
documentation and an active contributor community, which will help me move 
quickly in Phase II.


## Understand: The recurrence form has two keyboard accessibility bugs:
(1) WeekDay toggle buttons (S/M/T/W/T/F/S) can't be activated via 
Space/Enter, (2) the "Every" interval CaretInput ignores keyboard input.

Match: FreqSelect.tsx works with keyboard — compare its implementation 
to WeekDay.tsx and CaretInput.tsx to find what keyboard handlers are missing.

Plan:
1. Open WeekDay.tsx — add onKeyDown handler for Space/Enter to toggle 
   the day, add tabIndex={0} and role="checkbox" or role="button"
2. Open CaretInput.tsx — add onKeyDown for arrow keys (up/down) to 
   increment/decrement, add proper tabIndex
3. Open RecurringEventUpdateScopeDialog.tsx — add Tab/Enter/Space/Escape 
   keyboard navigation using floating-ui a11y utilities (per maintainer hint)
4. Add visible focus styles using existing Tailwind focus classes
5. Write tests in RecurrenceSection.test.tsx

Implement: [link to fix-issue-1326 branch]

Implementation Notes
Implementation Summary
Issue: #1326 — Cannot select recurrence option with keyboard

Root cause: Radio inputs in RecurringEventUpdateScopeDialog.tsx used Tailwind classes w-0 h-0 to visually hide them. Zero-dimension elements are non-focusable in most browsers, so Tab, Arrow keys, Space, and Enter all silently failed — the inputs existed in the DOM but couldn't receive keyboard events.

Files modified:

packages/web/src/views/Forms/EventForm/RecurringEventUpdateScopeDialog.tsx — main fix
packages/web/src/views/Forms/EventForm/RecurringEventUpdateScopeDialog.test.tsx — new test file (16 tests)
Root package.json + bun.lock — added @sinonjs/fake-timers which was a missing dep causing the entire web test suite to fail
What I built:

Replaced OverlayPanel with floating-ui's FloatingFocusManager + FloatingOverlay + FloatingPortal per the issue's guidance ("use floating-ui for the modal pattern"). This gives built-in focus trapping, return-focus-on-close, Escape key dismiss via useDismiss, and correct role="dialog" + aria-modal via useRole.
Changed radio input class from absolute opacity-0 w-0 h-0 → sr-only (Tailwind's 1px clip technique). The inputs remain visually hidden but are now properly focusable and in the tab order.
The pre-existing peer-focus-visible:ring-2 peer-focus-visible:ring-accent-primary styles on the radio dot visual already existed and activate correctly once the radio can receive focus — no new CSS needed.
Challenges faced:

@sinonjs/fake-timers was required by the test preload script but not listed as a dependency — every test file in the project failed with "Cannot find module." Discovered this by checking the preload file and searching .bun/ cache. Fixed by running bun add --dev @sinonjs/fake-timers at the root.
CSS.escape is not implemented in the project's jsdom version. @testing-library/user-event calls it when walking radio groups by name attribute for Arrow key simulation. Added a beforeAll polyfill in the test file using a simple regex escape.
The biome linter flagged aria-labelledby on the dialog div because it can't statically see that getFloatingProps() injects role="dialog" at runtime — resolved with a targeted biome-ignore comment matching the same pattern used elsewhere in OverlayPanel.tsx.
Code Changes
Branch: fix-issue-1326

Commits:

4d044fa — fix(web): enable keyboard navigation in recurring event scope dialog
167ebe4 — fix(dev): add @sinonjs/fake-timers to unblock web test suite
Testing Strategy
Tests added: RecurringEventUpdateScopeDialog.test.tsx — 16 tests using @testing-library/react + @testing-library/user-event + bun:test, matching the project's existing test patterns (see LogoutConfirmationDialog.test.tsx).

What the tests cover:

Rendering: dialog role="dialog", title, all three scope options present, default selection ("This Event"), Cancel/Ok buttons
Keyboard a11y (the regression guard): radio inputs are focusable, ArrowDown/ArrowUp navigate between options, Space selects, Enter submits with the active scope, Escape closes the dialog, Tab moves focus radio group → Cancel → Ok
Mouse interaction: click selection, Cancel closes dialog, Ok calls onUpdateScopeChange with the correct scope
Validation:

All 16 new tests pass: bun test src/views/Forms/EventForm/RecurringEventUpdateScopeDialog.test.tsx (run from packages/web/)
Existing LogoutConfirmationDialog and other web tests continue to pass
Biome linter: bunx @biomejs/biome check — 0 errors on both changed files

Review: Check CONTRIBUTING.md for PR conventions

Evaluate: Tab through all fields, Space/Enter activates each one. 
Run axe DevTools Chrome extension to audit a11y compliance.
