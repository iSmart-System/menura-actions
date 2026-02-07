# Como Configurar Preview Deploy em PRs

## Objetivo

Permitir que desenvolvedores façam deploy manual de preview de um PR no ambiente sandbox antes do merge, disparando automaticamente o deploy no repositório de infraestrutura.

## Visão Geral

```mermaid
sequenceDiagram
    autonumber
    participant Dev as Desenvolvedor
    participant PR as Pull Request
    participant CI as CI Pipeline
    participant Preview as Preview Deploy Job
    participant Issue as GitHub Issue
    participant Foundation as menura-cloud-foundation
    participant Sandbox as Sandbox Environment

    Dev->>PR: Abre PR
    PR->>CI: Dispara CI Pipeline
    CI->>CI: Build + Tests + Lint
    CI->>CI: Upload Artefato
    Preview->>Issue: Cria Issue de Aprovação
    Issue->>Dev: Notifica Aprovadores
    Note over Preview: Job aguarda aprovação (60min)
    Dev->>Issue: Comenta "approved"
    Issue->>Preview: Aprovação recebida
    Preview->>Issue: Fecha Issue automaticamente
    Preview->>Foundation: Repository Dispatch
    Foundation->>Sandbox: Deploy Efêmero
    Foundation->>PR: Comenta URL do Preview
    Note over Sandbox: Preview disponível
    Dev->>Sandbox: Testa mudanças
    PR->>Foundation: PR fechado
    Foundation->>Sandbox: Cleanup automático
```

## Pré-requisitos

### 1. Personal Access Token (PAT)

Crie um PAT com acesso ao repositório `menura-cloud-foundation`:

1. Acesse GitHub Settings → Developer settings → Personal access tokens → Fine-grained tokens
2. Clique em "Generate new token"
3. Configure:
   - **Token name:** `preview-deploy-token`
   - **Expiration:** 1 year (ou conforme política)
   - **Repository access:** Only select repositories
   - Selecione: `iSmart-System/menura-cloud-foundation`
   - **Permissions:**
     - Contents: Read and write
     - Metadata: Read-only
4. Copie o token gerado

### 2. Configurar Secret no Repositório

No repositório do projeto:

1. Settings → Secrets and variables → Actions
2. New repository secret
3. **Name:** `PREVIEW_DEPLOY_TOKEN`
4. **Secret:** Cole o PAT criado

### 3. Configurar Aprovadores

**Para Plano Free (Repos Privados):**

A aprovação é feita via **GitHub Issues** automaticamente (funciona no plano Free).

**A organização `iSmart-System` tem governança configurada centralmente:**

```bash
# Variables configuradas na organização (governança centralizada)
PREVIEW_DEPLOY_APPROVERS=nychollas09,YtaloCampos
PREVIEW_DEPLOY_MINIMUM_APPROVALS=2
```

**Como funciona:**

1. Os aprovadores e mínimo de aprovações são definidos **exclusivamente** nas Organization Variables
2. **TODOS** os repositórios usam automaticamente essas configurações (sem possibilidade de sobrescrever)
3. Quando preview deploy é solicitado, uma **issue é criada automaticamente**
4. Aprovadores comentam `approved` ou `denied` na issue
5. O workflow continua após **2 aprovações** (mínimo configurado)

**Governança e Segurança:**

✅ **Centralizado** - Única fonte de verdade para configurações de aprovação
✅ **Seguro** - Repositórios não podem contornar aprovação ou reduzir mínimo
✅ **Auditável** - Mudanças nas variables são rastreadas
✅ **Controlado** - Apenas admins da org podem atualizar

> **Nota:** Esta abordagem funciona em **qualquer plano do GitHub** (incluindo Free) para repositórios privados.

## Configuração do Workflow

### Node.js

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
      artifact-path: 'dist'
      upload-artifacts: true  # Necessário para preview deploy
    secrets: inherit

  preview-deploy:
    name: Preview Deploy (Manual)
    if: github.event_name == 'pull_request'
    needs: ci
    uses: iSmart-System/menura-actions/.github/workflows/codebase-preview-deploy.yml@main
    with:
      artifact-name: 'meu-app'  # Nome do seu projeto
      pr-number: ${{ github.event.pull_request.number }}
      repository-name: 'meu-app'
      branch-name: ${{ github.head_ref }}
    secrets:
      dispatch-token: ${{ secrets.PREVIEW_DEPLOY_TOKEN }}
```

### Bun

Igual ao Node.js, mas use `codebase-ci-bun.yml`:

```yaml
jobs:
  ci:
    uses: iSmart-System/menura-actions/.github/workflows/codebase-ci-bun.yml@main
    with:
      bun-version: 'latest'
      # ... resto igual
```

## Como Usar

### 1. Abrir Pull Request

```bash
git checkout -b feat/nova-funcionalidade
git add .
git commit -m "feat: adicionar nova funcionalidade"
git push origin feat/nova-funcionalidade
```

Abra o PR no GitHub.

### 2. Aguardar CI Pipeline

A pipeline de CI executará automaticamente:
- ✅ Lint
- ✅ Tests
- ✅ Build
- ✅ Upload de artefatos

### 3. Aprovar Preview Deploy (Manual)

Após o CI passar:

1. Uma **issue será criada automaticamente** solicitando aprovação
2. A issue será atribuída aos aprovadores configurados
3. Você receberá uma **notificação** da issue
4. Acesse a issue e leia os detalhes do preview deploy
5. Para **aprovar**, comente na issue:
   ```
   approved
   ```
6. Para **negar**, comente na issue:
   ```
   denied
   ```
7. O workflow continuará após aprovação ou falhará se negado
8. A issue será **fechada automaticamente** após a decisão

> **Timeout:** O workflow aguarda até 60 minutos por aprovação (configurável)

### 4. Acompanhar Deploy

O workflow:
1. ✅ Valida inputs
2. ✅ Prepara metadata do artefato efêmero
3. ✅ Dispara `repository_dispatch` para `menura-cloud-foundation`
4. ✅ Comenta no PR com detalhes

### 5. Acessar Preview Environment

Após o deploy no `menura-cloud-foundation`:
- URL do preview será comentada no PR
- Acesse e teste as mudanças
- Preview ID: `pr-{número}`

### 6. Cleanup Automático

Quando o PR for:
- **Merged** → Preview removido automaticamente
- **Closed** → Preview removido automaticamente

## Opções Avançadas

### Atualizar Configurações de Governança

⚠️ **Apenas admins da organização podem atualizar** (governança centralizada)

**Atualizar lista de aprovadores:**

```bash
# Atualizar variable da organização (requer permissão admin:org)
gh variable set PREVIEW_DEPLOY_APPROVERS \
  --org iSmart-System \
  --body "user1,user2,user3,user4" \
  --visibility all
