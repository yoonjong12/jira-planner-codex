---
name: subtask-to-nanotask
description: "Convert Subtasks to nanotask-level planning and execution tracked via status.md."
---

# SubtaskToNanotask Workflow

Convert Jira Subtasks into nanotask-level plans tracked via status.md.

---

## Quick Reference

### Hierarchy

```
Epic (WAO-180)
└── Story (WAO-252)
    └── Subtask (WAO-264)           ← Jira issue
        ├── Nanotask [commit]   WAO-264-1   ← status.md row
        ├── Nanotask [design]   WAO-264-2   ← status.md row
        ├── Nanotask [review]   WAO-264-3   ← status.md row
        └── Nanotask [docs]     WAO-264-4   ← status.md row
```

### Checkpoints (advisory — never block)

| CP | Gate | Verify |
| --- | --- | --- |
| CP1 | Plan files exist on disk | `ls space/{epic}/{story}/` shows plan.md + nanotask files |
| CP2 | User says "Approve" | Wait for explicit approval |
| CP3 | status.md created | `cat status.md` confirms nanotask table |

### Routing

**Read this first. Find your scenario, go to that section.**

| Scenario | How to detect | Go to |
| --- | --- | --- |
| Cold start | No plan.md | Cold Start |
| Partial plan | plan.md exists, no nanotask files | Cold Start (skip Select Story) |
| Ready for approval | plan.md + nanotask files, no status.md | Approval |
| Warm start | status.md exists | Implementation |
| Add nanotask mid-work | During Implementation, missing nanotask found | Amend Nanotask |
| Reopen completed nanotask | Completed nanotask needs follow-up under same goal | Reopen Nanotask |
| Add subtask mid-review | During Approval, missing subtask found | Escalate Subtask |
| Close out | All tasks done | Close & Report |

### Session Budget

Never mix planning and implementation in one session.

| Session | Sections | Output |
| --- | --- | --- |
| **Planning** | Cold Start → Approval → Status Creation | plan.md + nanotask files + status.md |
| **Implementation** | Implementation (+ Amend if needed) | Code / analysis / docs |

---

## Formats

### Jira Subtask Description (advisory hook)

Jira subtask descriptions should use this template. The `jira-description-guard.sh` hook warns if these 4 sections are missing.

```text
h2. Context
[Why this task exists — prior task, background. 1-2 sentences.]

h2. Objective
[What we're doing. 1-2 sentences.]

h2. Deliverables
* [Output 1]
* [Output 2]

h2. Acceptance Criteria
* [Condition 1]
* [Condition 2]
```

Purpose: concise plan that lets a manager understand context, scope, and expected results at a glance. No implementation details, no nanotask breakdowns, no internal architecture.

### File Structure

```text
$JIRA_PLANNER_SPACE_DIR/{epic-id}/{story-id}/
├── plan.md                    # Blueprint: objectives, nanotask definitions, decisions
├── status.md                  # Status tracker: nanotask current state (fast edit)
├── WAO-264-1.md               # Nanotask plan: subtask 264, nanotask 1
├── WAO-264-2.md               # Nanotask plan: subtask 264, nanotask 2
└── WAO-265-1.md               # Nanotask plan: subtask 265, nanotask 1
```

### plan.md Format

Blueprint — rarely modified after planning. Updated only when adding nanotasks or at Close.

```markdown
# Story WAO-252: [Summary]

## Objectives / Scope / Deliverables
[From Jira]

## Nanotasks

### WAO-264: [Summary]
[1] [commit] [Name]: [brief description]
[2] [analysis] [Name]: [brief description]

### WAO-265: [Summary]
[1] [docs] [Name]: [brief description]

## Dependency Chain
[Subtask/nanotask dependency graph]

## MUST READ
[Paths to reports, blueprints, or reference docs essential for this story's context.
These files are auto-loaded by jira-planner:onboarding at session start.]

## Decisions
[Architectural choices made during planning/implementation]
```

### status.md Format

Fast-edit status tracker — updated at every checkpoint and task completion. This is the **sole source of truth** for nanotask state.

```markdown
# WAO-252 Status

| ID | Type | Summary | Status | Hash | Updated |
|----|------|---------|--------|------|---------|
| 264-1 | commit | Add lifecycle tests | pending | - | - |
| 264-2 | analysis | Investigate feedback | pending | - | - |
| 265-1 | docs | Write integration guide | pending | - | - |
```

