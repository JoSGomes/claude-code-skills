# Project Specification: Data Science Skill + Agent Pipeline — 2026-04-21

**Data**: 2026-04-21
**Versão**: 1.0
**Status**: Draft

---

## 1. Visão Geral

Adição de um skill especializado em Data Science (`data-science-dev`) ao plugin `claude-code-skills`, acompanhado de dois agentes dedicados (`data-science-expert` e `data-science-reviewer`) e integração nas pipelines de especificação (`project-spec`) e execução (`spec-execution`).

O skill deve cobrir o espectro completo de Data Science científico: EDA, ML clássico, deep learning, NLP, computação científica e dados tabulares pesados — com foco em **reprodutibilidade**, **rigor estatístico** e **prevenção de data leakage**.

Uma adição transversal: quando o skill precisa fazer perguntas durante a execução, deve usar `AskUserQuestion` para apresentar opções interativamente no terminal em vez de perguntar em prosa no chat.

---

## 2. Objetivos e Critérios de Sucesso

| Objetivo | Critério mensurável |
|----------|---------------------|
| Skill DS carrega no contexto correto | Ativa para `.py`, `.ipynb`, projetos com `pyproject.toml` + deps DS |
| Pitfall checks roteados para Haiku | `data-science-reviewer` lançado em tarefas de review de código DS |
| Pipeline `spec-execution` classifica tarefas DS | Tabela de routing tem exemplos DS; sem ambiguidade |
| Pipeline `project-spec` detecta projeto científico | Quando projeto é DS/científico, routing para `research-validator` (Opus) é explícito |
| Perguntas com opções abrem no terminal | Skills usam `AskUserQuestion` — não prosa no chat |

---

## 3. Escopo

### In Scope

- `skills/data-science-dev/SKILL.md` — skill principal
- `skills/data-science-dev/references/patterns.md` — padrões DS (EDA, ML, DL, NLP, tabular, computação científica)
- `skills/data-science-dev/references/pitfalls.md` — bugs críticos em pesquisa DS (data leakage, seeds, temporal split, notebooks)
- `agents/data-science-expert.md` — agente Sonnet para tarefas DS complexas
- `agents/data-science-reviewer.md` — agente Haiku para review rápido de código DS
- Atualização de `skills/model-routing/SKILL.md` — linhas DS na tabela de routing
- Atualização de `skills/spec-execution/SKILL.md` e `references/task-classification.md` — exemplos DS
- Atualização de `commands/project-spec.md` — detecção de projeto científico/DS e roteamento
- Documentação do padrão `AskUserQuestion` no skill DS (e referência para outros skills)
- `skills/shared/changelog-readme-protocol.md` — protocolo compartilhado de atualização de README/CHANGELOG
- Atualização de todos os skills existentes (`python-dev`, `golang-dev`, `spec-execution`, `model-routing`) para incluir o protocolo README/CHANGELOG
- Templates de `README.md` e `CHANGELOG.md` para projetos que não os possuem

### Out of Scope

- MLOps pesado: Kubernetes, pipelines de deploy em produção, CI/CD de modelos
- Ferramentas de data engineering (Spark, Kafka, Airflow)
- Skill para R ou Julia
- Atualização retroativa de `python-dev` / `golang-dev` com `AskUserQuestion` (fica como item futuro)

---

## 4. Requisitos

### Funcionais

**RF-01** — O skill `data-science-dev` deve ativar quando: arquivos `.py` com imports de libs DS, arquivos `.ipynb`, projetos com `pyproject.toml` contendo deps DS (numpy, pandas, sklearn, torch, etc.), ou quando o usuário menciona EDA, modelo, treino, dataset, experimento.

**RF-02** — O skill deve definir routing explícito: `data-science-reviewer` (Haiku) para reviews de código DS; `data-science-expert` (Sonnet) para debug e arquitetura DS; `research-validator` (Opus) para validação científica.

**RF-03** — O skill deve incluir checklist de reprodutibilidade ativo: seeds, config salva, ambiente pinado, dados checksumados.

**RF-04** — O skill deve cobrir as seguintes áreas técnicas:
- EDA: pandas-profiling, seaborn, distribuições, correlações, outliers
- ML clássico: sklearn Pipeline obrigatório, validação cruzada, feature engineering
- Deep Learning: PyTorch (primário), AMP, gradient accumulation, reproducibilidade
- NLP/LLMs: HuggingFace Transformers, fine-tuning, embeddings
- Computação científica: SciPy, SymPy, simulações numéricas
- Tabular pesado: Polars, DuckDB, dados maiores que RAM
- Experiment tracking: MLflow, W&B, DVC (diretrizes básicas)

