---
description: Revisa aprendizados capturados no projeto, aplica melhorias nos arquivos de referência das skills e abre um PR no repositório do plugin. Síntese via Haiku (barato), edições e PR via git.
argument-hint: [skill-name para filtrar, ou vazio para todos]
allowed-tools: Read, Glob, Write, Edit, Bash(git:*), Bash(gh:*), Bash(mkdir:*), Bash(date:*), Bash(ls:*), Agent
---

# /review-learnings — Revisão de Aprendizados + PR de Melhoria

Lê os aprendizados capturados em `.claude/learnings/` do projeto atual, sintetiza via agente Haiku, aplica as melhorias nos arquivos do plugin e abre um Pull Request.

**Filtro (opcional):** `$ARGUMENTS`

**Plugin repo:** `C:/Users/pbexp/Documents/go/src/github.com/claude-code-skills`
**GitHub repo:** `JoSGomes/claude-code-skills`

---

## Fase 1 — Verificação

1. Verificar se `.claude/learnings/entries/` existe e tem arquivos:
   ```bash
   ls .claude/learnings/entries/ 2>/dev/null
   ```
   Se vazio ou não existir, informar:
   > "Nenhum aprendizado capturado neste projeto ainda. Use `/capture-learning` durante o trabalho."
   E parar.

2. Identificar o nome do projeto atual (nome do diretório cwd).

---

## Fase 2 — Síntese (Haiku)

Lançar agente **learning-synthesizer** (Haiku) com:
- Caminho: `.claude/learnings/entries/`
- Filtro de skill: `$ARGUMENTS` (ou "todos" se vazio)

O agente retorna:
- Agrupamento por skill
- Padrões recorrentes
- Correções necessárias na skill
- **Action items específicos** — quais arquivos editar e o que adicionar

Apresentar síntese ao usuário de forma compacta.

---

## Fase 3 — Confirmação das mudanças

Apresentar as melhorias propostas como checklist:

> "Baseado nos aprendizados, proponho as seguintes melhorias no plugin:
>
> **python-dev / references/pitfalls.md**
> - [ ] Adicionar: [conteúdo específico]
>
> **golang-dev / references/patterns.md**
> - [ ] Adicionar: [conteúdo específico]
>
> Quais itens deseja incluir no PR? (responda com números, 'todos', ou 'nenhum')"

Esperar confirmação antes de fazer qualquer edição.

---

## Fase 4 — Branch no plugin repo

Com os itens confirmados:

1. Ir para o plugin repo e verificar estado:
   ```bash
   git -C /c/Users/pbexp/Documents/go/src/github.com/claude-code-skills status
   git -C /c/Users/pbexp/Documents/go/src/github.com/claude-code-skills fetch origin
   ```

2. Verificar se há mudanças não commitadas. Se houver, alertar o usuário antes de prosseguir.

3. Criar branch a partir de `main`:
   ```bash
   git -C /c/Users/pbexp/Documents/go/src/github.com/claude-code-skills checkout main
   git -C /c/Users/pbexp/Documents/go/src/github.com/claude-code-skills pull origin main
   git -C /c/Users/pbexp/Documents/go/src/github.com/claude-code-skills checkout -b learnings/[project-name]-[YYYY-MM-DD]
   ```
   Exemplo de branch: `learnings/my-ml-pipeline-2026-04-20`

---

## Fase 5 — Aplicar melhorias

Para cada item confirmado, editar os arquivos correspondentes no plugin repo:

- Arquivos alvo ficam em:
  `/c/Users/pbexp/Documents/go/src/github.com/claude-code-skills/skills/[skill]/references/[arquivo].md`

**Regras de edição:**
- Adicionar ao final da seção mais relevante (não no topo)
- Manter o estilo do arquivo existente — ler antes de editar
- Incluir código de exemplo quando o aprendizado tiver código concreto
- Atribuir origem: `<!-- aprendizado: [project-name], [data] -->` como comentário HTML após o bloco adicionado

Após cada edição, confirmar o que foi alterado.

---

## Fase 6 — Commit e PR

1. Staged e commit:
   ```bash
   git -C /c/Users/pbexp/Documents/go/src/github.com/claude-code-skills add skills/
   git -C /c/Users/pbexp/Documents/go/src/github.com/claude-code-skills commit -m "$(cat <<'EOF'
   docs(skills): melhorias baseadas em aprendizados de [project-name]

   Aprendizados capturados durante trabalho no projeto [project-name].
   Skills atualizadas: [lista de skills].

   Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
   EOF
   )"
   ```

2. Push da branch:
   ```bash
   git -C /c/Users/pbexp/Documents/go/src/github.com/claude-code-skills push -u origin learnings/[project-name]-[YYYY-MM-DD]
   ```

3. Criar PR via `gh`:
   ```bash
   gh pr create \
     --repo JoSGomes/claude-code-skills \
     --title "docs(skills): melhorias de aprendizados — [project-name]" \
     --body "$(cat <<'EOF'
   ## Origem

   Aprendizados capturados durante trabalho no projeto **[project-name]** em [data].

   ## Skills atualizadas

   [lista com bullet por skill e o que foi adicionado]

   ## Mudanças

   [detalhe por arquivo: o que foi adicionado e por quê]

   ## Como revisar

   Verificar se os exemplos de código são corretos e se o estilo é consistente com o restante do arquivo.

   ---
   🤖 Gerado com [Claude Code](https://claude.ai/code) via `/review-learnings`
   EOF
   )"
   ```

4. Se `gh` não estiver disponível, mostrar:
   > "PR criado manualmente em: https://github.com/JoSGomes/claude-code-skills/compare/learnings/[branch]"

5. Informar o usuário com o link do PR.

---

## Fase 7 — Atualizar índice de aprendizados

Marcar as entradas processadas no índice `.claude/learnings/LEARNINGS.md` do projeto:

Adicionar ao topo do arquivo:
```markdown
## PR de melhorias gerado
**Data**: [data]
**Branch**: learnings/[project-name]-[data]
**PR**: https://github.com/JoSGomes/claude-code-skills/pull/[N]
**Skills atualizadas**: [lista]

---
```

Isso evita reprocessar os mesmos aprendizados em runs futuras.

---

## Regras de comportamento

- Nunca fazer commit ou push sem confirmação explícita do usuário (Fase 3)
- Ler cada arquivo de referência antes de editar — nunca sobrescrever cegamente
- Um PR por projeto/data — verificar se já existe branch para esse projeto antes de criar nova
- Se não houver nenhuma melhoria concreta (só confirmações de que a skill já está correta), não criar PR — informar ao usuário
