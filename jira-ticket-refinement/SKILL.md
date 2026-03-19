---
name: jira-ticket-refinement
description: Use when the user asks to refine, update, or write a description for a Jira ticket, mentions a ticket key like CTRL-XXX, or wants to add acceptance criteria to a Jira issue
---

# Jira Ticket Refinement

Workflow for refining Jira tickets in the factorial-io Atlassian instance via MCP tools.

## Cloud ID

`2c995649-395b-412d-80de-2f3dab5d35c8` for `factorial-io.atlassian.net`

If this stops working, re-discover with `getAccessibleAtlassianResources`.

## Quick Reference

| Action | Tool | Key params |
|--------|------|------------|
| Fetch ticket | `getJiraIssue` | `responseContentFormat: "markdown"` |
| Update ticket | `editJiraIssue` | `contentFormat: "markdown"` for description |
| Field metadata | `getJiraIssueTypeMetaWithFields` | `projectIdOrKey: "CTRL"` |
| Smart Checklist | `editJiraIssue` | `customfield_13523` with ADF-wrapped markdown |

## Workflow

1. **Fetch** the ticket with `getJiraIssue` (markdown format)
2. **Investigate** the codebase if description is empty or needs context
3. **Draft** description with: Context, Requirements, Technical Notes sections
4. **Update** description via `editJiraIssue` with `contentFormat: "markdown"`
5. **Set Smart Checklist** for acceptance criteria (see format below)

## Smart Checklist (customfield_13523)

The API requires ADF format, but Smart Checklist parses its own markdown syntax inside a single ADF paragraph.

**Format that works:**

```json
{
  "customfield_13523": {
    "version": 1,
    "type": "doc",
    "content": [{
      "type": "paragraph",
      "content": [{
        "type": "text",
        "text": "# Acceptance Criteria\n- First item\n- Second item\n- Third item"
      }]
    }]
  }
}
```

- `#` for section headings
- `- ` for checklist items (Smart Checklist adds its own checkboxes)

**Do NOT use:**
- ADF `taskList`/`taskItem` nodes — renders as bullet points, not checklist items
- `- []` syntax — shows literal `[]` text next to the checkbox
- Plain text strings — API rejects anything that isn't ADF for this field

## Description Template

```markdown
## Context
[Why this change is needed, current behavior, what's missing]

## Requirements
### [Primary requirement]
[Details, decision tables, severity levels as needed]

### Frontend / Backend
[Implementation specifics per layer]

## Technical Notes
- **Existing pattern**: [file path and what to reuse]
- **Key files**: [paths to modify]
```
