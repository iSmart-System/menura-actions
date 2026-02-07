# AGENTS.md

> Instruções para agentes de IA que trabalham neste repositório.
> Compatível com Claude Code, GitHub Copilot, Cursor, Codex e outros.

## Project Overview

Este é o **menura-pipelines** (anteriormente menura-actions), repositório central de governança de pipelines CI/CD para repositórios **Codebase** da organização Menura. Contém templates reutilizáveis para **GitHub Actions** e **GitLab CI/CD**.

### Propósito

- Padronizar pipelines de CI/CD para repositórios Codebase
- Automatizar gestão de releases e tags
- Garantir governança e qualidade nas releases
- Gerar artefatos e publicar releases
- **Suporte multi-plataforma:** GitHub Actions E GitLab CI/CD

> **Nota:** Repositórios de infraestrutura (Terraform/Terragrunt) mantêm suas próprias pipelines localmente.

### Stack Técnica

- **Plataformas:** GitHub Actions + GitLab CI/CD
- **Linguagem:** YAML (workflows/pipelines), Bash (scripts)
- **Padrão de Versionamento:** SemVer (vX.Y.Z)
- **Branches:** `sandbox` (staging), `main` (production)
- **Tech Stacks Suportadas:** Node.js, Bun

## Project Structure

```
m3nura/pipelines/
├── .github/workflows/          # GitHub Actions workflows
│   ├── codebase-ci-node.yml
│   ├── codebase-ci-bun.yml
│   ├── codebase-preview-deploy.yml
│   └── ...
├── .gitlab/                    # GitLab CI/CD pipelines
│   ├── ci/
│   │   ├── codebase-ci-node.yml
│   │   └── codebase-ci-bun.yml
│   ├── deploy/
│   │   └── codebase-preview-deploy.yml
│   └── release/
│       ├── create-rc.yml
│       └── qualify-release.yml
├── docs/
│   ├── tutorials/              # Guias de aprendizado passo-a-passo
│   ├── how-to/                 # Guias práticos para tarefas específicas
│   ├── reference/              # Documentação técnica detalhada
│   └── explanation/            # Conceitos e arquitetura
├── examples/
│   ├── github/                 # Exemplos GitHub Actions
│   └── gitlab/                 # Exemplos GitLab CI/CD
├── .gitlab-ci.yml              # Index de templates GitLab
├── GITLAB.md                   # Documentação GitLab
├── AGENTS.md                   # Este arquivo
└── README.md                   # Documentação principal (agnóstica)
```

## Multi-Platform Support

Este repositório suporta **DUAS plataformas de CI/CD** com templates equivalentes:

### GitHub Actions

**Localização:** `.github/workflows/`

**Como funciona:**
```yaml
# Projeto consome assim:
jobs:
  ci:
    uses: iSmart-System/menura-actions/.github/workflows/codebase-ci-node.yml@main
    with:
      node-version: '20'
    secrets: inherit
```

**Características:**
- Usa `workflow_call` para reutilização
- Requer configuração de permissões para repos privados
- Preview deploy via actions de terceiros
- Environments com proteção via required reviewers

### GitLab CI/CD

**Localização:** `.gitlab/`

**Como funciona:**
```yaml
# Projeto consome assim:
include:
  - project: 'm3nura/pipelines'
    ref: main
    file: '/.gitlab-ci.yml'

lint:
  extends: .node-lint
```

**Características:**
- Usa `include` + `extends` para reutilização
- Funciona out-of-the-box em repos privados
- Aprovação manual nativa (`when: manual`)
- Environments avançados (histórico, rollback)

## Build & Test Commands

### GitHub Actions

```bash
# Validar syntax YAML dos workflows
yamllint .github/workflows/

# Testar workflows localmente (requer 'act' instalado)
act -l                                # Listar workflows
act -j <job-name>                     # Executar job específico
act push                              # Simular evento push

# Verificar erros de formatação
prettier --check ".github/workflows/*.yml"
```

### GitLab CI/CD

```bash
# Validar syntax YAML das pipelines
yamllint .gitlab/

# Validar via GitLab CI Lint (requer projeto GitLab)
# Acesse: https://gitlab.com/m3nura/pipelines/-/ci/lint

# Testar localmente (requer gitlab-runner)
gitlab-runner exec docker <job-name>

# Verificar erros de formatação
prettier --check ".gitlab/**/*.yml"
```

## Code Style Guidelines

### YAML (Ambas Plataformas)

```yaml
# Indentar com 2 espaços
# Usar aspas simples para strings quando necessário
# Comentários em português, explicativos

# GitHub Actions
name: Nome do Workflow
on:
  workflow_call:
    inputs:
      exemplo:
        description: 'Descrição clara'
        required: false
        type: string
        default: 'valor-padrao'

# GitLab CI/CD
.template-name:
  stage: build
  script:
    - echo "Script aqui"
  variables:
    EXEMPLO: "valor-padrao"
```

