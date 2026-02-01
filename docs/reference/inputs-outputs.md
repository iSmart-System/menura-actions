# Reference: Inputs & Outputs

Referência completa de todos os inputs e outputs dos workflows disponíveis.

---

## Resumo de Inputs por Workflow

### codebase-ci-node.yml

| Input | Tipo | Default | Descrição |
|-------|------|---------|-----------|
| `node-version` | string | `'20'` | Versão do Node.js |
| `working-directory` | string | `'.'` | Diretório de trabalho |
| `skip-lint` | boolean | `false` | Pular etapa de lint |
| `skip-tests` | boolean | `false` | Pular etapa de testes |
| `skip-build` | boolean | `false` | Pular etapa de build |
| `artifact-path` | string | `'dist'` | Caminho dos artefatos de build |
| `upload-artifacts` | boolean | `true` | Fazer upload dos artefatos de build |

**Outputs:** Nenhum

---

### codebase-ci-bun.yml

| Input | Tipo | Default | Descrição |
|-------|------|---------|-----------|
| `bun-version` | string | `'latest'` | Versão do Bun |
| `working-directory` | string | `'.'` | Diretório de trabalho |
| `skip-lint` | boolean | `false` | Pular etapa de lint |
| `skip-tests` | boolean | `false` | Pular etapa de testes |
| `skip-build` | boolean | `false` | Pular etapa de build |
| `artifact-path` | string | `'dist'` | Caminho dos artefatos de build |
| `upload-artifacts` | boolean | `true` | Fazer upload dos artefatos de build |

**Outputs:** Nenhum

---

### codebase-release-node.yml

| Input | Tipo | Default | Descrição |
|-------|------|---------|-----------|
| `node-version` | string | `'20'` | Versão do Node.js |
| `working-directory` | string | `'.'` | Diretório de trabalho |
| `build-command` | string | `'npm run build'` | Comando de build |
| `artifact-name` | string | `'release'` | Nome do artefato |
| `artifact-path` | string | `'dist'` | Caminho dos arquivos |
| `include-source` | boolean | `false` | Incluir código fonte no artefato |

**Outputs:**

| Output | Descrição |
|--------|-----------|
| `version` | Versão da release (ex: `v1.0.0`) |
| `artifact-url` | URL do artefato gerado |
| `release-url` | URL da release no GitHub |11

---

### codebase-release-bun.yml

| Input | Tipo | Default | Descrição |
|-------|------|---------|-----------|
| `bun-version` | string | `'latest'` | Versão do Bun |
| `working-directory` | string | `'.'` | Diretório de trabalho |
| `build-command` | string | `'bun run build'` | Comando de build |
| `artifact-name` | string | `'release'` | Nome do artefato |
| `artifact-path` | string | `'dist'` | Caminho dos arquivos |
| `include-source` | boolean | `false` | Incluir código fonte no artefato |

**Outputs:**

| Output | Descrição |
|--------|-----------|
| `version` | Versão da release (ex: `v1.0.0`) |
| `artifact-url` | URL do artefato gerado |
| `release-url` | URL da release no GitHub |

---

### codebase-create-rc.yml

| Input | Tipo | Default | Descrição |
|-------|------|---------|-----------|
| `version-type` | choice | `'patch'` | Tipo de incremento: `patch`, `minor`, `major` |

**Outputs:**

| Output | Descrição |
|--------|-----------|
| `tag` | Tag criada (ex: `v1.2.0-rc.1`) |
| `version` | Versão base (ex: `v1.2.0`) |

---

### codebase-qualify-rc.yml

| Input | Tipo | Default | Descrição |
|-------|------|---------|-----------|
| `rc-tag` | string | - | Tag RC a promover (ex: `v1.2.0-rc.1`) ⚠️ **Obrigatório** |
| `auto-merge` | boolean | `false` | Fazer merge automático do PR |

**Outputs:**

| Output | Descrição |
|--------|-----------|
| `production-tag` | Tag de produção criada (ex: `v1.2.0`) |
| `pr-url` | URL do PR criado |

---

### codebase-validate-tag.yml

**Inputs:** Nenhum (usa automaticamente `github.ref_name`)

**Outputs:**

| Output | Descrição |
|--------|-----------|
| `is-valid` | `true` se tag é válida |
| `tag-type` | `production`, `rc`, ou `invalid` |
| `version` | Versão base extraída (ex: `1.2.0`) |

---

## Inputs Comuns

Inputs que aparecem em múltiplos workflows:

| Input | Workflows | Descrição |
|-------|-----------|-----------|
| `node-version` | CI Node, Release | Versão do Node.js |
| `working-directory` | CI Node, CI Bun, Release | Diretório de trabalho |
| `artifact-path` | CI Node, CI Bun, Release | Caminho dos artefatos |
| `upload-artifacts` | CI Node, CI Bun | Controlar upload de artefatos de build |

---

## Secrets

Todos os workflows usam `secrets: inherit` para passar secrets automaticamente.

Secrets comumente utilizados:

| Secret | Uso | Descrição |
|--------|-----|-----------|
| `GITHUB_TOKEN` | Todos | Token automático do GitHub Actions |
| `GH_TOKEN` | Release, Create RC, Promote | Token para criação de releases e PRs |

---

## Exemplos de Uso

### CI com inputs customizados

```yaml
jobs:
  ci:
    uses: iSmart-System/menura-actions/.github/workflows/codebase-ci-node.yml@main
    with:
      node-version: '18'
      working-directory: 'packages/api'
      artifact-path: 'build'
      skip-tests: false
      upload-artifacts: false  # Desabilitar upload em PRs
    secrets: inherit
```

### Release com configurações específicas

```yaml
jobs:
  release:
    uses: iSmart-System/menura-actions/.github/workflows/codebase-release-node.yml@main
    with:
      artifact-name: 'my-app'
      artifact-path: '.next'
      build-command: 'npm run build:production'
      include-source: true
    secrets: inherit
```

### Create RC com minor version

```yaml
jobs:
  create-rc:
    uses: iSmart-System/menura-actions/.github/workflows/codebase-create-rc.yml@main
    with:
      version-type: 'minor'  # v1.1.0-rc.1
    secrets: inherit
```

### Promote com auto-merge

```yaml
jobs:
  promote:
    uses: iSmart-System/menura-actions/.github/workflows/codebase-qualify-rc.yml@main
    with:
      rc-tag: 'v1.2.0-rc.3'
      auto-merge: true
    secrets: inherit
```

---

## Validação de Inputs

Todos os workflows validam inputs antes da execução:

- **Tipos:** Verificados pelo GitHub Actions
- **Formatos:** Tags devem seguir SemVer (`vX.Y.Z` ou `vX.Y.Z-rc.N`)
- **Paths:** Diretórios devem existir
- **Versões:** Node/Bun devem ser válidos

---

*Veja também: [Workflows Reference](workflows.md) para documentação completa de cada workflow.*
