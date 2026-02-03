# Tutorial: Setup Completo de Novo Repositório Codebase

## Objetivo

Este guia fornece um checklist completo para inicializar um novo repositório Codebase com **todas as melhores práticas e funcionalidades** do menura-actions.

Ao final deste guia, seu repositório terá:
- ✅ CI/CD completo (lint, test, build)
- ✅ Sistema de releases automático (RC → Produção)
- ✅ Preview deploy manual de PRs
- ✅ Proteção de branches configurada
- ✅ Code review obrigatório
- ✅ Secrets configurados
- ✅ Documentação básica

**Tempo estimado:** 30-45 minutos

---

## Pré-requisitos

- [ ] Repositório criado na organização `iSmart-System`
- [ ] Você tem permissões de **admin** no repositório
- [ ] Projeto com código Node.js ou Bun
- [ ] Scripts `lint`, `test`, `build` configurados no `package.json`

---

## Fase 1: Estrutura Básica

### 1.1 Criar Branches Principais

```bash
# No seu repositório local
git checkout -b main
git push -u origin main

git checkout -b sandbox
git push -u origin sandbox
```

### 1.2 Configurar Branch Padrão

1. GitHub → Settings → General → Default branch
2. Alterar para: `sandbox`
3. Salvar

> 💡 **Por quê?** Desenvolvimento acontece em sandbox, produção em main

### 1.3 Criar Estrutura de Diretórios

```bash
mkdir -p .github/workflows
touch .github/CODEOWNERS
touch .github/PULL_REQUEST_TEMPLATE.md
```

---

## Fase 2: Workflows CI/CD

### 2.1 Escolher Tech Stack

Copie os workflows apropriados de [examples/codebase-project/](../../examples/codebase-project/):

#### Para Node.js:

```bash
# CI básico
cp examples/codebase-project/ci-node.yml .github/workflows/ci.yml

# OU CI com preview deploy
cp examples/codebase-project/ci-with-preview-node.yml .github/workflows/ci.yml

# Release
cp examples/codebase-project/release-node.yml .github/workflows/release.yml

# Gestão de releases (opcional)
cp examples/codebase-project/release-management.yml .github/workflows/release-management.yml
```

#### Para Bun:

```bash
# CI básico
cp examples/codebase-project/ci-bun.yml .github/workflows/ci.yml

# OU CI com preview deploy
cp examples/codebase-project/ci-with-preview-bun.yml .github/workflows/ci.yml

# Release
cp examples/codebase-project/release-bun.yml .github/workflows/release.yml

# Gestão de releases (opcional)
cp examples/codebase-project/release-management.yml .github/workflows/release-management.yml
```

### 2.2 Customizar Workflows

Edite `.github/workflows/ci.yml`:

```yaml
with:
  node-version: '20'  # ou '18', '22', etc
  bun-version: 'latest'  # para Bun
  artifact-path: 'dist'  # ajuste: 'build', '.next', 'out', etc
  artifact-name: 'meu-projeto'  # nome do seu projeto
  repository-name: 'meu-projeto'  # mesmo nome
```

Edite `.github/workflows/release.yml`:

```yaml
with:
  artifact-name: 'meu-projeto'  # mesmo nome do CI
  artifact-path: 'dist'  # mesmo path do CI
```

### 2.3 Verificar Scripts

Certifique-se que seu `package.json` tem:

```json
{
  "scripts": {
    "lint": "eslint .",
    "test": "jest",  // ou "vitest", "bun test", etc
    "build": "tsc"   // ou "vite build", "next build", etc
  }
}
```

---

## Fase 3: Proteção de Branches

### 3.1 Proteger Branch `sandbox`

GitHub → Settings → Branches → Add rule

```
Branch name pattern: sandbox

✅ Require a pull request before merging
   ✅ Require approvals: 1
   ✅ Dismiss stale pull request approvals when new commits are pushed

✅ Require status checks to pass before merging
   ✅ Require branches to be up to date before merging
   Status checks:
      [x] ci / CI Pipeline  (aparece após primeiro CI rodar)

✅ Do not allow bypassing the above settings
```

