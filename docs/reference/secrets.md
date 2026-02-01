# Reference: Secrets

Referência completa de secrets utilizados pelos workflows do menura-actions.

---

## Resumo

**Na maioria dos casos, NENHUM secret adicional é necessário!**

O `GITHUB_TOKEN` automático fornecido pelo GitHub Actions é suficiente para:
- ✅ Executar CI (lint, test, build)
- ✅ Criar releases no GitHub
- ✅ Upload de artefatos
- ✅ Criar tags

---

## Secrets Disponíveis

### GITHUB_TOKEN (Automático)

**Tipo:** Automático (fornecido pelo GitHub Actions)  
**Configuração necessária:** Nenhuma  
**Disponível via:** `secrets.GITHUB_TOKEN` ou `${{ github.token }}`

#### Permissões

O `GITHUB_TOKEN` tem permissões para:
- Ler código do repositório
- Criar/atualizar releases
- Upload de artefatos
- Criar tags
- Comentar em issues/PRs

#### Limitações

O `GITHUB_TOKEN` **NÃO pode:**
- Disparar workflows quando cria PRs
- Acessar outros repositórios
- Realizar operações administrativas avançadas

#### Quando é suficiente

Para a **maioria dos projetos**, o `GITHUB_TOKEN` é suficiente:

```yaml
jobs:
  ci:
    uses: iSmart-System/menura-actions/.github/workflows/codebase-ci-node.yml@main
    secrets: inherit  # GITHUB_TOKEN passado automaticamente
```

---

### GH_TOKEN (Opcional)

**Tipo:** Personal Access Token (PAT)  
**Configuração necessária:** Manual (apenas se necessário)  
**Disponível via:** `secrets.GH_TOKEN`

#### Quando usar

Configure `GH_TOKEN` APENAS se precisar:

1. **PRs que disparam workflows**
   - O `codebase-promote.yml` cria um PR de sandbox → main
   - Se você quer que o CI rode automaticamente nesse PR, precisa de `GH_TOKEN`

2. **Acesso a múltiplos repositórios**
   - Workflows que precisam acessar outros repos da organização

3. **Operações administrativas**
   - Casos específicos que requerem permissões elevadas

#### Como criar

1. GitHub Settings (perfil) → Developer settings
2. Personal access tokens → Tokens (classic)
3. Generate new token (classic)
4. Selecione scopes:
   - ✅ `repo` (acesso a repositórios)
   - ✅ `workflow` (atualizar workflows)
5. Copie o token
6. Adicione como secret `GH_TOKEN` no repositório

---

### PREVIEW_DEPLOY_TOKEN (Opcional)

**Tipo:** Personal Access Token (PAT)
**Configuração necessária:** Manual (apenas para preview deploy)
**Disponível via:** `secrets.PREVIEW_DEPLOY_TOKEN`

#### Quando usar

Configure `PREVIEW_DEPLOY_TOKEN` APENAS se você quer habilitar **preview deploy manual de PRs**:

- Permite que desenvolvedores façam deploy de preview de um PR no ambiente sandbox antes do merge
- Dispara deploy automático no repositório `menura-cloud-foundation`
- Requer aprovação manual via GitHub Environment

#### Como criar

1. GitHub Settings (perfil) → Developer settings
2. Personal access tokens → Fine-grained tokens
3. Generate new token
4. Configure:
   - **Token name:** `preview-deploy-token`
   - **Expiration:** 1 year (ou conforme política)
   - **Repository access:** Only select repositories
   - Selecione: `iSmart-System/menura-cloud-foundation`
   - **Permissions:**
     - Contents: Read and write
     - Metadata: Read-only
5. Copie o token
6. Adicione como secret `PREVIEW_DEPLOY_TOKEN` no repositório

#### Uso

```yaml
jobs:
  preview-deploy:
    uses: iSmart-System/menura-actions/.github/workflows/codebase-preview-deploy.yml@main
    secrets:
      dispatch-token: ${{ secrets.PREVIEW_DEPLOY_TOKEN }}
```

---

## Comparação

| Aspecto | GITHUB_TOKEN | GH_TOKEN | PREVIEW_DEPLOY_TOKEN |
|---------|--------------|----------|----------------------|
| **Configuração** | Automático | Manual | Manual |
| **Criar releases** | ✅ Sim | ✅ Sim | ❌ Não |
| **Upload artefatos** | ✅ Sim | ✅ Sim | ❌ Não |
| **Criar tags** | ✅ Sim | ✅ Sim | ❌ Não |
| **PRs disparam workflows** | ❌ Não | ✅ Sim | ❌ Não |
| **Acesso outros repos** | ❌ Não | ✅ Sim | ✅ Sim (foundation) |
| **Preview deploy** | ❌ Não | ❌ Não | ✅ Sim |
| **Recomendado para** | Maioria dos casos | Casos específicos | Preview deploy |

---

## Uso nos Workflows

Todos os workflows do menura-actions usam `secrets: inherit`:

```yaml
jobs:
  ci:
    uses: iSmart-System/menura-actions/.github/workflows/codebase-ci-node.yml@main
    secrets: inherit  # ← Passa GITHUB_TOKEN e GH_TOKEN (se configurado)
```

Dentro dos workflows, priorizamos `GH_TOKEN` se disponível, com fallback para `GITHUB_TOKEN`:

```yaml
env:
  TOKEN: ${{ secrets.GH_TOKEN || github.token }}
```

---

## Secrets de Deploy/Infraestrutura

⚠️ **Importante:** O menura-actions **NÃO gerencia deploy ou provisionamento**.

Secrets como:
- AWS credentials (AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY)
- Vercel tokens (VERCEL_TOKEN)
- Database URLs (DATABASE_URL)
- API keys de serviços externos

Devem ser configurados nos **repositórios Architecture Live** (Terragrunt), **não** nos repositórios Codebase.

---

## Boas Práticas

1. **Use GITHUB_TOKEN sempre que possível**
   - É automático, seguro e suficiente para a maioria dos casos

2. **Configure GH_TOKEN apenas quando necessário**
   - Evita complexity desnecessária
   - Menos pontos de falha

3. **Rotacione PATs regularmente**
   - Se usar `GH_TOKEN`, rotacione periodicamente

4. **Limite scopes ao mínimo**
   - PATs devem ter apenas as permissões necessárias

5. **Nunca** commite secrets no código
   - Use sempre GitHub Secrets

---

*Veja também: [Como Configurar Secrets](../how-to/configure-secrets.md)*
