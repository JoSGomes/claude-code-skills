# Exemplos de Entradas de Aprendizado

## Exemplo 1: Pitfall Python (alta qualidade)

**Arquivo**: `2026-04-20_python-dev_sklearn-leakage-in-pipeline.md`

```markdown
---
date: 2026-04-20
skill: python-dev
type: pitfall
confidence: high
tags: [data-leakage, sklearn, preprocessing, pipeline]
---

## Contexto
Revisando um pipeline de classificação de sentimentos. O modelo tinha 98% de acurácia
no teste mas 72% em produção. Causa: StandardScaler fitted no dataset completo antes
do split.

## Aprendizado
```python
# Código encontrado (errado)
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X_all)   # vaza estatísticas do test para train
X_train, X_test = train_test_split(X_scaled, test_size=0.2)

# Correto
X_train, X_test = train_test_split(X_all, test_size=0.2)
pipe = Pipeline([("scaler", StandardScaler()), ("clf", LogisticRegression())])
pipe.fit(X_train, y_train)  # scaler só vê X_train
```

## Regra derivada
Nunca fazer fit de preprocessors antes do split — sempre usar Pipeline ou fit explícito só no X_train.

## Melhoria de skill sugerida
Adicionar à references/pitfalls.md: variante com fit_transform antes do split (a mais comum na prática).
```

---

## Exemplo 2: Pattern Go (média qualidade, mas útil)

**Arquivo**: `2026-04-21_golang-dev_errgroup-bounded-concurrency.md`

```markdown
---
date: 2026-04-21
skill: golang-dev
type: pattern
confidence: high
tags: [concurrency, errgroup, semaphore, worker-pool]
---

## Contexto
Precisava processar 500 arquivos em paralelo sem sobrecarregar a API externa (max 10 req/s).
O padrão de channel semaphore com errgroup funcionou perfeitamente.

## Aprendizado
Combinar errgroup + semaphore channel é mais limpo que WaitGroup manual para bounded concurrency:

```go
g, ctx := errgroup.WithContext(ctx)
sem := make(chan struct{}, 10)
for _, f := range files {
    f := f
    g.Go(func() error {
        sem <- struct{}{}
        defer func() { <-sem }()
        return process(ctx, f)
    })
}
return g.Wait()
```

## Regra derivada
Para fan-out com bounded concurrency em Go: errgroup + semaphore channel > WaitGroup manual.

## Melhoria de skill sugerida
Adicionar à references/patterns.md como exemplo preferido de worker pool.
```

---

## Exemplo 3: Correction (skill deu conselho errado)

**Arquivo**: `2026-04-22_model-routing_haiku-insufficient-for-architecture.md`

```markdown
---
date: 2026-04-22
skill: model-routing
type: correction
confidence: medium
tags: [model-selection, haiku, architecture]
---

## Contexto
Tentei usar Haiku para analisar a arquitetura de um sistema de microserviços com 15 serviços
e dependências complexas. O agente perdeu conexões importantes entre os serviços.

## Aprendizado
A skill diz "use Haiku para leitura de arquivos e summarização". Mas quando a tarefa
de leitura envolve raciocínio sobre dependências cruzadas (não só extração), Haiku
falha em identificar conexões não-óbvias.

## Regra derivada
Haiku: extração simples (existe? qual valor? quais arquivos?).
Sonnet: leitura que requer raciocínio sobre relações entre elementos.

## Melhoria de skill sugerida
Refinar a tabela de model-routing: adicionar coluna "nível de raciocínio necessário" como critério secundário.
```

---

## Exemplo 4: Project-Convention (específico do projeto)

**Arquivo**: `2026-04-23_general_project-uses-loguru-not-stdlib-logging.md`

```markdown
---
date: 2026-04-23
skill: general
type: project-convention
confidence: high
tags: [logging, python, convention]
---

## Contexto
Ao sugerir adicionar logging ao módulo de processamento, usei stdlib `logging`.
O dev corrigiu: este projeto usa `loguru` com formato estruturado.

## Aprendizado
Este projeto usa `loguru` com output JSON. Padrão:
```python
from loguru import logger
logger.add("app.log", format="{time} {level} {message}", serialize=True)
logger.info("processing", file=path, records=n)
```

## Regra derivada
Não usar stdlib logging neste projeto — sempre loguru com serialize=True.

## Melhoria de skill sugerida
Nenhuma (convenção específica do projeto, não generalizável).
```

---

## O que faz uma boa entrada

1. **Contexto específico** — o que estava sendo feito, não apenas "encontrei um bug"
2. **Código concreto** — mostrar o errado E o correto quando relevante
3. **Regra em uma frase** — generalizável, não acoplada ao projeto específico
4. **Melhoria específica** — onde exatamente na skill isso deveria aparecer
