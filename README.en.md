# design-to-implement

> 📖 **Languages**: [English](README.en.md) · [简体中文](README.md)

> An agent skill that distills the full engineering workflow—from "solution discussion" to "code landing"—into a structured, document-driven process.

[![Skill Type](https://img.shields.io/badge/type-agent%20skill-blue)](#) [![License](https://img.shields.io/badge/license-MIT-green)](#license)

## What is this

A skill for AI coding agents that forces a clean separation between **thinking** and **doing**:

- **Discussion phase**: Talk only, write no code. Conclusions land in a communication log.
- **Design phase**: Turn consensus into a design document (with acceptance criteria) — the single source of truth.
- **Implementation planning phase**: Break the design into executable steps + test cases.
- **Implementation phase**: Land code step by step, validating each step against the test cases.

The core idea: **discuss thoroughly before writing any code.** No code is written until the user explicitly says "start landing."

## Problems it solves

Common pain points when developing with AI agents:

- Coding starts before the approach is clear, only to discover halfway through that the direction was wrong
- Across multiple conversations/contexts, earlier conclusions get lost — leading to re-reading code or asking the user to repeat
- Design and implementation drift apart during changes; docs become after-the-fact records
- After implementation, there's no clear "done" definition; testing relies on gut feel
- Large tasks run in a single context, causing context bloat and quality decay toward the end

This skill systematically addresses these issues with a structured document workflow + multi-context memory mechanism.

## Core mechanisms

| Mechanism | Description |
|-----------|-------------|
| Four-phase workflow | Discussion → Design → Implementation planning → Implementation. Each phase transition requires explicit user confirmation. |
| Multi-context memory | Each task has a "task status" document as a memory anchor. New conversations read it first to recover context. |
| Single source of truth | Plan changes always update the design doc (with a change log), never the implementation doc directly. |
| Acceptance criteria + test cases | Design phase defines acceptance criteria (rules); implementation planning turns them into test cases (instances). Three-layer separation of concerns. |
| Modify / Rollback mechanisms | Discovered gaps during implementation go through "modify" (lightweight increment); fundamental errors go through "rollback" (scrap and redo). Both require user confirmation. |
| Task splitting (Mode A) | Large multi-module tasks can split into a parent task (interface contracts) + subtasks (each with its own full workflow). |
| Dual-agent implementation (Mode B) | A single task's implementation + testing phases can be split into two independent agents running in series, with isolated contexts. |

## When to use

- Design + implementation of medium-to-large features
- R&D tasks that span multiple conversations
- Backend projects (the design doc enforces an "interface change list" for frontend integration)
- Complex multi-module systems

**Not for**: One-off small requests, fixing a typo, adding a field — just do those directly, don't wrap them in process.

## Tool compatibility

**The skill body is tool-neutral.** The core workflow (document maintenance, phase transitions, change mechanisms) does not depend on any specific agent client — any client that can "read local files + write local files" can use it.

Extension modes (A/B) involve multi-agent scheduling, which is client-specific:

| Client | Status |
|--------|--------|
| ZCode | Verified. See [`assets/references/zcode-scheduling.md`](assets/references/zcode-scheduling.md) |
| Other clients | Unverified. Contributions welcome (see "Contributing" below) |

## Installation

Place this directory into the skills folder your agent client recognizes. Common paths:

- ZCode: `~/.agents/skills/design-to-implement/` or `<project>/.agents/skills/design-to-implement/`
- Other clients following the `.agents/skills/` convention: same as above

Once installed, invoke it explicitly in a conversation (e.g., `/design-to-implement` or "use the design-to-implement skill for this requirement"). This skill is **manual-trigger only** — it does not auto-activate based on keywords.

## Usage example

Start a new task (in an agent conversation):

```
User: /design-to-implement
      We need to build a file service SDK — wrap the existing HTTP API into an SDK.

Agent: (creates docs/design/01-file-service-sdk/ folder, generates 5 document skeletons,
        enters discussion phase, lists open questions)
       I need to confirm a few things first: 1. SDK language? 2. Keep the existing HTTP API?...

User: (answers each item)

Agent: (updates communication doc, marks closed questions)
       All questions resolved. Ready to enter the design phase?

User: Start design.

Agent: (generates 02-design.md, with module breakdown + interface contracts + acceptance criteria)
       ...
```

See SKILL.md for the full workflow.

## Structure

### Skill package structure (what you see after clone)

```
design-to-implement/
├── README.md                          ← Project intro (Chinese)
├── README.en.md                       ← Project intro (English, this file)
├── LICENSE                            ← MIT
├── SKILL.md                           ← Skill body (agent reads this when loaded)
└── assets/
    ├── readme.md                      ← Task overview template
    ├── task-status.md                 ← Task status anchor (regular/subtask)
    ├── parent-task-status.md          ← Parent task template (Mode A)
    ├── communication.md               ← Communication log template
    ├── design.md                      ← Design doc template (with acceptance criteria)
    ├── implementation.md              ← Implementation doc template (steps + case refs)
    ├── test.md                        ← Test cases template (four-layer cases)
    └── references/
        └── zcode-scheduling.md        ← ZCode scheduling reference (loaded on demand)
```

**Progressive disclosure convention**:
- `SKILL.md` is always in context (read as soon as the agent loads the skill)
- `assets/` templates are read on demand (when creating the corresponding document)
- `assets/references/` are loaded further on demand (only for Mode B + a specific client)

### User project output structure (what you get after using the skill)

```
your-project/
└── docs/design/
    ├── README.md                          ← Global task overview (entry point for all tasks)
    │
    ├── 01-single-module-task/             ← Regular task (one doc per phase)
    │   ├── 00-task-status.md              ← Memory anchor (read this first in new conversations)
    │   ├── 01-communication.md            ← Discussion conclusions + open questions list
    │   ├── 02-design.md                   ← Plan + acceptance criteria + interface change list
    │   ├── 03-implementation.md           ← Step checklist (each step refs test cases)
    │   └── 04-test.md                     ← Four-layer case set (F/EX/R/API)
    │
    └── 02-large-multi-module-task/        ← Parent task (Mode A)
        ├── 00-task-status.md              ← Includes subtask list + dependency relations
        ├── 02-design.md                   ← Overall architecture + module split + interface contracts
        └── subtasks/
            ├── 02-1-submodule-A/          ← Subtask (uses regular template, full 4 phases)
            │   └── ... (docs 00~04)
            └── 02-2-submodule-B/
                └── ...
```

**Memory mechanism diagram** (when a new conversation enters a task, the agent recovers context in this order):

```
1. docs/design/README.md              → locate the current task
2. <task>/00-task-status.md            → confirm current phase and next step
3. The doc for the current phase       → resume work
4. Read code only when necessary       → not the first means of context recovery
```

## Contributing

Contributions via issues / PRs are welcome:

- **Scheduling adaptation references for other agent clients** (e.g., Claude Code, Cursor): add `<client-name>-scheduling.md` under `assets/references/`, following the structure of `zcode-scheduling.md`
- **Bug fixes / mechanism improvements**: please open an issue describing the scenario; PRs should include the rationale for changes
- **New templates**: e.g., special templates for different project types (frontend, mobile)

## License

MIT (see [LICENSE](LICENSE))
