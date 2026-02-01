# Como Configurar Secrets

## Objetivo

Configurar os secrets necessários para os workflows de CI e Release funcionarem.

## Pré-requisitos

- Acesso de admin ao repositório
- Conhecimento básico de GitHub Actions

## Secrets Necessários

### GITHUB_TOKEN (Automático)

O GitHub Actions fornece automaticamente:

| Secret | Uso | Descrição |
|--------|-----|-----------|
| `GITHUB_TOKEN` | Automático | Token gerado automaticamente para cada workflow run |

**Não é necessário configurar** - está sempre disponível via `secrets.GITHUB_TOKEN`.

Este token é suficiente para:
- ✅ Criar releases no GitHub
- ✅ Upload de artefatos
- ✅ Ler código do repositório
- ✅ Executar workflows de CI

### GH_TOKEN (Opcional)

Em alguns casos específicos, você pode precisar de um Personal Access Token:

| Secret | Uso | Descrição |
|--------|-----|-----------|
| `GH_TOKEN` | Opcional | Personal Access Token para operações avançadas |

## Quando você precisa de GH_TOKEN

O `GITHUB_TOKEN` automático tem limitações. Configure `GH_TOKEN` apenas se precisar:

1. **Criar PRs que disparam workflows** - GITHUB_TOKEN não dispara workflows em PRs que ele cria (usado pelo `codebase-promote.yml`)
2. **Acessar outros repositórios** - GITHUB_TOKEN só tem acesso ao repositório atual
3. **Operações administrativas específicas** - Algumas operações requerem mais permissões

> **Para a maioria dos projetos:** Você NÃO precisa configurar GH_TOKEN. O GITHUB_TOKEN automático é suficiente.

## Como Criar GH_TOKEN (se necessário)

### 1. Acessar configurações do GitHub

1. Vá em **GitHub Settings** (seu perfil) → **Developer settings**
2. **Personal access tokens** → **Tokens (classic)**
3. **Generate new token (classic)**

### 2. Configurar o token

1. Dê um nome descritivo (ex: "menura-actions-releases")
2. Selecione scopes mínimos necessários:
   - ✅ `repo` (acesso a repositórios)
   - ✅ `workflow` (atualizar workflows)
3. **Generate token**
4. ⚠️ **Copie o token imediatamente** - ele só é mostrado uma vez!

### 3. Adicionar como secret no repositório

1. Vá ao repositório no GitHub
2. **Settings** → **Secrets and variables** → **Actions**
3. **New repository secret**
4. Name: `GH_TOKEN`
5. Secret: Cole o token
6. **Add secret**

## Usando nos Workflows

Os workflows do menura-actions já estão configurados para usar `secrets: inherit`:

```yaml
jobs:
  ci:
    uses: iSmart-System/menura-actions/.github/workflows/codebase-ci-node.yml@main
    secrets: inherit  # ← Passa todos os secrets automaticamente
```

Isso significa que:
- `GITHUB_TOKEN` é automaticamente disponibilizado
- `GH_TOKEN` (se configurado) também é passado

## Verificação

1. Acesse **Actions** no seu repositório
2. Execute um workflow (CI, Release, etc.)
3. Verifique que não há erros de permissão ou "secret not found"

## Troubleshooting

| Problema | Solução |
|----------|---------|
| "secret not found" | Verifique se o secret foi criado com o nome correto (`GH_TOKEN`) |
| PR criado não dispara workflow | Configure `GH_TOKEN` ao invés de usar apenas `GITHUB_TOKEN` |
| Permissão negada ao criar release | Verifique se `GITHUB_TOKEN` tem permissões ou configure `GH_TOKEN` |
| Token expirado | Gere um novo token e atualize o secret |

## Boas Práticas

1. **Use GITHUB_TOKEN sempre que possível** - É automático e seguro
2. **Configure GH_TOKEN apenas quando necessário** - Evita complexity desnecessária
3. **Rotacione tokens periodicamente** - Especialmente PATs (Personal Access Tokens)
4. **Limite scopes ao mínimo necessário** - Princípio do menor privilégio
5. **Nunca** commite secrets no código - Use sempre GitHub Secrets

## Sobre Secrets de Deploy

⚠️ **Importante:** O menura-actions **NÃO gerencia secrets de deploy** (AWS, Vercel, databases, etc.).

Deploy e provisionamento são responsabilidade dos repositórios **Architecture Live** (Terragrunt). Configure esses secrets diretamente nos repositórios de infraestrutura.

---

*Veja também: [Reference: Secrets](../reference/secrets.md)*
