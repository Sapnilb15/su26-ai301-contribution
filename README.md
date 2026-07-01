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

Files Modified
packages/web/src/views/Forms/EventForm/RecurringEventUpdateScopeDialog.tsx — main fix
packages/web/src/views/Forms/EventForm/RecurringEventUpdateScopeDialog.test.tsx — new test file (16 tests)
Root package.json + bun.lock — added @sinonjs/fake-timers, a missing dev dependency causing the entire web test suite to fail on import
What I Built
Replaced OverlayPanel with floating-ui's FloatingFocusManager + FloatingOverlay + FloatingPortal per the issue's guidance. This gives built-in focus trapping, return-focus-on-close, Escape dismiss via useDismiss, and correct role="dialog" + aria-modal via useRole.
Changed radio input class from absolute opacity-0 w-0 h-0 → sr-only (Tailwind's 1px clip technique). Inputs remain visually hidden but are now properly focusable and in the tab order.
The pre-existing peer-focus-visible:ring-2 peer-focus-visible:ring-accent-primary styles on the radio dot already existed and activate correctly once the radio can receive focus — no new CSS needed.
Challenges Faced
@sinonjs/fake-timers was required by the test preload script but not listed as a dependency — every test file failed with "Cannot find module." Discovered by checking the preload file. Fixed with bun add --dev @sinonjs/fake-timers at root.
CSS.escape is not implemented in the project's jsdom version. @testing-library/user-event calls it when walking radio groups by name attribute for Arrow key simulation. Added a beforeAll polyfill using a simple regex escape.
Biome linter flagged aria-labelledby on the dialog div because it can't statically see that getFloatingProps() injects role="dialog" at runtime — resolved with a targeted biome-ignore comment matching the same pattern used in OverlayPanel.tsx.
Code Changes
Branch: fix-issue-1326 on github.com/Sapnilb15/compass-calendar

Commits:

4d044fa — fix(web): enable keyboard navigation in recurring event scope dialog
167ebe4 — fix(dev): add @sinonjs/fake-timers to unblock web test suite
Testing Strategy
Tests added: RecurringEventUpdateScopeDialog.test.tsx — 16 tests using @testing-library/react + @testing-library/user-event + bun:test, modeled on LogoutConfirmationDialog.test.tsx.

Coverage:

Rendering: role="dialog", title, all three scope options, default selection, Cancel/Ok buttons
Keyboard a11y (regression guard): radio inputs are focusable, ArrowDown/ArrowUp navigate and select, Space selects, Enter submits, Escape closes, Tab moves focus radio group → Cancel → Ok
Mouse: click selection, Cancel closes, Ok calls onUpdateScopeChange with correct scope
Validation:

16/16 tests pass: bun test src/views/Forms/EventForm/RecurringEventUpdateScopeDialog.test.tsx
Existing tests unaffected
Biome lint: 0 errors on changed files
Pull Request
PR Link: https://github.com/SwitchbackTech/compass-calendar/pull/1900

PR Description: Fixes keyboard navigation in the recurring event update scope dialog. Changed radio inputs from w-0 h-0 to sr-only so they are focusable, and migrated the modal to floating-ui's FloatingFocusManager per the issue guidance. Full keyboard flow (Tab, Arrow keys, Space, Enter, Escape) now works. 16 tests added.

Status: Awaiting review

Note on project policy: After submitting, I discovered this project requires a formal contributor application before PRs are reviewed (contributors must commit 4 hrs/week for 6 months and apply via DM to the maintainer). The PR may be closed due to this process requirement. For the purposes of this program — where acceptance is not required for completion — the PR represents a professional, review-ready contribution demonstrating the full open source contribution cycle.

Learnings & Reflections
Read CONTRIBUTING.md before picking an issue, not after submitting. This project has a formal application process that I missed. Future me: check contributor requirements in Phase I, not Phase IV.
Zero-size != invisible for accessibility. opacity-0 hides visually but keeps elements focusable. w-0 h-0 removes focusability entirely. The right technique for visually-hidden-but-accessible is sr-only.
Existing tests are the best documentation. LogoutConfirmationDialog.test.tsx showed me exactly how to structure the test file — the pattern, the mocking approach, the userEvent setup — in one read.
Library-managed a11y beats manual a11y. floating-ui's FloatingFocusManager handled focus trapping, return focus, and Escape key in ~10 lines. The hand-rolled OverlayPanel did the same in ~30 lines with more edge cases to maintain.
