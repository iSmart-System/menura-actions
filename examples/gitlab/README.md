# GitLab CI/CD - Documentação Completa

Documentação e exemplos práticos para usar os templates do `m3nura/pipelines` no GitLab CI/CD.

---

## Arquivos de Exemplo

| Arquivo | Descrição | Quando Usar |
|---------|-----------|-------------|
| [`ci-node.yml`](ci-node.yml) | CI básico para Node.js | Projetos Node.js com lint, testes e build |
| [`ci-bun.yml`](ci-bun.yml) | CI básico para Bun | Projetos Bun com lint, testes e build |
| [`ci-node-with-preview.yml`](ci-node-with-preview.yml) | CI Node.js + Preview Deploy | Projetos que precisam preview em MRs |
| [`ci-node-skip-tests.yml`](ci-node-skip-tests.yml) | CI Node.js sem lint/tests | Projetos como Docusaurus (sem testes) |
| [`ci-node-with-release.yml`](ci-node-with-release.yml) | **CI + Preview + Release Management (Node.js)** | **Projetos prontos para produção (RECOMENDADO)** |
| [`ci-bun-with-release.yml`](ci-bun-with-release.yml) | **CI + Preview + Release Management (Bun)** | **Projetos Bun prontos para produção (RECOMENDADO)** |

---

## ⚠️ OBRIGATÓRIO: Workflow Rules

> **CRÍTICO:** Todos os projetos DEVEM incluir estas workflow rules para garantir governança.

**Problema:** Sem workflow rules, commits diretos em `sandbox`/`main` disparam builds desnecessários e desperdiçam minutos de CI.

**Solução:** Adicione no início do seu `.gitlab-ci.yml`:

```yaml
workflow:
  rules:
    # Permitir MRs (Merge Requests)
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
    # Evitar duplicação se já existe MR aberto para a branch
    - if: '$CI_COMMIT_BRANCH && $CI_OPEN_MERGE_REQUESTS'
      when: never
    # BLOQUEAR commits diretos em sandbox e main
    - if: '$CI_COMMIT_BRANCH == "sandbox" || $CI_COMMIT_BRANCH == "main"'
      when: never
    # Permitir outras branches (feature, fix, etc.)
    - if: '$CI_COMMIT_BRANCH'
```

**O que isso faz:**
- ✅ MRs executam build/lint/tests normalmente
- ✅ Feature branches executam build/lint/tests normalmente
- ❌ Commits diretos em `sandbox` NÃO executam (use Create RC)
- ❌ Commits diretos em `main` NÃO executam (use Qualify RC)

**Por quê:**
- Branches `sandbox`/`main` são protegidas
- Mudanças chegam via MRs ou release workflow
- Evita desperdício de minutos de CI
- Força governança no processo de release

---

## Quick Start

### 1. Incluir Templates no Seu Projeto

Crie `.gitlab-ci.yml` na raiz do seu projeto com **includes diretos**:

```yaml
include:
  # Include DIRETO do arquivo específico (best practice!)
  - project: 'm3nura/pipelines'
    ref: main
    file: '.gitlab/ci/codebase-ci-node.yml'

stages:
  - test
  - build

variables:
  NODE_VERSION: "20"
  ARTIFACT_PATH: "dist"

lint:
  extends: .node-lint

test:
  extends: .node-test

build:
  extends: .node-build
```

**Múltiplos arquivos** (GitLab 13.6+):

```yaml
include:
  - project: 'm3nura/pipelines'
    ref: main
    file:
      - '.gitlab/ci/codebase-ci-node.yml'
      - '.gitlab/deploy/codebase-preview-deploy.yml'
```

**Vantagens:**
- ✅ Incluir apenas o que você precisa
- ✅ Evita carregar templates desnecessários
- ✅ Melhor performance e clareza
- ✅ Não precisa de arquivo index na raiz

### 2. Configurar Variáveis Sensíveis

No Group/Project: **Settings → CI/CD → Variables**

