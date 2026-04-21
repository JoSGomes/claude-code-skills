# Spec Review — Fase 1: Core Skill DS + Agentes

**Data**: 2026-04-21
**Resultado**: APROVADO COM RESSALVAS — 1 problema crítico corrigido antes do PR

---

## Crítico Corrigido

**[C1]** `skills/data-science-dev/SKILL.md:3` — Triggers em português da spec (RF-01: `modelo`, `treino`, `experimento`) ausentes do frontmatter `description`. Corrigido adicionando os termos em português ao lado dos equivalentes em inglês.

---

## Avisos Corrigidos

**[W3]** `agents/data-science-expert.md` — Checklist de reprodutibilidade do agente tinha 6 itens vs. 8 do SKILL.md. Alinhado com o SKILL.md.

---

## Avisos Aceitos

**[W1]** `agents/data-science-expert.md:1` — "concurrent systems" na description do agente. Aceito: o campo description é usado pelo runtime para roteamento e a frase está em contexto de competências gerais — não cria ambiguidade com o perfil DS.

---

## Ambiguidades Documentadas

Ver `DECISIONS.md` desta fase para registro das decisões sobre:
- Língua dos triggers (inglês + português)
- Nível de detalhe dos exemplos em patterns.md (1 canônico por área)
- Sobreposição controlada do checklist com python-dev (expansão justificada)
