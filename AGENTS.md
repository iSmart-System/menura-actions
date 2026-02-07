# AGENTS.md

> Instruções para agentes de IA que trabalham neste repositório.
> Compatível com Claude Code, GitHub Copilot, Cursor, Codex e outros.

## Project Overview

Este é o **menura-pipelines** (anteriormente menura-actions), repositório central de governança de pipelines CI/CD para repositórios **Codebase** da organização Menura. Contém templates reutilizáveis para **GitHub Actions** e **GitLab CI/CD**.

> **⚠️ IMPORTANTE - Foco Atual:** A organização está migrando TODOS os repositórios para o GitLab. Portanto, **priorize sempre o GitLab CI/CD** ao trabalhar neste repositório. Templates do GitHub Actions são mantidos apenas para retrocompatibilidade, mas NÃO devem receber novas features ou melhorias significativas.

### Propósito

- Padronizar pipelines de CI/CD para repositórios Codebase
- Automatizar gestão de releases e tags
- Garantir governança e qualidade nas releases
- Gerar artefatos (.zip) e publicar releases
- **Plataforma principal:** GitLab CI/CD (GitHub Actions apenas para retrocompatibilidade)

> **Nota:** Repositórios de infraestrutura (Terraform/Terragrunt) mantêm suas próprias pipelines localmente.

### Stack Técnica

- **Plataforma Principal:** GitLab CI/CD (GitHub Actions mantido para retrocompatibilidade)
- **Linguagem:** YAML (workflows/pipelines), Bash (scripts)
- **Padrão de Versionamento:** SemVer (vX.Y.Z)
- **Branches:** `sandbox` (staging), `main` (production)
- **Tech Stacks Suportadas:** Node.js, Bun
- **Formato de Artefatos:** .zip (conteúdo direto na raiz, sem pasta pai)

## Project Structure

```
m3nura/pipelines/
├── .gitlab/                    # GitLab CI/CD pipelines (PRINCIPAL)
│   ├── ci/
│   │   ├── codebase-ci-node.yml       # Templates Node.js (.node-lint, .node-test, .node-build)
│   │   └── codebase-ci-bun.yml        # Templates Bun (.bun-lint, .bun-test, .bun-build)
│   ├── deploy/
│   │   └── codebase-preview-deploy.yml # Template preview deploy (.preview-deploy)
│   └── release/
│       ├── create-rc.yml               # Template criar RC (.create-release-candidate)
│       └── qualify-release.yml         # Template qualificar RC (.qualify-release)
├── .github/workflows/          # GitHub Actions workflows (RETROCOMPATIBILIDADE)
│   ├── codebase-ci-node.yml
│   ├── codebase-ci-bun.yml
│   ├── codebase-preview-deploy.yml
│   └── ...
├── examples/
│   ├── gitlab/                 # Exemplos GitLab CI/CD (PRINCIPAL)
│   │   ├── README.md          # Documentação completa GitLab
│   │   ├── ci-node.yml
│   │   ├── ci-node-with-preview.yml
│   │   └── ci-node-skip-tests.yml
│   └── github/                 # Exemplos GitHub Actions (RETROCOMPATIBILIDADE)
├── docs/
│   ├── tutorials/              # Guias de aprendizado passo-a-passo
│   ├── how-to/                 # Guias práticos para tarefas específicas
│   ├── reference/              # Documentação técnica detalhada
│   └── explanation/            # Conceitos e arquitetura
├── AGENTS.md                   # Este arquivo - instruções para agentes de IA
└── README.md                   # Documentação principal (agnóstica)
```

## Platform Support

### GitLab CI/CD (PRINCIPAL - FOCO ATUAL)

**Localização:** `.gitlab/`

