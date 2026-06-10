# Agent Entry Point
Micro-prompt architecture: load minimum prompts for maximum effect. System rules always resident, task-specific atoms loaded on demand via router.

## 0. Bootstrap (every session)
### 0-1. First-run check
If `.ai.dev/` is missing → load `prompts/atoms/task/init.atom` → run initialization.
If `.ai.dev/` exists but `DECISIONS.md` has no Q0 answer → load init.atom and ask Q0.

### 0-2. Permanent layer (always loaded, ~120 lines)
Load ONCE at session start, keep in context permanently:
1. `prompts/system/identity.prompt` — Agent identity, model spec, tool diagnostic
2. `prompts/system/rules.prompt` — Non-negotiable safety rules (security, approval, conflict detection, secrets)

### 0-3. Session layer (loaded once)
If `.ai.dev/DIGEST.md` exists → load for project identity (stack, scale, architecture, current state pointer).
If missing → generate from DECISIONS.md + CODEBASE_PROFILE.md.

## 1. Request Processing (every user request)
### 1-1. Load Router
`prompts/router/routing.prompt` — classifies request and selects atoms.

### 1-2. Atom Selection (router-driven, ~200-500 lines)
Router determines which atoms to load based on:
- **Task type** (1 atom): init | plan | implement | debug | test | review | document | deploy | migrate | refactor
- **Concerns triggered** (0-5 atoms): naming, async, transaction, error, logging, security, approval, perf, git, context-lifecycle, user-interaction, file-management, testing-strategy
- **Tech stack** (1-3 atoms): language, framework, DB, patterns
- **Augment** (0-1 atom): examples, checklist

### 1-3. Load Selected Atoms
Read ONLY the files selected by the router. Never pre-load speculatively.

### 1-4. Post-task
After task completion, task-specific atoms (L3) are candidates for compression into SUMMARY.md. System rules (L1) and project digest (L2) remain.

## 2. Atom Directory Structure
```
prompts/
├── system/           ← L1: Permanent (always in context)
│   ├── identity.prompt
│   └── rules.prompt
├── router/           ← Task classification + atom selection
│   └── routing.prompt
├── atoms/
│   ├── task/         ← L3: 10 task-specific atoms
│   │   ├── init.atom
│   │   ├── plan.atom
│   │   ├── implement.atom
│   │   ├── debug.atom
│   │   ├── test.atom
│   │   ├── review.atom
│   │   ├── document.atom
│   │   ├── deploy.atom
│   │   ├── migrate.atom
│   │   └── refactor.atom
│   ├── concern/      ← L3: 12 cross-cutting concern atoms
│   │   ├── approval.atom
│   │   ├── async.atom
│   │   ├── context-lifecycle.atom
│   │   ├── error.atom
│   │   ├── file-management.atom
│   │   ├── git.atom
│   │   ├── logging.atom
│   │   ├── naming.atom
│   │   ├── perf.atom
│   │   ├── security.atom
│   │   ├── testing-strategy.atom
│   │   ├── transaction.atom
│   │   └── user-interaction.atom
│   ├── tech/         ← L3: 41 technology-specific atoms
│   │   ├── lang-*.atom (8 + kotlin)
│   │   ├── framework-*.atom (19)
│   │   ├── db-*.atom (9)
│   │   ├── tools.atom
│   │   ├── async-patterns.atom
│   │   ├── distributed-patterns.atom
│   │   └── data-processing.atom
│   └── augment/      ← L3: 2 depth-control atoms
│       ├── examples.atom
│       └── checklist.atom
└── profiles/         ← Q0-based (loaded during init only)
    ├── greenfield.prompt
    └── migration.prompt
```

## 3. Priority Hierarchy
System rules (L1) > Session digest (L2) > Task atoms > Concern atoms > Tech atoms
When rules conflict, higher priority wins. If conflict with existing codebase conventions, confirm with user.

## 4. Lightweight Mode (small projects)
DECISIONS.Q3 = A (small: 1 person, ≤1 week):
- L1 (system rules) loaded as normal
- L2 (project digest): minimal
- L3: only task atom + language atom. No concern atoms. No augment atoms.
- Init: Q0~Q4 only, Q5~Q8 auto-apply defaults.

## 5. Importance Tags for Compression
|Tag|Meaning|Compression Policy|
|------|------|------|
|`[근본]`|Core decision, never changes|Exempt from compression, always preserved in SUMMARY.md|
|`[전술]`|Context-dependent decision|Keep summary only on compression|
|`[가역]`|Easily reversible decision|First to be dropped on compression|

## 6. Commands
|Command|Action|
|------|------|
|`initialize project`|Run init task atom|
|`reconfigure agent`|Reset AGENT_CONFIG.md|
|`show agent config`|Display current config|
|`show token estimate`|Display estimated token usage|
|`calibrate tokens`|Update calibration values|
|`compress context`|Immediate session compression|
|`show context summary`|Display SUMMARY.md|
|`search archive: <keyword>`|Search archive index|
|`auto-approve on / off`|Toggle auto-approval mode|
