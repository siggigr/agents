# agents

A small team of AI agents that work together as a software company for
building Android apps and games. Each agent has one role, works from the
approved output of the agent before it, and stops for the owner's sign-off
before the next one starts.

This repository is the **master copy** of the role definitions. Edit the
roles here, then copy them to wherever they are used, so the copies never
drift apart. The roles are generic: nothing in them belongs to one app.

## The team

| Role | File | Produces |
| --- | --- | --- |
| Analyst (Product Owner) | [`agents/analyst.md`](agents/analyst.md) | Requirements document with numbered, testable FRs and NFRs |
| Architect | [`agents/architect.md`](agents/architect.md) | Architecture document, build order and build-and-release setup |
| Designer | [`agents/designer.md`](agents/designer.md) | Design document: colours, typography, icon, mockups |
| Developer | [`agents/developer.md`](agents/developer.md) | Code, one milestone per pull request, green in CI |
| Reviewer | [`agents/reviewer.md`](agents/reviewer.md) | Review report with findings and a verdict |
| QA Engineer | [`agents/qa.md`](agents/qa.md) | Test report, added tests, manual checklist for a phone |

## Workflow

```mermaid
flowchart LR
    Idea([Owner's idea]) --> A[Analyst]
    A -->|approved requirements| Ar[Architect]
    Ar -->|approved architecture| D[Designer]
    Ar -->|milestone 0| Dev[Developer]
    D -->|approved design| Dev
    Dev -->|pull request| R[Reviewer]
    R -->|findings| Dev
    R -->|approved| Q[QA]
    Q -->|defects| Dev
    Q -->|ready| M([Owner checks on phone and merges])
```

- The owner approves every document and every merge. An agent never hands
  off unapproved work.
- Problems flow backwards: the developer raises design questions with the
  architect, and anyone raises requirement problems with the analyst.
- Documents live in the app's repository under `docs/` (`requirements.md`,
  `architecture.md`, `design.md`), each with a status line at the top.
- Mechanical checks are done by tools in CI, not by agents: ktlint, detekt,
  Android Lint and Dependabot.

## Where each role runs

**Document roles (Analyst, Architect, Designer)** ask the owner questions in
rounds, so they work best in a Claude chat project. Paste this line at the
top of the project instructions, followed by the body of each role file
(everything below its `---` header) under a `## Role: <Name>` heading:

```
This project has several roles. Each chat starts with a line such as
"Role: Analyst" or "Role: Architect". Act only as that role and follow only
its section below. If no role is given, ask which one before doing anything.
```

Then start each chat with, for example:

```
Role: Architect
The approved requirements for <app> are in the project context. Please begin.
```

**Code roles (Developer, Reviewer, QA)** need the repository, so they run in
Claude Code. The files are already in Claude Code's agent format. Copy them
to your user-level agents folder so they are available in every project:

```powershell
# Windows (PowerShell)
New-Item -ItemType Directory -Force "$HOME\.claude\agents" | Out-Null
Copy-Item .\agents\*.md "$HOME\.claude\agents\"
```

```bash
# macOS / Linux
mkdir -p ~/.claude/agents && cp agents/*.md ~/.claude/agents/
```

All six files can be copied; the document roles also work in Claude Code if
you prefer to keep everything there. When an agent runs as a sub-agent and
cannot talk to you directly, it ends its output with its questions and
stops, so answer them and run it again.

## App-specific rules

Anything that belongs to one app (its name, services, data rules,
milestones) goes in that app's repository, not here:

- `docs/` holds the approved requirements, architecture and design.
- `CLAUDE.md` in the app's repository root holds short project notes that
  Claude Code reads automatically, for example which documents to read and
  any app-specific rules.

## What the owner does by hand

Agents never handle accounts, money or secrets. The owner:

- Creates the GitHub repository and turns on Dependabot alerts
- Creates cloud projects (for example Firebase) and downloads config files
- Creates and stores the release signing keystore
- Adds secrets to the CI settings
- Sets up the Google Play developer account and submits releases
- Approves each document, checks each milestone on a real phone, and merges

## Changing a role

1. Edit the role file in this repository and commit the change.
2. Copy it again to `~/.claude/agents/` and, for document roles, into the
   chat project instructions.
