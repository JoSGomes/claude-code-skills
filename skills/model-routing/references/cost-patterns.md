# Token Cost Patterns & Anti-Patterns

## Anti-patterns (expensive, avoid)

### Reading files with large models
```
# WRONG: Opus reading 20 files to summarize them
# RIGHT: Haiku agents in parallel, each reading 1-2 files
```

### Sequential agents when parallel is possible
```
# WRONG: reviewer1 → wait → reviewer2 → wait → reviewer3
# RIGHT: launch all three, wait once, synthesize
```

### Re-reading context that's already in session
```
# WRONG: spawn an agent to "understand the codebase" when you already read it
# RIGHT: pass the relevant summary from your context to the agent prompt
```

### Opus for simple extraction
```
# WRONG: Opus to "check if this PR is a draft"
# RIGHT: Haiku for eligibility checks, gatekeeping
```

### Verbose bash output without RTK
```
# WRONG: go test ./... (full verbose output = many tokens)
# RIGHT: rtk go test ./... (failures only = 90% savings)
```

## Efficient agent prompt patterns

### Give agents what they need, not the whole conversation
```
# WRONG agent prompt: "Based on everything we discussed..."
# RIGHT agent prompt: self-contained with specific context

"Review file src/model.py for data leakage.
Context: this is a sklearn classifier pipeline.
The project uses train/val/test splits.
Return issues in format: [CRITICAL|WARNING] line:N — description"
```

### Constrain agent output format
```
# Agents that return unstructured prose force you to re-read everything
# Agents with structured output (JSON, tables, checklists) are cheaper to parse

"Return a JSON array of issues: [{file, line, severity, description, fix}]"
"Return only files that have issues. One per line."
"Respond YES or NO only, then one sentence explanation."
```

### Haiku for gating, Sonnet/Opus for work
```
Phase 1 (Haiku): Is this task actually needed?
  - Is the PR still open?
  - Does this file contain the pattern we're looking for?
  - Are there any existing tests?

Phase 2 (Sonnet/Opus): Do the actual work.
```

## Cost hierarchy (approximate, relative)

```
Haiku = 1x
Sonnet = ~5x
Opus = ~15x
```

Implications:
- 3 parallel Haiku agents ≈ 0.6x Sonnet
- 5 parallel Haiku agents ≈ Sonnet
- 1 Opus call ≈ 3 Sonnet calls ≈ 15 Haiku calls

## Caching benefits

Prompts with the same prefix benefit from prompt caching (5-min TTL).
- Skills loaded into context stay warm across rapid successive calls
- RTK instructions in CLAUDE.md are cached at session start
- Long system prompts amortized across the session

Keep frequently-used reference files stable (don't regenerate them each call).
