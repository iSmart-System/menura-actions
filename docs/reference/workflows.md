# Reference: Workflows

Documentação técnica de todos os workflows disponíveis no menura-actions para repositórios **Codebase**.

---

## codebase-ci-node.yml

**Propósito:** Workflow reutilizável para Continuous Integration em projetos Node.js.

**Arquivo:** `.github/workflows/codebase-ci-node.yml`

**Trigger:** `workflow_call` (chamado por outros workflows)

### Inputs

| Input | Tipo | Obrigatório | Default | Descrição |
|-------|------|-------------|---------|-----------|
| `node-version` | string | não | `'20'` | Versão do Node.js |
| `working-directory` | string | não | `'.'` | Diretório de trabalho |
| `skip-lint` | boolean | não | `false` | Pular etapa de lint |
| `skip-tests` | boolean | não | `false` | Pular etapa de testes |
| `skip-build` | boolean | não | `false` | Pular etapa de build |
| `artifact-path` | string | não | `'dist'` | Caminho dos artefatos de build |
| `upload-artifacts` | boolean | não | `true` | Fazer upload dos artefatos |

### Jobs

1. **ci** - Pipeline principal
   - Checkout código
   - Setup Node.js
   - Instalar dependências (`npm ci`)
   - Executar Lint (`npm run lint`)
   - Executar Testes (`npm test`)
   - Executar Build (`npm run build`)
   - Upload artefatos

### Exemplo de Uso

```yaml
jobs:
  ci:
    uses: iSmart-System/menura-actions/.github/workflows/codebase-ci-node.yml@main
    with:
      node-version: '20'
      artifact-path: 'dist'  # ou 'build', '.next', etc
    secrets: inherit
```

---

## codebase-ci-bun.yml

**Propósito:** Workflow reutilizável para Continuous Integration em projetos Bun.

**Arquivo:** `.github/workflows/codebase-ci-bun.yml`

**Trigger:** `workflow_call` (chamado por outros workflows)

### Inputs

| Input | Tipo | Obrigatório | Default | Descrição |
|-------|------|-------------|---------|-----------|
| `bun-version` | string | não | `'latest'` | Versão do Bun |
| `working-directory` | string | não | `'.'` | Diretório de trabalho |
| `skip-lint` | boolean | não | `false` | Pular etapa de lint |
| `skip-tests` | boolean | não | `false` | Pular etapa de testes |
| `skip-build` | boolean | não | `false` | Pular etapa de build |
| `artifact-path` | string | não | `'dist'` | Caminho dos artefatos de build |
| `upload-artifacts` | boolean | não | `true` | Fazer upload dos artefatos |

### Jobs

1. **ci** - Pipeline principal
   - Checkout código
   - Setup Bun
   - Instalar dependências (`bun install --frozen-lockfile`)
   - Executar Lint (`bun run lint`)
   - Executar Testes (`bun test`)
   - Executar Build (`bun run build`)
   - Upload artefatos

### Exemplo de Uso

```yaml
jobs:
  ci:
    uses: iSmart-System/menura-actions/.github/workflows/codebase-ci-bun.yml@main
    with:
      bun-version: 'latest'
      artifact-path: 'dist'
    secrets: inherit
```

---

## codebase-release-node.yml

**Propósito:** Gera artefatos (.zip) e publica no GitHub Releases para projetos Node.js.

**Arquivo:** `.github/workflows/codebase-release-node.yml`

**Trigger:** `workflow_call` (tipicamente via push de tags)

### Inputs

| Input | Tipo | Obrigatório | Default | Descrição |
|-------|------|-------------|---------|-----------|
| `node-version` | string | não | `'20'` | Versão do Node.js |
| `working-directory` | string | não | `'.'` | Diretório de trabalho |
| `build-command` | string | não | `'npm run build'` | Comando de build |
| `artifact-name` | string | não | `'release'` | Nome do artefato |
| `artifact-path` | string | não | `'dist'` | Caminho dos arquivos |
| `include-source` | boolean | não | `false` | Incluir código fonte |

### Outputs

| Output | Descrição |
|--------|-----------|
| `version` | Versão da release |
| `artifact-url` | URL do artefato |
| `release-url` | URL da release no GitHub |

### Jobs

1. **validate** - Valida formato da tag
2. **build** - Compila e empacota artefato (Node.js)
3. **release** - Publica no GitHub Releases

### Exemplo de Uso

```yaml
name: Release
on:
  push:
    tags: ['v*']
jobs:
  release:
    uses: iSmart-System/menura-actions/.github/workflows/codebase-release-node.yml@main
    with:
      artifact-name: 'meu-projeto'
      artifact-path: 'dist'
    secrets: inherit
```

---

## codebase-release-bun.yml

**Propósito:** Gera artefatos (.zip) e publica no GitHub Releases para projetos Bun.

**Arquivo:** `.github/workflows/codebase-release-bun.yml`