### 3.2 Proteger Branch `main`

GitHub → Settings → Branches → Add rule

```
Branch name pattern: main

✅ Require a pull request before merging
   ✅ Require approvals: 2
   ✅ Dismiss stale pull request approvals when new commits are pushed
   ✅ Require review from Code Owners

✅ Require status checks to pass before merging
   ✅ Require branches to be up to date before merging
   Status checks:
      [x] ci / CI Pipeline
      [x] validate / Validar Branch (para releases)

✅ Require signed commits (opcional)

✅ Do not allow bypassing the above settings

✅ Restrict who can push to matching branches
   Add: DevOps team, Tech Leads
```

---

## Fase 4: Code Owners

### 4.1 Configurar CODEOWNERS

Crie `.github/CODEOWNERS`:

```
# Formato: padrão @time-ou-usuario

# Owners globais (todos os arquivos)
* @iSmart-System/tech-leads

# Backend
/src/backend/ @iSmart-System/backend-team
/src/api/ @iSmart-System/backend-team

# Frontend
/src/frontend/ @iSmart-System/frontend-team
/src/components/ @iSmart-System/frontend-team

# Infraestrutura
/.github/ @iSmart-System/devops-team
/terraform/ @iSmart-System/devops-team
/docker/ @iSmart-System/devops-team

# Documentação
/docs/ @iSmart-System/tech-writers @iSmart-System/tech-leads

# Configurações críticas
package.json @iSmart-System/tech-leads
tsconfig.json @iSmart-System/tech-leads
.eslintrc* @iSmart-System/tech-leads
```

### 4.2 Criar Times no GitHub

Se os times não existirem:

1. Organization → Teams → New team
2. Criar: `backend-team`, `frontend-team`, `devops-team`, `tech-leads`
3. Adicionar membros apropriados

---

## Fase 5: Secrets (Opcional)

### 5.1 Secrets Básicos

**Nenhum secret é necessário para funcionalidade básica!** O `GITHUB_TOKEN` é suficiente.

### 5.2 Preview Deploy (Opcional)

Se você copiou `ci-with-preview-*.yml`, configure:

**1. Criar PAT:**