**Como funciona:**
```yaml
# Projeto consome assim (include direto, best practice):
include:
  - project: 'm3nura/pipelines'
    ref: main
    file: '.gitlab/ci/codebase-ci-node.yml'

lint:
  extends: .node-lint

# Para múltiplos arquivos (GitLab 13.6+):
include:
  - project: 'm3nura/pipelines'
    ref: main
    file:
      - '.gitlab/ci/codebase-ci-node.yml'
      - '.gitlab/deploy/codebase-preview-deploy.yml'
```

**Características:**
- Usa `include` + `extends` para reutilização
- Funciona out-of-the-box em repos privados
- Aprovação manual nativa (`when: manual`)
- Environments avançados (histórico, rollback)
- Include direto de arquivos específicos (sem index file)

## GitLab CI/CD - Variáveis Obrigatórias

> **⚠️ IMPORTANTE:** Todas as variáveis abaixo são **OBRIGATÓRIAS**. Se não forem definidas, a pipeline falhará com mensagem de erro clara.

### Variáveis de Build (Node.js)

| Variável | Descrição | Exemplo |
|----------|-----------|---------|
| `NODE_VERSION` | Versão do Node.js | `"20"`, `"18"`, `"latest"` |
| `ARTIFACT_PATH` | Diretório onde o build é gerado | `"dist"`, `"build"` |
| `ARTIFACT_NAME` | Nome do artefato/projeto (usado no zip) | `"meu-app"` |

### Variáveis de Build (Bun)

| Variável | Descrição | Exemplo |
|----------|-----------|---------|
| `BUN_VERSION` | Versão do Bun | `"latest"`, `"1.0.0"` |
| `ARTIFACT_PATH` | Diretório onde o build é gerado | `"dist"`, `"build"` |
| `ARTIFACT_NAME` | Nome do artefato/projeto (usado no zip) | `"meu-app"` |

### Variáveis de Preview Deploy

| Variável | Descrição | Exemplo |
|----------|-----------|---------|
| `ARTIFACT_NAME` | Nome do projeto | `"menura-documentation-portal"` |
| `PREVIEW_URL` | URL completa do preview | `"https://docs.sandbox.menura.com.br"` |
| `PREVIEW_DEPLOY_TOKEN` | Pipeline Trigger Token do cloud-foundation | Configurar no Group (masked) |

### Variáveis de Controle de Fluxo

| Variável | Valor | Efeito |
|----------|-------|--------|
| `SKIP_LINT` | `"true"` | Pula job de lint |
| `SKIP_TESTS` | `"true"` | Pula job de testes |
| `SKIP_BUILD` | `"true"` | Pula job de build |

### Geração de Artefatos (.zip)

Os templates `.node-build` e `.bun-build` geram automaticamente um arquivo .zip com:

**Características do zip:**
- ✅ Conteúdo do `ARTIFACT_PATH` diretamente na raiz (sem pasta pai)
- ✅ Nome único: `{ARTIFACT_NAME}-{branch}-{commit}-{timestamp}.zip`
- ✅ Evita colisões entre pipelines
- ✅ Exclui arquivos `.git*`

**Exemplo de nome gerado:**
```
menura-documentation-portal-main-a1b2c3d-1707318945.zip
```

**Exemplo de configuração (Node.js):**
```yaml
variables:
  NODE_VERSION: "20"               # Versão do Node.js (obrigatório)
  ARTIFACT_PATH: "dist"            # Diretório do build (obrigatório)
  ARTIFACT_NAME: "meu-app"         # Nome do projeto (obrigatório)
  PREVIEW_URL: "https://app.sandbox.menura.com.br"  # URL do preview (se usar preview deploy)
```

**Exemplo de configuração (Bun):**
```yaml
variables:
  BUN_VERSION: "latest"            # Versão do Bun (obrigatório)
  ARTIFACT_PATH: "dist"            # Diretório do build (obrigatório)
  ARTIFACT_NAME: "meu-app-bun"     # Nome do projeto (obrigatório)
  PREVIEW_URL: "https://app.sandbox.menura.com.br"  # URL do preview (se usar preview deploy)
```

