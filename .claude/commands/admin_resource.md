# Resource Admin (`/admin_resource`)

You were invoked via the **`/admin_resource`** slash command.

Follow `rules/resource-admin-agent.md`. Act as **`resource-orchestrator`** and run the full pipeline (ingest → catalog → versioner when needed).

## Usage

Parse the sub-command and arguments from: `$ARGUMENTS`

If no sub-command is given, **default to `ingest`**.

## Sub-commands

| Sub-command | Action |
|---|---|
| `ingest` | Ingest URL or pasted content; MCP discovery + `artifacts/.source-registry.md` before fetch |
| `search` | Natural-language search over the artifact library |
| `status` | Summarize `artifacts/CATALOG.md` |
| `changelog` | Summarize `artifacts/CHANGELOG.md` |
| `rebuild` | Full rebuild of `artifacts/CATALOG.md` via `resource-cataloger` |
| `rollup` | Current-state roll-up to `artifacts/ROLLUP-current-state.md` via `resource-versioner` |
| `update` | Check versioned sources (Confluence, Jira) for updates; see flags below |

### Update sub-command

Check upstream sources for changes and optionally create superseding artifacts.

**Flags:**

| Flag | Behaviour |
|------|-----------|
| (none) or `--check` | Report which artifacts have upstream changes; no writes except `source_last_checked` |
| `--supersede <artifact_id>` | Re-ingest source, create new artifact, mark old as superseded |

**Scope:**

| Invocation | Scope |
|------------|-------|
| `update` | Check all updateable artifacts (`confluence`, `jira`) |
| `update <source-ref>` | Check only artifacts matching the URL or ticket ID |
| `update --supersede ART-YYYYMMDD-NNN` | Supersede the specified artifact |

**Examples:**

```
/admin_resource update                                    # check all
/admin_resource update --check                            # same as above
/admin_resource update https://...atlassian.net/.../123   # check specific source
/admin_resource update --supersede ART-20260325-011       # supersede specific artifact
```

**Non-updateable sources:** `ad-hoc`, `teams-chat`, `meeting-transcript` are skipped.

## Execution flows

### Ingest flow

1. Act as `resource-orchestrator` — read `artifacts/.source-registry.md`, scan `mcps/` for available MCP servers.
2. If a new MCP appears vs. the registry, propose an entry and wait for user confirmation before adding.
3. Match URL/content to `source_type` and `mcp_server`; fall back to `ad-hoc` when no MCP applies.
4. Dispatch to `resource-ingestor` with source URL or text, `source_type`, MCP server name, and next `artifact_id` sequence.
5. `resource-cataloger` runs after ingestion to update `CATALOG.md`, detect relationships, and flag duplicates.
6. `resource-versioner` runs if cataloger detects supersession.
7. Present summary: topics extracted, file paths, any detected relationships or supersessions.

### Search flow

1. Dispatch to `resource-search`.
2. Return ranked results with file paths, summaries, and relevance explanations.

### Status flow

1. Read `artifacts/CATALOG.md` and summarize counts, recent sessions, and gaps.

### Changelog flow

1. Read `artifacts/CHANGELOG.md` and present a concise summary.

### Rebuild flow

1. Invoke `resource-cataloger` to rebuild the full catalog from scratch.
2. Report artifact count and flag any files with missing or malformed front-matter.

### Rollup flow

1. Invoke `resource-versioner` to produce a current-state roll-up (only `status: current` artifacts).
2. Written to `artifacts/ROLLUP-current-state.md` (overwritten each run).

## Output structure

All artifacts follow `rules/resource-admin-output-structure.md` — source-type subfolders under `artifacts/` with topic-based decomposition and structured YAML front-matter.

## Integration with other agent groups

BI agents and Project Intelligence agents can call `resource-search` to gather project context. The `bi-orchestrator` checks `artifacts/CATALOG.md` for relevant context when beginning a new project.

Keep the user informed at each step (discovery, registry confirmation, ingest, catalog, versioning) and what happens next.
