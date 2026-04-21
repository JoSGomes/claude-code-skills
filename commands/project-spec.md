---
description: Guided project specification — transforms a project plan into a structured, phased specification through interactive dialogue. Reads documents with Haiku, analyzes requirements and designs phases with Opus. Writes output to .specs/ in the project.
argument-hint: [path-to-plan-file | "project description" | URL]
allowed-tools: Read, Glob, Grep, Bash(mkdir:*), Write, Agent
---

# /project-spec — Project Specification Workflow

Guia o desenvolvedor por uma especificação estruturada. Transforma uma ideia ou plano em uma especificação faseada e documentada.

**Input**: `$ARGUMENTS`

**Output**: arquivos em `.specs/` no diretório atual do projeto.

---

## Workflow (6 fases)

```
Fase 1: Coleta de input        → entender o que o dev tem
Fase 2: Leitura de documentos  → Haiku (barato, paralelo)
Fase 3: Análise de requisitos  → Opus (profundo, uma vez)
Fase 4: Diálogo interativo     → confirmar antes de gastar mais
Fase 5: Planejamento de fases  → Opus (profundo, uma vez)
Fase 6: Output para .specs/    → gravar tudo no projeto
```

---

## Fase 1: Coleta de Input

1. Ler `$ARGUMENTS`. Identificar se é: caminho de arquivo, URL, texto livre, ou vazio.

2. Se vazio, perguntar:
   > "O que você está construindo? Cole uma descrição, caminho de arquivo ou URL do plano."

3. Perguntas iniciais (todas de uma vez):
   > "Algumas perguntas rápidas:
   > 1. É um produto de software, projeto de pesquisa, ou ambos?
   > 2. Há prazo ou horizonte de tempo?
   > 3. Público alvo: você mesmo, equipe, usuários externos, comunidade científica?
   > 4. Há arquivos, código existente, artigos ou URLs que devo ler para entender o contexto?"

4. Coletar respostas e avançar.

---

## Fase 2: Leitura de Documentos (Haiku)

Se arquivos, URLs ou documentos foram fornecidos:

1. Lançar agentes **document-reader** (Haiku) em paralelo — um por documento.
   - Cada agente extrai: requisitos, restrições, tecnologias, datasets, questões abertas.

2. Compilar os resumos e apresentar digest compacto ao desenvolvedor:
   > "Processei [N] documentos. Pontos relevantes: [digest]
   > Algo que devo priorizar mais nesses docs?"

---

## Fase 3: Análise de Requisitos (Opus)

**Detecção de projeto científico/DS**: antes de lançar o agente, verificar se o projeto tem natureza científica. Indicadores — qualquer um presente ativa o modo científico:
- Keywords no input: "pesquisa", "research", "dataset", "modelo", "model", "treino", "training", "experimento", "experiment", "EDA", "machine learning", "deep learning", "NLP", "dados", "data", "accuracy", "reprodutibilidade"
- Tipo de projeto informado na Fase 1: "projeto de pesquisa" ou "ambos"
- Documentos fornecidos contêm: datasets, métricas de avaliação, hipóteses, baselines

**Se projeto científico/DS detectado**, adicionar nota visível ao desenvolvedor antes de lançar o agente:
> "Detectei que este é um projeto científico/DS. A análise de requisitos será roteada para o `research-validator` (Opus) além do `requirements-analyst`, garantindo que requisitos de reprodutibilidade, rigor estatístico e data leakage sejam identificados explicitamente."

1. Lançar **requirements-analyst** (Opus) com:
   - Descrição do projeto + respostas da Fase 1 + resumos da Fase 2.
   - Se projeto científico/DS: incluir instrução explícita para identificar requisitos científicos (seeds, splits, reprodutibilidade, métricas de avaliação, data leakage risks).

2. O agente produz: funcionais, não-funcionais, requisitos científicos (se pesquisa), restrições, ambiguidades, riscos.

3. Não apresentar output bruto. Sintetizar internamente para a Fase 4.

---

## Fase 4: Diálogo Interativo

**Esta é a fase mais importante. Não pular.**

