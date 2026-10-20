---
name: android-lessons-learned
description: Use when building, reviewing or testing an Android app milestone, to check the work against problems found in earlier apps (text input, error states, data integrity, Firestore rules, layout, Icelandic text, release hygiene).
---

# Android lessons learned

Problems found and fixed in earlier apps (first Nextpost), written as
general checks. Use the sections that apply to the milestone.

- Developer: check your work against the relevant sections before opening
  the pull request.
- Reviewer: report a finding when a change breaks one of these checks.
- QA: turn the relevant checks into tests where possible, and put the rest
  on the owner's manual checklist.

Each check names its origin, so it can be traced back to the real problem.

## Rules and Guidelines

Every check is marked as one of two kinds:

- **[Rule]**: a lesson from a real bug in correctness, security, data or
  text. Rules are binding. If an approved app document (requirements,
  architecture or design) conflicts with a Rule, stop and raise it with the
  architect and the owner before building. Never pick a side yourself.
- **[Guideline]**: a good default for layout and interaction. The approved
  design document wins over a Guideline. Follow the design, and note the
  difference in the pull request so the owner can confirm it on the phone.

In both cases the owner decides, and the decision is recorded with its date
in the change log of the document concerned.

## 1. Text input and keyboards

- [ ] [Rule] Do not move focus from code in a way that can drop the first
  typed character. Test text entry on a physical Samsung phone with Samsung
  Keyboard, and with Gboard. The emulator alone did not show the problem.
  (Nextpost 1)
- [ ] [Rule] The on-screen keyboard never covers the focused field or the
  primary action: use IME padding and top-aligned forms. (Nextpost 33)
- [ ] [Rule] Validate trimmed input: a value of spaces only counts as empty.
  (Nextpost 6)
- [ ] [Rule] Every text input is tested with empty, spaces only, maximum
  length, different letter case, and pasted text with leading or trailing
  spaces. (Nextpost 9)
- [ ] [Rule] One validation function is used everywhere the same rule
  applies, so two screens can never disagree. (Nextpost 7)
- [ ] [Guideline] An untouched field shows no error until the user has
  interacted with it or tries to continue. (Nextpost 8)
- [ ] [Rule] A value prefilled from a previous session can still be edited.
  (Nextpost 10)

## 2. Loading, empty and error states

- [ ] [Rule] A failed load never looks like "nothing here yet". Show an
  error with Retry, and keep existing content visible when a refresh fails.
  (Nextpost 17)
- [ ] [Rule] A failed load never falls through to a "create new" flow. That
  is how orphaned duplicate records get created. (Nextpost 18)
- [ ] [Rule] Record something as loaded only after the load succeeds, so
  Retry actually retries. (Nextpost 19)
- [ ] [Rule] A missing record shows "not found". It is never turned into a
  blank new record or an active session. (Nextpost 21)
- [ ] [Rule] "Permission denied" and "can't reach the server" have different
  messages, on every read and write path. (Nextpost 20)
- [ ] [Rule] A brief loss of signal is handled silently from the offline
  cache. Only a definite failure shows an error. (Nextpost 22)
- [ ] [Rule] Temporary UI state, such as a selection or highlight, is
  cleared by any unrelated action. (Nextpost 12)
- [ ] [Rule] If the app is killed mid-task, reopening it resumes at the same
  place with the same data. A finished task reopens on its completion
  screen. (Nextpost 25)

## 3. Data integrity

- [ ] [Rule] Firestore does not delete child documents. Deleting a parent
  removes every document that belongs to it, and the test checks the
  console or the emulator for leftovers. (Nextpost 23)
- [ ] [Rule] Changes that span several documents happen in one batch or
  transaction: all or nothing. (Nextpost 24, 26)
- [ ] [Rule] Restarting or redoing something never silently overwrites
  history. Archive the old record first, in the same batch. (Nextpost 24)
- [ ] [Rule] Generated codes and IDs handle collisions, and a dropped
  connection halfway through creation leaves nothing half-made.
  (Nextpost 26)
- [ ] [Rule] Remove fields that nothing writes or reads. (Nextpost 27)
- [ ] [Rule] Display names are not unique. Never use a name as an ID or in a
  document path. (Nextpost 32)

