---
name: release-task-management
description: Manage project tasks, releases, implementation specs, and acceptance criteria in file-based markdown documents inside docs/tasks/. Use when creating tasks, initializing releases, planning tasks, picking up task execution, deferring or moving tasks, or verifying completed deliverables.
allowed-tools: Read Write Edit Glob Grep Bash
---

# Release & Task Management Contract

Governs how tasks and releases are created, planned, executed, and verified inside `docs/tasks/`, using slug-based filenames. Project-agnostic: project-specific commands, stack, and paths come from the project's own agent instructions (`AGENTS.md`, `CLAUDE.md`), never from this skill.

## 1. Directory Structure

```
docs/tasks/
├── releases.md          # master roadmap and release index
├── v1.0.0/              # one directory per release
│   ├── slug-of-task.md
│   └── another-task.md
└── v1.1.0/
    └── ...
```

- Slugs are kebab-case, derived from the task title, and are the task's only identifier. Refer to other tasks by slug, never by number.
- A task file lives in the directory of the release it is listed under in `releases.md`. Always keep the two in step.

## 2. File Schemas

### A. Release Index (`docs/tasks/releases.md`)

```markdown
# Release Roadmap

## Release v1.0.0

- [x] [User authentication setup](v1.0.0/auth-setup.md)
- [ ] [Stripe integration](v1.0.0/stripe-integration.md)
- [~] [Search indexing](v1.0.0/search-indexing.md)
- [!] [Billing emails](v1.0.0/billing-emails.md)
- [-] [Analytics integration](v1.0.0/analytics-integration.md)
- [ ] Audit log export - admins cannot get an audit trail out of the app
```

Entry states:

| Marker | Meaning |
|---|---|
| `[!]` | Not yet interrogated - task file drafted without interrogating the user; its spec is a proposal, not agreed. Not ready to execute |
| `[ ]` | Pending - interrogated and ready to execute, or an unplanned stub with no file |
| `[~]` | In progress - claimed by one agent or person; others do not pick it up |
| `[x]` | Completed - every acceptance criterion in the task file is `[x]` and verified |
| `[-]` | Deferred - kept in its release for history, not planned for delivery in it |

Link states:  
Record states only in the checkbox. Never append state to the title (e.g. "(draft)", "(ungrilled)").

- **No link** - unplanned stub, no task file exists yet. May carry a one-line problem statement after the title: `- [ ] Title - problem`.
- **Link** - task file exists. Every link must resolve to a real file.

Order within a release is priority: the first pending entry is the next to pick up. Place new entries by priority, not at the end by default.

### B. Task File (`docs/tasks/v<X.Y.Z>/<slug>.md`)

```markdown
# Task: [Title]

## 1. Purpose & Motivation

- [Context around this task]
- **Problem:** [Problem statement]
- **Value:** [Expected business or technical value]

## 2. Implementation Details

- **Affected Areas:** [files or modules]
- **Approach:** [Patterns, models, or abstractions]
- **Dependencies:** [Prerequisite task slugs or external libraries]
- **Out of scope:** [What this task deliberately does not do, and which task owns it if any]

## 3. Acceptance Criteria

- [ ] [Explicit, verifiable behaviour 1]
- [ ] [Explicit, verifiable behaviour 2]
- [ ] Tests written and passing
- [ ] Lint, type check, and build passing

## 4. Notes

- [Open question, option, or risk to settle when planning]
```

- Each criterion must be checkable by running a command or observing behaviour. No vague criteria ("works well").
- Sections 2 and 3 may be empty only while the task is unplanned (Workflow 1).
- **Notes** is optional. It holds material that is not yet spec: open questions, options to weigh, risks, and pointers to related tasks or docs. Omit the section when empty. Notes never add scope - behaviour counts only once it is in Sections 2 or 3.

## 3. Operational Workflows

### Workflow 1: Adding an Unplanned Task

1. Identify the target release. Ask if unknown; otherwise infer from `releases.md`.
2. Add the title to that release section in `releases.md` as `[ ]`, with no link, at the position matching its priority. Append ` - <problem>` when the problem is known in one line.
3. If the problem is already known, create `docs/tasks/v<X.Y.Z>/<slug>.md` with only **Purpose & Motivation** filled, turn the entry into a link, and mark it `[!]` in the same change. Record anything to decide when planning under **Notes**. Never leave a file unlinked or a link without a file.

