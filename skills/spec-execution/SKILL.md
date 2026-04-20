---
name: spec-execution
description: This skill should be used when a project has a .specs/execution/PROGRESS.md file, when the user is executing a project specification phase, when working on tasks defined in .specs/execution/phase-N/TASKS.md, when the user mentions "fase", "phase", "exec-spec", or asks "where are we" / "o que falta" in the context of an ongoing project. Provides execution context, model routing per task, and progress awareness.
version: 0.1.0
---

# Spec Execution Skill

Context loaded: there is an active specification being executed in this project.

---

## Comportamento esperado desta skill

Esta skill entra em ação quando há uma especificação ativa sendo executada. Seu papel é manter o Claude ciente do contexto de execução e garantir que cada ação respeite o plano estabelecido.

**O que fazer:**
- Sempre verificar o estado atual em `.specs/execution/PROGRESS.md` antes de começar qualquer trabalho
- Ler a tarefa corrente em `.specs/execution/phase-N/TASKS.md` para saber exatamente o que está em escopo
- Rotear cada tarefa para o modelo correto conforme a classificação no TASKS.md
- Commitar após cada deliverable concreto — não acumular trabalho não commitado
- Capturar aprendizados imediatamente com `/capture-learning` quando algo notável for descoberto
- Avisar quando uma ação vai além do escopo da fase corrente

**O que não fazer:**
- Não iniciar trabalho sem ler o TASKS.md da fase atual
- Não pular para a próxima fase sem validar os Success Criteria
- Não usar Opus para tarefas marcadas como Sonnet/Haiku no TASKS.md
- Não fazer commits genéricos como "work in progress" — cada commit deve descrever o deliverable
- Não perder o fio de progresso — PROGRESS.md é a fonte de verdade

**Princípio central:** a especificação foi pensada para economizar tempo total. Desviar do plano durante a execução é mais caro do que ajustar o plano antes de executar.

---

## Leitura de estado (fazer sempre ao iniciar sessão)

Ao abrir uma sessão de trabalho em um projeto com spec ativa:

```
1. Ler .specs/execution/PROGRESS.md   → fase atual, tarefas restantes, bloqueios
2. Ler .specs/execution/phase-N/TASKS.md  → próxima tarefa não concluída
3. Informar ao usuário: "Continuando Fase N — próxima tarefa: [T]"
```

---

## Routing de modelos por tarefa

O TASKS.md já define o modelo por tarefa (`[haiku]`, `[sonnet]`, `[opus]`). Respeitar sempre.

**Referência rápida:**

| Tarefa | Modelo | Como executar |
|--------|--------|---------------|
| Explorar/ler código existente | Haiku | Agente paralelo se múltiplos arquivos |
| Implementar/criar/corrigir | Sonnet | Inline, ler antes de escrever |
| Decisão de arquitetura | Opus | Agente único, registrar em DECISIONS.md |
| Validar critérios de sucesso | Haiku (verificar) + Sonnet (corrigir) | Testar com `rtk`, analisar resultado |

---

## Spec-reviewer — obrigatório antes de cada PR

O agente `spec-reviewer` (Sonnet) verifica conformidade com a spec **antes** do PR. Nunca pular.

O que ele checa (não é review de qualidade geral):
- Todos os deliverables da fase foram implementados?
- O código implementa o que foi especificado (requisitos funcionais e não-funcionais)?
- Há trabalho fora do escopo da fase?
- As interfaces/APIs estão conforme especificado?
- Os success criteria são atendidos?

Resultado vai para `.specs/execution/phase-N/REVIEW.md` e é incluído na descrição do PR.

---

## Commits durante execução

Padrão de mensagem de commit por tipo:

```bash
# Ao concluir um deliverable de implementação
rtk git commit -m "feat(phase-N): implementa [deliverable]"

# Ao corrigir algo encontrado durante validação
rtk git commit -m "fix(phase-N): corrige [problema] identificado na validação"

# Ao registrar decisão de arquitetura
rtk git commit -m "docs(phase-N): registra decisão sobre [tema]"

# Ao atualizar estado de progresso
rtk git commit -m "chore(phase-N): atualiza PROGRESS.md — [X/Y tarefas]"
```

---

## Quando sair do escopo da fase

Se durante a execução surgir trabalho relevante que pertence a outra fase:

1. **Não fazer** — registrar a nota no PROGRESS.md:
   ```markdown
   ## Notas para fases futuras
   - [Fase M] [Observação relevante descoberta durante Fase N]
   ```
2. Continuar a tarefa atual
3. Informar o usuário ao final da tarefa sobre o que foi anotado

---

## Additional Resources

- **`references/task-classification.md`** — Guia detalhado de como classificar tarefas por modelo
- **`references/commit-patterns.md`** — Padrões de commit por tipo de deliverable e fase
