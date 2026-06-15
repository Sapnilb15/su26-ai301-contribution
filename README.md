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


##Understand: The recurrence form has two keyboard accessibility bugs:
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

Review: Check CONTRIBUTING.md for PR conventions

Evaluate: Tab through all fields, Space/Enter activates each one. 
Run axe DevTools Chrome extension to audit a11y compliance.
