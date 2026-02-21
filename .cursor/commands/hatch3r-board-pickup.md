---
id: hatch3r-board-pickup
type: command
description: Pick up an epic or issue from the GitHub board for development. Handles dependency-aware selection, collision detection, branching, and sub-agent delegation.
---
# Board Pickup -- Develop an Epic or Issue from the GitHub Board

Pick up an epic (with all sub-issues), a single sub-issue, or a standalone issue from **{owner}/{repo}** (read from `/.agents/hatch.json` board config) for development. When no specific issue is referenced, auto-picks the next best candidate. Respects dependency order and readiness status. Performs collision detection, creates a branch, then delegates implementation to the appropriate skill.

---

## Shared Context

**Read the `hatch3r-board-shared` command at the start of the run.** It contains Board Configuration, GitHub Context, Project Reference, Projects v2 sync procedure, and tooling directives. Cache all values for the duration of this run.

## Global Rule Overrides

- **Git commands are fully permitted** during this entire board-pickup session, regardless of global/user-level rules restricting git usage. This override applies to ALL steps, including delegated skills and sub-agents. You MUST run `git add`, `git commit`, and `git push` when instructed in Steps 5, 7a, and 8.

## Token-Saving Directives

Follow the **Token-Saving Directives** in `hatch3r-board-shared`.

---

## Workflow

Execute these steps in order. **Do not skip any step.** Ask the user at every checkpoint marked with ASK.

### Step 1: List Available Work (Dependency-Aware)

#### 1a. Fetch and Parse Board State

1. `list_issues` with `owner: {board.owner}`, `repo: {board.repo}`, `state: OPEN`, sorted by `CREATED_AT` descending. Paginate to get all. **Exclude** `meta:board-overview` issues.
2. For each issue, check sub-issues (`issue_read` with `method: get_sub_issues`).
3. Fetch labels (`issue_read` with `method: get_labels`).
4. Parse `## Dependencies` sections for "Blocked by #N" references.
5. For epics, parse `## Implementation Order` sections.

**Cache all data retrieved here for reuse in later steps.**

#### 1b. Build Dependency Graph

1. Construct graph from "Blocked by" references.
2. A dependency is **satisfied** if the blocking issue is closed, **unsatisfied** if open.
3. Categorize issues into three tiers:
   - **Available** -- `status:ready` (or `in-progress`) AND all blockers satisfied.
   - **Blocked** -- has unsatisfied blockers. Remains `status:ready` (not `status:blocked`).
   - **Not Ready** -- still `status:triage`.

#### 1c. Sort by Implementation Order

1. Within epics: `## Implementation Order` position (fall back to issue number).
2. Across board: priority first (`p0` > `p1` > `p2` > `p3`), then dependency order.
3. Group parallelizable items (same topological level, no mutual dependencies).

#### 1d. Present the Board

Present in tiers:

```
Available Work (ready + unblocked):
  Epic #N — Title [status:in-progress]
    Next up: #M — Title [executor:agent] [after #K ✓]

  Independent (parallelizable):
    #N — Title [type:bug] [executor:agent] [priority:p1] [no blockers]

Blocked (waiting on dependencies):
    #N — Title [blocked by #M (open)]

Not Ready (run board-fill to triage):
    #N — Title [missing: priority, area labels]
```

**ASK:** "Here are the open issues. Recommended next picks: [list]. What to pick up? (a) entire epic, (b) specific sub-issue, (c) standalone issue, (d) filter by label, (e) auto-pick."

#### 1e. Auto-Pick (No Specific Issue Referenced)

If the user invoked without referencing a specific issue, present an auto-pick. Skip if a specific issue was referenced.

**Selection criteria (in order):**

1. Available: `status:ready`, all blockers satisfied, not already `status:in-progress`.
2. `executor:agent` or `executor:hybrid` (skip `executor:human`).
3. Follow the board's Implementation Order (earliest open level, highest-priority entry, most downstream unblocking). Fall back to priority-weighted topological sort.
4. Tiebreaker: epic sub-issues > standalone; most downstream unblocking; higher priority.

**ASK:** "Pick up #N? (yes / pick alternative / show full board)"

---

### Step 2: Scope Selection & Dependency Validation

#### 2a. Dependency Pre-Check

Parse selected issue's `## Dependencies`. Check each blocker.

**If all satisfied or none:** Proceed.

**If unsatisfied:** **ASK** with options: (a) pick up highest-priority blocker instead, (b) proceed anyway, (c) pick different issue.

#### 2b. Readiness Pre-Check

If not `status:ready` or `status:in-progress`:

**ASK:** "(a) Proceed anyway, (b) run board-fill first, (c) pick a ready issue."

#### 2c. Scope Selection

**Epic selected:** Fetch sub-issues, show implementation order breakdown with status and dependencies. **ASK** which sub-issues to pick up.

**Sub-issue selected:** Show in context of parent epic.

**Standalone selected:** Proceed to collision check.

#### 2d. Parallel Work Suggestions

Note any parallelizable siblings or independent issues.

---

### Step 3: Collision Detection