### GitHub Actions (RETROCOMPATIBILIDADE APENAS)

**Localização:** `.github/workflows/`

> **⚠️ IMPORTANTE:** GitHub Actions é mantido apenas para retrocompatibilidade. **NÃO adicione novas features** ou melhorias significativas. Todos os novos desenvolvimentos devem ser feitos no GitLab CI/CD.

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
- Artefatos .zip já implementados corretamente

## Build & Test Commands

### GitLab CI/CD (PRINCIPAL)

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

### GitHub Actions (Retrocompatibilidade)

```bash
# Validar syntax YAML dos workflows
yamllint .github/workflows/

# Verificar erros de formatação
prettier --check ".github/workflows/*.yml"

# NOTA: Testes locais com 'act' não são mais priorizados
```

## Code Style Guidelines

### YAML - GitLab CI/CD (PRINCIPAL)

```yaml
# Indentar com 2 espaços
# Usar aspas simples para strings quando necessário
# Comentários em português, explicativos

.template-name:
  stage: build
  script:
    - echo "Script aqui"
  variables:
    EXEMPLO: "valor-padrao"
```

### Nomenclatura

| Elemento | GitLab CI/CD | Observação |
|----------|--------------|------------|
| **Arquivos** | `codebase-{nome}-{tech}.yml` | Padrão consistente |
| **Templates** | `.kebab-case` | Prefixo `.` obrigatório para templates |
| **Jobs** | `kebab-case` | Jobs concretos sem prefixo `.` |
| **Steps** | Português, capitalizado | Comentários descritivos |
| **Variáveis** | `SCREAMING_SNAKE_CASE` | Sempre maiúsculas |

### Exemplo: Template GitLab CI/CD

```yaml
# .gitlab/ci/codebase-ci-node.yml
.node-lint:
  image: node:${NODE_VERSION}
  stage: test
  before_script:
    - npm ci
  script:
    - npm run lint
  rules:
    - if: '$SKIP_LINT == "true"'
      when: never
    - when: on_success

.node-build:
  extends: .node-base
  stage: build
  script:
    - npm run build
    - |
      # Criar zip do artefato com nome único
      BUILD_PATH="${ARTIFACT_PATH:-dist}"
      TIMESTAMP=$(date +%s)
      ZIP_NAME="${ARTIFACT_NAME:-artifact}-${CI_COMMIT_REF_SLUG}-${CI_COMMIT_SHORT_SHA}-${TIMESTAMP}.zip"

      cd "$BUILD_PATH"
      zip -r "../$ZIP_NAME" . -x "*.git*"
      cd ..
  artifacts:
    name: "$ARTIFACT_NAME-$CI_COMMIT_REF_SLUG-$CI_COMMIT_SHORT_SHA"
    paths:
      - "*.zip"
    expire_in: 7 days
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

### Checklist de Review (GitLab CI/CD)

- [ ] **Template:** Criado/atualizado em `.gitlab/`
- [ ] **Documentação:** Header descritivo no template
- [ ] **Variáveis:** Documentadas em AGENTS.md e `examples/gitlab/README.md`
- [ ] **Variáveis Obrigatórias:** Validadas no script com mensagens claras
- [ ] **Scripts:** Comentários em português, nomes descritivos
- [ ] **Secrets:** Nunca expostos em logs (masked variables)
- [ ] **Erros:** Mensagens claras com emojis (❌, ✅, 📦, etc.)
- [ ] **Nomenclatura:** Segue padrão (`.template-name` para templates)
- [ ] **Artefatos:** Se gera .zip, conteúdo deve estar na raiz
- [ ] **Exemplos:** Atualizados em `examples/gitlab/`
- [ ] **AGENTS.md:** Atualizado se for feature significativa

## Security Considerations

### Secrets (GitLab CI/CD)
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

### Validações (GitLab CI/CD)

```bash
# Validar tag SemVer
if [[ ! "$TAG" =~ ^v[0-9]+\.[0-9]+\.[0-9]+(-rc\.[0-9]+)?$ ]]; then
  echo "❌ Tag inválida: $TAG"
  echo "Formato esperado: vX.Y.Z ou vX.Y.Z-rc.N"
  exit 1