Adicione:
- `PREVIEW_DEPLOY_TOKEN` - Pipeline Trigger Token do `m3nura/cloud-foundation` (se usar preview deploy)

---

## Templates Disponíveis

### Node.js

| Template | Descrição |
|----------|-----------|
| `.node-base` | Configuração base (image, cache, npm ci) |
| `.node-lint` | Executar lint (ESLint, Prettier) |
| `.node-test` | Executar testes com coverage |
| `.node-build` | Executar build e gerar artefatos |

### Bun

| Template | Descrição |
|----------|-----------|
| `.bun-base` | Configuração base (image, cache, bun install) |
| `.bun-lint` | Executar lint |
| `.bun-test` | Executar testes |
| `.bun-build` | Executar build e gerar artefatos |

### Deploy

| Template | Descrição |
|----------|-----------|
| `.preview-deploy` | Preview deploy manual com environment nativo e auto-cleanup |

### Release

| Template | Descrição |
|----------|-----------|
| `.create-release-candidate` | Criar RC (manual, branch `sandbox`) |
| `.qualify-release` | Qualificar RC para produção (manual, branch `main`) |

---

## Variáveis Obrigatórias

> **⚠️ IMPORTANTE:** Todas as variáveis abaixo são **OBRIGATÓRIAS**. Se não forem definidas, a pipeline falhará com mensagem de erro clara.

### Node.js

```yaml
variables:
  NODE_VERSION: "20"            # Obrigatório - Versão do Node.js ("18", "20", "latest")
  ARTIFACT_PATH: "dist"         # Obrigatório - Caminho do build ("dist", "build")
  ARTIFACT_NAME: "meu-app"      # Obrigatório - Nome do artefato
```

### Bun

```yaml
variables:
  BUN_VERSION: "latest"         # Obrigatório - Versão do Bun ("latest", "1.0.0")
  ARTIFACT_PATH: "dist"         # Obrigatório - Caminho do build
  ARTIFACT_NAME: "meu-app-bun"  # Obrigatório - Nome do artefato
```

## Variáveis Opcionais

### Pular Stages Específicos

```yaml
variables:
  SKIP_LINT: "true"   # Pular lint
  SKIP_TESTS: "true"  # Pular testes
  SKIP_BUILD: "true"  # Pular build
```

### Preview Deploy

```yaml
variables:
  ARTIFACT_NAME: "meu-app"                          # Nome do artefato (obrigatório)
  PREVIEW_URL: "https://app.sandbox.menura.com.br"  # URL do preview (obrigatório)
```

O preview deploy é disparado automaticamente via **GitLab Pipeline Triggers** para o repositório `m3nura/cloud-foundation`.

---

## Exemplos Completos

### Projeto Node.js Simples

```yaml
include:
  - project: 'm3nura/pipelines'
    ref: main
    file: '.gitlab/ci/codebase-ci-node.yml'

stages:
  - test
  - build

variables:
  NODE_VERSION: "20"        # Obrigatório
  ARTIFACT_PATH: "dist"     # Obrigatório
  ARTIFACT_NAME: "meu-app"  # Obrigatório

lint:
  extends: .node-lint

test:
  extends: .node-test

build:
  extends: .node-build
```

### Projeto com Preview Deploy

```yaml
include:
  - project: 'm3nura/pipelines'
    ref: main
    file:
      - '.gitlab/ci/codebase-ci-node.yml'
      - '.gitlab/deploy/codebase-preview-deploy.yml'

stages:
  - test
  - build
  - deploy

variables:
  NODE_VERSION: "20"
  ARTIFACT_NAME: "meu-app"
  PREVIEW_URL: "https://app.sandbox.menura.com.br"

lint:
  extends: .node-lint

test:
  extends: .node-test

build:
  extends: .node-build

preview:
  extends: .preview-deploy
  needs:
    - job: build
      artifacts: true
```

### Projeto Docusaurus (Sem Lint/Tests)

