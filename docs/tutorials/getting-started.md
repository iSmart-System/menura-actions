# Tutorial: Primeiros Passos

> 🎓 Neste tutorial, você vai configurar seu primeiro projeto para usar os workflows padronizados do menura-actions.

## O que você vai aprender

- Como estruturar as branches do seu projeto
- Como adicionar os workflows reutilizáveis
- Como verificar se tudo está funcionando

## Pré-requisitos

- Um repositório na organização iSmart-System
- Permissões de admin no repositório
- Git instalado localmente

## Passo 1: Preparar as Branches

Primeiro, vamos garantir que seu repositório tenha as duas branches principais.

### 1.1 Criar branch sandbox

```bash
# Clone seu repositório
git clone git@github.com:iSmart-System/seu-projeto.git
cd seu-projeto

# Crie a branch sandbox a partir de main
git checkout main
git pull origin main
git checkout -b sandbox
git push -u origin sandbox
```

### 1.2 Verificar estrutura

Agora você deve ter:

```
Branches:
├── main      → Produção
└── sandbox   → Homologação
```

## Passo 2: Criar Estrutura de Workflows

Vamos criar a pasta de workflows no seu projeto.

### 2.1 Criar diretório

```bash
mkdir -p .github/workflows
```

### 2.2 Estrutura final

```
seu-projeto/
├── .github/
│   └── workflows/
│       ├── ci.yml                   # CI Pipeline
│       ├── release.yml              # Gera releases
│       └── release-management.yml   # Interface para criar RCs e promover
└── ... (seu código)
```

## Passo 3: Adicionar Workflow de CI

Escolha o workflow correspondente à sua tech stack:

### Para Projetos Node.js

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
      artifact-path: 'dist'  # Ajuste conforme seu projeto
    secrets: inherit
```

### Para Projetos Bun

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

### O que esse arquivo faz?

- **Trigger:** Roda em PRs e pushes para sandbox/main
- **Reutilização:** Usa o workflow de CI do menura-actions
- **Configuração:** Especifica runtime e versão
- **Secrets:** Passa secrets do repositório

## Passo 4: Adicionar Workflows de Release

### 4.1 Workflow de Release (automático)

Crie `.github/workflows/release.yml`:

```yaml
name: Release

on:
  push:
    tags: ['v*']

jobs:
  release:
    uses: iSmart-System/menura-actions/.github/workflows/codebase-release-node.yml@main
    with:
      artifact-name: 'seu-projeto'
      artifact-path: 'dist'
    secrets: inherit
```

### 4.2 Release Management (manual)

Copie de `examples/codebase-project/release-management.yml` para `.github/workflows/release-management.yml`.

Este workflow fornece interface manual para:
- Criar Release Candidates
- Promover RCs para produção

## Passo 5: Adicionar CODEOWNERS

Copie o arquivo CODEOWNERS dos exemplos:

```bash
cp examples/codebase-project/CODEOWNERS .github/CODEOWNERS
```

Customize conforme necessário para seu projeto.

## Passo 6: Testar a Configuração

### 6.1 Commit e push

```bash
git add .github/
git commit -m "feat(ci): adicionar workflows padronizados menura-actions"
git push origin sandbox
```

### 6.2 Verificar execução

1. Acesse seu repositório no GitHub
2. Vá em **Actions**
3. Você deve ver o workflow "CI" executando

### 6.3 Resultado esperado

```
✅ CI Pipeline
   ├── ✅ Checkout código
   ├── ✅ Setup Node.js/Bun
   ├── ✅ Instalar dependências
   ├── ✅ Executar Lint
   ├── ✅ Executar Testes
   └── ✅ Executar Build
```

## Passo 7: Configurar Proteção de Branches

No GitHub, configure as proteções:

### 7.1 Branch sandbox

1. Vá em **Settings** → **Branches**
2. Clique **Add rule**
3. Branch name pattern: `sandbox`
4. Marque:
   - ✅ Require a pull request before merging
   - ✅ Require status checks to pass
   - ✅ Require review from Code Owners
   - Selecione "CI Pipeline" como required

### 7.2 Branch main

1. Adicione outra regra para `main`
2. Marque as mesmas opções + mais restritivo:
   - ✅ Require 2 approvals
   - ✅ Require review from Code Owners

## Recapitulando

Neste tutorial você:

1. ✅ Criou a branch `sandbox`
2. ✅ Adicionou workflow de CI (Node.js ou Bun)
3. ✅ Adicionou workflows de Release
4. ✅ Adicionou CODEOWNERS
5. ✅ Testou a execução
6. ✅ Configurou proteção de branches

## Próximos Passos

- [Tutorial: Criando uma Release](first-release.md) - Aprenda o fluxo de release e tags
- [How-To: Setup CI](../how-to/setup-ci.md) - Opções avançadas de CI

---

*Teve problemas? Consulte os [How-To Guides](../how-to/) ou abra uma issue.*
