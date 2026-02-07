# Exemplos de Pipelines

Este diretório contém exemplos práticos de uso dos templates para GitHub Actions e GitLab CI/CD.

## Estrutura

```
examples/
├── github/           # Exemplos GitHub Actions
│   ├── ci-node.yml
│   ├── ci-bun.yml
│   ├── ci-with-preview-node.yml
│   ├── ci-with-preview-bun.yml
│   ├── release-node.yml
│   ├── release-bun.yml
│   ├── release-management.yml
│   └── CODEOWNERS
└── gitlab/           # Exemplos GitLab CI/CD
    ├── ci-node.yml
    ├── ci-bun.yml
    ├── ci-node-skip-tests.yml
    └── README.md
```

## GitHub Actions

Exemplos de workflows reutilizáveis para GitHub Actions.

📁 [Ver exemplos GitHub](./github/)

**Características:**
- Workflows em `.github/workflows/`
- Usa `workflow_call` para reutilização
- Requer configuração de permissões
- Preview deploy via actions de terceiros

## GitLab CI/CD

Exemplos de pipelines reutilizáveis para GitLab CI/CD.

📁 [Ver exemplos GitLab](./gitlab/)

**Características:**
- Pipelines em `.gitlab-ci.yml`
- Usa `include` e `extends` para reutilização
- Aprovação manual nativa
- Preview deploy com environments nativos

## Comparação

| Aspecto | GitHub Actions | GitLab CI/CD |
|---------|----------------|--------------|
| **Arquivo** | `.github/workflows/*.yml` | `.gitlab-ci.yml` |
| **Reutilização** | `workflow_call` | `include` + `extends` |
| **Aprovação manual** | Action terceira | Nativo (`when: manual`) |
| **Environments** | Básico | Avançado (histórico, rollback) |
| **Cache** | `actions/cache` | Nativo (`cache:`) |
| **Artifacts** | `actions/upload-artifact` | Nativo (`artifacts:`) |

## Como Usar

### GitHub Actions
1. Copie arquivos para `.github/workflows/` no seu projeto
2. Configure secrets no repositório
3. Ajuste inputs conforme necessário

### GitLab CI/CD
1. Copie arquivo para `.gitlab-ci.yml` na raiz do projeto
2. Configure variables no Group/Project
3. Ajuste variables conforme necessário

## Sobre Repositórios Codebase

Estes exemplos são para repositórios que contêm **código fonte de aplicações** (Codebase).

**Características:**
- Geram artefatos (`.zip`, Docker images, etc)
- Publicam releases (GitHub Releases, GitLab Releases)
- Executam CI/CD completo (lint, test, build, deploy)

**Não se aplica a:**
- Repositórios de infraestrutura (Terraform/Terragrunt)
- Repositórios de documentação pura
- Repositórios de configuração

> **Nota:** Repositórios de infraestrutura mantêm suas próprias pipelines localmente.