1. Da análise, compilar todas as questões abertas agrupadas por tema:
   - **Escopo** (o que está in/out)
   - **Técnico** (stack, arquitetura, restrições)
   - **Pesquisa** (hipóteses, dados, métricas) — só se projeto científico
   - **Prioridades** (o que mais importa se o tempo apertar)

2. Apresentar perguntas em grupos, com sugestão padrão para cada uma:
   > "Identifiquei [N] pontos para confirmar antes de desenhar as fases:
   >
   > **Escopo:**
   > 1. [Pergunta] — minha sugestão se não tiver preferência: [recomendação]
   >
   > **Técnico:**
   > 2. ..."

3. Esperar as respostas. **Não avançar sem elas.**

4. Para "tanto faz" ou "não sei": propor explicitamente e pedir confirmação.

5. Resumir o entendimento e confirmar:
   > "Antes de desenhar as fases, confirme se entendi certo:
   > - **O que estamos construindo**: ...
   > - **Fora de escopo**: ...
   > - **Restrições chave**: ...
   > - **Sucesso significa**: ...
   >
   > Correto?"

---

## Fase 5: Planejamento de Fases (Opus)

1. Lançar **phase-planner** (Opus) com: requisitos completos + respostas confirmadas + resumo do projeto.

2. O agente desenha fases com: deliverables, critérios de sucesso, dependências, estimativas.

3. Apresentar plano ao desenvolvedor com pontos de feedback:
   > "Aqui está o plano faseado: [resumo]
   > Quero seu feedback em: [fases com incerteza, trade-offs embutidos]"

4. Iterar até confirmação.

---

## Fase 6: Output em `.specs/`

Após confirmação do plano:

1. Criar diretório:
   ```bash
   mkdir -p .specs
   ```

2. Derivar o título simplificado do projeto para uso nos nomes de arquivo:
   - Pegar o nome do projeto confirmado na Fase 4
   - Remover artigos e preposições (o, a, os, as, de, do, da, em, e, para, the, of, in, for)
   - Substituir espaços e símbolos especiais por hifens
   - Converter para lowercase
   - Obter a data atual de `currentDate` no contexto (formato `YYYY-MM-DD`)
   - Exemplo: "Data Science Skill + Agent Pipeline" → `data-science-skill-agent-pipeline`
   - Nome final: `PROJECT_SPEC_data-science-skill-agent-pipeline_2026-04-21.md`

3. Gravar `.specs/PROJECT_SPEC_[titulo-simplificado]_[YYYY-MM-DD].md`:

```markdown
# Project Specification: [Nome do Projeto] — YYYY-MM-DD

**Data**: [data atual]
**Versão**: 1.0
**Status**: Draft

## 1. Visão Geral
## 2. Objetivos e Critérios de Sucesso
## 3. Escopo
### In Scope
### Out of Scope
## 4. Requisitos
### Funcionais
### Não-Funcionais
### Científicos/de Pesquisa (se projeto científico/DS detectado na Fase 3)
## 5. Restrições e Premissas
## 6. Decisões Técnicas
## 7. Plano Faseado
[breakdown completo da Fase 5]
## 8. Riscos e Mitigações
## 9. Questões em Aberto
```

4. Gravar `.specs/PHASES_[titulo-simplificado]_[YYYY-MM-DD].md` com apenas o breakdown de fases para referência rápida.

5. Gravar `.specs/README.md`:
```markdown
# Specs

Especificações geradas pelo workflow `/project-spec`.

| Arquivo | Descrição |
|---------|-----------|
| PROJECT_SPEC_[titulo]_[data].md | Especificação completa |
| PHASES_[titulo]_[data].md | Breakdown de fases para referência rápida |
```

6. Perguntar:
   > "Especificação gravada em `.specs/`. Deseja também:
   > - [ ] Gerar `.specs/TODO_PHASE1.md` com as tarefas da Fase 1?
   > - [ ] Criar estrutura de diretórios inicial do projeto?
   > - [ ] Capturar aprendizados desta especificação? (`/capture-learning`)"

7. Executar o que for confirmado.

---

## Regras de comportamento

- Perguntas em lotes, nunca uma a uma.
- Nunca assumir — ambiguidade não resolvida vira questão aberta documentada.
- Confirmar entendimento antes de cada fase cara (Opus).
- Specs vão sempre para `.specs/` — nunca na raiz do projeto.