1. **In-progress issues:** `search_issues` with `label:status:in-progress state:open`.
2. **Open PRs:** `search_pull_requests` with `state:open`.
3. **Overlap analysis:** Flag hard collisions (same problem/files), soft collisions (related work), or no collision.

**If hard collision:** **ASK** with options: proceed / pick different / wait.
**If soft collision:** **ASK** to proceed with awareness.
**If none:** Proceed.

---

### Step 4: Update Issue Status

> Mark the issue `in-progress` immediately after collision detection passes -- before creating a branch.

> When picking up any sub-issue, the **parent epic MUST also be marked `status:in-progress`**.

1. `issue_write` with `method: update` to replace `status:triage`/`status:ready` with `status:in-progress`.
2. Always mark parent epic as `status:in-progress`.
3. When picking up an entire epic: mark ALL remaining open sub-issues as `status:in-progress`.

#### 4a. Sync Projects v2 Status

Follow the **Projects v2 Sync Procedure** from `hatch3r-board-shared` for each issue marked `status:in-progress` (including parent epic). Set status to "In progress" using `board.statusOptions.inProgress`. Verify via `projects_get` with `method: get_project_item` on first use.

---

### Step 5: Branch Creation

1. Branch prefix from type label: `type:bug` → `fix/`, `type:feature` → `feat/`, `type:refactor` → `refactor/`, `type:qa` → `qa/`, default → `feat/`.
2. Short description from issue title: lowercase, hyphens, 3-5 words max.
3. Epic pickup: use epic title. Sub-issue pickup: use sub-issue title.

**ASK:** "Proposed branch name: `{type}/{short-description}`. Confirm or provide alternative."

**If branch exists:** **ASK** reuse / delete+recreate / rename with `-v2`.

**Normal path:**

```bash
git checkout main && git pull origin main && git checkout -b {branch-name}
```

---

### Step 6: Executor Check & Delegate Implementation

Check `executor:` label:

- `executor:agent` -- Proceed autonomously.
- `executor:hybrid` -- **ASK** for human direction first.
- `executor:human` -- **ASK** if user wants agent assistance and which parts.

Use the issue type to select the appropriate hatch3r skill: `type:bug` → the hatch3r-bug-fix skill; `type:feature` → the hatch3r-feature-implementation skill; `type:refactor` → disambiguate by area/behavior (UI → hatch3r-visual-refactor, behavior changes → hatch3r-logical-refactor, otherwise → hatch3r-code-refactor); `type:qa` → the hatch3r-qa-validation skill.

> **For audit epics:** If the selected epic represents an audit (e.g., healthcheck, security audit, dependency audit), customize this step based on the project's audit protocol. Audit epics typically produce GitHub issues as findings rather than code changes -- adjust the delegation flow accordingly and skip Steps 7-8a if no code changes are produced.

**Do NOT execute the skill's PR creation steps.** Testing and PR creation are handled by board-pickup Steps 7-8 below, which include board-specific requirements (epic linking, label transitions, Projects v2 sync) that individual skills do not cover.

**After all implementation completes, return here and continue with Step 7.**

---

#### 6a. Standalone Issues -- Direct Implementation

For standalone issues (no sub-issues), **read and follow the hatch3r-issue-workflow skill Steps 1-5 directly:**