Status values: `pending`, `in_progress`, `completed`, `blocked`, `reopened`.

**Architecture:**
- **plan.md** = blueprint (nanotask definitions, context, decisions) — static after planning
- **status.md** = current state (status, hashes) — frequently edited, sole source of truth
- **Nanotask hierarchy** is expressed via ID prefix: `264-*` belongs to subtask WAO-264

### Nanotask Plan Formats ({subtask-id}-{N}.md)

#### Type: commit (code change)

```markdown
# WAO-264-1: [Name]
**Type:** commit

## Goal
[One sentence]

## Diffs
### [Logical unit name]
- path/to/file.py:145-160
  contents: [What exactly doing]
  note: [Context if risky]

## Blocked by / Blocks
[WAO-XXX-N if any]

## Verify
[pytest command or make lint]
```

#### Type: design (conceptual / architecture)

Produces **design documents, architecture decisions, or conceptual explorations** — NOT code.

```markdown
# WAO-264-2: [Name]
**Type:** design

## Goal
[What design question to answer or architecture to define]

## Scope
- [Concepts / components / trade-offs to explore]
- [Existing code or docs to reference]
- [NOT in scope]

## Decision Points
- [Choices the user must make]

## Output
- [Document path or Jira comment]
```

#### Type: review (analysis / evaluation)

Investigates, compares, or evaluates existing code/data/artifacts.

```markdown
# WAO-264-3: [Name]
**Type:** review

## Goal
[What question to answer or what to evaluate]

## Scope
- [Files / modules / APIs to investigate]
- [Constraints: time, depth]

## Findings
[Filled after completion]

## Recommendation
[Action items or next nanotasks]
```

#### Type: docs (documentation)

```markdown
# WAO-264-4: [Name]
**Type:** docs

## Goal
[What document to produce]

## Outline
- [Section 1]
- [Section 2]

## Target Files
- path/to/docs/guide.md

## Review Criteria
[What makes this "done"]
```

#### Shared Rules

- `Blocked by` / `Blocks` support both subtask-level (`WAO-265`) and nanotask-level (`WAO-264-2`) references
- Nanotask numbers (`{N}`) are append-only; always increment, no gaps
- Execution order follows dependency graph, not number order
- Type `commit` requires `## Diffs` and `## Verify`
- Type `design` requires `## Decision Points` and `## Output`
- Type `review` requires `## Findings` (filled after completion)
- Type `docs` requires `## Outline` and `## Target Files`

---

## Cold Start

Full planning flow: select story → explore → write plans → approve → create status.

### Select Story

**Skip if Story already provided via args.**

```typescript
mcp__atlassian__jira_search({
  jql: "project = WAO AND type = Story AND assignee = currentUser() AND status IN (\"In Progress\")",
  limit: 5,
  fields: "summary,status,parent,updated"
})
```

Present options, user picks a story. Then fetch Story context using `references/common_patterns.md#context-fetch`.

### Sync

Check local state and route:

```text
planExists?
├─ YES → Read plan.md (cat)
│         ├─ status.md exists → Warm Start (→ Implementation)
│         ├─ Nanotask files exist → go to Approval
│         └─ No nanotask files → continue to Plan & Write below
└─ NO → continue to Plan & Write below
```

### Plan & Write

**Goal:** Design nanotask plans and write all files in one pass.

```text
1. UNDERSTAND: Extract objectives from each Jira subtask
2. EXPLORE: Investigate codebase (use shell: find, grep, cat)
3. VALIDATE SCOPE: Cross-reference design docs (see below)
4. DESIGN: Break each subtask into nanotasks, assign types (commit/analysis/docs)
5. WRITE: Create all plan files in one pass (use shell redirect/tee)
6. VERIFY: ls to confirm files exist (CP1)
```

**VALIDATE SCOPE (mandatory when design docs exist):**

If MUST READ or subtask references contain a design doc:

```text
1. Read the design doc fully
2. List every deliverable/component the design requires
3. Map each deliverable to a nanotask
4. If a deliverable has no nanotask → it is a gap
5. Present coverage table to user before proceeding to DESIGN:

   | Design doc deliverable | Nanotask | Status |
   |------------------------|----------|--------|
   | ROI benchmark harness  | 345-5    | OK     |
   | LLM feedback generator | —        | GAP    |
   | Noise skill injection  | —        | GAP    |

6. Gaps must be resolved: add nanotask, defer to another subtask, or mark out-of-scope with user approval
```

