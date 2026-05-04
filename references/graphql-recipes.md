# GraphQL Recipes

Use these `gh api graphql` snippets as templates. Prefer variables over string interpolation.

## 1. Check authentication

```bash
gh auth status
```

If the token lacks project permissions:

```bash
gh auth login --scopes "project"
```

## 2. Resolve a project ID

### Organization project

```bash
gh api graphql \
  -f query='query($login: String!, $number: Int!) { organization(login: $login) { projectV2(number: $number) { id title } } }' \
  -f login=ORG \
  -F number=5
```

### User project

```bash
gh api graphql \
  -f query='query($login: String!, $number: Int!) { user(login: $login) { projectV2(number: $number) { id title } } }' \
  -f login=USER \
  -F number=5
```

### List projects for an org

```bash
gh api graphql \
  -f query='query($login: String!) { organization(login: $login) { projectsV2(first: 20) { nodes { id title number } } } }' \
  -f login=ORG
```

## 3. Resolve owner node ID for project creation

```bash
gh api -H "Accept: application/vnd.github+json" /users/GITHUB_OWNER
```

Read the returned `node_id` and use it as `ownerId`.

## 4. Create a project

```bash
gh api graphql \
  -f query='mutation($owner: ID!, $title: String!) { createProjectV2(input: { ownerId: $owner, title: $title }) { projectV2 { id title } } }' \
  -f owner=OWNER_ID \
  -f title='Project Name'
```

## 5. Inspect project fields

This resolves field IDs plus single-select options and iteration IDs.

```bash
gh api graphql \
  -f query='query($project: ID!) { node(id: $project) { ... on ProjectV2 { fields(first: 50) { nodes { ... on ProjectV2Field { id name } ... on ProjectV2SingleSelectField { id name options { id name } } ... on ProjectV2IterationField { id name configuration { iterations { id title startDate duration } } } } } } } }' \
  -f project=PROJECT_ID
```

## 6. List project items and field values

```bash
gh api graphql \
  -f query='query($project: ID!) { node(id: $project) { ... on ProjectV2 { items(first: 50) { nodes { id fieldValues(first: 20) { nodes { ... on ProjectV2ItemFieldTextValue { text field { ... on ProjectV2FieldCommon { name } } } ... on ProjectV2ItemFieldDateValue { date field { ... on ProjectV2FieldCommon { name } } } ... on ProjectV2ItemFieldNumberValue { number field { ... on ProjectV2FieldCommon { name } } } ... on ProjectV2ItemFieldSingleSelectValue { name optionId field { ... on ProjectV2FieldCommon { name } } } ... on ProjectV2ItemFieldIterationValue { title iterationId field { ... on ProjectV2FieldCommon { name } } } } } content { __typename ... on DraftIssue { title body } ... on Issue { id number title url } ... on PullRequest { id number title url } } } } } } }' \
  -f project=PROJECT_ID
```

Prefer this recipe when the response should include browser-openable links. `Issue.url` and `PullRequest.url` are direct GitHub links. Draft issues do not expose an equivalent browser URL through project item content, so report that explicitly.

If the response includes `REDACTED`, the current principal cannot view that item.

## 7. List project items with browser links and repository context

```bash
gh api graphql \
  -f query='query($project: ID!) { node(id: $project) { ... on ProjectV2 { title items(first: 100) { nodes { id content { __typename ... on DraftIssue { title body } ... on Issue { id number title url repository { nameWithOwner } } ... on PullRequest { id number title url repository { nameWithOwner } } } fieldValues(first: 20) { nodes { ... on ProjectV2ItemFieldTextValue { text field { ... on ProjectV2FieldCommon { name } } } ... on ProjectV2ItemFieldDateValue { date field { ... on ProjectV2FieldCommon { name } } } ... on ProjectV2ItemFieldNumberValue { number field { ... on ProjectV2FieldCommon { name } } } ... on ProjectV2ItemFieldSingleSelectValue { name optionId field { ... on ProjectV2FieldCommon { name } } } ... on ProjectV2ItemFieldIterationValue { title iterationId field { ... on ProjectV2FieldCommon { name } } } } } } } } } }' \
  -f project=PROJECT_ID
```

Use the returned `url` for issues and pull requests in user-facing output. For draft issues, note that no direct browser link is available.

When the user asks to groom or improve an item, first ask whether they want `plan mode` or a direct `improve from current description` pass.

- `Improve from current description`: rewrite the existing body into a clearer, more actionable version using the current description as the source.
- `Plan mode`: ask short targeted questions first, then rewrite the item using the Good Card Anatomy format.

Good Card Anatomy:
- Title
- Context (optional but powerful)
- User Story
- Acceptance Criteria
- Technical Notes (optional)
- Dependencies

Recommended `plan mode` questions:
1. Who is the user or actor?
2. What problem are we solving?
3. What outcome should the user get?
4. What does success look like?
5. What constraints, edge cases, or non-goals matter?
6. Are there technical constraints or implementation expectations?
7. Does this depend on another item, service, API, team, or design?

## 8. Resolve an issue or PR content node ID

### Issue

```bash
gh api graphql \
  -f query='query($owner: String!, $repo: String!, $number: Int!) { repository(owner: $owner, name: $repo) { issue(number: $number) { id title url } } }' \
  -f owner=OWNER \
  -f repo=REPO \
  -F number=123
```

### Pull request

```bash
gh api graphql \
  -f query='query($owner: String!, $repo: String!, $number: Int!) { repository(owner: $owner, name: $repo) { pullRequest(number: $number) { id title url } } }' \
  -f owner=OWNER \
  -f repo=REPO \
  -F number=456
```