### Nomenclatura

| Elemento | GitHub Actions | GitLab CI/CD |
|----------|----------------|--------------|
| **Arquivos** | `codebase-{nome}-{tech}.yml` | `codebase-{nome}-{tech}.yml` |
| **Jobs/Templates** | `kebab-case` | `.kebab-case` (prefixo `.` para templates) |
| **Steps** | Português, capitalizado | Português, capitalizado |
| **Variáveis** | `SCREAMING_SNAKE_CASE` | `SCREAMING_SNAKE_CASE` |
| **Inputs/Variables** | `kebab-case` | `kebab-case` |

### Exemplo: Equivalência entre Plataformas

```yaml
# GitHub Actions (.github/workflows/codebase-ci-node.yml)
name: CI - Node.js
on:
  workflow_call:
    inputs:
      node-version:
        required: false
        type: string
        default: '20'
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
      - run: npm ci
      - run: npm run lint

# GitLab CI/CD (.gitlab/ci/codebase-ci-node.yml)
.node-lint:
  image: node:${NODE_VERSION}
  stage: test
  before_script:
    - npm ci
  script:
    - npm run lint
```

## Git Workflow

### Branches

- `main` → Produção (protegida, requer MR/PR aprovado)
- `sandbox` → Homologação (protegida, requer MR/PR aprovado)
- `feat/*` → Features (temporárias)
- `fix/*` → Correções (temporárias)

### Fluxo de Commits

```bash
# Formato: tipo(escopo): descrição
feat(github): adicionar validação de lint
feat(gitlab): adicionar template de preview deploy
fix(config): corrigir variável de ambiente
docs(readme): atualizar instruções de uso
refactor(workflows): simplificar job de build
```

### Tags

```
v1.0.0-rc.1  → Release Candidate (criada em sandbox)
v1.0.0       → Release de Produção (após merge em main)
```

## Testing Instructions

### Antes de Abrir PR/MR

**GitHub Actions:**
1. Validar syntax: `yamllint .github/workflows/`
2. Testar com `act` se possível
3. Verificar retrocompatibilidade de inputs

**GitLab CI/CD:**
1. Validar syntax: `yamllint .gitlab/`
2. Validar via GitLab CI Lint
3. Verificar retrocompatibilidade de variables

**Ambos:**
4. Atualizar documentação em `docs/` se necessário
5. Atualizar exemplos em `examples/github/` e `examples/gitlab/`
6. Manter paridade de features entre plataformas

### Checklist de Review

- [ ] **Paridade:** Feature existe em ambas plataformas (ou justificada)
- [ ] **Documentação:** Header descritivo em ambos
- [ ] **Inputs/Variables:** Documentados e com defaults
- [ ] **Steps/Scripts:** Nomes descritivos em português
- [ ] **Secrets:** Não expostos em logs
- [ ] **Erros:** Mensagens claras
- [ ] **Nomenclatura:** Segue padrão estabelecido
- [ ] **Exemplos:** Atualizados em `examples/github/` e `examples/gitlab/`

## Security Considerations

### Secrets (Ambas Plataformas)

**GitHub Actions:**
```yaml
secrets:
  dispatch-token:
    required: true

jobs:
  deploy:
    steps:
      - run: |
          # NUNCA logar secrets
          curl -H "Authorization: token ${{ secrets.dispatch-token }}"
```

**GitLab CI/CD:**
```yaml
variables:
  # Configure no Group/Project: Settings → CI/CD → Variables
  # Marque como Masked e Protected
  PREVIEW_DEPLOY_TOKEN: $PREVIEW_DEPLOY_TOKEN

script:
  - |
    # NUNCA logar secrets
    curl -H "Authorization: token $PREVIEW_DEPLOY_TOKEN"
```

### Validações

```bash
# GitHub Actions
if [[ ! "${{ inputs.tag }}" =~ ^v[0-9]+\.[0-9]+\.[0-9]+$ ]]; then
  echo "::error::Tag inválida"
  exit 1
fi

# GitLab CI/CD
if [[ ! "$TAG" =~ ^v[0-9]+\.[0-9]+\.[0-9]+$ ]]; then
  echo "❌ Tag inválida"
  exit 1
fi
```

## Boundaries & Constraints

### O que este repositório NÃO deve conter

- Código de aplicação (apenas workflows/pipelines)
- Secrets hardcoded
- Configurações específicas de projetos individuais
- Lógica de negócio
- Preferência por uma plataforma sobre a outra (mantenha paridade!)

### O que este repositório DEVE conter

- Templates reutilizáveis para **ambas** plataformas
- Documentação de uso (agnóstica quando possível)
- Exemplos para projetos (separados por plataforma)
- Scripts de automação de release
- Paridade de features entre GitHub Actions e GitLab CI/CD