Skipping this step when design docs exist violates CP1.

**Write all files:**

```text
1. mkdir -p $JIRA_PLANNER_SPACE_DIR/{epic-id}/{story-id}
2. Write plan.md (blueprint only)
3. Write each {subtask-id}-{N}.md
4. ls -la $JIRA_PLANNER_SPACE_DIR/{epic}/{story}/
```

**CP1 — files must exist on disk before proceeding.**

---

## Approval

**Goal:** Get user approval before creating status tracker.

Present summary:
1. Tree (subtasks → nanotasks with types → file counts)
2. Dependency chain
3. Critical files (3-5)

```text
Story WAO-252: Preparing MEGA Strategy integration
├── WAO-264: QA lifecycle-related functions
│   ├── [1] [commit] Add lifecycle tests (3 files)
│   └── [2] [analysis] Investigate feedback loop
├── WAO-265: Integrate wisdom graph [blocked by WAO-266]
│   └── [1] [commit] WG merge (5 files)
└── WAO-266: Refactor wisdom graph [PRIORITY]
    ├── [1] [commit] Extract storage interface (2 files)
    └── [2] [commit] Implement file backend (3 files)

Approve this plan? (Approve / Revise / Escalate / Cancel)
```

**CP2 — User must explicitly say "Approve".**

| Response | Action |
| --- | --- |
| Approve | → Status Creation |
| Revise | → back to Plan & Write |
| Escalate | → Escalate Subtask |
| Cancel | abort |

---

## Status Creation

**Goal:** Create status.md as the tracking hub.

### Procedure

```text
1. Write status.md with all nanotasks (initial: all pending)
   Include subtask grouping via ID prefix convention:
   - 264-* belongs to WAO-264
   - 265-* belongs to WAO-265

2. Update MEMORY.md with active_story and Next TODO
```

**CP3 — `cat status.md` confirms nanotask table exists on disk.**

---

## Implementation

**Goal:** Execute nanotask plans.

### Warm Start Entry

If status.md exists:

```text
1. Read status.md (cat) — source of truth for nanotask statuses
2. PRIOR ARTIFACTS: Before starting work on any nanotask, check for
   related completed nanotasks in status.md. If found:
   - Read their plan files (reports/, blueprint/, WAO-{id}-{N}.md)
   - These contain decisions, findings, and analysis already done
   - Do NOT re-investigate what prior nanotasks already documented
3. Continue below
```

### Process

```text
1. Read status.md → show available nanotasks (non-blocked first)
2. User selects nanotask (e.g., "264-1")
3. Edit status.md row → in_progress (use sed)
4. Read nanotask plan file (cat)
5. Execute based on type:
   - commit: Execute diffs in order, run verification
   - analysis: Investigate scope, fill Findings + Recommendation
   - docs: Write documents, check review criteria
6. On completion:
   - Edit status.md row → completed + hash (use sed)
7. Proceed to next
```

### Example

```text
status.md shows:

| ID | Type | Summary | Status | Hash | Updated |
|----|------|---------|--------|------|---------|
| 266-1 | commit | Extract storage | pending | - | - |
| 266-2 | commit | Implement backend | pending (blocked by 266-1) | - | - |
| 265-1 | commit | WG merge | pending (blocked by WAO-266) | - | - |
| 264-1 | commit | Add lifecycle tests | pending | - | - |

Available: 266-1, 264-1
Which nanotask? > 266-1
```

### Context Checkpoint

See `docs/checkpoint.md`. Triggered by user request.

Updates: MEMORY.md → status.md.

---

## Amend Nanotask

**When:** During Implementation, a missing nanotask is discovered.

```text
1. IDENTIFY: User describes what's missing
2. EXPLORE: Investigate target code (use grep, find, cat)
3. WRITE: Create {subtask-id}-{N+1}.md (next sequence number)
   → Choose type: commit / analysis / docs
4. UPDATE:
   - Append [N+1] entry to plan.md under the subtask
   - Append row to status.md (pending)
5. CONTINUE: Resume Implementation
```

