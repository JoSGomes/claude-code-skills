---
name: python-dev
description: This skill should be used when the user is working with Python code, asks about Python patterns, debugging Python, scientific computing with numpy/pandas/torch, ML pipelines, research code, pytest, async Python, packaging with pyproject.toml, or asks "how do I do X in Python". Activates for .py files, Jupyter notebooks, and Python project context.
version: 0.1.0
---

# Python Senior Dev & Researcher Skill

Context loaded: working in a Python project with a senior developer or researcher.

---

## Comportamento esperado desta skill

Esta skill foi projetada para um desenvolvedor ou pesquisador **sênior**. Isso muda como a interação deve funcionar:

**O que fazer:**
- Ir direto ao ponto — pular explicações de conceitos básicos não solicitadas
- Mostrar código ao invés de descrever — preferir `code over prose` para respostas técnicas
- Apontar o `file:line` exato ao discutir código existente
- Questionar escolhas ruins ativamente — se o código viola um princípio importante, dizer qual e por quê
- Consultar `references/pitfalls.md` antes de sugerir uma abordagem que possa ter armadilhas conhecidas
- Capturar aprendizados com `/capture-learning` quando encontrar padrão novo não coberto pelas referências

**O que não fazer:**
- Não repetir o que o CI já vai checar (linter, mypy, formatação)
- Não sugerir refatorações além do escopo pedido
- Não adicionar comentários explicativos em código que se explica sozinho
- Não fazer suposições sobre reprodutibilidade em código de pesquisa — verificar ativamente
- Não usar modelos pesados para tarefas de extração (ver routing abaixo)

**Princípio central:** em Python científico, um bug de data leakage é mais grave do que um bug de performance. Checar sempre antes de marcar como pronto.

---

## Agent routing (economia de tokens)

| Task | Agent | Model | Quando usar |
|------|-------|-------|-------------|
| Scan rápido de bugs, pre-commit | `python-reviewer` | Haiku | Sempre que a tarefa for "revise este arquivo/diff" |
| Arquitetura, debug complexo, ML | `python-expert` | Sonnet | Quando precisa de raciocínio sobre lógica |
| Validação científica, código de paper | `research-validator` | Opus | Quando rigor estatístico e reprodutibilidade são críticos |

**Padrão**: responder inline com Sonnet (sessão atual). Lançar agente só quando a tarefa for isolada ou precisar de outro modelo.

---

## Convenções assumidas para este projeto

- Type hints em todas as funções públicas
- `ruff` ou `flake8` no CI — não reportar o que o linter pega
- `pytest` para testes
- `pyproject.toml` para empacotamento (não `setup.py`)
- `uv` ou `hatch` preferido sobre `pip` bare

---

## RTK — sempre

```bash
rtk pytest                    # 90% de economia de tokens
rtk pip install <pkg>
rtk python -m <module>
```

---

## Padrões críticos

### Python científico
- Preferir NumPy vetorizado. Se há loop, perguntar por quê.
- DataFrames: checar dtypes antes (`df.dtypes`, `df.memory_usage(deep=True)`)
- RNG moderno: `np.random.default_rng(seed)` — não o legado `np.random.seed()`

### Checklist de reprodutibilidade
Antes de considerar qualquer resultado de experimento como final:
1. Seeds: `random`, `numpy`, `torch`, `sklearn` — todos setados e logados
2. Config salva junto com os resultados
3. Ambiente pinado (`requirements.txt` ou `pyproject.toml`)
4. Dados checksumados ou version-pinados

### Pipelines ML
- Sempre `sklearn.pipeline.Pipeline` — nunca fit de preprocessor fora dela
- `train_test_split` sempre com `random_state`
- Cross-validation: `StratifiedKFold` para classificação, `GroupKFold` para amostras correlacionadas

---

## README & CHANGELOG

Ao final de qualquer sessão que resulte em alteração de código, atualizar `README.md` e `CHANGELOG.md` do projeto. Ver protocolo completo em **`skills/shared/changelog-readme-protocol.md`**.

---

## Additional Resources

- **`references/patterns.md`** — Python científico, async, packaging, type hints avançados
- **`references/pitfalls.md`** — Bugs comuns em código de pesquisa (data leakage, seeds, generators)
