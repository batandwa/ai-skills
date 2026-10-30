AI Agents Guide
===============

## Project Docs

- `docs/context.md`
  - Domain language.
  - Use its terms in code, UI and docs.
  - New or changed domain term, or a rule about what a term means
- `docs/adr/`
  - Architecture decision records.
  - Read before changing anything they cover.
  - Hard-to-reverse choice with real trade-offs (storage, library, data model, layering).
  - Where a new decision goes.
- `docs/adr/index.md`
  - List and links of all ADR docs present in the directory.
- `docs/tasks/releases.md`
  - List and links of all task docs in the directory.
- `docs/tasks/`
  - Releases and task specs.
- `docs/architecture.md`
  - Current structure, stack, data flow or schema.

## Tech Stack

- Use the stack exactly as indicated in the architecture document, unless something genuinely breaks. 
- Make suggestions to update the stack where necessary.
- If a task modifies directory structure, introduces new core dependencies, or alters data flow patterns, update architecture document before finalizing the task.

## Design Direction

- Accessibility: full keyboard navigation, visible focus states, semantic
  headings, labelled form controls, aria labels on icon buttons, and a text
  summary alternative for every chart.

## Definition of Done

Every task is done only when all of these are true. Do not stop before.

1. If dependencies changed: installation completes with no errors.
2. Tests pass - **every test**. Do not delete or skip a failing test to make this true - fix the code.
3. Build completes with **zero errors**.
4. Linting reports **zero errors**.
5. Application starts successfully.
6. README.md still accurately describes what the app does and how to run it.

## Quality Bar

- Production-ready. No TODO comments, no stubbed functions, no `any` types, no "Lorem Ipsum", no dead buttons. 
- Every control does something. It should look like a high-end product a real company shipped, not a template. 
- If you hit an error, read it and fix it - do not work around it by deleting the feature.