**Trigger:** `workflow_call` (tipicamente via push de tags)

### Inputs

| Input | Tipo | Obrigatório | Default | Descrição |
|-------|------|-------------|---------|-----------|
| `bun-version` | string | não | `'latest'` | Versão do Bun |
| `working-directory` | string | não | `'.'` | Diretório de trabalho |
| `build-command` | string | não | `'bun run build'` | Comando de build |
| `artifact-name` | string | não | `'release'` | Nome do artefato |
| `artifact-path` | string | não | `'dist'` | Caminho dos arquivos |
| `include-source` | boolean | não | `false` | Incluir código fonte |

### Outputs

| Output | Descrição |
|--------|-----------|
| `version` | Versão da release |
| `artifact-url` | URL do artefato |
| `release-url` | URL da release no GitHub |

### Jobs

1. **validate** - Valida formato da tag
2. **build** - Compila e empacota artefato (Bun)
3. **release** - Publica no GitHub Releases

### Exemplo de Uso

```yaml
name: Release
on:
  push:
    tags: ['v*']
jobs:
  release:
    uses: iSmart-System/menura-actions/.github/workflows/codebase-release-bun.yml@main
    with:
      artifact-name: 'meu-projeto'
      artifact-path: 'dist'
    secrets: inherit
```

---

## codebase-create-rc.yml

**Propósito:** Criar tags de Release Candidate automaticamente.

**Arquivo:** `.github/workflows/codebase-create-rc.yml`

**Trigger:** `workflow_call`, `workflow_dispatch`

### Inputs

| Input | Tipo | Obrigatório | Default | Descrição |
|-------|------|-------------|---------|-----------|
| `version-type` | choice | não | `'patch'` | Tipo de incremento: `patch`, `minor`, `major` |

### Outputs

| Output | Descrição |
|--------|-----------|
| `tag` | Tag criada (ex: `v1.2.0-rc.1`) |
| `version` | Versão base (ex: `v1.2.0`) |

### Lógica de Versionamento

```
Última release: v1.0.0
Version type: patch → v1.0.1-rc.1
Version type: minor → v1.1.0-rc.1
Version type: major → v2.0.0-rc.1

Se já existe v1.0.1-rc.1 → v1.0.1-rc.2
```

### Exemplo de Uso

```yaml
jobs:
  create-rc:
    uses: iSmart-System/menura-actions/.github/workflows/codebase-create-rc.yml@main
    with:
      version-type: minor
    secrets: inherit
```

---

## codebase-qualify-rc.yml

**Propósito:** Qualificar Release Candidate como release de produção.

**Arquivo:** `.github/workflows/codebase-qualify-rc.yml`

**Trigger:** `workflow_call`, `workflow_dispatch`

### Inputs

| Input | Tipo | Obrigatório | Default | Descrição |
|-------|------|-------------|---------|-----------|
| `rc-tag` | string | **sim** | - | Tag RC a promover (ex: `v1.2.0-rc.1`) |
| `auto-merge` | boolean | não | `false` | Fazer merge automático do PR |

### Outputs

| Output | Descrição |
|--------|-----------|
| `production-tag` | Tag de produção criada |
| `pr-url` | URL do PR criado |

### Validações

- Tag deve seguir formato `vX.Y.Z-rc.N`
- Tag deve existir no repositório
- Tag de produção (`vX.Y.Z`) não pode existir

### Exemplo de Uso

```yaml
jobs:
  promote:
    uses: iSmart-System/menura-actions/.github/workflows/codebase-qualify-rc.yml@main
    with:
      rc-tag: 'v1.2.0-rc.3'
      auto-merge: false
    secrets: inherit
```

---

## codebase-validate-tag.yml

**Propósito:** Validar nomenclatura de tags.

**Arquivo:** `.github/workflows/codebase-validate-tag.yml`

**Trigger:** `workflow_call`, `push` (tags `v*`)

### Outputs

| Output | Descrição |
|--------|-----------|
| `is-valid` | `true` se tag é válida |
| `tag-type` | `production`, `rc`, ou `invalid` |
| `version` | Versão base extraída |

### Validações

- **Produção:** `vX.Y.Z` (ex: `v1.2.3`)
- **RC:** `vX.Y.Z-rc.N` (ex: `v1.2.3-rc.1`)

---

# Matriz de Compatibilidade

| Workflow | Branch sandbox | Branch main | Tag RC | Tag Prod |
|----------|---------------|-------------|--------|----------|
| codebase-ci-node.yml | ✅ | ✅ | - | - |
| codebase-ci-bun.yml | ✅ | ✅ | - | - |
| codebase-release-node.yml | - | - | ✅ | ✅ |
| codebase-release-bun.yml | - | - | ✅ | ✅ |
| codebase-create-rc.yml | ✅ | ❌ | - | - |
| codebase-qualify-rc.yml | - | - | ✅ | - |
| codebase-validate-tag.yml | - | - | ✅ | ✅ |