```

**Dica:** Mantenha sincronizado com o team `root`:

```bash
# Listar membros do team root
gh api /orgs/iSmart-System/teams/root/members --jq '.[].login' | tr '\n' ',' | sed 's/,$//'

# Copiar output e atualizar a variable
```

**Atualizar mínimo de aprovações:**

```bash
# Atualizar mínimo de aprovações (requer permissão admin:org)
gh variable set PREVIEW_DEPLOY_MINIMUM_APPROVALS \
  --org iSmart-System \
  --body "2" \
  --visibility all
```

**Verificar configurações:**

```bash
gh variable list --org iSmart-System | grep PREVIEW_DEPLOY
```

> **Importante:** Repositórios individuais **NÃO podem** sobrescrever essas configurações. Isso é intencional para manter governança e segurança centralizadas.

### Customizar Timeout de Aprovação

O timeout padrão é 60 minutos. Não é configurável via inputs (limitação do GitHub Actions), mas pode ser ajustado editando o workflow diretamente se necessário.

### Customizar Mensagem da Issue

```yaml
preview-deploy:
  uses: iSmart-System/menura-actions/.github/workflows/codebase-preview-deploy.yml@main
  with:
    issue-title: '🚀 [URGENTE] Aprovação de Preview Deploy Produção'
    # ... outros inputs
```

### Customizar Repositório de Infraestrutura

```yaml
preview-deploy:
  uses: iSmart-System/menura-actions/.github/workflows/codebase-preview-deploy.yml@main
  with:
    foundation-repo: 'minha-org/meu-infra-repo'
    # ... outros inputs
```

## Troubleshooting

| Problema | Causa | Solução |
|----------|-------|---------|
| Job não aparece | `if: github.event_name == 'pull_request'` | Verifique se está em um PR |
| Issue não é criada | Falta permissão `issues: write` | Workflow já tem, verifique token |
| Aprovadores não notificados | Usernames incorretos | Verifique usernames no input `approvers` |
| Timeout após 60min | Ninguém aprovou | Reduza timeout ou aprove mais rápido |
| Erro ao criar issue | Problemas com GITHUB_TOKEN | Use `secrets: inherit` no workflow |
| Deploy não dispara | Secret não configurado | Adicione `PREVIEW_DEPLOY_TOKEN` |
| Artefato não encontrado | `upload-artifacts: false` | Altere para `true` no CI |
| Workflow continua sem aprovação | Comentário incorreto | Use exatamente `approved` (minúsculo) |

## Payload Enviado ao Foundation

O workflow envia o seguinte payload via `repository_dispatch`:

```json
{
  "event_type": "preview-deploy",
  "client_payload": {
    "source_repo": "iSmart-System/meu-app",
    "source_repo_name": "meu-app",
    "pr_number": "123",
    "branch": "feat/nova-funcionalidade",
    "commit_sha": "abc123...",
    "artifact_name": "meu-app-pr-123",
    "artifact_url": "https://github.com/.../actions/runs/...",
    "preview_id": "pr-123",
    "environment": "sandbox",
    "triggered_by": "username"
  }
}
```

## Workflow no menura-cloud-foundation

O repositório de infraestrutura deve ter um workflow que escuta o evento:

```yaml
name: Preview Deploy

on:
  repository_dispatch:
    types: [preview-deploy]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy Preview
        run: |
          echo "Deploying ${{ github.event.client_payload.artifact_name }}"
          echo "PR: ${{ github.event.client_payload.pr_number }}"
          # ... lógica de deploy
```

## Boas Práticas

1. **Limite Aprovadores:** Use apenas times/pessoas autorizados
2. **Self-Review Prevention:** Habilite para maior segurança
3. **Cleanup:** Sempre implemente cleanup automático de previews
4. **Monitoramento:** Configure alertas para previews órfãos
5. **Custos:** Monitore custos de recursos efêmeros
6. **Nomenclatura:** Use padrão consistente (`pr-{número}`)
7. **Labels:** Adicione labels Kubernetes para rastreamento
8. **TTL:** Configure timeout para previews antigos (7 dias)

## Segurança

- ✅ PAT com minimal scope (apenas foundation repo)
- ✅ Aprovação manual via issue (apenas aprovadores podem aprovar)
- ✅ Audit trail completo via issues e workflow logs
- ✅ Validação de inputs antes do dispatch
- ✅ Secrets via GitHub Secrets (nunca hardcoded)
- ✅ Timeout configurável para prevenir workflows órfãos
- ✅ Lista explícita de aprovadores (controle de acesso)

---

*Veja também:*
- [Reference: Preview Deploy Workflow](../reference/workflows.md#codebase-preview-deploy)
- [Configurar Secrets](configure-secrets.md)
