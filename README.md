# GitHub Projects Skill

> GitHub Projects v2 workflow for OpenCode, powered by `gh` + GraphQL.

Use this skill to inspect, organize, and update GitHub Projects items without leaving the terminal workflow.

## At a Glance

| Area | What it does |
| --- | --- |
| Project visibility | Lists projects for users and organizations |
| Item inspection | Shows items, field values, and browser links |
| Item management | Adds issues and PRs, creates draft items, updates fields |
| Project updates | Changes metadata like title, visibility, README, and description |
| Grooming | Expands lightweight cards into actionable work items |

## Capabilities

- List projects for a user or organization
- Show project items and field values
- Include browser links when listing or showing items
- Add issues or pull requests to a project
- Create project draft items
- Move items by updating fields such as `Status`
- Update project metadata
- Groom item descriptions

## Requirements

Before using the skill:

- `gh` must be authenticated
- token scope should include `project` for mutations
- `read:project` is enough for read-only requests

Check auth with:

```bash
gh auth status
```

## Default Workflow

The skill follows a small, safe mutation flow:

1. Resolve the target owner and project.
2. Resolve item IDs, field IDs, and content IDs as needed.
3. Query current state before making changes.
4. Apply the smallest correct mutation.
5. Re-query and report the final state.

## Item Output Rules

When listing or showing items, the response should always include a browser-openable link.

| Item type | Link behavior |
| --- | --- |
| Issue | Use the GitHub `url` |
| Pull request | Use the GitHub `url` |
| Draft issue | State clearly when no direct browser link exists |

## Grooming Workflow

The grooming flow should activate for requests like:

- `groom`
- `improve`
- `refine`
- `expand`
- `flesh out`
- `clarify`
- `detail`
- `make this more actionable`
- `break this down`

### First Decision

When grooming is requested, ask the user to choose one path first:

1. `Plan mode`
2. `Improve from current description`

### Path A: Improve From Current Description

Use the existing body or description as the source material and rewrite it into a clearer, more actionable version.

### Path B: Plan Mode

Ask short, targeted questions first, then rewrite the item using the required card structure.

#### Good Card Anatomy

| Section | Notes |
| --- | --- |
| `Title` | Clear and specific |
| `Context` | Optional, but strongly encouraged |
| `User Story` | Who, what, and why |
| `Acceptance Criteria` | Observable success conditions |
| `Technical Notes` | Optional implementation guidance |
| `Dependencies` | Related items, APIs, teams, or blockers |

#### Preferred Plan-Mode Questions

- Who is the user or actor?
- What problem are we solving?
- What outcome should the user get?
- What does success look like?
- What constraints, edge cases, or non-goals matter?
- Are there technical constraints or implementation expectations?
- Does this depend on another item, service, API, team, or design?

## Example Requests

```text
list all items from project Buteco Games
show item #6 from Buteco Games
move item #6 in Buteco Games to In Progress
add Funnie-Tech/butecogames#12 to Buteco Games
create a draft item in Buteco Games called Improve onboarding flow
improve item #6 in Buteco Games
```

## Example Grooming Flow

```text
User: improve item #6 in Buteco Games
Skill: Plan mode or improve from current description?
User: Plan mode
Skill: asks targeted questions
Skill: rewrites the item using Good Card Anatomy
Skill: updates the item and returns the browser link
```

## Repository Files

| File | Purpose |
| --- | --- |
| `SKILL.md` | Main instructions and workflow rules |
| `references/graphql-recipes.md` | Reusable GraphQL query and mutation examples |

## Notes

> Prefer the smallest correct mutation.
>
> Query before mutating, and re-query after mutating.
