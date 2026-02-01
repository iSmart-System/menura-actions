# Como Configurar CI em um Novo Projeto

## Objetivo

Adicionar o workflow de CI padronizado a um novo projeto da organização.

## Pré-requisitos

- Repositório criado na organização iSmart-System
- Branches `main` e `sandbox` existentes
- Projeto com código Node.js ou Bun
- Scripts de lint, test e build configurados no projeto

## Passos

### 1. Criar diretório de workflows

```bash
mkdir -p .github/workflows
```

### 2. Criar arquivo de CI

Escolha o workflow correspondente à sua tech stack:

#### Para Projetos Node.js

Crie `.github/workflows/ci.yml`:

```yaml
name: CI

on:
  pull_request:
    branches: [sandbox, main]
  push:
    branches: [sandbox, main]

jobs:
  ci:
    uses: iSmart-System/menura-actions/.github/workflows/codebase-ci-node.yml@main
    with:
      node-version: '20'
      artifact-path: 'dist'  # ou 'build', '.next', etc
    secrets: inherit
```

#### Para Projetos Bun

Crie `.github/workflows/ci.yml`:

```yaml
name: CI

on:
  pull_request:
    branches: [sandbox, main]
  push:
    branches: [sandbox, main]

jobs:
  ci:
    uses: iSmart-System/menura-actions/.github/workflows/codebase-ci-bun.yml@main
    with:
      bun-version: 'latest'
      artifact-path: 'dist'
    secrets: inherit
```

### 3. Configurar scripts no projeto

#### Node.js - package.json

Certifique-se que seu `package.json` tem:

```json
{
  "scripts": {
    "lint": "eslint .",
    "test": "jest",
    "build": "tsc"
  }
}
```

#### Bun - package.json

Para Bun, configure:

```json
{
  "scripts": {
    "lint": "eslint .",
    "test": "bun test",
    "build": "bun run build:script"
  }
}
```

### 4. Commit e push

```bash
git add .github/workflows/ci.yml
git commit -m "feat(ci): adicionar workflow de CI padronizado"
git push
```

## Verificação

1. Acesse **Actions** no GitHub
2. O workflow "CI" deve estar executando
3. Todos os steps devem passar ✅

## Opções Avançadas

### Node.js: Mudar versão

```yaml
with:
  node-version: '18'  # ou '20', '22', etc
```

### Bun: Especificar versão

```yaml
with:
  bun-version: '1.0.0'  # ou 'latest'
```

### Customizar path dos artefatos

```yaml
with:
  artifact-path: 'build'  # Padrão: 'dist'
```

### Pular etapas

```yaml
with:
  skip-lint: false
  skip-tests: false
  skip-build: false
```

### Monorepo (subdiretório)

```yaml
with:
  working-directory: 'packages/api'
```

## Troubleshooting

| Problema | Solução |
|----------|---------|
| Script não encontrado | Adicione o script ao `package.json` |
| Erro de permissão | Verifique se `secrets: inherit` está presente |
| Node/Bun version errada | Especifique a versão correta via input |
| Artefatos não encontrados | Ajuste `artifact-path` para o diretório correto |

---

*Veja também: [Reference: CI Workflows](../reference/workflows.md)*
