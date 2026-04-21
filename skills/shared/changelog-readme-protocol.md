# Protocolo README & CHANGELOG

Protocolo compartilhado por todos os skills do plugin `claude-code-skills`. Seguir sempre que qualquer skill realizar uma alteração de código em um projeto do usuário.

---

## Gatilho

Atualizar `README.md` e `CHANGELOG.md` **ao final de qualquer sessão de trabalho que resulte em alteração de código**. Isso inclui:
- Adição de nova funcionalidade
- Correção de bug
- Refatoração que muda comportamento observável
- Adição ou remoção de dependências

Não atualizar para:
- Mudanças puramente internas sem efeito externo (renomear variável local, reformatar)
- Atualização de comentários sem mudança de comportamento
- Atualização dos próprios arquivos de spec (`.specs/`)

---

## CHANGELOG.md — formato Keep a Changelog

Seguir [keepachangelog.com](https://keepachangelog.com/pt-BR/1.0.0/).

### Estrutura

```markdown
# Changelog

Todas as mudanças notáveis neste projeto serão documentadas aqui.

O formato é baseado em [Keep a Changelog](https://keepachangelog.com/pt-BR/1.0.0/).

## [Unreleased]

### Added
- [feature adicionada]

### Changed
- [comportamento alterado]

### Fixed
- [bug corrigido]

### Removed
- [feature removida]

## [1.0.0] — YYYY-MM-DD

### Added
- Versão inicial
```

### Regras

- Sempre adicionar a mudança em `## [Unreleased]` — nunca em uma versão existente
- Usar a seção correta: `Added` para novo, `Changed` para alterado, `Fixed` para corrigido, `Removed` para removido
- Uma linha por mudança — concisa e orientada ao usuário, não ao implementador
- Se `## [Unreleased]` não existir no arquivo, criar acima da versão mais recente
- Data no formato `YYYY-MM-DD` (ex: `2026-04-21`)

---

## README.md — o que atualizar

O README deve refletir as **funcionalidades presentes** após a mudança — não funcionalidades planejadas.

### O que atualizar

- Seção de funcionalidades: adicionar nova feature ou marcar feature removida
- Exemplos de uso: atualizar se a interface mudou
- Instalação/configuração: atualizar se novos passos são necessários
- Seção de status/badges: se houver

### O que NÃO incluir

- Features planejadas ou "em breve"
- Roadmap futuro
- Mudanças internas de implementação sem efeito na interface

### Estrutura mínima de README

Se o projeto não tiver README, criar com esta estrutura mínima:

```markdown
# [Nome do Projeto]

[Uma linha descrevendo o que o projeto faz]

## O que faz

[2-3 bullet points das funcionalidades principais]

## Como usar

[Exemplo mínimo de uso]

## Instalação

[Passos necessários]
```

---

## Comportamento quando os arquivos não existem

### CHANGELOG.md ausente

Criar o arquivo com a estrutura completa (cabeçalho + `## [Unreleased]` + primeira entrada) antes de registrar a mudança atual.

### README.md ausente

Criar o arquivo com a estrutura mínima acima, preenchendo com as funcionalidades **já existentes** no projeto (não só a mudança atual).

### Ambos ausentes

Criar README primeiro, depois CHANGELOG. Commitar os dois junto com as mudanças de código.

---

## Exemplo de uso por skill

Ao final de uma sessão de trabalho com qualquer skill deste plugin:

```bash
# 1. Verificar se README.md e CHANGELOG.md existem
ls README.md CHANGELOG.md

# 2. Se CHANGELOG.md existir, adicionar entrada em [Unreleased]
# Se não existir, criar com estrutura completa

# 3. Se README.md existir, atualizar seção de funcionalidades se necessário
# Se não existir, criar com estrutura mínima

# 4. Commitar junto com as mudanças de código
git add README.md CHANGELOG.md
git commit -m "docs: update README and CHANGELOG"
```
