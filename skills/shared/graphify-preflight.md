# Shared Pre-flight: Graphify / code-review-graph Detection

Canonical graphify detection block. Skills reference this file instead of
duplicating detection logic.

---

## Detection

Check for graphify availability in this order:

### 1. MCP tools available in session

Check if `mcp__code-review-graph__query_graph` or equivalent tools are listed
in the current session's deferred tools or available tools. If yes:

- **Mode: `graph-available`** — use MCP tools directly
- Tools: `query_graph`, `get_node`, `get_neighbors`, `get_community`,
  `god_nodes`, `graph_stats`, `shortest_path`

### 2. Graphify installed but MCP not configured

```bash
python -c "import graphify; print('installed')" 2>/dev/null || echo "not installed"
```

If installed but MCP tools are not in the session:

- **Mode: `graphify-installed`** — MCP not available, but can be set up
- Suggest: "Graphify is installed. Run `/graphify . --mcp` to build the
  graph, then add to MCP config for graph-accelerated analysis."

### 3. Nothing available

- **Mode: `standard`** — proceed with file reads and grep
- No suggestion needed

## Query-First Principle (when `graph-available`)

For structural questions (architecture, connections, dependencies, schemas,
"how is X related to Y"):

1. **Query the graph first** — one MCP call replaces 5-10 file reads
2. **Read files to confirm** — graph gives structure, files give depth
3. **Never do both in full** — graph queries replace grep chains, not
   add to them (rule E3a)

Budget: defer to skill-specific limits. Default per E3a: max 3 queries
per skill invocation unless the skill specifies otherwise.
