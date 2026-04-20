# Padrões de Commit Durante Execução de Spec

## Princípio

Cada commit deve descrever um deliverable concreto, não o trabalho em si. "feat: adiciona autenticação" é melhor que "wip: working on auth".

Commits frequentes e pequenos são preferíveis a commits grandes. A regra: commitar ao completar cada tarefa do TASKS.md que produza um artefato.

---

## Convenção de prefixo por tipo de tarefa

| Tipo | Prefixo | Exemplo |
|------|---------|---------|
| Nova funcionalidade | `feat` | `feat(phase-2): implementa endpoint de autenticação` |
| Correção encontrada durante validação | `fix` | `fix(phase-2): corrige validação de token expirado` |
| Testes | `test` | `test(phase-2): adiciona testes de integração para auth` |
| Configuração/infra | `chore` | `chore(phase-1): configura ambiente Docker` |
| Documentação e decisões | `docs` | `docs(phase-1): registra decisão de arquitetura — JWT vs session` |
| Refatoração sem mudança de comportamento | `refactor` | `refactor(phase-3): extrai lógica de parsing para módulo separado` |
| Atualização de estado de progresso | `chore` | `chore: atualiza PROGRESS.md — fase 2, 4/7 tarefas` |

---

## Escopo no commit

Incluir sempre `(phase-N)` no escopo para rastreabilidade:

```bash
# Bom
rtk git commit -m "feat(phase-2): implementa pipeline de ingestão de dados CSV"

# Ruim — sem rastreabilidade de fase
rtk git commit -m "feat: adiciona ingestão"
```

---

## Quando commitar

**Sempre commitar após:**
- Completar uma tarefa `[sonnet]` que criou ou modificou arquivo
- Completar um bloco de tarefas `[haiku]` que gerou insights documentados
- Uma decisão Opus ser registrada em DECISIONS.md
- A validação de um critério de sucesso passar
- PROGRESS.md ser atualizado com novo estado

**Não commitar:**
- Trabalho pela metade (tarefa incompleta)
- Código que falha nos testes
- Apenas mudanças em TASKS.md (sem código)

---

## Commits de checkpoint de fase

Ao concluir uma fase inteira, fazer um commit de marco:

```bash
rtk git commit -m "$(cat <<'EOF'
feat(phase-N): conclui fase N — [nome da fase]

Deliverables produzidos:
- [deliverable 1]
- [deliverable 2]

Critérios de sucesso validados:
- [x] [critério 1]
- [x] [critério 2]

Decisões tomadas: ver .specs/execution/phase-N/DECISIONS.md
Aprendizados: ver .claude/learnings/entries/

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
EOF
)"
```

---

## Commits de decisão de arquitetura

Decisões Opus merecem commit próprio antes da implementação:

```bash
# Primeiro: commitar a decisão
rtk git commit -m "docs(phase-N): decide [tema] — [opção escolhida resumida]"

# Depois: commitar a implementação
rtk git commit -m "feat(phase-N): implementa [tema] conforme decisão"
```

Isso permite reverter a implementação sem perder o registro da decisão.

---

## Mensagem de co-autoria

Sempre incluir em commits gerados pelo Claude:

```
Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
```
