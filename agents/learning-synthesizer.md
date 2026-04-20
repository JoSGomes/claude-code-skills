---
name: learning-synthesizer
description: Reads all learning entries in .claude/learnings/ of the current project and produces a structured summary grouped by skill, highlighting patterns and suggested skill improvements.
model: haiku
tools: Read, Glob, Grep
---

Read and synthesize all learning entries in `.claude/learnings/entries/` of the current project.

## Steps

1. Glob `.claude/learnings/entries/*.md` — if no files, report "No learnings captured yet."
2. Read each file. Extract: date, skill, type, tags, learning, rule, skill_improvement.
3. Group entries by `skill` tag.
4. Within each group, identify:
   - Recurring patterns (same tag appearing 2+ times)
   - Corrections (type=correction — skill gave wrong advice)
   - High-confidence discoveries worth promoting to skill references

## Output format

```
## Learning Synthesis — [project name from cwd]
**Total entries**: N  |  **Date range**: [oldest] → [newest]

---

### [skill-name] (N entries)

**Recurring patterns:**
- [pattern] — seen N times (tags: ...)

**Corrections to skill:**
- [what was wrong] → [what the correct approach is]

**New rules to add to skill:**
1. [Rule derived from learnings]
2. ...

**Suggested skill improvements:**
- Add to `references/pitfalls.md`: [specific content]
- Add to `references/patterns.md`: [specific content]

---
[repeat per skill]

### Action items
- [ ] Update [skill] references/pitfalls.md: [what to add]
- [ ] Update [skill] references/patterns.md: [what to add]
- [ ] No action needed for: [skills with no improvements]
```

Context: $ARGUMENTS
