---
name: github-projects
description: Manage GitHub Projects v2 with GraphQL through the GitHub CLI. Use when the user wants to create or update projects, move cards, add issues or PRs, create draft items, inspect fields, set status, iterations, dates, or other project metadata.
compatibility: opencode
metadata:
  workflow: github-projects-v2
  transport: gh-graphql
---

# GitHub Projects

Use this skill for GitHub Projects v2 work involving project boards, items, fields, draft issues, and project automation through GraphQL.

## Default behavior

Treat project changes as side-effecting operations.

1. Parse the user's request into the exact project operation needed.
2. Verify GitHub CLI authentication before making changes.
3. Resolve all required node IDs first.
4. Query current state before mutating anything.
5. If the request is ambiguous or could affect multiple items, summarize the intended mutation briefly before acting.
6. Execute the smallest correct mutation.
7. Query the result afterward and report the updated state.

If the user only wants information, stop after the read-only queries.

When listing or showing project items, always include a browser-openable link for each item. Prefer the underlying issue or pull request URL when the item is backed by repository content. If the item is a draft issue and no direct browser URL exists, say that explicitly instead of omitting the link information.

When the user asks to groom an item, or uses equivalent wording such as improve, refine, expand, flesh out, clarify, detail, or break down an item description, do not immediately rewrite it. First ask whether they want `plan mode` or a direct `improve from current description` pass.

- If they choose `improve from current description`, use the current item description or body as the source material for a short planning pass and write back a clearer, more actionable version unless the user asks for a preview only.
- If they choose `plan mode`, ask targeted questions first, then rewrite the item using the required Good Card Anatomy format below.

Good Card Anatomy must contain:
- Title
- Context (optional but powerful)
- User Story
- Acceptance Criteria
- Technical Notes (optional)
- Dependencies

## Authentication

Prefer `gh` over raw `curl` unless the user explicitly asks otherwise.

Check auth with:

```bash
gh auth status
```

If auth is missing or the token scope is insufficient:

```bash
gh auth login --scopes "project"
```

Use `read:project` only for read-only requests.

## Preferred workflow

1. Identify whether the project owner is a user or organization.
2. Resolve the project ID from owner login and project number.
3. Resolve field IDs and, when needed, single-select option IDs or iteration IDs.
4. Resolve the content ID or existing project item ID.
5. Perform the mutation.
6. Re-query the item or field values to confirm final state.

## Important constraints

- GitHub Projects v2 uses GraphQL.
- You cannot add an item and update its project field values in the same mutation. Add it first, then update it.
- `updateProjectV2ItemFieldValue` works for project field values such as text, number, date, single select, and iteration.
- It does not update issue or pull request properties like assignees, labels, milestone, or repository. Use issue or pull request mutations for those.
- If adding an issue or PR that already exists in the project, GitHub returns the existing item ID.
- Some items may be `REDACTED` if the authenticated principal cannot view them.

## Query and mutation patterns

Use `gh api graphql -f query='...'` and pass variables with `-f` or `-F`.

Example project lookup:

```bash
gh api graphql -f query='query($login: String!, $number: Int!) { organization(login: $login) { projectV2(number: $number) { id title } } }' -f login=ORG -F number=5
```

Example add-item mutation:

```bash
gh api graphql -f query='mutation($project: ID!, $content: ID!) { addProjectV2ItemById(input: { projectId: $project, contentId: $content }) { item { id } } }' -f project=PROJECT_ID -f content=CONTENT_ID
```

For reusable recipes, read `references/graphql-recipes.md`.

## Common requests

### Move a card or change status

1. Find the project ID.
2. Find the item ID.
3. Find the `Status` field ID and target option ID.
4. Run `updateProjectV2ItemFieldValue`.

### Add an issue or PR to a project

1. Resolve the issue or pull request content node ID.
2. Run `addProjectV2ItemById`.
3. If needed, update status, iteration, or dates in separate mutations.

### Create a task or card

1. If the user wants a project-only card, create a draft issue with `addProjectV2DraftIssue`.
2. If the user wants a repository issue, create the issue first, then add it to the project.

### List cards or show project state

1. Query project items.
2. Include content type, title, assignees when available, key field values, and a browser-openable link for each item.
3. Call out missing permissions or redacted items explicitly.

### Create a new project

1. Resolve the user or organization owner node ID.
2. Run `createProjectV2`.
3. If requested, follow up with `updateProjectV2` for title, visibility, README, or short description.

### Groom an item

Trigger this workflow not only for the word `groom`, but also for nearby intent such as `improve`, `refine`, `expand`, `flesh out`, `clarify`, `detail`, `make this more actionable`, or `break this down`.

1. Resolve the project, item, and underlying content type.
2. Query the current title and body or description.
3. Ask the user to choose between `plan mode` and `improve from current description`.
4. If the user chooses `improve from current description`, preserve the original intent while improving clarity. Prefer adding scope, acceptance criteria, assumptions, dependencies, and implementation notes over changing the request.
5. If the user chooses `plan mode`, ask targeted questions needed to produce a strong card. Keep the questions short and focused on missing details.
6. In `plan mode`, gather enough detail to produce the following Good Card Anatomy:
   Title
   Context (optional but powerful)
   User Story
   Acceptance Criteria
   Technical Notes (optional)
   Dependencies
7. Preferred `plan mode` questions should cover, when missing:
   Who is the user or actor?
   What problem are we solving?
   What outcome should the user get?
   What does success look like?
   What constraints, edge cases, or non-goals matter?
   Are there technical constraints or implementation expectations?
   Does this depend on another item, service, API, team, or design?
8. After the user answers, rewrite the item into the Good Card Anatomy format.
9. If the content is an issue or pull request, update the body on the underlying GitHub item.
10. If the content is a draft issue, update the draft issue body in the project.
11. Re-query the item and report the updated body plus the browser link when available.

## Response expectations

Be explicit about the IDs and names you resolved when they matter for follow-up work.

When responding with item lists or item details, include the browser link for every item when available.

After each mutation, report:

- Project title and number if known
- Item title or draft title
- Item ID or project item ID returned by GitHub
- Field changes applied
- Body or description changes applied, when grooming or editing details
- Whether the grooming path used was `plan mode` or `improve from current description`
- Any limitations, such as issue fields that require separate mutations

## Additional resources

- Read `references/graphql-recipes.md` for ready-to-use GraphQL snippets and lookup sequences.
