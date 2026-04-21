# claude-code-skills

Plugin para Claude Code com workflows e skills para desenvolvimento de software e pesquisa científica. Projetado para economizar tokens usando o modelo certo para cada tarefa.

## Instalação

```bash
claude plugin marketplace add JoSGomes/claude-code-skills
claude plugin install claude-code-skills
```

## Como funciona

O plugin oferece um ciclo completo de desenvolvimento orientado por especificação:

```
/project-spec   →  especificação interativa com divisão em fases
/exec-spec N    →  executa a fase N em uma branch dedicada
                   └─ roteia cada tarefa: Haiku / Sonnet / Opus
                   └─ spec-reviewer verifica conformidade antes do PR
                   └─ abre PR com descrição completa (com aprovação)
/capture-learning  →  registra aprendizados em .claude/learnings/
/review-learnings  →  sintetiza aprendizados e abre PR de melhoria no plugin
```

## Workflows

| Comando | O que faz | Modelos usados |
|---------|-----------|----------------|
| `/project-spec` | Especificação interativa: coleta input, lê documentos, analisa requisitos, divide em fases, grava em `.specs/` | Haiku (docs) · Opus (análise + fases) |
| `/exec-spec [N]` | Executa a fase N: cria branch, decompõe em tarefas, executa com roteamento de modelo, revisa conformidade com a spec, abre PR | Haiku · Sonnet · Opus por tarefa |
| `/capture-learning` | Registra um aprendizado descoberto durante o trabalho em `.claude/learnings/` do projeto | — |
| `/review-learnings` | Sintetiza aprendizados capturados, aplica melhorias nos `references/` das skills e abre PR neste repositório | Haiku (síntese) |

## Skills contextuais

Ativadas automaticamente pelo Claude de acordo com o contexto — sem precisar invocar.

| Skill | Quando ativa | O que carrega |
|-------|--------------|---------------|
| `python-dev` | Arquivos `.py`, projetos Python, computação científica | Convenções, routing de agentes, checklist de reprodutibilidade |
| `golang-dev` | Arquivos `.go`, `go.mod`, projetos Go | Idioms, modelo mental de concorrência, routing |
| `model-routing` | Planejamento multi-agent, otimização de tokens | Tabela de custo, anti-patterns de gasto |
| `learning-capture` | Descoberta de padrões, bugs, correções durante trabalho | Quando capturar, formato, localização |
| `spec-execution` | `PROGRESS.md` presente, menção a "fase" ou "exec-spec" | Estado de progresso, routing por tarefa, padrões de commit |

## Agentes

Lançados pelos workflows ou diretamente — cada um usa o modelo mais barato para o que faz.

| Agente | Modelo | Especialidade |
|--------|--------|---------------|
| `document-reader` | Haiku | Leitura e resumo de arquivos, URLs, documentos |
| `python-reviewer` | Haiku | Review rápido Python — só issues reais (>80% confiança) |
| `golang-reviewer` | Haiku | Review rápido Go — goroutine leaks, erros ignorados, idioms |
| `learning-synthesizer` | Haiku | Sintetiza entradas de aprendizado e gera action items |
| `python-expert` | Sonnet | Arquitetura, debug, ML pipelines, async, NumPy/PyTorch |
| `golang-expert` | Sonnet | Concorrência, profiling, interfaces, gRPC, CLI |
| `spec-reviewer` | Sonnet | Conformidade do código com a especificação (não qualidade geral) |
| `requirements-analyst` | Opus | Análise de requisitos: gaps, ambiguidades, riscos |
| `phase-planner` | Opus | Plano faseado com deliverables, critérios de sucesso e routing de modelos |
| `research-validator` | Opus | Rigor científico: data leakage, seeds, validade estatística |

## Estratégia de tokens

```
Leitura, extração, review rápido  →  Haiku   (paralelo, muito barato)
Código, debug, diálogo            →  Sonnet  (padrão, inline)
Raciocínio profundo, pesquisa     →  Opus    (máx. 2-3× por sessão)
Todo bash                         →  prefixar com rtk
```

## Outputs gerados nos projetos

| Workflow | Onde grava |
|----------|------------|
| `/project-spec` | `.specs/PROJECT_SPEC.md` · `.specs/PHASES.md` |
| `/exec-spec` | `.specs/execution/PROGRESS.md` · `.specs/execution/phase-N/TASKS.md` · `DECISIONS.md` · `REVIEW.md` |
| `/capture-learning` | `.claude/learnings/entries/YYYY-MM-DD_skill_slug.md` |
| `/review-learnings` | PR em `JoSGomes/claude-code-skills` |

## Estrutura do repositório

```
claude-code-skills/
├── .claude-plugin/
│   ├── plugin.json              ← manifesto do plugin
│   └── marketplace.json         ← manifesto do marketplace
├── commands/                    ← workflows invocados com /comando
│   ├── project-spec.md
│   ├── exec-spec.md
│   ├── capture-learning.md
│   └── review-learnings.md
├── agents/                      ← agentes especializados por modelo
│   ├── document-reader.md       (Haiku)
│   ├── python-reviewer.md       (Haiku)
│   ├── golang-reviewer.md       (Haiku)
│   ├── learning-synthesizer.md  (Haiku)
│   ├── python-expert.md         (Sonnet)
│   ├── golang-expert.md         (Sonnet)
│   ├── spec-reviewer.md         (Sonnet)
│   ├── requirements-analyst.md  (Opus)
│   ├── phase-planner.md         (Opus)
│   └── research-validator.md    (Opus)
└── skills/                      ← skills contextuais (auto-ativadas)
    ├── python-dev/
    ├── golang-dev/
    ├── model-routing/
    ├── learning-capture/
    └── spec-execution/
```

## Roadmap

- `research-pipeline` — estruturação de pipelines de pesquisa científica
- `paper-review` — leitura e síntese de artigos científicos para embasar decisões
- `dataset-audit` — análise de datasets: qualidade, viés, distribuição
- `experiment-tracker` — setup de rastreamento de experimentos com logging estruturado
