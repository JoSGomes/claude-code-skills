---
description: Executa uma especificação de projeto fase a fase em uma branch git dedicada. Roteia tarefas por modelo (Haiku/Sonnet/Opus), revisa conformidade com a spec antes de cada PR, e abre PR com descrição completa ao fim de cada fase.
argument-hint: [N — número da fase, ou vazio para status]
allowed-tools: Read, Glob, Grep, Write, Edit, Bash(git:*), Bash(gh:*), Bash(rtk:*), Bash(mkdir:*), Bash(ls:*), Agent
---

# /exec-spec — Executor de Especificação por Fases

Lê `.specs/PHASES.md`, cria branch por fase, executa tarefa a tarefa com o modelo certo, revisa conformidade com a spec e abre PR com aprovação do desenvolvedor.

**Argumento:** `$ARGUMENTS` (número de fase, ou vazio para status)

---

## Estrutura de estado

```
.specs/
└── execution/
    ├── PROGRESS.md              ← fase atual, branches, PRs, bloqueios
    ├── GIT_STRATEGY.md          ← estratégia de branch acordada na inicialização
    ├── phase-N/
    │   ├── TASKS.md             ← tarefas com checkboxes + modelo
    │   ├── DECISIONS.md         ← decisões tomadas durante execução
    │   └── REVIEW.md            ← resultado do spec-reviewer antes do PR
    └── ...
```

---

## Dispatch inicial

### Sem argumento — Status

1. Verificar se `.specs/` existe. Se não:
   > "Nenhuma especificação encontrada. Execute `/project-spec` primeiro."
   Parar.

2. Ler `.specs/execution/PROGRESS.md`. Mostrar dashboard:

```
## Status de Execução — [nome do projeto]

Branch strategy: [por fase | única]
Fase atual:      [N] — [Nome da Fase]
Branch ativa:    spec/phase-N-[slug]
PR aberto:       [URL ou "nenhum"]
Progresso:       [X/Y tarefas]
Próxima tarefa:  [descrição]

Fases:
  ✅ Fase 1 — [nome]  PR #[N] mergeado
  🔄 Fase 2 — [nome]  spec/phase-2-... (4/7 tarefas)
  ⬜ Fase 3 — [nome]
```

3. Se PROGRESS.md não existir mas `.specs/PHASES.md` existir:
   > "Especificação encontrada mas execução não iniciada. Execute `/exec-spec 1` para começar."

---

### Com argumento N — Executar fase N

Seguir o workflow abaixo.

---

## Workflow de Execução de Fase

### Etapa 0 — Inicialização (apenas na primeira vez)

Se `GIT_STRATEGY.md` não existir:

1. Detectar branch padrão:
   ```bash
   git symbolic-ref refs/remotes/origin/HEAD | sed 's@^refs/remotes/origin/@@'
   ```

2. Perguntar ao desenvolvedor:
   > "Como prefere organizar as branches desta execução?
   >
   > **A) Uma branch por fase** (recomendado)
   >    `spec/phase-1-[slug]` → PR → merge → `spec/phase-2-[slug]` → PR → ...
   >    Cada fase tem seu próprio PR — mais fácil de revisar incrementalmente.
   >
   > **B) Uma branch única**
   >    `spec/[project-slug]` com commits de todas as fases → um PR ao final.
   >    Mais simples, mas PR maior e mais difícil de revisar.
   >
   > Responda A ou B."

3. Gravar `.specs/execution/GIT_STRATEGY.md`:
   ```markdown
   # Git Strategy
   mode: [por-fase | unica]
   base-branch: [main|master|develop]
   project-slug: [slug do nome do projeto]
   ```

---

### Etapa 1 — Ler a fase e criar branch

1. Ler `.specs/PHASES.md`, extrair Fase N:
   - Goal, Deliverables, Key Tasks, Success Criteria, Dependencies, Recommended Model Usage

2. Verificar dependências em PROGRESS.md. Se fase anterior não concluída, alertar e aguardar confirmação.

3. **Criar branch para esta fase:**

   **Estratégia por fase:**
   ```bash
   # Base: main (ou a branch da fase anterior se não foi mergeada)
   git checkout [base-branch]
   git pull origin [base-branch]
   git checkout -b spec/phase-N-[slug-do-goal]
   ```
   Exemplos: `spec/phase-1-setup`, `spec/phase-2-data-pipeline`, `spec/phase-3-model-training`

   **Estratégia única:**
   ```bash
   # Só cria se ainda não existir
   git checkout [base-branch] && git pull
   git checkout -b spec/[project-slug] 2>/dev/null || git checkout spec/[project-slug]
   ```