Siga o guia: [Como Criar PAT](../how-to/preview-deploy.md#1-personal-access-token-pat)

**2. Adicionar Secret:**

```
Organization Settings → Secrets → Actions
Name: PREVIEW_DEPLOY_TOKEN
Value: [seu token]
Repository access: [selecione este repositório]
```

**3. Criar Environment:**

```
Repository → Settings → Environments → New environment
Name: sandbox-preview

✅ Required reviewers
   Add: Tech Leads, DevOps team

✅ Prevent self-review (recomendado)

Save protection rules
```

### 5.3 Secrets de Deploy (Não aqui!)

⚠️ **Importante:** Secrets de AWS, Vercel, databases, etc. vão no repositório **Architecture Live** (Terragrunt), **NÃO** aqui!

---

## Fase 6: Templates

### 6.1 Pull Request Template

Crie `.github/PULL_REQUEST_TEMPLATE.md`:

```markdown
## Resumo

[Descreva brevemente o que esse PR faz]

## Tipo de Mudança

- [ ] 🐛 Bug fix
- [ ] ✨ Nova feature
- [ ] 💥 Breaking change
- [ ] 📝 Documentação
- [ ] 🎨 Refactoring
- [ ] ⚡ Performance
- [ ] ✅ Testes

## Mudanças

- [Lista as principais mudanças]

## Como Testar

1. [Passo 1]
2. [Passo 2]
3. [Verificar...]

## Checklist

- [ ] Código segue os padrões do projeto
- [ ] Testes adicionados/atualizados
- [ ] Documentação atualizada
- [ ] CI passando
- [ ] Self-review feito
- [ ] Breaking changes documentados

## Screenshots (se aplicável)

[Adicione screenshots ou GIFs]

## Notas Adicionais

[Informações adicionais para os reviewers]
```

### 6.2 Issue Templates (Opcional)

Crie `.github/ISSUE_TEMPLATE/bug_report.md`:

```markdown
---
name: Bug Report
about: Reportar um bug
title: '[BUG] '
labels: bug
assignees: ''
---

## Descrição do Bug
[Descrição clara e concisa do bug]

## Como Reproduzir
1. [Passo 1]
2. [Passo 2]
3. [Ver erro]

## Comportamento Esperado
[O que deveria acontecer]

## Screenshots
[Se aplicável]

## Ambiente
- OS: [ex: Ubuntu 22.04]
- Browser: [ex: Chrome 120]
- Versão: [ex: v1.2.3]

## Contexto Adicional
[Qualquer outra informação relevante]
```

Crie `.github/ISSUE_TEMPLATE/feature_request.md`:

```markdown
---
name: Feature Request
about: Sugerir nova funcionalidade
title: '[FEATURE] '
labels: enhancement
assignees: ''
---

## Descrição da Feature
[Descrição clara do que você quer]

## Problema que Resolve
[Qual problema essa feature resolve?]

## Solução Proposta
[Como você imagina que funcione]

## Alternativas Consideradas
[Outras soluções que você pensou]

## Contexto Adicional
[Screenshots, mockups, referências, etc]
```

---

## Fase 7: Documentação Básica

### 7.1 README.md

Crie um README.md mínimo:

```markdown
# Nome do Projeto

Breve descrição do projeto.

## Tech Stack

- Node.js 20 / Bun latest
- [Outras tecnologias]

## Desenvolvimento

\`\`\`bash
# Instalar dependências
npm install  # ou: bun install

# Rodar em desenvolvimento
npm run dev

# Lint
npm run lint

# Testes
npm test

# Build
npm run build
\`\`\`

## Estrutura do Projeto

\`\`\`
src/
├── components/  # Componentes
├── services/    # Lógica de negócio
├── utils/       # Utilitários
└── types/       # TypeScript types
\`\`\`

## CI/CD

Este projeto usa workflows do [menura-actions](https://github.com/iSmart-System/menura-actions):

- **CI:** Executa em PRs (lint, test, build)
- **Release:** Executa em tags (gera artefatos)
- **Preview Deploy:** Manual via aprovação (opcional)

## Contribuindo

1. Crie branch a partir de `sandbox`
2. Faça suas mudanças
3. Abra PR para `sandbox`
4. Aguarde review (1 aprovação)
5. Merge

## Releases

Releases são gerenciadas via tags:

- `v1.0.0-rc.1` → Release Candidate (sandbox)
- `v1.0.0` → Release de Produção (main)

Veja: [Como Criar Releases](https://github.com/iSmart-System/menura-actions/blob/main/docs/how-to/create-rc.md)

## Licença

Propriedade da Menura/iSmart-System.
\`\`\`

### 7.2 CONTRIBUTING.md (Opcional)

Crie `CONTRIBUTING.md` com guidelines de contribuição.

---

## Fase 8: Primeiro Commit

### 8.1 Commit Inicial

```bash
# Adicione todos os arquivos
git add .

# Commit inicial
git commit -m "chore: setup inicial do projeto

- Adiciona workflows CI/CD (menura-actions)
- Configura proteção de branches
- Adiciona CODEOWNERS
- Adiciona templates de PR/Issue
- Adiciona documentação básica

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>"

# Push para sandbox
git push origin sandbox
```

### 8.2 Verificar CI

1. Vá para **Actions** no GitHub
2. Verifique se o workflow "CI" executou
3. Confirme que todos os jobs passaram ✅

---

## Fase 9: Primeira Release

### 9.1 Merge Inicial para Main

```bash
# Criar PR de sandbox → main
gh pr create \
  --base main \
  --head sandbox \
  --title "chore: setup inicial" \
  --body "Setup inicial do repositório com workflows menura-actions"

# Aguardar aprovações (2) e fazer merge
```

### 9.2 Criar Primeira Tag

```bash
# Após merge em main
git checkout main
git pull

# Criar tag inicial
git tag -a v0.1.0 -m "Release inicial"
git push origin v0.1.0
```

### 9.3 Verificar Release

1. Actions → Workflow "Release"
2. Confirme que release foi criada ✅
3. Releases → Verifique artefato `.zip` ✅

---

## Checklist Final

Use este checklist para garantir que nada foi esquecido:

### Estrutura
- [ ] Branch `sandbox` criada e configurada como default
- [ ] Branch `main` criada
- [ ] `.github/workflows/` com workflows
- [ ] `.github/CODEOWNERS` configurado
- [ ] `.github/PULL_REQUEST_TEMPLATE.md` criado

### Workflows
- [ ] CI configurado (Node.js ou Bun)
- [ ] Release configurado
- [ ] Workflows testados e passando
- [ ] Artifact path correto nos workflows
- [ ] Preview deploy configurado (opcional)

### Proteção
- [ ] Branch `sandbox` protegida (1 review)
- [ ] Branch `main` protegida (2 reviews)
- [ ] Status checks configurados
- [ ] Code Owners habilitado

### Secrets
- [ ] `GITHUB_TOKEN` funcionando (automático)
- [ ] `PREVIEW_DEPLOY_TOKEN` configurado (se preview)
- [ ] Environment `sandbox-preview` criado (se preview)

### Documentação
- [ ] README.md atualizado
- [ ] Scripts no package.json documentados
- [ ] CONTRIBUTING.md criado (opcional)
- [ ] Issue templates criados (opcional)

### Validação
- [ ] Primeiro commit feito
- [ ] CI executou com sucesso
- [ ] PR de sandbox → main funcionou
- [ ] Primeira release criada
- [ ] Artefato gerado corretamente

---

## Próximos Passos

Agora que seu repositório está configurado, você pode:

1. **Desenvolver features:** Crie branches `feat/` a partir de `sandbox`
2. **Criar RCs:** Use o workflow "Release Management" para criar tags RC
3. **Preview Deploy:** Teste PRs em sandbox antes do merge (se configurado)
4. **Releases:** Qualifique RCs para produção

### Guias Úteis

- [Criar Release Candidate](../how-to/create-rc.md)
- [Qualificar RC](../how-to/qualify-rc.md)
- [Preview Deploy em PRs](../how-to/preview-deploy.md)
- [Configurar Secrets](../how-to/configure-secrets.md)

---

## Troubleshooting

### CI não executou

**Causa:** Workflow pode ter erro de syntax
**Solução:** Valide com `yamllint .github/workflows/ci.yml`

### Status check não aparece na proteção

**Causa:** CI precisa rodar pelo menos uma vez
**Solução:** Faça um commit, aguarde CI, depois configure status check

### Preview deploy não funciona

**Causa:** Secret ou environment não configurado
**Solução:** Siga [Preview Deploy Guide](../how-to/preview-deploy.md) passo-a-passo

### Artefato não foi gerado

**Causa:** `artifact-path` incorreto
**Solução:** Verifique onde seu build gera arquivos (dist, build, out, .next)

---

## Conclusão

Parabéns! 🎉 Seu repositório Codebase está configurado com **todas as melhores práticas**:

✅ CI/CD automático
✅ Sistema de releases robusto
✅ Proteção de branches
✅ Code review obrigatório
✅ Preview deploy (opcional)
✅ Documentação completa

Seu repositório está pronto para **crescer com qualidade e governança**!

---

*Para mais informações, veja a [documentação completa](../README.md) do menura-actions.*
