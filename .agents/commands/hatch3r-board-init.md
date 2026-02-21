---
id: hatch3r-board-init
type: command
description: Initialize a GitHub Projects V2 board with hatch3r's label taxonomy, status fields, and board structure. Optionally migrate issues from an existing project.
---

# Board Init -- Bootstrap a GitHub Projects V2 Board

Initialize a new or existing GitHub Projects V2 board for **{owner}/{repo}** (read from `/.agents/hatch.json` board config). Sets up status fields, creates the full hatch3r label taxonomy, optionally migrates issues from another project, and writes all IDs back to `/.agents/hatch.json` so subsequent board commands work out of the box. AI proposes configuration; user confirms before any mutation.

---

## Shared Context

**Read the `hatch3r-board-shared` command at the start of the run.** It contains Board Configuration, GitHub Context, Project Reference, Projects v2 sync procedure, and tooling directives. Cache all values for the duration of this run.

## Token-Saving Directives

Follow the **Token-Saving Directives** in `hatch3r-board-shared`.

---

## Workflow

Execute these steps in order. **Do not skip any step.** Ask the user at every checkpoint marked with ASK.

### Step 1: Read Configuration

1. Read `/.agents/hatch.json` and cache the `board` config.
2. Check whether `board.owner` and `board.repo` are set (non-null, non-empty).
3. If both are set, confirm: "Using owner=`{owner}`, repo=`{repo}`."
4. If either is missing:

**ASK:** "I need the GitHub owner and repository for this board. Please provide: (1) owner (org or username), (2) repo name."

Update the in-memory config with the provided values.

---

### Step 2: Choose Mode

**ASK:** "How would you like to set up the Projects V2 board?

- **A** — Create a new GitHub Projects V2 board
- **B** — Connect to an existing project (provide the project number)"

Record the user's choice and, for option B, the project number.

---

### Step 3: Create or Connect Project

#### Option A — Create New Project

1. Fetch the repository node ID:
   ```graphql
   query {
     repository(owner: "{owner}", name: "{repo}") {
       id
     }
   }
   ```
   Use the GitHub MCP `graphql` tool with `owner: {board.owner}`, `repo: {board.repo}`.
2. Create the project:
   ```graphql
   mutation {
     createProjectV2(
       input: { ownerId: "<repo_owner_node_id>", title: "{repo} Board" }
     ) {
       projectV2 {
         id
         number
       }
     }
   }
   ```
   The `ownerId` must be the **owner's** node ID (org or user), not the repository node ID. Fetch the owner node ID first if needed:
   ```graphql
   query {
     repositoryOwner(login: "{owner}") {
       id
     }
   }
   ```
3. Capture the project `id` (node ID) and `number` from the response.

#### Option B — Connect to Existing Project

1. Query the existing project:
   ```graphql
   query { user(login: "{owner}") { projectV2(number: {N}) { id number } } }
   ```
   Use `organization` instead of `user` if the owner is an org. Try `user` first; if it fails, retry with `organization`.
2. Capture the project `id` and `number`.

---

### Step 4: Configure Status Field

1. Query the project's fields to find the "Status" single-select field:
   ```graphql
   query {
     node(id: "<project_id>") {
       ... on ProjectV2 {
         fields(first: 50) {
           nodes {
             ... on ProjectV2SingleSelectField {
               id
               name
               options {
                 id
                 name
               }
             }
           }
         }
       }
     }
   }
   ```
2. Look for a field named "Status" (case-insensitive match).
3. If no Status field exists, create one via the `createProjectV2Field` mutation with type `SINGLE_SELECT`.
4. Ensure these status options exist on the field: **Backlog**, **Ready**, **In Progress**, **In Review**, **Done**.
   - For missing options, use the `updateProjectV2Field` mutation (or the appropriate mutation for adding options to a single-select field) to add them.
5. Capture the field ID and each option's ID.

Present the configured status options in a table:

```
Status Field: {fieldId}
  Backlog      → {optionId}
  Ready        → {optionId}
  In Progress  → {optionId}
  In Review    → {optionId}
  Done         → {optionId}
```

**ASK:** "Here are the status options configured on the project board. Confirm or adjust?"

---

### Step 5: Create Label Taxonomy

1. Read the label taxonomy from `board.labels` in `/.agents/hatch.json`.
2. If labels are not defined or empty, use these defaults:

| Category | Labels                                                                                      |
| -------- | ------------------------------------------------------------------------------------------- |
| Type     | `type:bug`, `type:feature`, `type:refactor`, `type:qa`, `type:docs`, `type:infra`           |
| Executor | `executor:agent`, `executor:human`, `executor:hybrid`                                       |
| Status   | `status:triage`, `status:ready`, `status:in-progress`, `status:in-review`, `status:blocked` |
| Priority | `priority:p0`, `priority:p1`, `priority:p2`, `priority:p3`                                  |
| Risk     | `risk:low`, `risk:med`, `risk:high`                                                         |
| Meta     | `meta:board-overview`, `has-dependencies`                                                   |

3. For each label, check if it already exists on the repository using `get_label` (or `list_labels` to fetch all at once). Create only missing labels via `create_label`.
4. Use consistent colors per category:

| Category                      | Color scheme            | Hex examples                                                     |
| ----------------------------- | ----------------------- | ---------------------------------------------------------------- |
| `type:*`                      | Blue shades             | `#0052CC`, `#1D76DB`, `#5319E7`, `#0075CA`, `#006B75`, `#0E8A16` |
| `executor:*`                  | Green shades            | `#0E8A16`, `#2EA44F`, `#7CFC00`                                  |
| `status:*`                    | Yellow/Orange shades    | `#FBCA04`, `#F9D0C4`, `#E4E669`, `#FFA500`, `#D93F0B`            |
| `priority:*`                  | Red shades (p0 darkest) | `#B60205`, `#D93F0B`, `#E99695`, `#F9D0C4`                       |
| `risk:*`                      | Purple shades           | `#5319E7`, `#7B68EE`, `#D4C5F9`                                  |
| `meta:*` / `has-dependencies` | Gray                    | `#BFD4F2`, `#C5DEF5`                                             |

5. Present a summary of created vs. existing labels:

```
Labels:
  Created (N): type:bug, executor:agent, ...
  Already existed (M): type:feature, ...
```

**ASK:** "Label taxonomy created. Would you also like to create area labels? If yes, list area names (e.g., frontend, backend, infra)."

6. If the user provides area names, create `area:{name}` labels for each (teal/cyan color, e.g., `#006B75`). Add area names to `board.areas` in the in-memory config.

---

### Step 6: Migrate from Existing Project (Optional)

**ASK:** "Do you have an existing GitHub project to migrate issues from? (yes + project number / no)"

#### If yes:

1. Query the source project to get all items:
   ```graphql
   query {
     node(id: "<source_project_id>") {
       ... on ProjectV2 {
         items(first: 100, after: <cursor>) {
           pageInfo { hasNextPage endCursor }
           nodes {
             id
             content { ... on Issue { id number title } ... on PullRequest { id number title } }
             fieldValues(first: 20) {
               nodes {
                 ... on ProjectV2ItemFieldSingleSelectValue { name field { ... on ProjectV2SingleSelectField { name } } }
               }
             }
           }
         }
       }
     }
   }
   ```
   Paginate until all items are retrieved. Resolve the source project node ID from the project number first (same approach as Step 3 Option B).
2. For each item with linked issue content, add it to the new project board via the `addProjectV2ItemById` mutation:
   ```graphql
   mutation {
     addProjectV2ItemById(
       input: { projectId: "<new_project_id>", contentId: "<issue_node_id>" }
     ) {
       item {
         id
       }
     }
   }
   ```
3. Map the source project's status to the new project's status options (best-effort string matching: exact match first, then case-insensitive, then substring). Update each migrated item's status on the new board using the `updateProjectV2ItemFieldValue` mutation.
4. Present a migration summary:

```
Migration Summary:
  Items migrated: N
  Status mappings:
    "{source_status}" → {target_status} (X items)
    ...
  Unmapped statuses: "{status}" (Y items, defaulted to Backlog)
  Failures: Z items (listed below)
    - #{number}: {error}
```

#### If no: skip to Step 7.

---

### Step 7: Write Configuration Back

1. Prepare the updated `board` config with all captured IDs:

   - `board.owner` — set if it was missing
   - `board.repo` — set if it was missing
   - `board.projectNumber` — from the created/connected project
   - `board.statusFieldId` — from the Status field
   - `board.statusOptions.backlog` — option ID
   - `board.statusOptions.ready` — option ID
   - `board.statusOptions.inProgress` — option ID
   - `board.statusOptions.inReview` — option ID
   - `board.statusOptions.done` — option ID
   - `board.areas` — if area labels were created

2. Present the updated `/.agents/hatch.json` content to the user.

**ASK:** "Here is the updated configuration. Confirm before I write it to `/.agents/hatch.json`?"

3. On confirmation, write the file. Preserve any keys outside the `board` section.

---

### Step 8: Create Board Overview Issue (Optional)

**ASK:** "Create an initial board overview issue (labeled `meta:board-overview`) to serve as a dashboard? (yes / no)"

#### If yes:

1. Create an issue via `issue_write` with `method: create`, `owner: {board.owner}`, `repo: {board.repo}`:
   - **Title:** `[Board Overview] {repo} Project Board`
   - **Labels:** `meta:board-overview`
   - **Body:**

```markdown
## Board Overview

**Project:** {owner}/{repo} (Project #{projectNumber})
**Status:** Active

## Current Sprint

| Status      | Count |
| ----------- | ----- |
| Backlog     | 0     |
| Ready       | 0     |
| In Progress | 0     |
| In Review   | 0     |
| Done        | 0     |

## Labels

All labels from the hatch3r taxonomy have been configured.

---

_This issue is auto-maintained by the board management commands. Do not close._
```

2. Add the issue to the project board and set its status to **Backlog** using the Projects v2 Sync Procedure from `hatch3r-board-shared`.

#### If no: skip.

---

### Step 9: Summary

Print a complete summary:

```
Board Init Complete:
  Project: {owner}/{repo} (Project #{number})
  Status field: configured (5 options)
  Labels created: N new, M existing
  Areas: [list or "none"]
  Migration: N issues migrated from Project #X (or "skipped")
  Board overview: #{issueNumber} (or "skipped")
  Config: /.agents/hatch.json updated
```

---

## Error Handling

- **GraphQL mutation failure:** Report the error and suggest checking GitHub PAT permissions (must include `project` scope for Projects V2 operations).
- **Label creation failure:** Report the failing label, continue with remaining labels. Summarize failures at end.
- **Migration failure:** Report per-item, continue with remaining items. Summarize at end.
- **Never create or mutate without user confirmation.**

## Guardrails

- **Never modify or delete existing labels.** Only create missing ones.
- **Never remove issues from existing projects.** Migration is additive only.
- **Always ASK before any write operation.**
- **Never skip ASK checkpoints.**
- **Require `project` scope** on the GitHub token for Projects V2 operations. If mutations fail with permission errors, surface this requirement.
- **Preserve existing `/.agents/hatch.json` content** outside the `board` key when writing config back.