**Constraints:**
- No Jira changes (nanotasks are SubtaskToNanotask's domain)
- Existing nanotask files untouched
- Sequence numbers always increment

---

## Reopen Nanotask

**When:** A completed nanotask needs follow-up work under the **same goal**.

**Trigger:** Post-completion issue found (e.g., review reveals gaps, test regression, follow-up analysis needed).

**Judgment — Reopen vs Amend (new):**

| Signal | Action |
| --- | --- |
| Follow-up serves the **same goal** as the original nanotask | **Reopen** the completed nanotask |
| Follow-up has a **different goal** (new scope, new deliverable) | **Amend** — create a new nanotask |

```text
1. IDENTIFY: Describe what follow-up is needed and why
2. VERIFY GOAL: Is this the same goal as the original nanotask?
   YES → continue (Reopen)
   NO  → go to Amend Nanotask instead
3. UPDATE status.md: completed → reopened (use sed)
4. UPDATE nanotask plan file: Add "## Follow-up" section with new scope
5. EXECUTE: Do the follow-up work
6. On completion:
   - Edit status.md row → completed + new hash
```

**Constraints:**
- Do NOT create a new nanotask file — extend the existing one
- The nanotask plan file gets a `## Follow-up` section appended (original content preserved)
- status.md shows `reopened` then `completed` (audit trail)

---

## Escalate Subtask

**When:** During Approval, user identifies a missing subtask.

```text
1. → StoryToSubtask Steps 2-5 (reuse existing context)
     → Identify Subtask (user describes gap)
     → Collaborative Planning (Objectives/Scope/Deliverables)
     → Draft Content (user approval)
     → Create in Jira

2. → Return to Cold Start (Plan & Write)
     → Existing plans preserved
     → Re-run exploration for new subtask only
     → Update plan.md + write new {subtask-id}-{N}.md files

3. → Back to Approval
```

---

## Close & Report

**When:** All nanotasks done in Implementation.

### Process

```text
1. COLLECT: git log + plan files + status.md
2. DRAFT: Per-subtask report → present to user
3. REFINE: User feedback → incorporate
4. PUBLISH: Jira comment per subtask
```

### Report Format

```markdown
## WAO-264: [Subtask Summary]

### Summary
[1-2 sentences]

### Lessons Learned
- [Insight]

### Issues & Resolutions
- **Issue:** [What] → **Fix:** {hash}

### Nanotasks
| # | Type | Summary | Hash |
|---|------|---------|------|
| 1 | commit | Add lifecycle tests | abc123 |
| 2 | analysis | Investigate feedback loop | — |
```

Uses disk-based context, so works regardless of session state.

---

## Reference

### Quality Checklists

**Plan & Write:**
- [ ] Each subtask has a nanotask list with types
- [ ] Nanotask names are specific (include file/module names)
- [ ] Type `commit` has Diffs with file:line + Verify command
- [ ] Dependencies specified
- [ ] `## MUST READ` section lists reference docs (reports, blueprints)
- [ ] If design docs exist in MUST READ: every deliverable maps to a nanotask (scope coverage verified)
- [ ] All files exist on disk (`ls` passed)

**Approval:**
- [ ] Tree summary with types shown
- [ ] Dependency chain + MUST READ references listed
- [ ] User explicitly said "Approve"

**Status Creation:**
- [ ] status.md created with all nanotasks (initial: pending)
- [ ] Nanotask grouping via ID prefix convention

**Close & Report:**
- [ ] Per-subtask report with Summary + Lessons + Nanotasks table
- [ ] Jira comments posted

### Common Issues

**Nanotask plan too abstract:**

Bad: `contents: add tests` → Good: `contents: Add test_lifecycle() with 3-phase test (generate→inject→evaluate)`

**Wrong nanotask type:**

`commit` = code change. `design` = conceptual/architecture (no code). `review` = analysis/evaluation. `docs` = documentation.

**Explore results insufficient:**

Agent found paths but no line numbers → Read file directly, identify function locations, add line numbers to plan.

**User wants to skip planning:**

Block. "Complete planning first. Current: {section}. Continue?"

### Links

- `references/plan_mode.md` — plan mode guide
- `references/common_patterns.md` — shared patterns
- `references/jira_commands.md` — JQL/MCP reference
- `references/guardrails.md` — hook-based enforcement
- `docs/checkpoint.md` — context checkpoint protocol
