# Fase 2 — Integração nas Pipelines: Task Breakdown

**Goal**: Integrar o skill DS nas pipelines de routing e especificação existentes
**Branch**: spec/phase-2-integracao-pipelines
**Gerado em**: 2026-04-21

## Tarefas

- [ ] **T1** `sonnet` — Atualizar `skills/model-routing/SKILL.md` — adicionar ≥3 linhas DS na tabela de budget (EDA, implementação de modelo, validação científica)
- [ ] **T2** `sonnet` — Atualizar `skills/spec-execution/SKILL.md` — adicionar exemplos DS no routing de tarefas (coluna de tabela e exemplos concretos)
- [ ] **T3** `sonnet` — Atualizar `skills/spec-execution/references/task-classification.md` — adicionar seção com exemplos de tarefas DS classificadas por modelo
- [ ] **T4** `sonnet` — Atualizar `commands/project-spec.md` — (a) detecção de projeto científico/DS + roteamento para research-validator; (b) output com nome de arquivo datado (`PROJECT_SPEC_titulo-simplificado_YYYY-MM-DD.md`)
- [ ] **T5** `revisão` — Executar spec-reviewer e validar conformidade com a spec

## Decisões pendentes
- Seção nova vs. linhas adicionadas na tabela existente em model-routing (preferir linhas adicionadas para não duplicar estrutura)

## Success Criteria (da spec)
- [ ] `model-routing` tem ≥3 linhas DS na tabela
- [ ] `spec-execution` tem exemplos concretos de tarefas DS classificadas
- [ ] `project-spec` detecta keywords DS e indica `research-validator` para análise científica
- [ ] `project-spec` gera arquivos com nome `PROJECT_SPEC_titulo-simplificado_YYYY-MM-DD.md` e título com data