4. Confirmar:
   > "Branch `spec/phase-N-[slug]` criada a partir de `[base]`. Iniciando Fase N — [nome]."

---

### Etapa 2 — Task breakdown

Se `.specs/execution/phase-N/TASKS.md` não existir, criar com Sonnet inline:

```markdown
# Fase N — [Nome]: Task Breakdown

**Goal**: [goal]
**Branch**: spec/phase-N-[slug]
**Gerado em**: [data]

## Tarefas

- [ ] **T1** `[haiku|sonnet|opus]` — [Descrição atômica e executável]
- [ ] **T2** `[sonnet]` — ...
- [ ] **TN** `[revisão]` — Executar spec-reviewer e validar conformidade

## Decisões pendentes
[lista]

## Success Criteria (da spec)
- [ ] [critério 1]
- [ ] [critério 2]
```

Apresentar e aguardar confirmação antes de executar.

---

### Etapa 3 — Execução tarefa a tarefa

Para cada tarefa não concluída, em sequência:

#### 3a. Anunciar
```
▶ Fase N / T[X] — [modelo] — [descrição]
```

#### 3b. Executar com o modelo correto

**`[haiku]`** — agente paralelo para tarefas independentes:
- Output format estruturado, sem prosa longa
- Lançar juntos quando independentes, aguardar todos

**`[sonnet]`** — inline na sessão atual:
- Ler arquivos relevantes antes de escrever
- Usar `rtk` em todo bash
- Não lançar agente separado se a tarefa cabe no contexto

**`[opus]`** — agente único por decisão real:
- Prompt autocontido com contexto + opções + critérios
- Resultado registrado em `.specs/execution/phase-N/DECISIONS.md`
- Máximo 1-2 por fase

#### 3c. Marcar concluída em TASKS.md
```markdown
- [x] **T1** `haiku` — [descrição] ✓
```

#### 3d. Commit por deliverable
```bash
rtk git add .
rtk git commit -m "feat(phase-N): [descrição do deliverable]

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>"
```

Na primeira execução da sessão, perguntar: commits automáticos (a cada deliverable) ou manuais?

#### 3e. Captura de aprendizados
Ao encontrar padrão novo, bug ou correção — capturar imediatamente:
```
/capture-learning "[descrição]"
```

---

### Etapa 4 — Validação de sucesso

Após todas as tarefas concluídas:

1. Para cada Success Criteria:
   - Mensurável → testar: `rtk [comando de teste/verificação]`
   - Qualitativo → revisar o deliverable produzido

2. Critério que falha → criar tarefa corretiva em TASKS.md, executar, re-validar.

3. Quando todos passarem → avançar para revisão.

---

### Etapa 5 — Revisão de conformidade com a spec

**Lançar agente `spec-reviewer` (Sonnet)** com:
- Conteúdo de `.specs/PROJECT_SPEC.md` e `.specs/PHASES.md` (fase N)
- `git diff [base-branch]...HEAD` — todas as mudanças desta fase
- Lista de deliverables esperados da fase

O agente verifica:
- ✅/❌ Cada deliverable da spec foi implementado?
- ✅/❌ O código está alinhado com os requisitos (funcionais e não-funcionais)?
- ✅/❌ Há trabalho fora do escopo desta fase?
- ✅/❌ As interfaces/APIs estão conforme especificado?
- Problemas de conformidade encontrados (com file:line)

Gravar resultado em `.specs/execution/phase-N/REVIEW.md`.

**Se o reviewer encontrar problemas de conformidade:**
```
⚠ Spec-reviewer encontrou N problemas de conformidade:
  1. [problema] — [arquivo:linha]
  2. ...

Deseja corrigir antes de abrir o PR? (recomendado) ou abrir o PR com estas pendências documentadas?
```

Aguardar decisão do desenvolvedor.

---

### Etapa 6 — Aprovação e abertura do PR

Apresentar resumo final ao desenvolvedor:

