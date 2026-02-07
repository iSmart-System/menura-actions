# Exemplos GitLab CI/CD

Exemplos práticos de como usar os templates do `m3nura/pipelines`.

## Arquivos Disponíveis

| Arquivo | Descrição | Quando Usar |
|---------|-----------|-------------|
| [`ci-node.yml`](ci-node.yml) | CI completo para Node.js | Projetos Node.js com lint, testes e build |
| [`ci-bun.yml`](ci-bun.yml) | CI completo para Bun | Projetos Bun com lint, testes e build |
| [`ci-node-skip-tests.yml`](ci-node-skip-tests.yml) | CI Node.js sem lint/tests | Projetos como Docusaurus |

## Como Usar

1. **Escolha o exemplo** que se aproxima do seu projeto
2. **Copie o conteúdo** para `.gitlab-ci.yml` na raiz do seu projeto
3. **Customize as variáveis:**
   ```yaml
   variables:
     NODE_VERSION: "20"      # Sua versão do Node.js
     ARTIFACT_NAME: "meu-app"  # Nome do seu projeto
     ARTIFACT_PATH: "dist"   # Caminho dos artefatos de build
   ```
4. **Commit e push** para GitLab
5. **Configure variáveis sensíveis** no Group/Project:
   - Settings → CI/CD → Variables
   - Adicione `PREVIEW_DEPLOY_TOKEN` (se usar preview deploy)

## Como Funciona o Include

O `m3nura/pipelines` usa **includes diretos** (best practice):

```yaml
# Incluir arquivo específico
include:
  - project: 'm3nura/pipelines'
    ref: main
    file: '.gitlab/ci/codebase-ci-node.yml'

# Incluir múltiplos arquivos (GitLab 13.6+)
include:
  - project: 'm3nura/pipelines'
    ref: main
    file:
      - '.gitlab/ci/codebase-ci-node.yml'
      - '.gitlab/deploy/codebase-preview-deploy.yml'
```

**Vantagens:**
- Incluir apenas o que você precisa
- Evita carregar templates desnecessários
- Melhor performance e clareza

## Customizações Comuns

### Pular Stages Específicos

```yaml
variables:
  SKIP_LINT: "true"   # Pular lint
  SKIP_TESTS: "true"  # Pular testes
  SKIP_BUILD: "true"  # Pular build
```

### Mudar Versão do Node.js

```yaml
variables:
  NODE_VERSION: "18"  # ou "20", "latest"
```

### Customizar Caminho de Artefatos

```yaml
variables:
  ARTIFACT_PATH: "build"  # Para Docusaurus, Next.js, etc
```

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

### Preview Deploy com Configuração Custom

```yaml
preview:
  extends: .preview-deploy
  variables:
    ARTIFACT_NAME: "meu-app"
    FOUNDATION_REPO: "m3nura/custom-foundation"  # Repo customizado
  environment:
    url: https://preview-mr-$CI_MERGE_REQUEST_IID.custom.com  # URL customizada
```

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

## Verificação

Após configurar, verifique:

1. ✅ Pipeline aparece em **CI/CD → Pipelines**
2. ✅ Jobs executam na ordem correta
3. ✅ Artefatos são gerados (`build` job)
4. ✅ Preview deploy aparece como **manual** em MRs
5. ✅ Environment é criado em **Deployments → Environments**

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
      file: '.gitlab/ci/codebase-ci-node.yml'  # Caminho específico do template
  ```
- Verifique se você tem acesso ao projeto `m3nura/pipelines`
- Confirme que a branch `main` existe no repositório de pipelines

### Preview deploy não aparece

**Verificar:**
- Está em um Merge Request (MR)?
- Job `build` completou com sucesso?
- Variable `ARTIFACT_NAME` está definida?

## Links Úteis

- [Documentação completa](../../GITLAB.md)
- [Templates CI](../../.gitlab/ci/)
- [Templates Deploy](../../.gitlab/deploy/)
- [Templates Release](../../.gitlab/release/)
- [GitLab CI/CD Docs](https://docs.gitlab.com/ee/ci/)
