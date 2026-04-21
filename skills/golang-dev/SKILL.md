---
name: golang-dev
description: This skill should be used when the user is working with Go code, asks about Go patterns, debugging goroutine issues, designing interfaces, writing concurrent systems, gRPC APIs, CLI tools in Go, performance profiling, or asks "how do I do X in Go". Activates for .go files and Go project context (go.mod present).
version: 0.1.0
---

# Go Senior Dev Skill

Context loaded: working in a Go project with a senior developer.

---

## Comportamento esperado desta skill

Esta skill foi projetada para um desenvolvedor **sênior em Go**. O foco é idiomatismo, correção de concorrência e código que escala.

**O que fazer:**
- Escrever Go idiomático — não "Go que funciona" mas "Go que um Gopher reconhece como bem escrito"
- Raciocinar sobre tempo de vida de goroutines em qualquer código concorrente, sempre
- Apontar `file:line` ao discutir código existente
- Questionar uso de `interface{}` / `any` desnecessário — tipo concreto quase sempre é melhor
- Preferir stdlib antes de sugerir dependência externa
- Usar `rtk go test ./...` e `rtk go build ./...` para qualquer verificação bash

**O que não fazer:**
- Não reportar o que `gofmt`, `goimports` ou `golangci-lint` já pegariam
- Não sugerir abstrações para um único caso de uso
- Não usar `panic` em código de biblioteca — panics são para programação defensiva em main
- Não esquecer de checar o goroutine ownership antes de qualquer sugestão de concorrência
- Não usar Opus para tarefas de leitura de código simples — Haiku para review, Sonnet para design

**Princípio central:** em Go, um goroutine leak é silencioso e acumula. Concorrência sem modelo mental claro de ownership é código com bug esperando para manifestar.

---

## Agent routing (economia de tokens)

| Task | Agent | Model | Quando usar |
|------|-------|-------|-------------|
| Review rápido, pre-commit | `golang-reviewer` | Haiku | Sempre que a tarefa for "revise este arquivo/diff" |
| Arquitetura, debug, design | `golang-expert` | Sonnet | Quando precisa de raciocínio sobre design |
| Ferramental científico em Go | `research-validator` | Opus | Se o código Go tem função científica crítica |

**Padrão**: responder inline com Sonnet. Lançar agente para tarefas isoladas.

---

## Convenções assumidas

- `gofmt`/`goimports` no CI — não reportar formatação
- `golangci-lint` rodando — não reportar o que ele já pega
- Módulos via `go.mod`
- Stdlib preferida onde equivalente a third-party
- Erros sempre encapsulados: `fmt.Errorf("context: %w", err)`

---

## RTK — sempre

```bash
rtk go test ./...
rtk go build ./...
rtk git diff
```

---

## Modelo mental de concorrência

Antes de escrever qualquer goroutine, responder:
1. **Quem é o dono?** (quem inicia, quem para)
2. **Como ela termina?** (context cancel, done channel, exit do processo)
3. **O que acontece se der panic?** (recover? crash intencional?)
4. **O que ela compartilha?** (estado compartilhado precisa de proteção)

Se não der para responder as 4 perguntas, a goroutine não foi projetada ainda.

---

## Regras de interface

- Definir interfaces **onde são usadas**, não onde são implementadas
- Interfaces pequenas: 1-3 métodos é o ideal
- Retornar tipo concreto de constructors, aceitar interface em funções
- Compor `io.Reader`, `io.Writer`, `io.Closer`

---

## Regras de tratamento de erros

- Nunca `_` um erro não-trivial
- Encapsular com contexto: `fmt.Errorf("user.Find(%d): %w", id, err)`
- Sentinels: `var ErrNotFound = errors.New("not found")` — unexported se interno
- Tipos de erro: só quando callers precisam inspecionar programaticamente

---

## README & CHANGELOG

Ao final de qualquer sessão que resulte em alteração de código, atualizar `README.md` e `CHANGELOG.md` do projeto. Ver protocolo completo em **`skills/shared/changelog-readme-protocol.md`**.

---

## Additional Resources

- **`references/patterns.md`** — Concorrência, interfaces, CLI, gRPC, HTTP middleware
- **`references/pitfalls.md`** — Goroutine leaks, nil maps, defer em loop, variável de loop capturada
