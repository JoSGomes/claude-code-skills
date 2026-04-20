# Classificação de Tarefas por Modelo

## Princípio

O modelo mais barato que resolve o problema é sempre o correto. A escala de custo relativo:
- Haiku = 1×
- Sonnet = ~5×
- Opus = ~15×

Antes de escalar para um modelo mais caro, perguntar: "um modelo mais leve consegue fazer isso com qualidade suficiente?"

---

## Haiku — Extração e verificação

**Usar quando a tarefa é determinar um fato ou extrair informação estruturada.**

Haiku é excelente para:
- Encontrar arquivos por padrão (`Glob`, `Grep`)
- Ler um arquivo e extrair seções específicas
- Checar se uma condição é verdadeira ("este arquivo tem testes?")
- Resumir um documento longo em pontos-chave
- Verificar critérios de sucesso objetivos ("os testes passam?")
- Revisar diff para issues de alta confiança

Haiku falha quando:
- A tarefa requer raciocínio sobre relações não-óbvias entre componentes
- A extração depende de julgamento sobre o que é relevante (ambíguo)
- O output precisa de código gerado, não só encontrado

**Padrão de execução:** lançar como agente com output format estruturado.

```
Prompt pattern:
"[tarefa específica]. Retornar em formato:
[campo1]: valor
[campo2]: valor
Não retornar prosa."
```

**Paralelismo:** tarefas Haiku independentes rodam em paralelo — lançar juntas, aguardar todas.

---

## Sonnet — Implementação e debug

**Usar quando a tarefa requer geração, modificação, ou raciocínio sobre código.**

Sonnet é o padrão para:
- Escrever código novo (funções, módulos, classes)
- Refatorar código existente
- Debugar bugs conhecidos
- Escrever testes
- Adaptar código a uma nova interface
- Explicar ou documentar código
- Qualquer tarefa onde o resultado é um arquivo modificado

Sonnet **inline** (sessão atual) é preferível a agente Sonnet separado quando:
- Os arquivos relevantes cabem no contexto (já foram lidos)
- A tarefa é uma continuação do trabalho em andamento
- Não há necessidade de isolamento

Agente Sonnet é preferível quando:
- A tarefa é genuinamente isolada (ex: "revise apenas este módulo")
- É necessário um contexto limpo sem o histórico da sessão atual

---

## Opus — Decisões e raciocínio profundo

**Usar apenas quando a decisão tem consequências de longo prazo e há genuína ambiguidade.**

Opus é justificado para:
- Escolher entre duas ou mais arquiteturas com trade-offs reais
- Analisar requisitos ambíguos com múltiplas interpretações válidas
- Validar rigor científico de um pipeline de pesquisa
- Resolver conflitos entre requisitos que afetam múltiplas fases

Opus **não** é justificado para:
- Leitura de código (use Haiku)
- Implementação de código onde a solução é clara (use Sonnet)
- Revisão de PR simples (use Haiku/Sonnet)
- Qualquer tarefa onde Sonnet produziria o mesmo resultado

**Padrão de execução:** sempre como agente, com prompt autocontido.

```
Prompt pattern:
"CONTEXTO: [descrição completa do sistema e estado atual]
DECISÃO NECESSÁRIA: [o que precisa ser decidido]
OPÇÕES DISPONÍVEIS:
  A) [opção com trade-offs]
  B) [opção com trade-offs]
CRITÉRIOS: [o que importa para esta decisão]
Retornar: decisão escolhida + justificativa + o que foi descartado e por quê."
```

Resultado vai para `.specs/execution/phase-N/DECISIONS.md`.

---

## Casos limítrofes

### "Explorar e depois implementar" — Haiku → Sonnet

```
T1 [haiku] — Mapear estrutura do módulo X e identificar pontos de extensão
T2 [sonnet] — Implementar nova feature Y nos pontos identificados em T1
```

Sequencial: esperar T1 antes de iniciar T2.

### "Debug complexo" — Sonnet com escalada para Opus

Iniciar com Sonnet. Se após 2 tentativas a causa raiz não estiver clara:
- Parar
- Preparar contexto completo (logs, código, hipóteses testadas)
- Escalar para Opus com esse contexto
- Registrar causa raiz em DECISIONS.md para evitar repetição

### "Validação científica" — Haiku (verificar) + Opus (julgar)

```
T_val_haiku [haiku]  — Rodar suite de testes, coletar métricas, verificar seeds
T_val_opus  [opus]   — Avaliar se os resultados são estatisticamente válidos
```

Haiku coleta os fatos; Opus julga se são suficientes.

---

## Indicadores de classificação errada

| Sinal | Problema | Correção |
|-------|----------|----------|
| Haiku retornou resposta vaga sem estrutura | Tarefa requer raciocínio, não extração | Reescalar para Sonnet |
| Sonnet propôs 3 abordagens diferentes sem escolher | Decisão com trade-offs reais | Escalar para Opus |
| Opus usado para "ler este arquivo" | Custo desnecessário | Rebaixar para Haiku |
| Agente Sonnet lançado para tarefa que cabe no contexto | Overhead desnecessário | Executar inline |