```yaml
include:
  - project: 'm3nura/pipelines'
    ref: main
    file:
      - '.gitlab/ci/codebase-ci-node.yml'
      - '.gitlab/deploy/codebase-preview-deploy.yml'

stages:
  - build
  - deploy

variables:
  NODE_VERSION: "20"
  ARTIFACT_PATH: "build"
  SKIP_LINT: "true"
  SKIP_TESTS: "true"
  ARTIFACT_NAME: "menura-documentation-portal"
  PREVIEW_URL: "https://docs.sandbox.menura.com.br"

build:
  extends: .node-build

preview:
  extends: .preview-deploy
  needs:
    - job: build
      artifacts: true
```

---

## Release Management

> 🚀 **Fluxo completo de release com Release Candidates (RC) e deploy automatizado**

### Visão Geral

```
Feature → Sandbox → RC → Produção

1. Feature branch → MR para sandbox → Build/Tests → Merge
2. Manual: Create RC → Build → Artefato → Tag → Deploy Sandbox
3. Validação/Homologação
4. Manual: Qualify RC → Tag Produção → Deploy Produção
```

### Templates Disponíveis

| Template | Para | Onde Executar | O que faz |
|----------|------|---------------|-----------|
| `.create-rc-node` | Node.js | Branch `sandbox` | Build → Artefato RC → Tag → Deploy |
| `.create-rc-bun` | Bun | Branch `sandbox` | Build → Artefato RC → Tag → Deploy |
| `.qualify-rc-to-release` | Qualquer | Branch `main` | Tag Produção → Remove RC → Deploy |

### Passo 1: Criar Release Candidate

**Quando:** Após features prontas em sandbox, pronto para homologação.

**Como executar:**
1. Vá na pipeline da branch `sandbox` no GitLab
2. Clique em "Run pipeline"
3. Adicione variable: `RC_VERSION=1.0.0-rc.1`
4. Execute o job `create-rc` manualmente

**Exemplo de configuração:**
```yaml
include:
  - project: 'm3nura/pipelines'
    ref: main
    file:
      - '.gitlab/ci/codebase-ci-node.yml'
      - '.gitlab/release/create-rc-node.yml'  # Para Node.js

stages:
  - test
  - build
  - release

variables:
  NODE_VERSION: "20"
  ARTIFACT_PATH: "dist"
  ARTIFACT_NAME: "meu-app"

# Jobs de CI (lint, test, build)
lint:
  extends: .node-lint

test:
  extends: .node-test

build:
  extends: .node-build

# Job de Release
create-rc:
  extends: .create-rc-node
  stage: release
```

**O que acontece:**
- ✅ Build executado (npm run build)
- ✅ Artefato criado: `meu-app-v1.0.0-rc.1-abc123-1234567890.zip`
- ✅ Conteúdo direto na raiz do zip (sem pasta pai)
- ✅ Tag Git criada: `v1.0.0-rc.1`
- ✅ Trigger disparado para `m3nura/cloud-foundation`
- ✅ Deploy em sandbox executado

### Passo 2: Validar RC

Após deploy em sandbox:
- Executar testes de homologação
- Validar funcionalidades
- Obter aprovações de stakeholders

Se encontrar bugs:
1. Corrigir na feature branch
2. MR para sandbox
3. Criar nova RC (ex: `v1.0.0-rc.2`)

### Passo 3: Qualificar RC para Produção

**Quando:** Após RC validada e homologada, com GMUD aprovada.

**Pré-requisitos:**
1. RC criada e testada (ex: `v1.0.0-rc.1`)
2. MR de `sandbox` → `main` criado e aprovado
3. MR merged

**Como executar:**
1. Vá na pipeline da branch `main` no GitLab
2. Clique em "Run pipeline"
3. Adicione variable: `RC_TAG=v1.0.0-rc.1`
4. Execute o job `qualify-rc` manualmente

**Exemplo de configuração:**
```yaml
include:
  - project: 'm3nura/pipelines'
    ref: main
    file:
      - '.gitlab/ci/codebase-ci-node.yml'
      - '.gitlab/release/qualify-rc-to-release.yml'

stages:
  - test
  - build
  - release

# ... outros jobs ...

qualify-rc:
  extends: .qualify-rc-to-release
  stage: release
```

