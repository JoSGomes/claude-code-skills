# Fase 1 — Decisões

## D1 — Língua dos triggers no frontmatter

**Decisão**: usar inglês + português no campo `description` do frontmatter.

**Why:** A spec (RF-01) lista keywords em português (`modelo`, `treino`, `experimento`), mas o restante do repositório usa inglês. O Claude Code usa o `description` para ativação — incluir ambas as línguas garante cobertura para usuários que escrevem em português (língua principal de interação do projeto, RNF-02) sem perder cobertura para inglês.

---

## D2 — Nível de detalhe em patterns.md (1 exemplo canônico por área)

**Decisão**: 1 exemplo canônico por área técnica, sem variações.

**Why:** A spec (Seção 8, Riscos) alerta explicitamente que `patterns.md` pode "ficar grande demais e perder valor como referência rápida". Mitigação documentada: "Limitar cada seção a 1 exemplo canônico; sem variações".

---

## D3 — Sobreposição controlada do checklist com python-dev

**Decisão**: reexpandir o checklist de reprodutibilidade em `data-science-dev/SKILL.md` em vez de referenciar o de `python-dev`.

**Why:** O checklist DS tem 8 itens (vs. 4 do python-dev), incluindo itens DS-específicos (preprocessor leakage, test set único, notebook restart). Referenciar python-dev criaria uma dependência de navegação desnecessária. A expansão é justificada pelo escopo maior — não é duplicação, é especialização.
