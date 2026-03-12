---
name: context-first-lookup
description: Use when encountering anything unknown, unfamiliar, or missing — before grepping, catting, listing directories, or running any exploratory shell command. Triggers on symptoms like "I don't know where this is", "let me search for", "let me check", "I need to find". Enforces structured lookup hierarchy over random exploration.
---

# Context-First Lookup

## Overview

**Never produce junk output hunting for information.** When you encounter something unknown, follow the lookup hierarchy: local context first, then semantic tools, then indexes, then — only as a last resort — targeted file search. If everything fails, propose creating an index so the question never requires exploration again.

**Core principle:** Every lookup should either find the answer immediately or improve the system so the next lookup will.

## Lookup Hierarchy

```dot
digraph lookup {
    rankdir=TB;
    node [shape=box];

    unknown [label="Unknown reference\nencountered" shape=doublecircle];
    local [label="1. Local Context\nCLAUDE.md, memory files,\nloaded skills, conversation"];
    tools [label="2. Semantic Tools\nstatement-mcp retrieve,\nMCP tool discovery"];
    indexes [label="3. Indexed Collections\nstatement-mcp collections,\ningested documents"];
    structured [label="4. Structured Search\nGlob with known patterns,\ntargeted Grep on known paths"];
    report [label="5. Report & Plan\nExhaust report → propose\nindex creation"];

    found [label="Answer found" shape=doublecircle];

    unknown -> local;
    local -> found [label="found"];
    local -> tools [label="not found"];
    tools -> found [label="found"];
    tools -> indexes [label="not found"];
    indexes -> found [label="found"];
    indexes -> structured [label="not found"];
    structured -> found [label="found"];
    structured -> report [label="not found"];
}
```

## Level 1: Local Context

Check what's already loaded or immediately available:

| Source | How to check |
| ------ | ------------ |
| CLAUDE.md (global + project) | Already in context — re-read it |
| Memory files | Read MEMORY.md index, then relevant files |
| Loaded skills | Already in context — re-read them |
| Conversation history | Scroll up / recall from current session |
| Plan files | Check `docs/plans/` if a plan is active |

**No tool calls needed.** This is a mental check of what you already have.

## Level 2: Semantic Tools

Use tools designed for knowledge retrieval:

```
mcp__statement-mcp__retrieve
  query: "the thing you're looking for"
  project: "current-project"        # if known
  content_type: "all"               # or "knowledge", "document", "chat"
  mode: "semantic"                  # default hybrid search
```

Also check:
- `mcp__statement-mcp__status` — see what's indexed for this project
- `mcp__memory__search_nodes` — knowledge graph entries
- Other MCP tools that might have relevant data

**Key:** Semantic search finds conceptually related content even with different wording. Always try 2-3 query phrasings before concluding nothing exists.

## Level 3: Indexed Collections

Check if the content is in an ingested collection:

```
mcp__statement-mcp__retrieve
  query: "your search"
  content_type: "document"
  tags: { "collection": "project-name" }
```

If the project should have a collection but doesn't, **stop and ingest it first:**

```
mcp__statement-mcp__ingest_collection
  path: "/absolute/path/to/project"
  collection: "project-name"
  glob_pattern: "**/*.md"    # or appropriate pattern
```

Then search the newly created index. This is not wasted time — it permanently solves the lookup problem.

## Level 4: Structured Search (Last Code-Level Resort)

Only reach this level after exhausting 1-3. When you do:

- **Glob** for specific file patterns you expect to exist (not `**/*`)
- **Grep** for exact strings, function names, or error messages
- **Read** specific files you have reason to believe contain the answer

**Rules:**
- Never `ls` a directory to "see what's there"
- Never `grep -r` across an entire codebase speculatively
- Never `cat` a file to "take a look"
- Every search must have a specific hypothesis: "I believe X is defined in files matching Y"

## Level 5: Report & Plan

If all levels fail, **do not keep fishing.** Instead:

1. **Report** what you tried at each level and why it didn't work
2. **Propose** an index creation plan so this lookup succeeds next time:
   - What content should be ingested?
   - What collection name and glob pattern?
   - What tags/context should be added?
   - Should a memory file be created?
3. **Ask** the user whether to create the index now or proceed differently

This turns every failed lookup into a system improvement.

## Red Flags — STOP and Use the Hierarchy

| Thought | Reality |
| ------- | ------- |
| "Let me just quickly grep for it" | Grep produces junk. Check indexes first. |
| "I'll cat a few files to see" | Reading random files is not a strategy. Check context. |
| "Let me ls the directory" | You don't need a listing. You need the answer. Use semantic search. |
| "I'll search the codebase" | Search the INDEX. The codebase is not a search engine. |
| "Let me explore a bit" | Exploration without structure wastes context. Follow the hierarchy. |
| "This will be faster than the index" | It won't. And it won't help next time either. |
| "The index probably doesn't have this" | Check. Don't assume. `status` takes 1 second. |
| "I know roughly where this is" | If you know exactly, go there. If roughly, use semantic search. |

## Common Mistakes

| Mistake | Fix |
| ------- | --- |
| Grepping before checking statement-mcp | Always `retrieve` first, even if you think grep would be faster |
| Using `grep -r . -l` across entire repo | Use Glob with specific pattern or semantic search |
| Reading 10 files "to understand the codebase" | Read the index, the CLAUDE.md, the memory. Then read specific files. |
| Not trying multiple query phrasings | Semantic search is fuzzy — rephrase 2-3 times before giving up |
| Skipping Level 5 when stuck | Every failed lookup should create an index. Don't just move on. |
| Ingesting but not adding context | Always `qmd context add` or add appropriate tags after ingesting |

## The Bottom Line

**Grep is not a knowledge system. Indexes are.**

Every time you grep instead of querying an index, you:
1. Produce noisy output that wastes context
2. Get results without semantic understanding
3. Leave the system no better for next time
4. Train yourself (and the user) to accept slow, unreliable lookups

Follow the hierarchy. Build indexes. Make every lookup improve the system.