**RF-05** — O skill deve ter comportamento específico para notebooks Jupyter: detectar execução fora de ordem, estado oculto em variáveis, células não executadas.

**RF-06** — Quando o skill precisar fazer perguntas com opções durante execução, deve usar `AskUserQuestion` para apresentar escolhas interativamente no terminal.

**RF-07** — O agente `data-science-reviewer` deve verificar prioritariamente: data leakage, seed incompleto, temporal leakage em time-series, uso múltiplo do test set, generator exhaustion.

**RF-08** — O `project-spec` deve detectar explicitamente projetos DS/científicos e indicar que a análise de requisitos científicos passa pelo `research-validator` (Opus).

**RF-09** — O `spec-execution` deve ter exemplos de classificação DS nas tabelas: EDA, implementação de modelo, validação de experimento.

**RF-10** — Os arquivos de spec gerados pelo `project-spec` devem incluir o título da spec simplificado (lowercase, hifenizado) e a data no nome do arquivo. Formato: `PROJECT_SPEC_titulo-simplificado_YYYY-MM-DD.md`, `PHASES_titulo-simplificado_YYYY-MM-DD.md`. Formato do título interno: `# Project Specification: [Nome] — YYYY-MM-DD`. O título simplificado é derivado do nome do projeto: remover artigos/preposições, substituir espaços e símbolos por hifens, lowercase. A data usada é a data corrente no momento da execução, disponível via `currentDate` no contexto.