**O que acontece:**
- ✅ Tag de produção criada: `v1.0.0`
- ✅ Tag RC antiga removida: `v1.0.0-rc.1`
- ✅ Trigger disparado para `m3nura/cloud-foundation`
- ✅ Deploy em produção executado

### Exemplo Completo com Release Management

Ver:
- [`ci-node-with-release.yml`](ci-node-with-release.yml) - Node.js
- [`ci-bun-with-release.yml`](ci-bun-with-release.yml) - Bun

### Variável Necessária

Configure no **Group level** (Settings → CI/CD → Variables):

| Variable | Value | Protected | Masked | Description |
|----------|-------|-----------|--------|-------------|
| `CLOUD_FOUNDATION_TOKEN` | `glptt-xxx...` | ✅ | ✅ | Pipeline Trigger Token do m3nura/cloud-foundation |

**Como criar o token:**
1. Vá em `m3nura/cloud-foundation`
2. Settings → CI/CD → Pipeline triggers
3. Clique em "Add trigger"
4. Description: "Deploy from Codebase repos"
5. Copie o token (`glptt-xxx...`)
6. Adicione no Group `m3nura` como `CLOUD_FOUNDATION_TOKEN`

---

## Customizações Avançadas

### Adicionar Jobs Customizados

```yaml
# Depois dos jobs padrão (lint, test, build)
deploy-staging:
  stage: deploy
  script:
    - echo "Deploy para staging"
  environment:
    name: staging
    url: https://staging.example.com
  rules:
    - if: '$CI_COMMIT_BRANCH == "develop"'
```

---

## Governança e Configuração

### Group-level Variables (Recomendado)

Configure no Group `m3nura`: **Settings → CI/CD → Variables**

**Variáveis recomendadas:**

| Variable | Value | Protected | Masked | Description |
|----------|-------|-----------|--------|-------------|
| `PREVIEW_DEPLOY_TOKEN` | `glptt-xxx...` | ✅ | ✅ | Pipeline Trigger Token do cloud-foundation |
| `PREVIEW_DEPLOY_APPROVERS` | `user1,user2` | ❌ | ❌ | Lista de aprovadores |
| `DEPLOY_NOTIFICATION_WEBHOOK` | `https://...` | ✅ | ✅ | Webhook para notificações |

#### Como Criar o Pipeline Trigger Token

No repositório **`m3nura/cloud-foundation`**:

1. Acesse: **Settings → CI/CD → Pipeline triggers**
2. Clique em **Add trigger**
3. **Description:** `Preview Deploy from Codebase repos`
4. Copie o token gerado (`glptt-xxx...`)
5. Adicione como variable `PREVIEW_DEPLOY_TOKEN` no Group `m3nura`

**Benefícios:**
- ✅ Herdadas por todos os projects do Group
- ✅ **Não podem ser sobrescritas** por projects (governança!)
- ✅ Protected variables só acessíveis em branches protegidas
- ✅ Masked variables nunca aparecem em logs

### Protected Branches

Configure: **Settings → Repository → Protected Branches**

**Recomendado:**
- `main` - Apenas maintainers podem push, requer MR aprovado
- `sandbox` - Developers podem push, requer MR aprovado
- `develop` - Developers podem push

### Approval Rules (Merge Requests)

Configure no **Group level**: **Settings → General → Merge request approvals**

**Configuração recomendada:**
```
Approvals required: 2
Approvers: @root (team)
Prevent approval by author: ✅
Require new approvals when new commits: ✅
```

**No GitLab Free:**
- ✅ Approval rules funcionam nativamente!
- ✅ Code owners nativos
- ✅ Não precisa pagar (no GitHub só no Team $4/user/month)

---

## Migração do GitHub Actions

### Diferenças Principais

