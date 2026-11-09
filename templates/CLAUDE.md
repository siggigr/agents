# <App name>: notes for Claude

Claude Code reads this file at the start of every session in this
repository. It holds the working rules every role follows here. Role
instructions come from the agent files; the approved documents are in
`docs/`.

## Sources of truth

- `docs/requirements.md`, `docs/architecture.md` and `docs/design.md` are
  the approved documents. Only the role that owns a document changes it
  (analyst, architect, designer), and only with the owner's approval.
- `docs/status.md` is the current state. Read it first in every session.
- `docs/privacy-register.md`, if present, lists the personal data the app
  handles.

## When sources disagree

The android-lessons-learned skill marks each check as a Rule or a
Guideline.

- A Rule (correctness, security, data, text) is binding. If an approved
  document conflicts with a Rule, stop and raise it with the architect and
  the owner before building.
- A Guideline (layout and interaction) gives way to the approved design.
  Follow `docs/design.md` and note the difference in the pull request.
- No agent picks a side by itself. The owner decides, and the decision is
  recorded with its date in the change log of the document concerned.

## Working rules

- Run `git fetch` before reporting the repository's state, and before
  saying that something does or does not exist.
- One Claude Code session per repository folder at a time.
- Run agents in the foreground, so the owner can approve their commands.
- Never push to `main`. Every change goes through a pull request, and only
  the owner merges, unless the owner allows an agent to for that task.
- Run the full local check before every push:
  `<build, lint, format and test command>`
- Report a check as passed only from a run made after the last change in
  the same session, judged by that command's own exit code. Old result
  files, or test results Gradle restored from its cache (FROM-CACHE,
  UP-TO-DATE, a build in seconds), are not a run: force execution, or say
  the result is cached (Tillslip M2 A).
- Keep pull requests reviewable: when a milestone grows large, split it by
  layer (for example rules, data, UI), each through its own review.
- Never print or ask for secrets. Secret and service config files stay
  untracked, with `.template` files showing their format.
- If the app has server security rules (for example Firestore rules), only
  the owner deploys them, from the reviewed commit, before the phone check,
  and the deployed commit is recorded in `docs/status.md`.

## Reporting

- The reviewer's and QA's reports are posted unchanged as comments on the
  pull request.
- Every role ends its report with: what was done, what is still open, and
  any questions for the owner.
- `docs/status.md` gets a one-line Log entry (date, what was done, the
  outcome) each time work is handed on: a pull request opened, each round
  of fixes, each review and QA verdict, and the merge. The developer
  commits the lines; the reviewer and QA end their report with theirs,
  and the developer adds them in the next round.
- Every role also saves its full final report, unchanged, as
  `.agent-reports/<YYYY-MM-DD>-<branch>-<role>[-round-N].md` (slashes in
  the branch name become dashes). `.agent-reports/` is in `.gitignore`,
  so the reports are never committed; the Tech Lead chat reads them
  there.

## App-specific notes

<Short facts every session needs for this app: services and regions, test
devices and accounts, who deploys what, anything easy to get wrong.>
