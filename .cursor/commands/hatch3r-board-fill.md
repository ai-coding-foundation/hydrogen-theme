---
id: hatch3r-board-fill
type: command
description: Create GitHub epics and issues from todo.md, reorganize the board with dependency analysis, readiness assessment, and implementation ordering.
---

# Board Fill -- Create Epics & Issues from todo.md + Board Reorganization

Create GitHub epics (with sub-issues) or standalone issues from items in `todo.md`, using the GitHub MCP tools against **{owner}/{repo}** (read from `/.agents/hatch.json` board config). On every run, board-fill also performs a **full board reorganization**: grouping standalone issues into epics, analyzing dependencies, setting implementation order, identifying parallel work, and marking issues as `status:ready` when all readiness criteria are met. AI proposes groupings, dependencies, and ordering; user confirms before anything is created or updated. Duplicate topics are detected and skipped.

---

## Shared Context

**Read the `hatch3r-board-shared` command at the start of the run.** It contains Board Configuration, GitHub Context, Project Reference, Projects v2 sync procedure, and tooling directives. Cache all values for the duration of this run.

## Token-Saving Directives

Follow the **Token-Saving Directives** in `hatch3r-board-shared`.

---

## Workflow

Execute these steps in order. **Do not skip any step.** Ask the user at every checkpoint marked with ASK.

### Step 1: Read and Parse todo.md

1. Read `todo.md` at the project root.
2. Parse each non-empty line as a separate todo item. Strip leading `- ` or `* ` markers.
3. Present the parsed list numbered.

**ASK:** "Here are the items I found in todo.md. Which items should I process? (all / specific numbers / exclude specific numbers)"

---

### Step 1.5: Full Board Scan

Scan the entire board to build an inventory of all existing work. This scan feeds into all subsequent steps. **Cache everything retrieved here.**

1. Fetch ALL open issues using `list_issues` with `owner: {board.owner}`, `repo: {board.repo}`, `state: OPEN`. Paginate to retrieve every issue. **Exclude** any issue with the `meta:board-overview` label from all subsequent processing.
2. For each issue, fetch labels (`issue_read` with `method: get_labels`) and check for sub-issues (`issue_read` with `method: get_sub_issues`).
3. Categorize every open issue:
   - **Epic** -- has sub-issues
   - **Sub-issue** -- is a child of an epic
   - **Standalone** -- neither parent nor child
4. For each issue, note presence of: `has-dependencies` label, `## Dependencies` section, required labels (`type:*`, `priority:*`, `area:*`, `executor:*`), acceptance criteria, documentation references, `## Implementation Order` (epics only).
5. Present a board health summary:

```
Board Health:
  Total open issues: N (X epics, Y sub-issues, Z standalone)
  Missing dependency metadata: #N, #M ...
  Missing required labels: #N — no priority, no area ...
  Potential epic grouping candidates: #N + #M (shared theme) ...
```

**ASK:** "Here is the current board state. I will process the selected todo.md items AND reorganize existing issues. Continue?"

---

### Step 2: Duplicate Detection

#### 2a. New Items vs. Existing Board

For each selected todo item:

1. `search_issues` with keywords derived from the item.
2. Compare against cached board inventory semantically.
3. Classify: **Duplicate** / **Partial overlap** / **No match**.

Present findings in a table and ask.

**ASK:** "These items appear already covered. Skip / create anyway / add as sub-issue? For partial overlaps, create new or link?"

#### 2b. Existing Board Cross-Deduplication

Compare existing open issues pairwise for semantic duplicates. Present any findings.

**ASK (only if duplicates found):** "These existing issues overlap. Merge / keep both / convert one to sub-issue?"

---

### Step 3: Issue Type & Executor Classification

For each remaining item, classify across all dimensions using these mapping tables:

**Type:**

| Signal                          | Type             | Label                       |
| ------------------------------- | ---------------- | --------------------------- |
| bug, broken, not working, fix   | Bug Report       | `type:bug`                  |
| add, implement, create, new     | Feature Request  | `type:feature`              |
| refactor, simplify, clean up    | Code Refactor    | `type:refactor`             |
| rework flow, change behavior    | Logical Refactor | `type:refactor`             |
| UI, visual, layout, dark mode   | Visual Refactor  | `type:refactor` + `area:ui` |
| test, QA, validate              | QA Validation    | `type:qa`                   |
| docs, document, README          | Documentation    | `type:docs`                 |
| CI, CD, pipeline, deploy, infra | Infrastructure   | `type:infra`                |

**Executor:** `executor:agent` (clear criteria, bounded scope) / `executor:human` (decisions, infra, external setup) / `executor:hybrid` (agent implements after human direction).