### Workflow 2: Planning a Task

1. Locate the entry in `releases.md`. Create the task file and release directory if missing.
2. Interrogate problem context, constraints, and desired outcomes with the user. Use a grilling/interview skill if one is installed. If asked to draft without interrogating, or the user is unavailable, draft anyway and mark the entry `[!]`.
3. Read the project's domain docs (e.g. `CONTEXT.md`, ADRs, architecture doc) and use their terms. Flag conflicts instead of silently diverging.
4. Fill all three sections per the schema. List prerequisite tasks by slug under **Dependencies**. Work through every item in **Notes**: fold each settled decision into Sections 2 or 3 and remove it from Notes. Keep only items that stay open on purpose, stating why.
5. Ensure the `releases.md` entry links to the file.
6. Set the entry to `[ ]` only once the user has been interrogated and has agreed the spec. Otherwise it stays `[!]`.

To clear an existing `[!]`: walk the user through the drafted spec and its **Notes**, interrogate every decision they did not make themselves, update the file, then set `[ ]`.

### Workflow 3: Executing a Task

1. If the entry is `[!]`, stop - run the interrogation in Workflow 2 first. Mark the entry `[~]` in `releases.md`. If it is already `[~]`, stop and ask - someone else has it. Read the task file and every task listed under its **Dependencies**.
2. If a dependency is pending or deferred, stop and report it before writing code.
3. Implement strictly within **Implementation Details**.
4. Do not scope-creep: no changes to **Purpose** and no features outside **Acceptance Criteria** without explicit approval. If the spec turns out wrong, report it and propose an edit.

### Workflow 4: Completing a Task

1. **Verify:** run the project's test, lint, type check, and build commands, as defined in its agent instructions. If they are not defined, ask.
2. **Check off criteria:** mark each criterion `[x]` only after verifying it against the code or a command result.
3. **Promote durable decisions:** anything in the task file that must outlive the task - domain terms, architectural patterns, schema, hard-to-reverse choices - goes into the project's domain docs (glossary, architecture doc, ADR). Task files are history, not the source of truth.
4. **Update the index:** mark the entry `[x]` in `releases.md` only when every criterion is `[x]`. If work stops unfinished, set it back to `[ ]`.
5. **Report:** short summary of criteria met and the verification command results.

### Workflow 5: Deferring or Moving a Task

- **Defer within a release:** mark the entry `[-]`. Keep the file. Check other task files that list it under **Dependencies** and report every one now blocked or needing a spec change.
- **Move to another release:** `git mv` the file to the new release directory, move the entry in `releases.md` to its priority position, and update the link.

### Workflow 6: Starting and Closing a Release

- **Start:** add a `## Release v<X.Y.Z>` section to `releases.md` in version order. Create the directory when its first task file is created.
- **Close:** a release is closed when every entry is `[x]` or `[-]`. Before closing:
  1. No entry may be `[!]`, `[ ]` or `[~]`. Finish it, defer it, or move it to a later release (Workflow 5).
  2. For each `[-]` entry, decide with the user: keep it deferred in this release as history, or move it to a later release.
  3. Report the release summary: completed, deferred, moved.

## 4. Guardrails

### Allowed

- Creating, reading, and modifying files under `docs/tasks/`.
- Moving task files between release directories with `git mv`.
- Checking off criteria after actual implementation and verification.

### Forbidden

- **NEVER** mark a criterion or task `[x]` without verification.
- **NEVER** mark a release entry `[x]` while any criterion in its file is `[ ]`.
- **NEVER** pick up an entry marked `[~]` by someone else.
- **NEVER** execute an entry marked `[!]`, or clear `[!]` without interrogating the user.
- **NEVER** edit a task's **Purpose & Motivation** during execution unless asked.
- **NEVER** leave a task file unregistered in `releases.md`, or a link pointing to a missing file.
- **NEVER** store scratchpads or logs in `docs/tasks/`. Notes go only in a task file's **Notes** section.
- **NEVER** implement something listed only in **Notes**.