## 9. Add an issue or PR to a project

```bash
gh api graphql \
  -f query='mutation($project: ID!, $content: ID!) { addProjectV2ItemById(input: { projectId: $project, contentId: $content }) { item { id } } }' \
  -f project=PROJECT_ID \
  -f content=CONTENT_ID
```

Note: if the content is already in the project, GitHub returns the existing item ID.

## 10. Add a draft issue to a project

```bash
gh api graphql \
  -f query='mutation($project: ID!, $title: String!, $body: String!) { addProjectV2DraftIssue(input: { projectId: $project, title: $title, body: $body }) { projectItem { id } } }' \
  -f project=PROJECT_ID \
  -f title='Draft title' \
  -f body='Draft body'
```

## 11. Update project metadata

```bash
gh api graphql \
  -f query='mutation($project: ID!, $title: String!, $public: Boolean!, $readme: String!, $short: String!) { updateProjectV2(input: { projectId: $project, title: $title, public: $public, readme: $readme, shortDescription: $short }) { projectV2 { id title readme shortDescription public } } }' \
  -f project=PROJECT_ID \
  -f title='Project title' \
  -F public=false \
  -f readme='# Project README' \
  -f short='Short description'
```

## 12. Update a text field

```bash
gh api graphql \
  -f query='mutation($project: ID!, $item: ID!, $field: ID!, $value: String!) { updateProjectV2ItemFieldValue(input: { projectId: $project, itemId: $item, fieldId: $field, value: { text: $value } }) { projectV2Item { id } } }' \
  -f project=PROJECT_ID \
  -f item=ITEM_ID \
  -f field=FIELD_ID \
  -f value='Updated text'
```

## 13. Update a number field

```bash
gh api graphql \
  -f query='mutation($project: ID!, $item: ID!, $field: ID!, $value: Float!) { updateProjectV2ItemFieldValue(input: { projectId: $project, itemId: $item, fieldId: $field, value: { number: $value } }) { projectV2Item { id } } }' \
  -f project=PROJECT_ID \
  -f item=ITEM_ID \
  -f field=FIELD_ID \
  -F value=3.5
```

## 14. Update a date field

```bash
gh api graphql \
  -f query='mutation($project: ID!, $item: ID!, $field: ID!, $value: Date!) { updateProjectV2ItemFieldValue(input: { projectId: $project, itemId: $item, fieldId: $field, value: { date: $value } }) { projectV2Item { id } } }' \
  -f project=PROJECT_ID \
  -f item=ITEM_ID \
  -f field=FIELD_ID \
  -f value=2026-05-04
```

## 15. Update a single-select field such as Status

```bash
gh api graphql \
  -f query='mutation($project: ID!, $item: ID!, $field: ID!, $option: String!) { updateProjectV2ItemFieldValue(input: { projectId: $project, itemId: $item, fieldId: $field, value: { singleSelectOptionId: $option } }) { projectV2Item { id } } }' \
  -f project=PROJECT_ID \
  -f item=ITEM_ID \
  -f field=FIELD_ID \
  -f option=OPTION_ID
```

## 16. Update an iteration field

```bash
gh api graphql \
  -f query='mutation($project: ID!, $item: ID!, $field: ID!, $iteration: String!) { updateProjectV2ItemFieldValue(input: { projectId: $project, itemId: $item, fieldId: $field, value: { iterationId: $iteration } }) { projectV2Item { id } } }' \
  -f project=PROJECT_ID \
  -f item=ITEM_ID \
  -f field=FIELD_ID \
  -f iteration=ITERATION_ID
```

## 17. Delete an item from a project

```bash
gh api graphql \
  -f query='mutation($project: ID!, $item: ID!) { deleteProjectV2Item(input: { projectId: $project, itemId: $item }) { deletedItemId } }' \
  -f project=PROJECT_ID \
  -f item=ITEM_ID
```

## 18. Update an issue body for grooming

```bash
gh api graphql \
  -f query='mutation($id: ID!, $body: String!) { updateIssue(input: { id: $id, body: $body }) { issue { id title url body } } }' \
  -f id=ISSUE_ID \
  -f body='Updated issue body'
```

Use this after producing the groomed plan for a repository issue.

## 19. Update a draft issue body for grooming

```bash
gh api graphql \
  -f query='mutation($project: ID!, $item: ID!, $title: String!, $body: String!) { updateProjectV2DraftIssue(input: { projectId: $project, itemId: $item, title: $title, body: $body }) { draftIssue { title body } } }' \
  -f project=PROJECT_ID \
  -f item=ITEM_ID \
  -f title='Draft title' \
  -f body='Updated draft body'
```

Use this when grooming a project-only draft item.

## 20. Mutations you need for issue or PR properties

Do not use `updateProjectV2ItemFieldValue` for these:

- Assignees
- Labels
- Milestone
- Repository

Use GitHub GraphQL issue or pull request mutations instead, such as:

- `addAssigneesToAssignable`
- `removeAssigneesFromAssignable`
- `addLabelsToLabelable`
- `removeLabelsFromLabelable`
- `updateIssue`
- `updatePullRequest`
- `transferIssue`

## 21. Safe execution checklist

Before mutating:

1. Confirm the target owner, project number, and item identity.
2. Resolve project ID, item ID, and field IDs using queries.
3. Query the current field values so the user can see the before state.
4. Apply exactly one focused mutation at a time.
5. Re-query the item after the mutation and report the after state.