### Retrocompatibilidade

- Novos inputs/variables DEVEM ter defaults
- Breaking changes requerem nova major version
- Deprecações devem ser documentadas e comunicadas
- **Mudanças devem ser propagadas para ambas plataformas**

## Workflows/Pipelines Reference

### GitHub Actions (`.github/workflows/`)

| Workflow | Propósito |
|----------|-----------|
| `codebase-ci-node.yml` | CI Node.js (lint, test, build) |
| `codebase-ci-bun.yml` | CI Bun (lint, test, build) |
| `codebase-release-node.yml` | Release Node.js (artefatos + GitHub Release) |
| `codebase-release-bun.yml` | Release Bun (artefatos + GitHub Release) |
| `codebase-create-rc.yml` | Criar Release Candidate |
| `codebase-qualify-rc.yml` | Qualificar RC para produção |
| `codebase-preview-deploy.yml` | Preview deploy manual (via repository dispatch) |

### GitLab CI/CD (`.gitlab/`)

| Pipeline | Propósito |
|----------|-----------|
| `ci/codebase-ci-node.yml` | Templates CI Node.js (.node-lint, .node-test, .node-build) |
| `ci/codebase-ci-bun.yml` | Templates CI Bun (.bun-lint, .bun-test, .bun-build) |
| `deploy/codebase-preview-deploy.yml` | Preview deploy manual (.preview-deploy, .stop-preview) |
| `release/create-rc.yml` | Criar Release Candidate (.create-release-candidate) |
| `release/qualify-release.yml` | Qualificar RC (.qualify-release) |

## Common Tasks

### Adicionar Nova Feature

**IMPORTANTE:** Mantenha paridade entre plataformas!

1. **GitHub Actions:**
   - Criar/atualizar workflow em `.github/workflows/`
   - Adicionar exemplo em `examples/github/`

2. **GitLab CI/CD:**
   - Criar/atualizar pipeline em `.gitlab/`
   - Adicionar exemplo em `examples/gitlab/`

3. **Documentação:**
   - Atualizar `docs/reference/` (incluir ambas plataformas)
   - Atualizar `README.md` (agnóstico)
   - Se específico de plataforma, documentar em `GITLAB.md` ou docs específicas

### Adicionar Suporte a Nova Tech Stack

1. **GitHub Actions:** `.github/workflows/codebase-ci-{tech}.yml`
2. **GitLab CI/CD:** `.gitlab/ci/codebase-ci-{tech}.yml`
3. Adicionar exemplos em `examples/github/` e `examples/gitlab/`
4. Documentar em `docs/reference/workflows.md`
5. Manter comandos equivalentes entre plataformas

### Testar Mudança em Projeto Real

**GitHub Actions:**
```yaml
uses: iSmart-System/menura-actions/.github/workflows/codebase-ci-node.yml@sua-branch
```

**GitLab CI/CD:**
```yaml
include:
  - project: 'm3nura/pipelines'
    ref: sua-branch
    file: '/.gitlab-ci.yml'
```

### Migrar Feature entre Plataformas

Se uma feature existe apenas em uma plataforma:

1. Analisar implementação existente
2. Adaptar para a outra plataforma (respeitando idiomas nativos)
3. Testar equivalência funcional
4. Atualizar exemplos e documentação
5. Marcar paridade no changelog

## Development Environment

### Ferramentas Recomendadas

```bash
# Para ambas plataformas
pip install yamllint
brew install prettier

# GitHub Actions
brew install act
brew install gh

# GitLab CI/CD
brew install gitlab-runner  # Opcional, para testes locais
```

### VS Code Extensions

- YAML (Red Hat)
- GitHub Actions (GitHub)
- GitLab Workflow (GitLab)
- EditorConfig

## Pull Request / Merge Request Guidelines

### Título

```
tipo(plataforma/escopo): descrição breve

Exemplos:
feat(github): adicionar suporte a Python
feat(gitlab): adicionar template de preview deploy
feat(both): adicionar validação de tags
fix(gitlab): corrigir timeout em produção
docs(readme): melhorar seção de instalação
```

### Checklist no PR/MR

```markdown
## Resumo
[Descrição clara da mudança]

## Plataforma(s) Afetada(s)
- [ ] GitHub Actions
- [ ] GitLab CI/CD
- [ ] Ambas (paridade mantida)

## Mudanças
- [Lista de alterações]

## Testes
- [ ] Testado no GitHub Actions
- [ ] Testado no GitLab CI/CD
- [ ] Exemplos atualizados
- [ ] Documentação atualizada
```

---

*Este arquivo segue a especificação [AGENTS.md](https://agents.md/), mantida pela Agentic AI Foundation sob a Linux Foundation.*