```
## Fase N — [Nome] pronta para PR

Branch: spec/phase-N-[slug]
Base:   [base-branch]
Commits: N commits

Deliverables:
  ✅ [deliverable 1]
  ✅ [deliverable 2]

Success Criteria:
  ✅ [critério 1]
  ✅ [critério 2]

Spec Review: [N problemas / Aprovado]

Deseja abrir o PR agora? (s/n)
Se não, execute `/exec-spec N` novamente quando estiver pronto.
```

**Aguardar aprovação explícita antes de qualquer push ou PR.**

Com aprovação:

1. Push da branch:
   ```bash
   git push -u origin spec/phase-N-[slug]
   ```

2. Criar PR com descrição completa:
   ```bash
   gh pr create \
     --base [base-branch] \
     --head spec/phase-N-[slug] \
     --title "feat(phase-N): [nome da fase] — [goal em uma linha]" \
     --body "$(cat <<'PREOF'
   ## Fase N — [Nome da Fase]

   > **Spec:** `.specs/PHASES.md` → Phase N
   > **Branch:** `spec/phase-N-[slug]`
   > **Execução:** [data início] → [data conclusão]

   ### Goal
   [goal da fase copiado da spec]

   ### Deliverables

   | Deliverable | Status |
   |-------------|--------|
   | [deliverable 1] | ✅ Implementado |
   | [deliverable 2] | ✅ Implementado |

   ### Success Criteria

   - [x] [critério 1]
   - [x] [critério 2]

   ### Decisões de arquitetura
   [lista das decisões do DECISIONS.md com justificativa resumida]
   > Detalhes completos: `.specs/execution/phase-N/DECISIONS.md`

   ### Spec Review
   [resultado resumido do REVIEW.md — aprovado ou pendências documentadas]
   > Relatório completo: `.specs/execution/phase-N/REVIEW.md`

   ### Mudanças
   [lista dos principais arquivos criados/modificados com uma linha de descrição cada]

   ### Aprendizados capturados
   [N aprendizados registrados em `.claude/learnings/` durante esta fase]

   ---
   🤖 Gerado com [Claude Code](https://claude.ai/code) via `/exec-spec`
   PREOF
   )"
   ```

3. Exibir URL do PR ao desenvolvedor.

---

### Etapa 7 — Atualizar PROGRESS.md e transição

Atualizar `.specs/execution/PROGRESS.md`:

```markdown
# Execution Progress — [nome do projeto]

**Última atualização**: [data]

## Git

| Fase | Branch | PR | Status |
|------|--------|----|--------|
| 1 | spec/phase-1-[slug] | #[N] — [URL] | ✅ Mergeado |
| 2 | spec/phase-2-[slug] | #[N] — [URL] | 🔍 Em revisão |
| 3 | — | — | ⬜ Não iniciada |

## Status por fase

| Fase | Nome | Status | Tarefas | Início | Conclusão |
|------|------|--------|---------|--------|-----------|
| 1 | [nome] | ✅ PR aberto | X/X | [data] | [data] |
| 2 | [nome] | 🔄 Em progresso | X/Y | [data] | — |

## Decisões tomadas
- Fase 1: [decisão resumida] → `.specs/execution/phase-1/DECISIONS.md`

## Notas para fases futuras
[observações registradas durante execução que afetam fases seguintes]
```

Informar ao desenvolvedor:
```
✅ PR aberto: [URL]

Próxima fase disponível: Fase N+1 — [nome]
  Goal: [goal]
  Depende de: [se aguarda merge desta fase ou pode rodar em paralelo]

Execute `/exec-spec [N+1]` quando estiver pronto.
```

**Não iniciar próxima fase automaticamente.**

---

## Regras gerais de comportamento

- **Branch antes de código** — nunca commitar na base-branch, sempre na branch da fase
- **Push e PR somente com aprovação** — a palavra "sim" ou "s" do desenvolvedor é obrigatória
- **Spec-reviewer é obrigatório** — não pular a Etapa 5, mesmo que pareça desnecessário
- **Modelo mais barato que resolve** — não escalar sem necessidade
- **Haiku paralelo** — tarefas independentes lançadas juntas
- **Opus máximo 1-2 por fase** — só para decisões reais com trade-offs
- **Commits frequentes e descritivos** — um por deliverable
- **Aprendizados imediatos** — capturar enquanto o contexto está fresco
- **Trabalho fora de escopo** → registrar em PROGRESS.md, não implementar agora