**RF-11** — Toda atualização de código realizada com qualquer skill deste plugin deve, ao final, atualizar o `README.md` e o `CHANGELOG.md` do repositório. Se esses arquivos não existirem, o skill deve criá-los antes de atualizar. O `CHANGELOG.md` segue o formato [Keep a Changelog](https://keepachangelog.com) (`## [Unreleased]` → seções `Added / Changed / Fixed / Removed`). O `README.md` deve refletir as funcionalidades presentes após a mudança. O protocolo completo fica em `skills/shared/changelog-readme-protocol.md` e é referenciado por todos os skills.

### Não-Funcionais

**RNF-01** — Estrutura de arquivos idêntica aos skills existentes (`python-dev`, `golang-dev`): `SKILL.md` + `references/patterns.md` + `references/pitfalls.md`.

**RNF-02** — Instruções comportamentais em português; exemplos de código em inglês (padrão atual do projeto).

**RNF-03** — Agentes novos seguem o formato dos existentes (`python-expert`, `python-reviewer`).

**RNF-04** — Nenhum skill deve duplicar conteúdo de `python-dev`; referenciá-lo quando necessário.

### Científicos / de Pesquisa

**RC-01** — Data leakage é o bug mais crítico; deve ser o primeiro item verificado em qualquer review DS.

**RC-02** — Reprodutibilidade não é opcional: seed incompleto = resultado inválido.

**RC-03** — Test set usado mais de uma vez = p-hacking acidental; o skill deve tornar esse erro óbvio.

---

## 5. Restrições e Premissas

- Plugin usa Markdown como formato de skills e agentes — sem código executável nos arquivos de skill
- O padrão de routing Haiku/Sonnet/Opus já está estabelecido; o skill DS deve seguir sem desvios
- `AskUserQuestion` é uma ferramenta deferida no ambiente Claude Code — o skill deve instruir seu carregamento via `ToolSearch` antes de chamar
- `python-dev` permanece ativo para Python genérico; `data-science-dev` é complementar, não substituto total

---

## 6. Decisões Técnicas

| Decisão | Escolha | Alternativa descartada | Motivo |
|---------|---------|------------------------|--------|
| Relação com `python-dev` | Complementar — DS ativo para contexto científico; python-dev para Python genérico | Substituir python-dev | python-dev tem valor para Flask, FastAPI, scripts — não só DS |
| Reviewer DS | Novo agente `data-science-reviewer` (Haiku) | Reutilizar `python-reviewer` | python-reviewer não tem pitfalls DS específicos |
| Cobertura DL | PyTorch primário, TF como nota | Paridade PyTorch/TF | Uso pessoal favorece PyTorch; evita doc inflada |
| Perguntas interativas | `AskUserQuestion` no skill DS | Prosa no chat | Terminal é mais rápido e menos interruptivo |
| Deploy de modelos | Fora do escopo v1 | Incluir ONNX/FastAPI | Escopo inicial focado em pesquisa, não produção |

---

## 7. Plano Faseado

### Fase 1 — Core: Skill DS + Agentes

**Deliverables:**
- `skills/data-science-dev/SKILL.md`
- `skills/data-science-dev/references/patterns.md`
- `skills/data-science-dev/references/pitfalls.md`
- `agents/data-science-expert.md`
- `agents/data-science-reviewer.md`

**Success Criteria:**
- Skill ativa nas condições corretas (descrição do frontmatter cobre todos os triggers)
- Routing interno completo (tabela Haiku/Sonnet/Opus)
- Checklist de reprodutibilidade presente e acionável
- Cobertura das 7 áreas técnicas em `patterns.md`
- Top 8 pitfalls DS em `pitfalls.md` (data leakage, temporal leakage, seeds, test set reutilizado, generator exhaustion, Pandas SettingWithCopy, float comparison, notebook state)
- Padrão `AskUserQuestion` documentado e exemplificado no skill

**Modelo**: Sonnet (inline)
**Estimativa**: 1 sessão

---

### Fase 2 — Integração nas Pipelines

**Deliverables:**
- `skills/model-routing/SKILL.md` — atualizado com linhas DS
- `skills/spec-execution/SKILL.md` — atualizado com exemplos DS
- `skills/spec-execution/references/task-classification.md` — exemplos de tarefas DS classificadas
- `commands/project-spec.md` — detecção de projeto científico/DS + routing explícito para `research-validator`

**Success Criteria:**
- Tabela de routing em `model-routing` tem pelo menos 3 linhas DS (EDA, implementação de modelo, validação científica)
- `spec-execution` tem exemplos concretos: "Implementar pipeline de EDA → Sonnet", "Validar reprodutibilidade do experimento → Haiku + Opus"
- `project-spec` detecta "pesquisa", "dataset", "modelo", "experimento" como triggers para roteamento científico
- `project-spec` gera arquivos com nome no formato `PROJECT_SPEC_titulo-simplificado_YYYY-MM-DD.md` e título com data (`# ... — YYYY-MM-DD`)

**Modelo**: Sonnet (inline)
**Estimativa**: 1 sessão

---

### Fase 3 — Protocolo README/CHANGELOG (transversal)

**Deliverables:**
- `skills/shared/changelog-readme-protocol.md` — protocolo compartilhado: quando atualizar, formato Keep a Changelog, template de README, comportamento quando arquivos não existem
- `skills/data-science-dev/SKILL.md` — seção referenciando o protocolo (complementa Fase 1)
- `skills/python-dev/SKILL.md` — seção referenciando o protocolo
- `skills/golang-dev/SKILL.md` — seção referenciando o protocolo
- `skills/spec-execution/SKILL.md` — seção referenciando o protocolo (complementa Fase 2)
- `skills/model-routing/SKILL.md` — seção referenciando o protocolo (complementa Fase 2)

**Success Criteria:**
- `changelog-readme-protocol.md` existe e define: gatilho (qualquer mudança de código), formato CHANGELOG (Keep a Changelog), comportamento de criação quando ausente, o que atualizar no README (features presentes, não features planejadas)
- Todos os 5 skills têm seção `## README & CHANGELOG` apontando para o protocolo compartilhado
- Não há duplicação do protocolo nos skills — cada um referencia, não repete

**Modelo**: Sonnet (inline)
**Estimativa**: 1 sessão
**Dependência**: Fase 1 e Fase 2 concluídas (os skills precisam existir antes de serem atualizados)

---

## 8. Riscos e Mitigações

| Risco | Probabilidade | Impacto | Mitigação |
|-------|--------------|---------|-----------|
| Sobreposição com `python-dev` causa ambiguidade de ativação | Média | Médio | Triggers de ativação mutuamente exclusivos na description do frontmatter |
| `patterns.md` fica grande demais e perde valor como referência rápida | Alta | Médio | Limitar cada seção a 1 exemplo canônico; sem variações |
| `AskUserQuestion` não disponível em todos os contextos Claude Code | Baixa | Baixo | Instrução de fallback: se ToolSearch falhar, usar lista numerada em prosa |

---

## 9. Questões em Aberto

- (Futuro) Atualizar `python-dev` e `golang-dev` para também usarem `AskUserQuestion` — postergado para v1.1
- (Futuro) Skill `data-science-dev` para R ou Julia — fora de escopo, registrado para avaliação futura
- (Futuro) Cobertura de deploy: ONNX, TorchServe, FastAPI + modelo — pós v1