fi

# Validar variável obrigatória
if [ -z "$ARTIFACT_NAME" ]; then
  echo "❌ ARTIFACT_NAME não definido"
  echo "Configure: variables.ARTIFACT_NAME no seu .gitlab-ci.yml"
  exit 1
fi

# Validar diretório existe
if [ ! -d "$BUILD_PATH" ]; then
  echo "❌ Diretório $BUILD_PATH não encontrado"
  exit 1
fi
```

## Boundaries & Constraints

### O que este repositório NÃO deve conter

- Código de aplicação (apenas workflows/pipelines)
- Secrets hardcoded
- Configurações específicas de projetos individuais
- Lógica de negócio
- **Novas features no GitHub Actions** (apenas GitLab)

### O que este repositório DEVE conter (GitLab CI/CD)

- Templates reutilizáveis e bem documentados
- Documentação completa em `examples/gitlab/README.md`
- Exemplos práticos em `examples/gitlab/`
- Variáveis obrigatórias documentadas
- Scripts com validações robustas
- Mensagens de erro claras com emojis

### Retrocompatibilidade (GitLab CI/CD)

- Novos inputs/variables DEVEM ter defaults OU validação explícita
- Breaking changes requerem nova major version
- Deprecações devem ser documentadas em AGENTS.md
- GitHub Actions: mantido sem alterações (exceto bugs críticos)

## Workflows/Pipelines Reference

### GitLab CI/CD (`.gitlab/`) - PRINCIPAL

| Pipeline | Propósito | Templates Disponíveis |
|----------|-----------|----------------------|
| `ci/codebase-ci-node.yml` | CI Node.js (lint, test, build com .zip) | `.node-base`, `.node-lint`, `.node-test`, `.node-build` |
| `ci/codebase-ci-bun.yml` | CI Bun (lint, test, build com .zip) | `.bun-base`, `.bun-lint`, `.bun-test`, `.bun-build` |
| `deploy/codebase-preview-deploy.yml` | Preview deploy com Pipeline Triggers | `.preview-deploy-base`, `.preview-deploy` |
| `release/create-rc.yml` | Criar Release Candidate (sandbox) | `.create-release-candidate` |
| `release/qualify-release.yml` | Qualificar RC para produção (main) | `.qualify-release` |

**Características:**
- ✅ Artefatos gerados como .zip (conteúdo na raiz)
- ✅ Preview deploy 100% GitLab nativo (Pipeline Triggers)
- ✅ Environments com auto-cleanup (7 dias)
- ✅ Aprovação manual nativa (`when: manual`)

### GitHub Actions (`.github/workflows/`) - RETROCOMPATIBILIDADE

| Workflow | Propósito | Status |
|----------|-----------|--------|
| `codebase-ci-node.yml` | CI Node.js | ⚠️ Mantido, sem novas features |
| `codebase-ci-bun.yml` | CI Bun | ⚠️ Mantido, sem novas features |
| `codebase-release-node.yml` | Release Node.js | ⚠️ Mantido, sem novas features |
| `codebase-release-bun.yml` | Release Bun | ⚠️ Mantido, sem novas features |
| `codebase-preview-deploy.yml` | Preview deploy | ⚠️ Mantido, sem novas features |

## Common Tasks

### Adicionar Nova Feature

**⚠️ IMPORTANTE:** Adicione novas features APENAS no GitLab CI/CD. GitHub Actions não deve receber novas funcionalidades.

1. **GitLab CI/CD (OBRIGATÓRIO):**
   - Criar/atualizar template em `.gitlab/`
   - Adicionar exemplo em `examples/gitlab/`
   - Documentar em `examples/gitlab/README.md`
   - Atualizar AGENTS.md se for feature significativa

2. **Documentação:**
   - Atualizar `README.md` se afetar uso geral
   - Documentar variáveis obrigatórias em AGENTS.md
   - Adicionar exemplos práticos em `examples/gitlab/`

3. **GitHub Actions (OPCIONAL - apenas se absolutamente necessário):**
   - Não adicionar novas features
   - Apenas correções críticas de bugs

### Adicionar Suporte a Nova Tech Stack

1. **GitLab CI/CD:** `.gitlab/ci/codebase-ci-{tech}.yml`
2. Adicionar exemplo em `examples/gitlab/`
3. Documentar em `examples/gitlab/README.md`
4. Atualizar AGENTS.md com templates e variáveis
5. GitHub Actions: NÃO é necessário adicionar

### Testar Mudança em Projeto Real (GitLab CI/CD)

```yaml
# No .gitlab-ci.yml do projeto de teste
include:
  - project: 'm3nura/pipelines'
    ref: sua-branch-de-desenvolvimento  # Branch com suas mudanças
    file:
      - '.gitlab/ci/codebase-ci-node.yml'
      - '.gitlab/deploy/codebase-preview-deploy.yml'