## 4. Security rules

- [ ] [Rule] No rule of the form "any signed-in user may read X" for user
  data. Access requires membership (household, session, ownership).
  (Nextpost 28)
- [ ] [Rule] Rules tests include a second signed-in account that is not a
  member, and prove it can read and write nothing. (Nextpost 28)
- [ ] [Rule] Anything the app hides until later (partial disclosure) is
  enforced by the rules, not only by the UI. (Nextpost 29)
- [ ] [Rule] Ownership fields (creator, owner, member user IDs) and
  finalised records cannot be changed through the rules. (Nextpost 30)
- [ ] [Rule] If the app uses anonymous sign-in, warn the user that clearing
  app data loses access to their data. (Nextpost 31)

## 5. Calculations (scores, money, quantities)

- [ ] [Rule] Every worked example in the requirements is a unit test.
  (Nextpost 5)
- [ ] [Rule] Property tests cover the rules a formula must always keep, for
  example "always decreases", "never below the floor" or "parts add up to
  the total". (Nextpost 5)
- [ ] [Rule] Caps and floors are tested at their exact edges. (Nextpost 5)

## 6. Location, sensors and maps (when used)

- [ ] [Rule] A GPS accuracy gate scales with the trigger radius (about 1.5
  times), and a poor fix inside the circle does not count. (Nextpost 3)
- [ ] [Rule] Manual overrides go through the same gate, and their failure
  message shows the measured distance and accuracy. (Nextpost 4)
- [ ] [Rule] Distances and thresholds are tuned outdoors on a real phone
  before they are final. (Nextpost 2)
- [ ] [Rule] Map markers sit exactly on their coordinate (check the anchor),
  and radii are drawn to scale. (Nextpost 11, 16)
- [ ] [Rule] The camera never jumps unexpectedly: placed items don't move
  when the map pans, and repositioning returns to the item first.
  (Nextpost 13–15)

## 7. Layout and interaction

- [ ] [Guideline] Primary actions sit in the bottom third of the screen.
  (Nextpost 34)
- [ ] [Guideline] Touch targets are at least 48 dp, and 56 dp for primary
  actions. (Nextpost 34)
- [ ] [Guideline] Delete and Cancel are not placed next to Save on a screen.
  Standard Material dialogs, which put dismiss next to confirm, are the
  expected exception. (Nextpost 35)
- [ ] [Rule] Leaving with unsaved changes asks for confirmation.
  (Nextpost 35)
- [ ] [Rule] No screen is a dead end: there is always a visible way to
  finish or go back. (Nextpost 36)
- [ ] [Guideline] Button labels follow the state ("Set location" becomes
  "Reposition"). (Nextpost 37)
- [ ] [Guideline] An action with a cost says the cost before it runs ("This
  post drops to 50 points"). (Nextpost 38)

## 8. Icelandic and other languages

- [ ] [Rule] Every Icelandic string is reviewed for grammar and gender
  agreement, with one consistent word for each core term. (Nextpost 39)
- [ ] [Rule] No circular or self-referring messages. Each message says what
  happened and what to do. (Nextpost 40)
- [ ] [Rule] Þ, Ð, Æ, Ö and the accented vowels render correctly in the
  app's actual font, in every weight used. (Nextpost 41)

## 9. Release hygiene

- [ ] [Rule] Check `git status` before every commit. No stray debug or repro
  files are committed. (Nextpost 42)
- [ ] [Rule] Secrets stay untracked, with a `.template` file showing the
  format. A release build works from a fresh clone. (Nextpost 43)
- [ ] [Rule] Every claim in the README is checked against the repository.
  (Nextpost 44)
- [ ] [Rule] Logic lives in the layer the architecture says, not in
  ViewModels or mappers. (Nextpost 45)
- [ ] [Guideline] A cloud API key without billing may be throttled and look
  like a broken key. Check quota and billing before debugging the code.
  (Nextpost 46)

## Adding a lesson

When a new app finds a problem that could happen again, add one check here
in the matching section, written generally, marked [Rule] or [Guideline],
with its origin, for example "(Tillslip M2)". Commit it to the agents
repository.
