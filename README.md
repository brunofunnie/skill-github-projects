# github-projects

GitHub Projects v2 skill for OpenCode.

This skill uses the GitHub CLI and GraphQL to inspect and update GitHub Projects items, fields, and metadata.

## What it covers

- List projects for a user or organization
- Show project items and field values
- Include browser links when listing or showing items
- Add issues or pull requests to a project
- Create project draft items
- Move items by updating fields such as `Status`
- Update project metadata
- Groom item descriptions

## Requirements

- `gh` must be authenticated
- Token scope should include `project` for mutations
- `read:project` is enough for read-only requests

Check auth with:

```bash
gh auth status
```

## Main behavior

The skill follows a small, safe workflow:

1. Resolve the target owner and project.
2. Resolve item IDs, field IDs, and content IDs as needed.
3. Query current state before making changes.
4. Apply the smallest correct mutation.
5. Re-query and report the final state.

## Item links

When listing or showing items, the skill should include a browser-openable link for each item.

- For issues and pull requests, use the GitHub `url`.
- For draft issues, report when no direct browser link is available.

## Grooming workflow

The skill supports grooming requests triggered by words such as:

- `groom`
- `improve`
- `refine`
- `expand`
- `flesh out`
- `clarify`
- `detail`
- `make this more actionable`
- `break this down`

When grooming is requested, the skill must first ask which path to use:

1. `plan mode`
2. `improve from current description`

### Improve from current description

Use the existing body or description as source material and rewrite it into a clearer, more actionable version.

### Plan mode

Ask short, targeted questions first, then rewrite the item using this Good Card Anatomy:

- `Title`
- `Context` (optional but powerful)
- `User Story`
- `Acceptance Criteria`
- `Technical Notes` (optional)
- `Dependencies`

Preferred planning questions:

- Who is the user or actor?
- What problem are we solving?
- What outcome should the user get?
- What does success look like?
- What constraints, edge cases, or non-goals matter?
- Are there technical constraints or implementation expectations?
- Does this depend on another item, service, API, team, or design?

## Files

- `SKILL.md`: main instructions and workflow rules
- `references/graphql-recipes.md`: reusable GraphQL query and mutation examples

## Example requests

- `list all items from project Buteco Games`
- `show item #6 from Buteco Games`
- `move item #6 in Buteco Games to In Progress`
- `add Funnie-Tech/butecogames#12 to Buteco Games`
- `create a draft item in Buteco Games called Improve onboarding flow`
- `improve item #6 in Buteco Games`

Example grooming flow:

1. User: `improve item #6 in Buteco Games`
2. Skill: asks whether to use `plan mode` or `improve from current description`
3. If `plan mode` is selected, the skill asks targeted questions
4. The skill rewrites the card using Good Card Anatomy
5. The skill updates the item and returns the browser link