# Configure variáveis obrigatórias
variables:
  NODE_VERSION: "20"
  ARTIFACT_NAME: "projeto-teste"
  ARTIFACT_PATH: "dist"
  PREVIEW_URL: "https://test.sandbox.menura.com.br"

# Use os templates
lint:
  extends: .node-lint

build:
  extends: .node-build

preview:
  extends: .preview-deploy
```

**Passos para testar:**
1. Criar branch de desenvolvimento em `m3nura/pipelines`
2. Fazer as mudanças nos templates
3. Em um projeto real, apontar o `ref:` para sua branch
4. Executar pipeline e validar comportamento
5. Após validação, merge para `main`

## Development Environment

### Ferramentas Recomendadas (GitLab CI/CD)

```bash
# Essenciais
pip install yamllint
brew install prettier

# GitLab CLI
brew install glab

# Opcional (para testes locais)
brew install gitlab-runner
```

### VS Code Extensions

**Essenciais:**
- YAML (Red Hat)
- GitLab Workflow (GitLab)
- EditorConfig

**Opcionais:**
- GitHub Actions (GitHub) - para manutenção de workflows legados

## Merge Request Guidelines (GitLab)

### Título

```
tipo(escopo): descrição breve

Exemplos:
feat(gitlab): adicionar template de preview deploy
feat(ci): adicionar suporte a Python
fix(preview): corrigir validação de PREVIEW_URL
refactor(build): otimizar geração de zip
docs(agents): atualizar variáveis obrigatórias
```

**Tipos permitidos:** `feat`, `fix`, `refactor`, `docs`, `chore`

### Checklist no MR

```markdown
## Resumo
[Descrição clara da mudança]

## Tipo de Mudança
- [ ] Nova feature (GitLab CI/CD)
- [ ] Bug fix (GitLab CI/CD)
- [ ] Bug fix crítico (GitHub Actions - apenas se absolutamente necessário)
- [ ] Documentação
- [ ] Refatoração

## Mudanças
- [Lista detalhada de alterações]

## Testes
- [ ] Validado YAML com yamllint
- [ ] Testado em projeto real (qual?)
- [ ] Exemplos atualizados em `examples/gitlab/`
- [ ] Documentação atualizada (AGENTS.md e/ou examples/gitlab/README.md)
- [ ] Variáveis obrigatórias documentadas

## Impacto
- [ ] Breaking change? (Se sim, justifique e documente)
- [ ] Requer atualização de projetos? (Se sim, descreva)
```

---

*Este arquivo segue a especificação [AGENTS.md](https://agents.md/), mantida pela Agentic AI Foundation sob a Linux Foundation.*
