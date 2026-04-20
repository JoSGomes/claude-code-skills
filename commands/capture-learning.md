---
description: Captura um aprendizado da sessão atual em .claude/learnings/ do projeto. Use quando encontrar um padrão, bug, abordagem eficaz ou correção a uma skill.
argument-hint: [descrição breve do aprendizado]
allowed-tools: Read, Write, Bash(mkdir:*), Bash(date:*)
---

# /capture-learning — Captura de Aprendizado

Registra um aprendizado descoberto durante o trabalho no projeto atual.

Argumento recebido: `$ARGUMENTS`

## Passos

1. Determinar o diretório do projeto (cwd).

2. Criar `.claude/learnings/entries/` se não existir:
   ```bash
   mkdir -p .claude/learnings/entries
   ```

3. Identificar o skill relevante com base no contexto da sessão:
   - Trabalhando com `.py` → `python-dev`
   - Trabalhando com `.go` → `golang-dev`
   - Especificação de projeto → `project-spec`
   - Sobre custo/modelos → `model-routing`
   - Genérico → `general`

4. Determinar o tipo:
   - `pitfall` — bug ou erro encontrado no código
   - `pattern` — abordagem que funcionou bem
   - `correction` — skill deu conselho errado ou incompleto
   - `discovery` — algo novo não coberto pela skill
   - `project-convention` — convenção específica deste projeto

5. Se `$ARGUMENTS` for vazio ou insuficiente, perguntar:
   > "O que foi o aprendizado? Descreva brevemente o que aconteceu, por que importa e a regra que deriva disso."

6. Gerar nome do arquivo: `YYYY-MM-DD_[skill]_[slug].md`
   Exemplo: `2026-04-20_python-dev_data-leakage-in-pipeline.md`

7. Escrever o arquivo de aprendizado:

```markdown
---
date: [YYYY-MM-DD]
skill: [skill-name]
type: [pitfall|pattern|correction|discovery|project-convention]
confidence: [high|medium]
tags: [tag1, tag2, tag3]
---

## Contexto
[O que estava sendo feito quando este aprendizado foi descoberto]

## Aprendizado
[O insight específico — concreto, com código se relevante]

## Regra derivada
[A regra generalizável em uma frase]

## Melhoria de skill sugerida
[O que deveria ser adicionado/modificado na skill — ou "nenhuma"]
```

8. Atualizar `.claude/learnings/LEARNINGS.md` (criar se não existir) adicionando uma linha no topo:
   ```
   - [YYYY-MM-DD] [type] [skill]: [regra derivada em uma linha] → [arquivo]
   ```

9. Confirmar:
   > "Aprendizado capturado em `.claude/learnings/entries/[filename]`."
   > "Para revisar todos os aprendizados e sugestões de melhoria de skills: `/review-learnings`"
