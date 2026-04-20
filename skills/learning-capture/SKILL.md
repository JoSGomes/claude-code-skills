---
name: learning-capture
description: This skill should be used when Claude discovers a bug pattern, anti-pattern, effective approach, or correction to an existing skill during work on a project. Activates when finding something noteworthy in code review, debugging, architecture, research validation, or any workflow. Guides when and how to capture learnings to .claude/learnings/ in the project.
version: 0.1.0
---

# Learning Capture Skill

Context: documenting insights discovered during work so they can improve the skills over time.

## When to capture a learning

Capture when you encounter any of the following:

| Trigger | Type | Example |
|---------|------|---------|
| Found a bug pattern not in the skill's pitfalls | `pitfall` | Data leakage via preprocessor fitted on all data |
| An approach worked particularly well | `pattern` | Using errgroup for bounded parallelism |
| Skill gave incorrect or incomplete advice | `correction` | Skill said X, but Y is correct here |
| Project-specific convention discovered | `project-convention` | This project uses X instead of Y |
| New technique or library worth noting | `discovery` | Found a cleaner way to do Z |

**Do NOT capture:**
- Things already in the skill references (check first)
- Trivial style issues
- Project-specific bugs with no generalizable lesson

## How to capture

Use `/capture-learning [brief description]` at the end of a relevant task, or inline when the discovery is fresh.

## Storage location

All learnings go to `.claude/learnings/` **in the project being worked on** — not in the plugin repo.

```
project-root/
└── .claude/
    └── learnings/
        ├── LEARNINGS.md        ← index (newest first)
        └── entries/
            ├── 2026-04-20_python-dev_data-leakage.md
            └── 2026-04-20_golang-dev_goroutine-leak.md
```

## Entry format

Each entry in `entries/YYYY-MM-DD_[skill]_[slug].md`:

```markdown
---
date: YYYY-MM-DD
skill: [python-dev|golang-dev|project-spec|model-routing|general]
type: [pitfall|pattern|correction|discovery|project-convention]
confidence: [high|medium]
tags: [tag1, tag2]
---

## Contexto
O que estava sendo feito quando este aprendizado foi descoberto.

## Aprendizado
O insight específico — concreto, com código se relevante.

## Regra derivada
A regra generalizável em uma frase.

## Melhoria de skill sugerida
O que deveria ser adicionado/modificado na skill — ou "nenhuma".
```

## Reviewing learnings

Use `/review-learnings` to:
1. Synthesize all entries (Haiku agent — cheap)
2. Identify patterns and corrections
3. Produce action items for skill improvement
4. Apply improvements directly to skill reference files

## Additional Resources

- **`references/entry-examples.md`** — Exemplos de entradas bem escritas