| GitHub Actions | GitLab CI | Equivalente |
|----------------|-----------|-------------|
| `workflow_call` | `include` + `extends` | Templates reutilizáveis |
| `workflow_dispatch` | `when: manual` | Execução manual |
| `environment` | `environment` | Environments nativos |
| `needs` | `needs` | Dependências entre jobs |
| `actions/checkout@v4` | Automático | Git clone nativo |
| `actions/setup-node@v4` | `image: node:20` | Docker image |
| `trstringer/manual-approval` | `when: manual` | Aprovação nativa! |

### Vantagens do GitLab

1. ✅ **Aprovação manual nativa** - não precisa de actions de terceiros
2. ✅ **Templates mais simples** - `include` e `extends` são nativos
3. ✅ **Environments nativos** - histórico, rollback, auto-cleanup
4. ✅ **Sem problemas de permissões** - templates em repos privados funcionam
5. ✅ **Governança mais forte** - Group variables não sobrescrevíveis
6. ✅ **Approval rules no Free** - GitHub só tem no Team ($4/user/month)

---

## Self-hosted Runners (Opcional)

Se 400 minutos/mês não for suficiente, configure runners próprios.

### Docker Runner (Recomendado)

```bash
# Instalar GitLab Runner
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh | sudo bash
sudo apt-get install gitlab-runner

# Registrar runner
sudo gitlab-runner register \
  --url https://gitlab.com \
  --token <seu-token> \
  --executor docker \
  --docker-image alpine:latest

# Iniciar runner
sudo gitlab-runner start
```

**Custo:** ~$5-10/mês (VPS básico) vs $4/user/month GitHub Team

---

## Verificação

Após configurar, verifique:

1. ✅ Pipeline aparece em **CI/CD → Pipelines**
2. ✅ Jobs executam na ordem correta
3. ✅ Artefatos são gerados (`build` job)
4. ✅ Preview deploy aparece como **manual** em MRs
5. ✅ Environment é criado em **Deployments → Environments**

---

## Troubleshooting

### Pipeline não inicia

**Verificar:**
- Arquivo `.gitlab-ci.yml` está na **raiz** do projeto
- Sintaxe YAML está correta (use [GitLab CI Lint](https://gitlab.com/m3nura/seu-projeto/-/ci/lint))
- Você tem acesso ao projeto `m3nura/pipelines`

### Job falha com "template not found"

**Solução:**
- Verifique se o `include` está correto:
  ```yaml
  include:
    - project: 'm3nura/pipelines'
      ref: main
      file: '.gitlab/ci/codebase-ci-node.yml'  # Caminho específico
  ```
- Verifique se você tem acesso ao projeto `m3nura/pipelines`
- Confirme que a branch `main` existe no repositório de pipelines

### Preview deploy não aparece

**Verificar:**
- Está em um Merge Request (MR)?
- Job `build` completou com sucesso?
- Variable `PREVIEW_DEPLOY_TOKEN` está configurada?
- Variable `ARTIFACT_NAME` está definida?
- Variable `PREVIEW_URL` está definida?

### Variable não está disponível

**Erro:** Variable `PREVIEW_DEPLOY_TOKEN` não definida

**Solução:**
1. Configure no Group level: **Settings → CI/CD → Variables**
2. Marque como **Protected** se usar em branches protegidas
3. Marque como **Masked** para não aparecer em logs

---

## Links Úteis

- 📖 [README Principal](../../README.md) - Documentação vendor-agnostic
- 🔧 [Templates CI](../../.gitlab/ci/) - Código-fonte dos templates
- 🚀 [Templates Deploy](../../.gitlab/deploy/) - Templates de deploy
- 📦 [Templates Release](../../.gitlab/release/) - Templates de release
- 📚 [GitLab CI/CD Docs](https://docs.gitlab.com/ee/ci/) - Documentação oficial
- 🔄 [Migração GitHub → GitLab](https://docs.gitlab.com/ee/ci/migration/github_actions.html)

---

**Mantido por:** Menura DevOps Team
**Contato:** Via [GitLab Issues](https://gitlab.com/m3nura/pipelines/-/issues) ou MRs
