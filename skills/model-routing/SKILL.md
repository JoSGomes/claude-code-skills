---
name: model-routing
description: This skill should be used when deciding which Claude model or agent to spawn for a task, when the user asks "which model should I use", when planning a multi-agent workflow, when optimizing for token cost vs quality, or when a task involves spawning sub-agents. Provides model selection rules and token economy guidance.
version: 0.1.0
---

# Model Routing & Token Economy Skill

Reference: seleção de modelo e economia de tokens para este workflow de desenvolvimento.

---

## Comportamento esperado desta skill

Esta skill define **como tomar decisões de roteamento** — qual modelo usar, quando lançar agente vs responder inline, como estruturar trabalho paralelo.

**O que fazer:**
- Antes de lançar qualquer agente, verificar se a tarefa é realmente isolada ou se já está no contexto
- Usar Haiku em paralelo para tarefas de extração — é a operação de menor custo disponível
- Reservar Opus para momentos onde raciocínio profundo sobre ambiguidade **muda o resultado**
- Estruturar prompts de agentes com output format definido — agentes que retornam prosa são caros de processar
- Verificar se o resultado de um Haiku barato pode funcionar antes de chamar Sonnet

**O que não fazer:**
- Não lançar agente Opus para tarefas de leitura ou extração simples
- Não sequenciar agentes que podem rodar em paralelo
- Não lançar agente quando a resposta cabe no contexto atual
- Não usar bash sem prefixo `rtk` — cada linha de output desnecessária custa tokens

**Princípio central:** o modelo mais barato que resolve o problema é sempre o correto. Custo não é só dinheiro — é velocidade e janela de contexto.

---

## Regras de seleção de modelo

### Haiku — extração e verificação
- Ler arquivos e extrair informação pontual
- Verificações de elegibilidade (está aberto? tem testes? toca este arquivo?)
- Review rápido com output estruturado
- Resumir outputs longos de comandos ou documentos
- Tarefas repetíveis em paralelo (mesma lógica, muitos arquivos)

### Sonnet — padrão para código e diálogo
- Geração e refatoração de código
- Debug — precisa entender lógica
- Explicação de arquitetura
- Diálogo interativo com o desenvolvedor
- Orquestração de comandos e agents
- Quando não tiver certeza qual usar

### Opus — raciocínio profundo (máximo 2-3×/sessão)
- Análise de requisitos com inputs ambíguos e múltiplas dependências
- Validação científica — rigor estatístico e reprodutibilidade
- Decisões de arquitetura com trade-offs de longo prazo
- Resolução de ambiguidade onde escolha errada é cara

---

## Padrão de agentes paralelos

Agentes independentes → paralelo. Agentes dependentes → sequencial.

```
# ERRADO: sequencial quando paralelo é possível
agent1 → wait → agent2 → wait → agent3

# CORRETO: paralelo quando independentes
[agent1, agent2, agent3] → wait → sintetizar inline
```

Agentes Haiku em paralelo para leitura. Sonnet/Opus para síntese.

---

## RTK — sem exceções

```bash
rtk go test ./...         # não: go test ./...
rtk pytest                # não: pytest
rtk git diff              # não: git diff
rtk gh pr view 42         # não: gh pr view 42
rtk cargo build           # não: cargo build
```

RTK passa comandos desconhecidos sem filtrar — sempre seguro usar.

---

## Budget por tipo de tarefa

| Tarefa | Abordagem | Custo relativo |
|--------|-----------|----------------|
| Leitura + resumo de arquivo | Agente Haiku | Muito baixo |
| Review rápido de código | Agente Haiku | Muito baixo |
| Review de código DS (pitfall check) | `data-science-reviewer` Haiku | Muito baixo |
| Geração de código | Sonnet inline | Médio |
| Implementação de pipeline de EDA | `data-science-expert` Sonnet | Médio |
| Implementação/debug de modelo DS | `data-science-expert` Sonnet | Médio |
| Review de PR multi-arquivo | 3-5 agentes Sonnet paralelos | Médio-alto |
| Validação científica | `research-validator` Opus | Alto |
| Validação de reprodutibilidade de experimento | Haiku (verificar seeds/leakage) + `research-validator` Opus | Alto |
| Especificação de projeto DS/científico | `document-reader` Haiku + `requirements-analyst` Opus + `phase-planner` Opus | Alto, one-time |
| Especificação de projeto genérico | Haiku (docs) + Opus×2 | Alto, one-time |

---

## Quando NÃO lançar agente

- Pergunta simples e focada → responder inline
- Edição de arquivo único → editar diretamente
- Lookup rápido → usar Grep/Read direto
- Tudo que cabe no contexto atual → não externalizar

Lançar agente tem overhead de cold start. Só vale quando a tarefa é isolada ou precisa de modelo diferente.

---

## Additional Resources

- **`references/cost-patterns.md`** — Anti-patterns de gasto de tokens, custo relativo por operação, caching
