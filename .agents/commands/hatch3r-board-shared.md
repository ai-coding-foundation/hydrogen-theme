---
id: hatch3r-board-shared
type: command
description: Shared context and procedures for all board commands. Provides GitHub context from hatch.json, Projects v2 sync, and tooling directives.
---
# Board Shared Reference

Shared context for `hatch3r-board-fill`, `hatch3r-board-pickup`, and related board commands. Read once per run and cache.

---

## Board Configuration

All board commands read project-specific configuration from `/.agents/hatch.json` under the `board` key. This file defines the GitHub owner/repo, Projects v2 IDs, label taxonomy, branch conventions, and area labels. **Read `/.agents/hatch.json` at the start of every run and cache the `board` config for the duration.**

```json
{
  "board": {
    "owner": "{github-org-or-user}",
    "repo": "{repository-name}",
    "projectNumber": null,
    "statusFieldId": null,
    "statusOptions": {
      "backlog": null,
      "ready": null,
      "inProgress": null,
      "inReview": null,
      "done": null
    },
    "labels": {
      "types": ["type:bug", "type:feature", "type:refactor", "type:qa", "type:docs", "type:infra"],
      "executors": ["executor:agent", "executor:human", "executor:hybrid"],
      "statuses": ["status:triage", "status:ready", "status:in-progress", "status:in-review", "status:blocked"],
      "meta": ["meta:board-overview"]
    },
    "branchConvention": "{type}/{short-description}",
    "areas": []
  }
}
```

If any field is `null` or missing, the corresponding feature is disabled (e.g., null `projectNumber` → skip Projects v2 sync).

---

## GitHub Context

Derived from `/.agents/hatch.json` board config:

- **Owner:** `board.owner`
- **Repository:** `board.repo`
- **Type labels:** `board.labels.types`
- **Executor labels:** `board.labels.executors`
- **Status labels:** `board.labels.statuses`
- **Dependency label:** `has-dependencies`
- **Meta labels:** `board.labels.meta`
- **Branch convention:** `board.branchConvention`
- **Issue templates:** Check `.github/ISSUE_TEMPLATE/` if present in the repository.
- **PR template:** Check `.github/PULL_REQUEST_TEMPLATE.md` if present.

### Project Reference (cache for the full run)

If `board.projectNumber` is not null, verify via `projects_list` with `list_project_fields` on first use.

- **Owner:** `board.owner`, **owner type:** infer from context (`org` or `user`)
- **Project number:** `board.projectNumber`
- **Status field ID:** `board.statusFieldId`
- **Status option IDs:** Read from `board.statusOptions` (keys: `backlog`, `ready`, `inProgress`, `inReview`, `done`)

---

## Projects v2 Sync Procedure

> **Skip entirely if `board.projectNumber` is null.**

Use this procedure whenever a status label is set or changes and the board needs to reflect it. Labels are the source of truth; Projects v2 sync keeps the board view consistent. This includes newly created issues -- sync their initial status immediately after adding them to the board.

**Status label → Projects v2 option mapping:**

Read the mapping from `board.statusOptions` in `/.agents/hatch.json`:

| Label                | Option ID from hatch.json          |
| -------------------- | ---------------------------------- |
| `status:triage`      | `board.statusOptions.backlog`      |
| `status:ready`       | `board.statusOptions.ready`        |
| `status:in-progress` | `board.statusOptions.inProgress`   |
| `status:in-review`   | `board.statusOptions.inReview`     |
| `status:blocked`     | `board.statusOptions.backlog`      |

**Steps for each issue to sync:**

1. **Add to board + capture item ID:** `projects_write` with `method: add_project_item`, `owner: {board.owner}`, `owner_type: {inferred}`, `project_number: {board.projectNumber}`, `item_type: issue`, `issue_number: <N>`, `item_owner: {board.owner}`, `item_repo: {board.repo}`. **Capture the `item_id` from the response.** This call is idempotent -- if the item already exists on the board it returns the existing item with its ID.
2. **Update status:** `projects_write` with `method: update_project_item`, `owner: {board.owner}`, `owner_type: {inferred}`, `project_number: {board.projectNumber}`, `item_id: <captured_item_id>`, `updated_field: { "id": {board.statusFieldId}, "value": "<option_id>" }`. The `id` field must be a number; the `value` is the option ID string from the mapping table above.
3. **Verify (first sync per run only):** `projects_get` with `method: get_project_item`, `owner: {board.owner}`, `owner_type: {inferred}`, `project_number: {board.projectNumber}`, `item_id: <item_id>`, `fields: ["{board.statusFieldId}"]`. Confirm the returned status matches the expected option ID. If it matches, skip verification for subsequent syncs in this run. If it does not match, retry step 2 once.

