---
name: data-science-dev
description: This skill should be used when the user is working on Data Science, ML, or scientific research code — EDA, model training, experiment design, deep learning, NLP, tabular data, or scientific computing. Activates for .py files with DS imports (numpy, pandas, sklearn, torch, transformers), .ipynb Jupyter notebooks, projects with pyproject.toml containing DS dependencies, or when the user mentions EDA, model/modelo, training/treino, dataset, experiment/experimento, reprodutibilidade/reproducibility, data leakage, or feature engineering. Complements python-dev; use this skill when scientific rigor and research reproducibility are the primary concerns.
version: 0.1.0
---

# Data Science Dev & Researcher Skill

Context loaded: working in a Data Science or scientific research project with a senior developer or researcher.

---

## Comportamento esperado desta skill

Esta skill é para **pesquisa e desenvolvimento DS** — reprodutibilidade e rigor estatístico são não-negociáveis.

**O que fazer:**
- Verificar data leakage antes de qualquer diagnóstico de performance de modelo
- Checar seeds incompletos antes de aceitar resultados como válidos
- Apontar `file:line` exato ao discutir código existente
- Consultar `references/pitfalls.md` antes de sugerir qualquer abordagem de treino/validação
- Usar `AskUserQuestion` para perguntas com opções durante execução (ver seção abaixo)
- Capturar aprendizados com `/capture-learning` ao encontrar padrão novo não coberto pelas referências

**O que não fazer:**
- Não aceitar resultados sem verificar reprodutibilidade primeiro
- Não sugerir arquiteturas mais complexas antes de confirmar que o baseline está correto
- Não duplicar conteúdo de `python-dev` — referenciar quando necessário
- Não tratar test set como validation set — são papéis distintos

**Princípio central:** um modelo com 99% de acurácia que tem data leakage não tem valor científico. Verificar sempre antes de reportar resultados.

---

## Agent routing (economia de tokens)

| Task | Agent | Model | Quando usar |
|------|-------|-------|-------------|
| Review de código DS, pitfall check | `data-science-reviewer` | Haiku | Sempre que a tarefa for "revise este arquivo/diff" — rápido e barato |
| Debug de modelo, arquitetura de pipeline, NLP | `data-science-expert` | Sonnet | Quando precisa raciocínio sobre lógica DS ou debugging profundo |
| Validação científica, código de paper, rigor estatístico | `research-validator` | Opus | Quando reprodutibilidade e rigor de pesquisa são críticos |

**Padrão**: responder inline com Sonnet (sessão atual). Lançar agente só quando a tarefa for isolada ou precisar de outro modelo.

---

## Checklist de reprodutibilidade (obrigatório antes de reportar resultados)

```
[ ] Seeds: random, numpy, torch, sklearn — todos setados e logados
[ ] Config salva junto com os resultados (não só os pesos do modelo)
[ ] Ambiente pinado (requirements.txt / pyproject.toml com versões exatas)
[ ] Dados checksumados ou version-pinados
[ ] Preprocessor fitted ONLY on train — nunca no dataset completo
[ ] Test set usado exatamente UMA vez — nunca em loops de seleção
[ ] Split temporal (se dados com dimensão de tempo)
[ ] Notebook: Restart + Run All executado antes de publicar resultados
```

---

## Cobertura técnica

Esta skill cobre as seguintes áreas — ver `references/patterns.md` para exemplos canônicos:

| Área | Foco |
|------|------|
| **EDA** | pandas-profiling, distribuições, correlações, outliers, dados faltantes |
| **ML Clássico** | sklearn Pipeline obrigatório, validação cruzada, feature engineering |
| **Deep Learning** | PyTorch primário, AMP, gradient accumulation, reprodutibilidade |
| **NLP / LLMs** | HuggingFace Transformers, fine-tuning, embeddings, sentence-transformers |
| **Computação Científica** | SciPy, SymPy, bootstrap CI, testes estatísticos |
| **Tabular Pesado** | Polars (lazy), DuckDB, chunked pandas para dados maiores que RAM |
| **Experiment Tracking** | MLflow, W&B, DVC — config sempre salva junto com resultados |

---

## Padrão AskUserQuestion (perguntas interativas no terminal)

Quando esta skill precisar fazer perguntas com opções durante a execução, usar `AskUserQuestion` para apresentar as escolhas no terminal em vez de prosa no chat. Isso é mais rápido e menos interruptivo.

**Como usar:**

```markdown
<!-- Passo 1: carregar o schema da ferramenta via ToolSearch -->
ToolSearch({ query: "select:AskUserQuestion", max_results: 1 })

<!-- Passo 2: chamar AskUserQuestion com as opções -->
AskUserQuestion({
  question: "Qual estratégia de validação usar?",
  options: [
    "StratifiedKFold (classificação com classes balanceadas)",
    "TimeSeriesSplit (dados temporais)",
    "GroupKFold (amostras correlacionadas por grupo)",
    "Hold-out simples (dataset grande, sem necessidade de CV)"
  ]
})
```

**Fallback**: se `ToolSearch` falhar ou `AskUserQuestion` não estiver disponível no contexto, apresentar as opções como lista numerada em prosa no chat.

**Exemplos de quando usar:**
- Escolha de estratégia de validação (CV vs hold-out vs temporal)
- Estratégia de split (aleatório vs temporal vs por grupo)
- Framework de DL (PyTorch vs TensorFlow)
- Método de experiment tracking (MLflow vs W&B vs sem tracking)

---

## Comportamento específico para Jupyter Notebooks

Ao trabalhar com `.ipynb`:

1. **Detectar execução fora de ordem**: verificar se o número de execução das células está crescendo monotonicamente
2. **Estado oculto em variáveis**: confirmar se variáveis-chave (splits, scalers, modelos) foram criadas na sessão atual
3. **Células não executadas**: alertar se a célula de setup (seeds, imports) não foi executada antes de células de treino
4. **Resultado final**: sempre instruir `Kernel → Restart & Run All` antes de publicar ou compartilhar resultados

---

## Convenções assumidas

- `sklearn.pipeline.Pipeline` obrigatório — nunca preprocessor fora dela
- `pyproject.toml` para projetos (não `setup.py`)
- `uv` ou `hatch` preferido sobre `pip` bare
- Resultados salvos com timestamp e config — nunca sobrescrevendo runs anteriores

---

## RTK — sempre

```bash
rtk pytest                    # 90% de economia de tokens
rtk python -m <module>
rtk pip install <pkg>
```

---

## Additional Resources

- **`references/patterns.md`** — Exemplos canônicos para as 7 áreas técnicas DS
- **`references/pitfalls.md`** — 8 bugs críticos em pesquisa DS (data leakage, seeds, temporal leakage, test set reuse, generator exhaustion, SettingWithCopy, float comparison, notebook state)
- **`agents/data-science-expert.md`** — Agente Sonnet para tarefas DS complexas
- **`agents/data-science-reviewer.md`** — Agente Haiku para review rápido de código DS
- **`agents/research-validator.md`** — Agente Opus para validação científica rigorosa
- **`skills/python-dev/SKILL.md`** — Para Python genérico fora de contexto DS
