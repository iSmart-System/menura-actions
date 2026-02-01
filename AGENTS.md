# AGENTS.md

> Instruções para agentes de IA que trabalham neste repositório.
> Compatível com Claude Code, GitHub Copilot, Cursor, Codex e outros.

## Project Overview

Este é o **menura-actions**, repositório central de governança de pipelines CI/CD para repositórios **Codebase** da organização Menura (iSmart-System). Contém workflows reutilizáveis do GitHub Actions para repositórios que contêm código fonte de aplicações.

### Propósito

- Padronizar pipelines de CI/CD para repositórios Codebase
- Automatizar gestão de releases e tags
- Garantir governança e qualidade nas releases
- Gerar artefatos (.zip) e publicar no GitHub Releases

> **Nota:** Repositórios de infraestrutura (Terraform/Terragrunt) mantêm suas próprias pipelines localmente.

### Stack Técnica

- **Plataforma:** GitHub Actions
- **Linguagem:** YAML (workflows), Bash (scripts)
- **Padrão de Versionamento:** SemVer (vX.Y.Z)
- **Branches:** `sandbox` (staging), `main` (production)
- **Tech Stacks Suportadas:** Node.js, Bun

## Project Structure

```
menura-actions/
├── .github/
│   └── workflows/
│       ├── codebase-ci-node.yml           # CI para projetos Node.js
│       ├── codebase-ci-bun.yml            # CI para projetos Bun
│       ├── codebase-release-node.yml      # Gera artefatos e GitHub Release (Node.js)
│       ├── codebase-release-bun.yml       # Gera artefatos e GitHub Release (Bun)
│       ├── codebase-create-rc.yml         # Cria tags de Release Candidate
│       ├── codebase-qualify-rc.yml           # Qualifica RC como release de produção
│       └── codebase-validate-tag.yml      # Valida nomenclatura de tags
├── docs/
│   ├── tutorials/                         # Guias de aprendizado passo-a-passo
│   ├── how-to/                            # Guias práticos para tarefas específicas
│   ├── reference/                         # Documentação técnica detalhada
│   └── explanation/                       # Conceitos e arquitetura
├── examples/
│   └── codebase-project/                  # Exemplos para copiar nos projetos
├── .gitignore
├── AGENTS.md                              # Este arquivo
└── README.md                              # Documentação principal
```

## Build & Test Commands

```bash
# Validar syntax YAML dos workflows
yamllint .github/workflows/

# Testar workflows localmente (requer 'act' instalado)
act -l                                # Listar workflows
act -j <job-name>                     # Executar job específico
act push                              # Simular evento push
act workflow_dispatch                 # Simular dispatch manual

# Verificar erros de formatação
prettier --check "**/*.yml"
```

## Code Style Guidelines

### YAML

```yaml
# Indentar com 2 espaços
# Usar aspas simples para strings quando necessário
# Comentários em português, explicativos

name: Nome do Workflow

on:
  workflow_call:
    inputs:
      exemplo:
        description: 'Descrição clara do input'
        required: false
        type: string
        default: 'valor-padrao'
```

### Nomenclatura

| Elemento | Padrão | Exemplo |
|----------|--------|---------|
| Arquivos de workflow | `{arquétipo}-{nome}.yml` ou `{arquétipo}-{nome}-{tech}.yml` | `codebase-ci-node.yml`, `codebase-ci-bun.yml` |
| Nome de jobs | `kebab-case` | `validate-branch` |
| Nome de steps | Português, capitalizado | `Validar Branch` |
| Variáveis de ambiente | `SCREAMING_SNAKE_CASE` | `PROD_TAG` |
| Inputs de workflow | `kebab-case` | `node-version`, `bun-version` |

### Scripts Bash em Workflows

```yaml
- name: Exemplo de Script
  run: |
    # Sempre adicionar comentários explicativos
    echo "🚀 Iniciando processo..."

    # Usar variáveis entre chaves para clareza
    TAG="${{ github.ref_name }}"

    # Validações com mensagens claras
    if [[ ! "$TAG" =~ ^v[0-9]+\.[0-9]+\.[0-9]+$ ]]; then
      echo "::error::Tag inválida: $TAG"
      exit 1
    fi
```

## Git Workflow

### Branches

- `main` → Produção (protegida, requer PR aprovado)
- `sandbox` → Homologação (protegida, requer PR aprovado) - apenas Codebase
- `feat/*` → Features (temporárias)
- `fix/*` → Correções (temporárias)

### Fluxo de Commits

```bash
# Formato: tipo(escopo): descrição
feat(codebase): adicionar validação de lint
fix(config): corrigir variável de ambiente
docs(readme): atualizar instruções de uso
refactor(workflows): simplificar job de build
```

### Tags (Apenas Codebase)

```
v1.0.0-rc.1  → Release Candidate (criada em sandbox)
v1.0.0       → Release de Produção (após merge em main)
```

## Testing Instructions

### Antes de Abrir PR

1. Validar syntax YAML: `yamllint .github/workflows/`
2. Verificar se inputs têm valores default (retrocompatibilidade)
3. Testar localmente com `act` se possível
4. Atualizar documentação em `docs/` se necessário
5. Atualizar exemplos em `examples/` se necessário

