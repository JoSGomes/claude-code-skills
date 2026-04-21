# Fase 1 — Core: Skill DS + Agentes: Task Breakdown

**Goal**: Criar o skill `data-science-dev` completo com seus dois agentes dedicados
**Branch**: spec/phase-1-core-skill-ds
**Gerado em**: 2026-04-21

## Tarefas

- [ ] **T1** `sonnet` — Criar `agents/data-science-expert.md` — agente Sonnet para tarefas DS complexas (debug, arquitetura, modelagem)
- [ ] **T2** `sonnet` — Criar `agents/data-science-reviewer.md` — agente Haiku para review rápido de código DS (pitfall checks, data leakage, seeds)
- [ ] **T3** `sonnet` — Criar `skills/data-science-dev/references/pitfalls.md` — 8+ bugs críticos em pesquisa DS
- [ ] **T4** `sonnet` — Criar `skills/data-science-dev/references/patterns.md` — padrões DS: 7 áreas técnicas (EDA, ML, DL, NLP, tabular, computação científica, experiment tracking)
- [ ] **T5** `sonnet` — Criar `skills/data-science-dev/SKILL.md` — skill principal com routing, triggers de ativação, checklist de reprodutibilidade e padrão AskUserQuestion
- [ ] **T6** `revisão` — Executar spec-reviewer e validar conformidade com a spec

## Decisões pendentes
- Nível de detalhe dos exemplos em `patterns.md` (1 canônico por área vs. múltiplos)
- Estrutura exata do frontmatter do SKILL.md para triggers de ativação

## Success Criteria (da spec)
- [ ] Skill ativa nos triggers corretos (`.py` DS, `.ipynb`, keywords DS)
- [ ] Tabela de routing completa (Haiku/Sonnet/Opus)
- [ ] Checklist de reprodutibilidade acionável
- [ ] 7 áreas técnicas cobertas em `patterns.md`
- [ ] 8+ pitfalls DS em `pitfalls.md`
- [ ] Padrão `AskUserQuestion` exemplificado no skill