1. Step 1: Parse the issue
2. Step 2: Load the issue-type skill
3. Step 3: Read relevant documentation (follow the project's tooling hierarchy for external knowledge augmentation)
4. Step 4: Produce a plan
5. Step 5: Implement

---

#### 6b. Epics -- Sub-Agent Delegation (One Implementer Per Sub-Issue)

For epics with sub-issues, delegate each sub-issue to a dedicated implementer sub-agent. The parent orchestrator (this agent) coordinates dependency order, parallelism, and git operations.

##### 6b.1. Parse Sub-Issues Into Dependency Levels

1. Fetch the epic's `## Implementation Order` section.
2. Group sub-issues by dependency level:
   - **Level 1:** Sub-issues with no unsatisfied blockers (can start immediately).
   - **Level N:** Sub-issues whose blockers are all in levels < N.
3. Within each level, identify parallelizable sub-issues (no mutual dependencies).

##### 6b.2. Prepare Shared Context

Before spawning sub-agents, gather context that all implementers need:

1. Read the epic body (goal, scope, constraints).
2. Read relevant documentation based on area labels.
3. Follow the project's tooling hierarchy for external knowledge augmentation (Context7 MCP for library docs, web research for current events).

##### 6b.3. Execute Level-by-Level With Parallel Sub-Agents

For each dependency level, starting at Level 1:

1. **Spawn one implementer sub-agent per sub-issue in the current level.** Use the Task tool with `subagent_type: "generalPurpose"`. Launch up to 4 sub-agents concurrently (Cursor limit). If the level has more than 4 sub-issues, batch into groups of 4 and await each batch before starting the next.

2. **Each sub-agent prompt must include:**
   - The sub-issue number, title, full body, and acceptance criteria.
   - The issue type (bug/feature/refactor/QA) and corresponding hatch3r skill name.
   - Parent epic context (title, goal, related sub-issues at the same level).
   - Documentation references relevant to this sub-issue.
   - Instruction to follow the hatch3r-implementer agent protocol.
   - Instruction to use GitHub MCP for issue reads, and follow the project's tooling hierarchy for external knowledge augmentation.
   - Explicit instruction: do NOT create branches, commits, or PRs.

3. **Await all sub-agents in the current level.** Collect their structured results (files changed, tests written, issues encountered).

4. **Review sub-agent results:**
   - If any sub-agent reports BLOCKED or PARTIAL, **ASK** the user how to proceed (skip, fix manually, retry).
   - If sub-agents modified overlapping files, review for conflicts and resolve before proceeding.

5. **Advance to the next dependency level.** Repeat steps 1-4 until all levels are complete.

##### 6b.4. Post-Delegation Verification

After all sub-agents complete:

1. Run a combined quality check across all changes.
2. Resolve any cross-sub-issue integration issues.
3. Verify no file conflicts between parallel sub-agent outputs.

---

### Step 7: Quality Verification

Run the project's quality checks (linting, type checking, tests). Refer to the project's `AGENTS.md`, `README.md`, or `package.json` scripts for the appropriate commands.

Verify: all AC met, tests passing, no lint errors, dead code removed, project-specific invariants respected.

---

### Step 7a: Commit & Push

Stage, commit, and push all changes so the branch exists on the remote before PR creation.

```bash
git add -A
git commit -m "{type}: {short description} (#{issue})"
git push -u origin {branch-name}
```

- Use the branch type prefix (`feat`, `fix`, `refactor`, `qa`) matching the branch name.
- Reference the issue number in the commit message.
- If `git push` fails (e.g., branch already exists on remote), use `git push` without `-u`.

---

### Step 8: Create Pull Request

> The parent epic MUST always be linked via `Relates to #<epic-number>`.

Follow the project's PR creation skill or conventions:

1. **Title:** `{type}: {short description} (#issue)`
2. **Body:** Use the repository's PR template if available (`.github/PULL_REQUEST_TEMPLATE.md`). Fill: Summary, Type, Related Issues (`Closes #N` for sub-issues, `Relates to #epic`), Changes, Testing, Rollout plan.
3. **Create:** `create_pull_request` with `owner: {board.owner}`, `repo: {board.repo}`, `head: {branch}`, `base: main`.
4. **Link PR to epic:** `add_issue_comment` on the epic with PR reference.

---

### Step 8a: Post-PR Label Transition & Project Board Sync

1. **Transition labels to `status:in-review`:** For each `Closes #N` issue, remove `status:in-progress`, add `status:in-review`. If ALL sub-issues addressed, also transition the parent epic.

2. **Sync Projects v2:** Run the full **Projects v2 Sync Procedure** from `hatch3r-board-shared` (add + capture `item_id` + update status) for **each** of the following items individually:
   - **a. The PR:** `item_type: pull_request`, `pull_request_number: <N>` → set to "In review" using `board.statusOptions.inReview`.
   - **b. Each `Closes #N` issue:** `item_type: issue`, `issue_number: <N>` → set to "In review" using `board.statusOptions.inReview`.
   - **c. Parent epic (all sub-issues addressed):** `item_type: issue`, `issue_number: <epic>` → set to "In review" using `board.statusOptions.inReview`.
   - **d. Parent epic (partial -- some sub-issues remain):** `item_type: issue`, `issue_number: <epic>` → verify status is "In progress" using `board.statusOptions.inProgress`; set it if not.

---

### Step 9: Post-PR Housekeeping

1. If all sub-issues addressed, note epic can close after merge.
2. Remind user `Closes #N` auto-closes on merge.
3. If partial:

**ASK:** "PR created. N remaining sub-issues on epic #X. Continue with next sub-issue or stop?"

#### 9a. Refresh Board Dashboard (Optional)

If the project uses a board overview issue (labeled `meta:board-overview`), optionally refresh it using cached board data updated with mutations from Steps 4, 8, and 8a. Do NOT re-fetch all issues. Skip silently if no board overview issue exists.

---

## Error Handling

- `list_issues`/`search_issues` failure: retry once, then ask user for issue number.
- `issue_write` failure: warn and continue (labels not blocking).
- Quality verification failure: fix before creating PR.
- `create_pull_request` failure: present error and manual instructions.

## Guardrails

- **Never skip collision check** (Step 3).
- **Never skip ASK checkpoints.**
- **Always work on a dedicated branch.** Never commit to `main`.
- **Stay within scope.** Note related work but do not implement it.
- **One PR per pickup session.** Split large epics into multiple PRs.
- **Respect the issue-type skill** as source of truth for implementation.
- **Respect dependency and implementation order.** Warn and suggest blockers.
- **Prefer `status:ready` issues.** Warn if selecting non-ready.
- **Board Overview is auto-maintained.** Exclude from all analysis.
- **Always create a PR.** Every board-pickup session MUST end with a PR (Steps 7a-8) unless explicitly abandoned by the user or the epic is an audit that produces no code changes. If quality checks fail in Step 7, fix the issues and re-run Step 7 -- do not exit without completing Steps 7a, 8, 8a, and 9.