### Checklist de Review

- [ ] Workflow tem header descritivo
- [ ] Todos os inputs estão documentados
- [ ] Steps têm nomes descritivos em português
- [ ] Secrets não são expostos em logs
- [ ] Erros têm mensagens claras (`::error::`)
- [ ] Workflow segue o prefixo `codebase-`

## Security Considerations

### Secrets

- NUNCA fazer log de secrets
- Usar `secrets: inherit` para passar secrets aos workflows chamados
- Secrets sensíveis devem estar em GitHub Secrets, não no código

### Validações

- Sempre validar inputs antes de usar
- Sanitizar variáveis usadas em comandos shell
- Verificar formato de tags e branches antes de operações críticas

### Exemplo de Validação Segura

```yaml
- name: Validar Input
  run: |
    # Validar formato antes de usar
    if [[ ! "${{ inputs.tag }}" =~ ^v[0-9]+\.[0-9]+\.[0-9]+(-rc\.[0-9]+)?$ ]]; then
      echo "::error::Formato de tag inválido"
      exit 1
    fi
```

## Boundaries & Constraints

### O que este repositório NÃO deve conter

- Código de aplicação (apenas workflows)
- Secrets hardcoded
- Configurações específicas de projetos individuais
- Lógica de negócio

### O que este repositório DEVE conter

- Workflows reutilizáveis genéricos
- Documentação de uso
- Exemplos para projetos
- Scripts de automação de release

### Retrocompatibilidade

- Novos inputs DEVEM ter `required: false` e `default`
- Breaking changes requerem nova major version
- Deprecações devem ser documentadas e comunicadas

## Workflows Reference

| Workflow | Propósito |
|----------|-----------|
| `codebase-ci-node.yml` | Pipeline de CI para projetos Node.js (lint, tests, build) |
| `codebase-ci-bun.yml` | Pipeline de CI para projetos Bun (lint, tests, build) |
| `codebase-release-node.yml` | Gera artefatos .zip e publica no GitHub Releases (Node.js) |
| `codebase-release-bun.yml` | Gera artefatos .zip e publica no GitHub Releases (Bun) |
| `codebase-create-rc.yml` | Cria tag de Release Candidate |
| `codebase-qualify-rc.yml` | Qualifica RC como release de produção |
| `codebase-validate-tag.yml` | Valida nomenclatura de tags |

## Development Environment

### Ferramentas Recomendadas

```bash
# Instalar yamllint para validação
pip install yamllint

# Instalar act para testar workflows localmente
brew install act  # macOS
# ou
curl https://raw.githubusercontent.com/nektos/act/master/install.sh | sudo bash

# Instalar GitHub CLI
brew install gh
```

### VS Code Extensions

- YAML (Red Hat)
- GitHub Actions (GitHub)
- EditorConfig

## Pull Request Guidelines

### Título do PR

```
tipo(escopo): descrição breve

Exemplos:
feat(codebase): adicionar suporte a Python
fix(config): corrigir timeout em produção
docs(readme): melhorar seção de instalação
```

### Descrição do PR

```markdown
## Resumo
[Descrição clara da mudança]

## Motivação
[Por que essa mudança é necessária]

## Mudanças
- [Lista de alterações]

## Checklist
- [ ] Testado localmente
- [ ] Documentação atualizada
- [ ] Retrocompatível
```

## Common Tasks

### Adicionar Novo Workflow

1. Criar arquivo em `.github/workflows/codebase-{nome}.yml` ou `.github/workflows/codebase-{nome}-{tech}.yml`
2. Usar `on: workflow_call` para torná-lo reutilizável
3. Documentar todos os inputs com `description`
4. Criar exemplo em `examples/codebase-project/`
5. Adicionar entrada em `docs/reference/workflows.md`
6. Atualizar README.md

### Adicionar Suporte a Nova Tech Stack

1. Criar workflow em `.github/workflows/codebase-ci-{tech}.yml`
2. Configurar setup da runtime específica (Node, Bun, Java, etc.)
3. Adaptar comandos de instalação, lint, teste e build
4. Documentar inputs específicos (ex: `bun-version`)
5. Adicionar exemplos para a nova stack
6. Atualizar documentação de referência

### Adicionar Novo Input a Workflow Existente

1. Adicionar input com `required: false` e `default`
2. Atualizar header do workflow com documentação
3. Atualizar exemplos em `examples/`
4. Atualizar `docs/reference/` correspondente

### Testar Mudança em Projeto Real

1. Criar branch com a mudança
2. Em um projeto piloto, referenciar a branch:
   ```yaml
   # Para Node.js
   uses: iSmart-System/menura-actions/.github/workflows/codebase-ci-node.yml@sua-branch

   # Para Bun
   uses: iSmart-System/menura-actions/.github/workflows/codebase-ci-bun.yml@sua-branch
   ```
3. Validar comportamento
4. Após aprovação, fazer merge

---

*Este arquivo segue a especificação [AGENTS.md](https://agents.md/), mantida pela Agentic AI Foundation sob a Linux Foundation.*
