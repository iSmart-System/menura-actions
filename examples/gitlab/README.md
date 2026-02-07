# GitLab CI/CD - Documentação Completa

Documentação e exemplos práticos para usar os templates do `m3nura/pipelines` no GitLab CI/CD.

---

## Arquivos de Exemplo

| Arquivo | Descrição | Quando Usar |
|---------|-----------|-------------|
| [`ci-node.yml`](ci-node.yml) | CI completo para Node.js | Projetos Node.js com lint, testes e build |
| [`ci-bun.yml`](ci-bun.yml) | CI completo para Bun | Projetos Bun com lint, testes e build |
| [`ci-node-with-preview.yml`](ci-node-with-preview.yml) | CI Node.js + Preview Deploy | Projetos que precisam preview em MRs |
| [`ci-node-skip-tests.yml`](ci-node-skip-tests.yml) | CI Node.js sem lint/tests | Projetos como Docusaurus |

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
- `PREVIEW_DEPLOY_TOKEN` (se usar preview deploy)

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
| `.preview-deploy` | Preview deploy manual com environment nativo |
| `.stop-preview` | Cleanup de preview environment |

### Release

| Template | Descrição |
|----------|-----------|
| `.create-release-candidate` | Criar RC (manual, branch `sandbox`) |
| `.qualify-release` | Qualificar RC para produção (manual, branch `main`) |

---

## Variáveis de Controle

### Pular Stages Específicos

```yaml
variables:
  SKIP_LINT: "true"   # Pular lint
  SKIP_TESTS: "true"  # Pular testes
  SKIP_BUILD: "true"  # Pular build
```

### Configurar Versões

```yaml
variables:
  NODE_VERSION: "20"      # Node.js: "18", "20", "latest"
  BUN_VERSION: "latest"   # Bun: "latest", "1.0.0", etc
```

### Configurar Artefatos

```yaml
variables:
  ARTIFACT_PATH: "dist"         # Caminho do build
  ARTIFACT_NAME: "meu-app"      # Nome do artefato
```

### Preview Deploy

```yaml
variables:
  FOUNDATION_REPO: "m3nura/custom-foundation"  # Repo de deploy customizado
```

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
  NODE_VERSION: "20"
  ARTIFACT_PATH: "dist"

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

stop_preview:
  extends: .stop-preview
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
  ARTIFACT_NAME: "docs"

build:
  extends: .node-build

preview:
  extends: .preview-deploy
  needs:
    - job: build
      artifacts: true

stop_preview:
  extends: .stop-preview
```

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

### Preview Deploy Customizado

```yaml
preview:
  extends: .preview-deploy
  variables:
    ARTIFACT_NAME: "meu-app"
    FOUNDATION_REPO: "m3nura/custom-foundation"
  environment:
    url: https://preview-mr-$CI_MERGE_REQUEST_IID.custom.com
```

---

## Governança e Configuração

### Group-level Variables (Recomendado)

Configure no Group `m3nura`: **Settings → CI/CD → Variables**

**Variáveis recomendadas:**

| Variable | Value | Protected | Masked | Description |
|----------|-------|-----------|--------|-------------|
| `PREVIEW_DEPLOY_TOKEN` | `ghp_xxx...` | ✅ | ✅ | PAT para foundation |
| `PREVIEW_DEPLOY_APPROVERS` | `user1,user2` | ❌ | ❌ | Lista de aprovadores |
| `DEPLOY_NOTIFICATION_WEBHOOK` | `https://...` | ✅ | ✅ | Webhook para notificações |

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
