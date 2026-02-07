# Menura Pipelines - GitLab CI/CD

Repositório central de templates reutilizáveis de CI/CD para a organização Menura no GitLab.

## Estrutura

```
m3nura/
├── pipelines/              # Este repositório (templates)
├── menura-documentation-portal/
├── menura-cloud-foundation/
└── ...outros projetos
```

## Como Usar

### 1. Incluir Templates no Seu Projeto

Crie `.gitlab-ci.yml` na raiz do seu projeto com **includes diretos** dos arquivos específicos:

```yaml
include:
  # Include DIRETO dos arquivos específicos (best practice!)
  - project: 'm3nura/pipelines'
    ref: main
    file: '.gitlab/ci/codebase-ci-node.yml'

stages:
  - test
  - build
  - deploy

variables:
  NODE_VERSION: "20"
  ARTIFACT_PATH: "dist"

# Estender templates
lint:
  extends: .node-lint

test:
  extends: .node-test

build:
  extends: .node-build

preview:
  extends: .preview-deploy
  variables:
    ARTIFACT_NAME: "meu-app"
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

### 2. Templates Disponíveis

#### Node.js
- `.node-base` - Configuração base (image, cache, npm ci)
- `.node-lint` - Executar lint
- `.node-test` - Executar testes com coverage
- `.node-build` - Executar build e gerar artefatos

#### Bun
- `.bun-base` - Configuração base (image, cache, bun install)
- `.bun-lint` - Executar lint
- `.bun-test` - Executar testes
- `.bun-build` - Executar build e gerar artefatos

#### Deploy
- `.preview-deploy` - Preview deploy manual com environment nativo
- `.stop-preview` - Cleanup de preview environment

#### Release
- `.create-release-candidate` - Criar RC (manual, branch sandbox)
- `.qualify-release` - Qualificar RC para produção (manual, branch main)

### 3. Variáveis de Controle

Você pode pular stages específicos definindo variáveis:

```yaml
variables:
  SKIP_LINT: "true"   # Pular lint
  SKIP_TESTS: "true"  # Pular testes
  SKIP_BUILD: "true"  # Pular build
```

### 4. Preview Deploy

O preview deploy é **manual** (botão na UI do GitLab) e cria environments nativos:

```yaml
preview:
  extends: .preview-deploy
  variables:
    ARTIFACT_NAME: "meu-app"
  needs:
    - job: build
      artifacts: true
```

**Aprovação:**
- ✅ Nativa do GitLab (botão "Play")
- ✅ Não precisa de actions de terceiros
- ✅ Environment com histórico e rollback

**Cleanup automático:**
- ✅ `auto_stop_in: 7 days` - Remove após 7 dias
- ✅ `on_stop: stop_preview` - Botão manual de cleanup

## Governança Centralizada

### Group-level Variables (Recomendado)

Configure no Group `m3nura`:
- Settings → CI/CD → Variables

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

Configure branches protegidas:
- Settings → Repository → Protected Branches

**Recomendado:**
- `main` - Apenas maintainers podem push, requer MR
- `sandbox` - Developers podem push, requer MR
- `develop` - Developers podem push

## Approval Rules (Merge Requests)

Configure approval rules no **Group level** (herança automática):
- Settings → General → Merge request approvals

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

## Exemplos

### CI Simples (Node.js)

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

lint:
  extends: .node-lint

test:
  extends: .node-test

build:
  extends: .node-build
```

### CI com Preview Deploy

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

# Preview deploy manual
preview:
  extends: .preview-deploy
  needs:
    - job: build
      artifacts: true

# Cleanup do preview
stop_preview:
  extends: .stop-preview
```

### CI sem Lint/Tests (Docusaurus)

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

build:
  extends: .node-build

preview:
  extends: .preview-deploy
  variables:
    ARTIFACT_NAME: "docs"
```

## Migração do GitHub Actions

### Diferenças principais

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

1. **Aprovação manual nativa** - não precisa de actions de terceiros
2. **Templates mais simples** - `include` e `extends` são nativos
3. **Environments nativos** - histórico, rollback, auto-cleanup
4. **Sem problemas de permissões** - templates em repos privados funcionam
5. **Governança mais forte** - Group variables não sobrescrevíveis
6. **Approval rules no Free** - GitHub só tem no Team ($$$)

## Self-hosted Runners (Opcional)

Se 400 minutos/mês não for suficiente, configure runners próprios:

### Docker runner (recomendado)

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

## Troubleshooting

### Job não encontra template

**Erro:** `Include file '.gitlab/ci/codebase-ci-node.yml' does not exist`

**Solução:**
1. Verifique se o projeto `m3nura/pipelines` existe
2. Verifique se você tem acesso ao projeto (visibilidade)
3. Confirme que o caminho do arquivo está correto:
   - `.gitlab/ci/codebase-ci-node.yml` para templates Node.js
   - `.gitlab/ci/codebase-ci-bun.yml` para templates Bun
   - `.gitlab/deploy/codebase-preview-deploy.yml` para preview deploy
4. Verifique se a branch `main` existe no repositório de pipelines

### Preview deploy não dispara

**Erro:** Job não aparece na pipeline

**Verificar:**
1. Está em um Merge Request? (`if: '$CI_PIPELINE_SOURCE == "merge_request_event"'`)
2. Job `build` completou com sucesso? (`needs: [build]`)
3. Artefatos do build foram gerados? (`artifacts: paths`)

### Variable não está disponível

**Erro:** Variable `PREVIEW_DEPLOY_TOKEN` não definida

**Solução:**
1. Configure no Group level: Settings → CI/CD → Variables
2. Marque como **Protected** se usar em branches protegidas
3. Marque como **Masked** para não aparecer em logs

## Links Úteis

- [GitLab CI/CD Docs](https://docs.gitlab.com/ee/ci/)
- [Templates CI](./.gitlab/ci/)
- [Templates Deploy](./.gitlab/deploy/)
- [Templates Release](./.gitlab/release/)
- [Exemplos](./examples/gitlab/)
- [Migração do GitHub Actions](https://docs.gitlab.com/ee/ci/migration/github_actions.html)

---

**Mantido por:** Menura DevOps Team
**Contato:** Via GitLab Issues ou MRs