**For PRs:** Use `item_type: pull_request`, `pull_request_number: <N>` in step 1.

**Fallback (rare):** If `add_project_item` does not return an `item_id`, use `projects_list` with `method: list_project_items`, `owner: {board.owner}`, `owner_type: {inferred}`, `project_number: {board.projectNumber}`, `fields: ["{board.statusFieldId}"]`, paginate, and match the item by its issue/PR content to obtain the ID. Then proceed with step 2.

**Resilience:** If any call fails, retry once. If it still fails, surface a warning to the user and continue with the next item. If the `projects` toolset is unavailable, skip sync silently and warn: "Projects v2 sync skipped -- enable the `projects` toolset in your MCP configuration."

---

## Board Overview

Teams can define their own board overview format and dashboard issue. If `meta:board-overview` is included in `board.labels.meta`, board commands will look for an open issue with that label to use as a live dashboard. The format and content of this dashboard is project-specific -- define it in your project's documentation or shared context.

---

## Cross-Cutting Tooling Directives

These directives apply to ALL board commands. They supplement the project's tooling hierarchy.

### GitHub MCP-First

All board commands MUST use GitHub MCP tools as the primary interface for GitHub operations. Key tools used by board commands:

| Operation            | MCP Tool                                            | NOT this               |
| -------------------- | --------------------------------------------------- | ---------------------- |
| List issues          | `list_issues`                                       | `gh issue list`        |
| Read issue details   | `issue_read`                                        | `gh issue view`        |
| Create/update issues | `issue_write`                                       | `gh issue create/edit` |
| Search issues        | `search_issues` / `semantic_issues_search`          | `gh search issues`     |
| Manage sub-issues    | `sub_issue_write`                                   | `gh api`               |
| Add comments         | `add_issue_comment`                                 | `gh issue comment`     |
| Create PRs           | `create_pull_request`                               | `gh pr create`         |
| Read PR details      | `pull_request_read`                                 | `gh pr view`           |
| Manage labels        | `issue_write` (with labels)                         | `gh label`             |
| Projects v2          | `projects_write` / `projects_get` / `projects_list` | `gh project`           |

Fallback to `gh` CLI only for operations outside the MCP catalog (e.g., `gh run` for Actions, `gh release create` with asset uploads).

### Context7 MCP + Web Research

During **board-fill Step 4c** (external research) and **board-pickup Step 6** (implementation):

1. Use **Context7 MCP** (`resolve-library-id` then `query-docs`) whenever an issue references an external library, framework, or SDK. This retrieves current, version-specific documentation to inform issue scoping and implementation.
2. Use **web research** for novel technical challenges, current best practices, security advisories, or breaking changes not covered by Context7 or local docs.
3. Follow the project's tooling hierarchy for knowledge augmentation priority.

---

## Formatting Rules

- Task list: `- [ ] #{number} {short title} *({type tag, }{priority})*`
- Type tag: `Epic` for epics, omitted for standalone.
- Short title: max ~50 chars, strip `[Type]:` prefix.
- Priority: `P0`-`P3` or `--`.
- The board overview issue itself is never listed.

---

## Token-Saving Directives

These apply to all board commands. Follow them to minimize token consumption.

1. **Single board scan.** Perform ONE full board scan per run. Cache all issue data. Reuse for all subsequent steps. Only re-fetch an issue if you mutated it.
2. **Do NOT re-read shared context files** -- their content is available via always-applied rules, this shared context file, or inline in the command.
3. **Single board dashboard regeneration.** If using a board overview, regenerate it ONCE at the end of the run, not after intermediate status changes.
4. **Limit documentation reads.** Read project documentation selectively -- TOC/headers first, full content only for relevant sections.
5. **Do NOT read issue templates.** Required structure is provided inline in the command.
6. **Follow the project's tooling hierarchy** for knowledge augmentation (Context7 MCP, web research).
