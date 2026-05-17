# Resource Admin (`/admin_resource`)

You were invoked via the **`/admin_resource`** slash command.

Route to the **`collector` agent** (`.claude/agents/collector.md`). The collector agent owns all artifact library operations — ingestion, cataloging, versioning, and search.

## Usage

Parse the sub-command and arguments from: `$ARGUMENTS`

If no sub-command is given, **default to `ingest`**.

## Sub-commands

| Sub-command | Collector skill invoked | Action |
|---|---|---|
| `ingest <url>` | `collect/ingest-confluence` or `collect/ingest-jira` | Fetch and ingest a URL via MCP |
| `ingest` (no URL) | `collect/ingest-adhoc` | Ingest pasted content from conversation |
| `search <query>` | `collect/search` | Search artifact library; returns ranked results |
| `status` | — | Read `artifacts/CATALOG.md` and summarize counts and recent artifacts |
| `changelog` | — | Read `artifacts/CHANGELOG.md` and summarize recent changes |
| `supersede <old_id> <new_id>` | `collect/version-supersede` | Mark old artifact as superseded by new |
| `rebuild` | `collect/catalog-index` | Rebuild full CATALOG.md by re-reading all artifact front-matter |
| `update` | `collect/ingest-confluence` or `collect/ingest-jira` | Re-check upstream sources for changes |

## Routing Logic

The collector agent follows this routing for ingest:

```
URL provided?
  YES → contains "atlassian.net" or "confluence"? → collect/ingest-confluence
      → contains "atlassian.net" and "browse/" or "issues/"? → collect/ingest-jira
      → other URL → ask user: "Is this a Confluence page, Jira issue, or other source?"
  NO  → content pasted in conversation? → collect/ingest-adhoc
      → no URL, no content → ask user to provide URL or paste content
```

After every ingest:
1. Run `collect/catalog-index` to update CATALOG.md and CHANGELOG.md
2. Check if existing artifacts are superseded by new content; if so, run `collect/version-supersede`

## Source Registry

Before fetching from a URL source, the collector agent:
1. Reads `artifacts/.source-registry.md` to check if the MCP server is registered
2. If a new MCP appears, proposes an entry and waits for user confirmation before adding to registry

## Artifact ID Sequencing

artifact_ids follow the pattern `ART-YYYYMMDD-NNN` where NNN is a zero-padded 3-digit sequence number, incrementing from the highest NNN used today. If no artifacts exist for today's date, start at 001.

To determine next ID: read CATALOG.md, find the highest artifact_id for today's date, add 1.

## Update Sub-command

Re-fetch a previously ingested source to check for changes.

| Invocation | Behaviour |
|---|---|
| `update` | Check all `confluence` and `jira` artifacts for upstream changes (no writes) |
| `update <url>` | Check only artifacts matching this source URL |
| `update --supersede ART-YYYYMMDD-NNN` | Re-ingest source, create new artifact, supersede old one |

Non-updateable sources (`adhoc`) are skipped.

## Output

After any operation, report:
- What was done (artifacts written, catalog updated, supersessions recorded)
- File paths of all written/modified files
- Total artifact count in the library
- Any open questions or missing source descriptions that need user input

## Integration with Other Agents

Planner and generator agents call `/admin_resource search` to retrieve project context before planning. The collector agent's `artifacts/CATALOG.md` is the single source of truth for what knowledge has been ingested.

Keep the user informed at each step: discovery → registry confirmation → ingest → catalog update → any supersession.
