# Exemplos de Workflows

Este diretório contém exemplos de workflows para repositórios **Codebase**.

## Sobre Repositórios Codebase

Repositórios que contêm código fonte de aplicações. Geram artefatos `.zip` e publicam no GitHub Releases.

**Diretório:** `codebase-project/`

| Arquivo | Descrição |
|---------|-----------|
| `ci-node.yml` | Pipeline de CI para projetos Node.js |
| `ci-bun.yml` | Pipeline de CI para projetos Bun |
| `ci-with-preview-node.yml` | CI Node.js com preview deploy opcional |
| `ci-with-preview-bun.yml` | CI Bun com preview deploy opcional |
| `release-node.yml` | Gera artefatos e publica releases (Node.js) |
| `release-bun.yml` | Gera artefatos e publica releases (Bun) |
| `release-management.yml` | Interface para criar RCs e qualificar |
| `CODEOWNERS` | Define ownership para code review |

## Como Usar

1. Copie os arquivos de workflow para `.github/workflows/` no seu projeto
2. Copie o arquivo `CODEOWNERS` para `.github/CODEOWNERS` no seu projeto
3. Escolha os workflows correspondentes à sua tech stack:
   - Node.js: `ci-node.yml` + `release-node.yml`
   - Node.js com preview: `ci-with-preview-node.yml` + `release-node.yml`
   - Bun: `ci-bun.yml` + `release-bun.yml`
   - Bun com preview: `ci-with-preview-bun.yml` + `release-bun.yml`
   - Ambos: `release-management.yml`
4. Ajuste os inputs conforme necessário (node-version/bun-version, artifact-path, etc.)
5. Customize o CODEOWNERS conforme a estrutura de times do seu projeto
6. Configure os secrets no repositório
7. Para preview deploy: Configure environment `sandbox-preview` + secret `PREVIEW_DEPLOY_TOKEN`
8. Faça commit e push

## Dicas

- Mantenha `secrets: inherit` para passar secrets automaticamente
- Use `@main` para sempre usar a versão mais recente dos workflows
- Use `@v1.0.0` para fixar em uma versão específica

## Infraestrutura (Terraform/Terragrunt)

Repositórios de infraestrutura mantêm suas próprias pipelines localmente usando Architecture Live.

O menura-actions foca em repositórios **Codebase** (aplicações que geram artefatos).
