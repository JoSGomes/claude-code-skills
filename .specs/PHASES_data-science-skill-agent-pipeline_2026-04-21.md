# Plano Faseado — Data Science Skill + Agent Pipeline — 2026-04-21

## Fase 1 — Core: Skill DS + Agentes
**Modelo**: Sonnet inline | **Status**: Pendente

### Deliverables
| Arquivo | Descrição |
|---------|-----------|
| `skills/data-science-dev/SKILL.md` | Skill principal — comportamento, routing, convenções, padrão AskUserQuestion |
| `skills/data-science-dev/references/patterns.md` | Padrões DS: EDA, ML, DL, NLP, tabular, computação científica |
| `skills/data-science-dev/references/pitfalls.md` | Bugs críticos em pesquisa DS |
| `agents/data-science-expert.md` | Agente Sonnet para tarefas DS complexas |
| `agents/data-science-reviewer.md` | Agente Haiku para review rápido de código DS |

### Success Criteria
- [ ] Skill ativa nos triggers corretos (`.py` DS, `.ipynb`, keywords DS)
- [ ] Tabela de routing completa (Haiku/Sonnet/Opus)
- [ ] Checklist de reprodutibilidade acionável
- [ ] 7 áreas técnicas cobertas em `patterns.md`
- [ ] 8+ pitfalls DS em `pitfalls.md`
- [ ] Padrão `AskUserQuestion` exemplificado

---

## Fase 2 — Integração nas Pipelines
**Modelo**: Sonnet inline | **Status**: Pendente (depende de Fase 1)

### Deliverables
| Arquivo | Tipo | Descrição |
|---------|------|-----------|
| `skills/model-routing/SKILL.md` | Atualização | Linhas DS na tabela de routing |
| `skills/spec-execution/SKILL.md` | Atualização | Exemplos DS no routing de tarefas |
| `skills/spec-execution/references/task-classification.md` | Atualização | Classificação de tarefas DS por modelo |
| `commands/project-spec.md` | Atualização | Detecção de projeto científico/DS + roteamento + arquivos datados |

### Success Criteria
- [ ] `model-routing` tem ≥3 linhas DS na tabela
- [ ] `spec-execution` tem exemplos concretos de tarefas DS classificadas
- [ ] `project-spec` detecta keywords DS e indica `research-validator` para análise científica
- [ ] `project-spec` gera arquivos com nome no formato `PROJECT_SPEC_titulo-simplificado_YYYY-MM-DD.md` e título com data (`# ... — YYYY-MM-DD`)

---

## Fase 3 — Protocolo README/CHANGELOG (transversal)
**Modelo**: Sonnet inline | **Status**: Pendente (depende de Fase 1 e Fase 2)

### Deliverables
| Arquivo | Tipo | Descrição |
|---------|------|-----------|
| `skills/shared/changelog-readme-protocol.md` | Criação | Protocolo: gatilho, formato Keep a Changelog, template README, criação quando ausente |
| `skills/data-science-dev/SKILL.md` | Atualização | Seção `## README & CHANGELOG` referenciando o protocolo |
| `skills/python-dev/SKILL.md` | Atualização | Seção `## README & CHANGELOG` referenciando o protocolo |
| `skills/golang-dev/SKILL.md` | Atualização | Seção `## README & CHANGELOG` referenciando o protocolo |
| `skills/spec-execution/SKILL.md` | Atualização | Seção `## README & CHANGELOG` referenciando o protocolo |
| `skills/model-routing/SKILL.md` | Atualização | Seção `## README & CHANGELOG` referenciando o protocolo |

### Success Criteria
- [ ] `changelog-readme-protocol.md` define: gatilho, formato CHANGELOG, comportamento de criação, o que atualizar no README
- [ ] Todos os 5 skills têm seção `## README & CHANGELOG` apontando para o protocolo
- [ ] Nenhum skill duplica o conteúdo do protocolo — apenas referencia
