# MCP Story Fetching Guide

This reference describes how to fetch stories from TrackerBoot via MCP tools.

Story input supports two modes:
1. **TrackerBoot story ID** — fetch via MCP
2. **Pasted story content** — use directly, no MCP needed

If neither is provided, the session does not start.

---

## Discovery Strategy

Do NOT hard-code specific MCP tool names. Instead:

1. Review the list of available tools in the current session
2. Look for tools matching TrackerBoot patterns
3. If none found, report the error and stop

---

## TrackerBoot MCP Patterns

### Tool Name Variants (try in order)

```
mcp__trackerboot__get_story
mcp__trackerboot__fetch_story
mcp__trackerboot__story
mcp__pivotal__get_story
mcp__tracker__get_story
```

### Typical Call

```json
{
  "id": "12345678"
}
```

or

```json
{
  "story_id": "12345678"
}
```

Strip a leading `#` if present before sending (`#12345678` → `12345678`).

### Response Field Extraction

| Target field | TrackerBoot field | Notes |
|-------------|------------------|-------|
| Title | `name` | Always present |
| Description | `description` | Plain text |
| Status | `current_state` | `unstarted`, `started`, `finished`, etc. |
| Story Type | `story_type` | `feature`, `bug`, `chore` |
| Estimate | `estimate` | Story points — optional |
| Labels | `labels[].name` | Tags — optional |
| Acceptance Criteria | `tasks[]` or embedded in `description` | Check both |

### TrackerBoot Tasks vs Acceptance Criteria

TrackerBoot has a `tasks` array for checklist items. Use these as acceptance criteria when present:

```json
{
  "tasks": [
    { "description": "User can update name", "complete": false },
    { "description": "User can update email", "complete": false }
  ]
}
```

If `tasks` is empty or absent, parse the `description` field for an "Acceptance Criteria:" or "AC:" section.

---

## Failure Handling

### MCP tool not found

```
"Could not find a TrackerBoot MCP tool in the current session.

Please check that the TrackerBoot MCP server is configured, then try again,
or paste the story content directly."
```

**STOP — do not proceed to Phase 2.**

### MCP call returned an error

```
"Could not fetch story [ID] from TrackerBoot: [error message].

Please verify the story ID and TrackerBoot configuration, then try again,
or paste the story content directly."
```

**STOP — do not proceed to Phase 2.**

### Never fail silently

Always explain why the fetch failed. Never guess or invent story content.

---

## Parsed Story Display Format

After a successful fetch, display the story and PAUSE for confirmation:

```markdown
## Story

**ID:** 12345678
**Title:** User Profile Update

**Description:**
A logged-in user should be able to update their name and email address.
Changes take effect immediately, and a success message is shown on save.

**Acceptance Criteria:**
- [ ] User can update their name
- [ ] User can update their email
- [ ] An error message is shown when the email format is invalid
- [ ] A confirmation message "Profile updated successfully" is shown on save

**Status:** started

---
Does this look correct? Type "ok" to continue, or let me know what to fix.
```

---

## Story ID Format Recognition

Recognize these as TrackerBoot story IDs (trigger MCP fetch):
- `12345678` — numeric ID
- `#12345678` — numeric ID with hash prefix

Recognize these as direct story content (skip Phase 1, parse and display directly):
- Long text paragraphs
- "As a user..." user story format
- Multiline text with acceptance criteria listed

Anything else (e.g. short alphanumeric codes like `PROJ-123`) is not a supported format — display the error and stop.