**Priority:** `priority:p0` (critical/security) / `priority:p1` (broken, no workaround) / `priority:p2` (degraded, default) / `priority:p3` (cosmetic, nice-to-have). Default `p2` when ambiguous; security defaults to `p1`+.

**Area:** Read area labels from `board.areas` in `/.agents/hatch.json`. If the list is empty, infer areas from the repository's directory structure. Assign all relevant area labels.

**Risk:** `risk:low` (isolated, easy rollback) / `risk:med` (shared modules, moderate scope) / `risk:high` (architectural, security-critical). Defaults: low for bug fixes; med for features; high for security/auth.

Present the full classification table with all label categories.

**ASK:** "Confirm or adjust the type, executor, priority, area, and risk for any item."

---

### Step 4: Retrieve Application Context

#### 4a. Project Documentation

1. Read project documentation relevant to the areas touched by the todo items (e.g., specs, ADRs, design docs).
2. Prefer reading TOC/headers first (first ~30 lines), then expand only relevant sections. Do NOT read full document bodies unless necessary.
3. Read ADRs or architectural documents if items touch architectural decisions.

#### 4b. Codebase Exploration

Use explore subagents or direct file reads to understand the current state of source areas touched by the todo items.

#### 4c. External Research (When Needed)

For items referencing external libraries, frameworks, or services not fully covered by local docs:

Follow the project's tooling hierarchy for external knowledge augmentation (Context7 MCP for library docs, web research for current events).

Skip if all items are purely internal (no external library involvement).

#### Output

Present a brief **Context Summary**: key constraints from documentation, current implementation state, external findings (if any).

---

### Step 5: Propose Grouping (Epics vs. Standalone)

**Grouping philosophy:** Minimize standalone issues to near-zero. Group aggressively into epics. Standalone only if topically isolated from every other issue AND substantial enough to stand alone.

#### 5a. Group New Items

1. **Absorb into existing epics first.**
2. **Form new epics** from 2+ items sharing any connection (area, subsystem, category).
3. **Adopt orphans** into broad thematic epics (e.g., "Security & Auth Hardening", "Infrastructure & Tooling").
4. **Standalone only as last resort.**

#### 5b. Regroup Existing Standalone Issues

Evaluate existing standalones for grouping into existing or new epics. Same aggressive philosophy.

Present grouping proposals for new items and existing regrouping separately.

**ASK:** "Confirm grouping, or: move items between groups / merge-split epics / convert epic↔standalone / reject existing regrouping."

---

### Step 5.5: Dependency Analysis & Implementation Order

> **Note on `status:blocked`:** Do NOT mark dependency-blocked issues as `status:blocked`. Reserve `status:blocked` for external blockers only.

#### 5.5a. Infer Dependencies

For every open issue, determine blocking relationships using:

1. Explicit references ("Depends on", "Blocked by", "After #N").
2. Semantic analysis (A creates what B consumes → A blocks B).
3. Epic internal ordering.
4. Area overlap (soft ordering, flag as recommendation).
5. Cross-epic dependencies.

#### 5.5b. Build the Dependency DAG

Directed graph: nodes = issues, edges = "blocked by". Validate: no cycles, identify orphans. Compute topological ordering.

#### 5.5c. Determine Implementation Order

1. Within epics: topological order, then priority at same level. Mark parallel items.
2. Across board: interleave by priority at each topological depth.
3. Identify parallel lanes.

#### 5.5d. Present

Present the dependency graph, parallel groups, and implementation order by priority lane. Draft `## Dependencies` and `## Implementation Order` sections for each issue/epic.

**ASK:** "Confirm or adjust dependencies and implementation order."

---

### Step 5.6: Readiness Assessment

#### Readiness Criteria (all must hold)

1. `## Dependencies` section present.
2. Issue appears in an `## Implementation Order` section or has a global position.
3. All required labels: one `type:*`, one `priority:*`, 1+ `area:*`, one `executor:*`, one `risk:*`.
4. Acceptance criteria present.
5. Issue body follows template structure.

#### Process

Evaluate each open issue at `status:triage`. Classify as Ready (all criteria met → `status:ready`) or Not Ready (list gaps). Do not downgrade issues already at `status:ready`/`in-progress`/`in-review`.

**ASK:** "These issues will be marked `status:ready`. Not-ready issues have gaps listed. Options: (a) confirm, (b) fill in gaps for specific issues, (c) skip readiness marking."

---

### Step 6: Refine Issue Content

Issue bodies must follow the structure for their type. Use this condensed reference:

| Type                           | Body sections                                                                                        |
| ------------------------------ | ---------------------------------------------------------------------------------------------------- |
| Bug Report                     | Summary, Steps to Reproduce, Expected/Actual Behavior, Acceptance Criteria, References, Dependencies |
| Feature Request                | Summary, Motivation, Proposed Solution, Acceptance Criteria, References, Dependencies                |
| Code/Logical/Visual Refactor   | Summary, Current State, Proposed Change, Acceptance Criteria, References, Dependencies               |
| QA Validation                  | Summary, Test Scope, Pass/Fail Criteria, Coverage Targets, Dependencies                              |
| Documentation / Infrastructure | Use Feature Request structure                                                                        |

**Epics** get: Overview (2-3 sentences), Sub-issues checklist, Acceptance criteria ("done when all sub-issues resolved"), all classified labels.

**Sub-issues** get: Problem/Goal, Acceptance criteria, Parent epic reference, all classified labels.

**Standalone** issues follow the sub-issue pattern without parent reference.

**ASK:** "Here are the drafted titles and acceptance criteria for each issue. Review and adjust before I create them."

---

### Step 7: Create & Update Issues via GitHub MCP

#### 7a. Create New Issues

Execute in dependency order (parents before children):

1. **Epics first:** `issue_write` with `method: create`, `owner: {board.owner}`, `repo: {board.repo}`. Include `## Dependencies` section and `has-dependencies` label. Record the returned `number` and internal numeric `id` field.
2. **Sub-issues:** Create each, then link via `sub_issue_write` with `method: add` using the parent `issue_number` and child's internal numeric `id` (NOT the issue number or node_id).
3. **Standalone issues:** Create with `## Dependencies` and `has-dependencies`.
4. **Add to project board + sync initial status:** For each created issue, run the full **Projects v2 Sync Procedure** from `hatch3r-board-shared`. This adds the item to the board, captures its `item_id` from the response, and sets the Projects v2 status to match the issue's `status:*` label (typically `status:ready` or `status:triage`). Use the label → option ID mapping from the sync procedure.

#### 7b. Update Existing Issues (Board Reorganization)

For issues needing updates (from Steps 5, 5.5, 5.6):

1. **Add/update `## Dependencies`:** Read current body, append/replace section, update via `issue_write`. Add `has-dependencies` label.
2. **Add/update `## Implementation Order`** (epics only): Append/replace section.
3. **Apply epic regrouping:** Link standalones to epics via `sub_issue_write`. Update epic body.
4. **Mark `status:ready`:** Remove `status:triage`, add `status:ready`. Do not downgrade existing statuses.
5. **Add missing labels** if user opted to fill gaps.
6. **Sync Projects v2 Status:** For each issue whose status label was set or changed in this run (including issues newly marked `status:ready` in step 4 above), run the full **Projects v2 Sync Procedure** from `hatch3r-board-shared`. This ensures the Projects v2 board column matches the label. Skip issues already synced in Step 7a.4.

#### 7c. Present Summary

```
New Issues Created:
| Type | Title | Issue # | Executor | Parent | Status |

Existing Issues Updated:
| Issue # | Title | Updates Applied |

Board Summary: N created, M updated, X marked ready, Y still triage, Z parallel lanes
```

---

### Step 7.5: Refresh Board Dashboard (Optional)

If the project uses a board overview issue (labeled `meta:board-overview`), optionally refresh it now using cached board data updated with mutations from Step 7. The format of the board dashboard is project-specific -- refer to the project's documentation or shared context for the template.

Do NOT re-fetch all issues; use cached data.

---

### Step 8: Cleanup

**ASK:** "All issues created. Should I remove processed items from `todo.md`? (yes / no / only created ones)"

If yes, edit `todo.md` to remove lines for created issues. Preserve skipped/excluded lines.

---

## Error Handling

- `search_issues` failure: retry once, then warn and proceed without dedup.
- `issue_write` failure: report, skip that issue, continue. Summarize failures at end.
- `sub_issue_write` failure: report but do not delete the created issue.
- Never create an issue without user confirmation in Step 6.

## Guardrails

- **Never create issues for topics already covered** without explicit user approval.
- **Never skip ASK checkpoints.**
- **Use correct labels** from the label taxonomy defined in `/.agents/hatch.json` (type, priority, area, risk, executor, status, has-dependencies, meta labels).
- **Keep issue bodies concise.** Minimal acceptance criteria directly derivable from the todo item.
- **No dependency cycles.** Flag and resolve before proceeding.
- **Never downgrade issue status.** Only upgrade `status:triage` → `status:ready`.
- **Always perform the full board scan** in Step 1.5.
- **Preserve existing issue body content** when appending sections.
- **Board Overview is auto-maintained.** Exclude it from all analysis. One board overview issue at a time.
