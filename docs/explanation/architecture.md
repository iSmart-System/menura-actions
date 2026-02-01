# Arquitetura do menura-actions

Este documento explica a arquitetura geral do sistema de governança de pipelines e como os componentes se relacionam.

---

## Visão Geral

O menura-actions é um **repositório central de governança** que define e padroniza os processos de CI/CD para repositórios **Codebase** da organização Menura.

```mermaid
flowchart TB
    subgraph ORG["🏢 ORGANIZAÇÃO MENURA"]
        direction TB
        MA["📦 menura-actions<br/><i>Repositório Central de Governança</i>"]

        subgraph CODEBASE["🔷 Repositórios Codebase"]
            direction LR
            C1["App Frontend"]
            C2["API Backend"]
            C3["Microservices"]
            C4["Libs Compartilhadas"]
        end

        MA -->|"codebase-*"| CODEBASE
        CODEBASE -->|"Releases"| GHR["📦 GitHub Releases"]
    end

    style MA fill:#f4a261,stroke:#264653,color:#000,stroke-width:3px
    style CODEBASE fill:#a8dadc,stroke:#264653
    style GHR fill:#2a9d8f,stroke:#264653,color:#fff
```

> **Nota:** Repositórios de infraestrutura (Terraform/Terragrunt) mantêm suas próprias pipelines localmente usando Architecture Live.

---

## Repositórios Codebase

Repositórios que contêm código fonte de aplicações.

| Característica | Descrição |
|----------------|-----------|
| **Propósito** | Desenvolver e empacotar aplicações |
| **Artefatos** | Gera arquivos `.zip` |
| **Publicação** | GitHub Releases |
| **Branches** | `sandbox` (staging) + `main` (production) |
| **Tags** | Release Candidates (`-rc.N`) + Produção |
| **Workflows** | `codebase-*` (centralizados) |

---

## Modelo Hub-and-Spoke

Utilizamos um modelo **hub-and-spoke** (centro e raios):

### Hub (Centro)

O repositório `menura-actions` é o hub que:

- Define workflows padronizados para repos Codebase
- Controla políticas de CI/CD
- Gerencia versões e releases
- Documenta padrões

### Spokes (Raios)

Cada projeto Codebase é um spoke que:

- Consome workflows do hub (`codebase-*`)
- Customiza via inputs
- Mantém código de negócio
- Segue as políticas definidas

### Benefícios

| Aspecto | Benefício |
|---------|-----------|
| **Consistência** | Todos os projetos seguem o mesmo padrão |
| **Manutenção** | Atualização centralizada propaga para todos |
| **Governança** | Políticas aplicadas uniformemente |
| **Onboarding** | Novos projetos começam com padrão pronto |

---

## Fluxo de Dados

```mermaid
sequenceDiagram
    autonumber
    participant Dev as Desenvolvedor
    participant FB as Feature Branch
    participant SB as sandbox
    participant MA as menura-actions
    participant GHR as GitHub Releases
    participant Main as main

    Note over Dev,Main: FLUXO CODEBASE

    Dev->>FB: Push código
    FB->>SB: Pull Request

    rect rgb(230, 240, 250)
        Note over SB,MA: CI Pipeline
        SB->>MA: Chama codebase-ci.yml
        MA-->>SB: Resultado
    end

    SB->>SB: Merge
    Note over SB: Testes em sandbox

    rect rgb(250, 245, 230)
        Note over SB,GHR: Create RC + Release
        Dev->>MA: Chama codebase-create-rc.yml
        MA->>GHR: Tag v1.0.0-rc.1 + Release
    end

    Note over GHR: Artefato disponível

    rect rgb(250, 235, 235)
        Note over SB,Main: Promote to Production
        Dev->>MA: Chama codebase-promote.yml
        MA-->>Main: PR + Tag v1.0.0
    end

    Main->>Main: Merge PR

    rect rgb(235, 250, 235)
        Note over Main,GHR: Release de Produção
        Main->>GHR: Tag v1.0.0 + Release
    end

    Note over GHR: Artefato pronto para consumo
```

---

## Componentes do Sistema

### Workflows

```mermaid
flowchart LR
    subgraph CI["codebase-ci.yml"]
        direction TB
        A1[Checkout] --> B1[Setup]
        B1 --> C1[Lint]
        C1 --> D1[Testes]
        D1 --> E1[Build]
    end

    subgraph RELEASE["codebase-release-{tech}.yml"]
        direction TB
        A3[Validar Tag] --> B3[Build]
        B3 --> C3[Empacotar]
        C3 --> D3[Publicar Release]
    end

    subgraph RC["codebase-create-rc.yml"]
        direction TB
        A4[Calcular Versão] --> B4[Criar Tag]
        B4 --> C4[Criar Release]
    end

    CI --> RC
    RC --> RELEASE

    style E1 fill:#2a9d8f,color:#fff
    style D3 fill:#f4a261
    style C4 fill:#e9c46a
```

**Princípios:**
- CI: Falhar rápido (fail fast), feedback claro
- Release: Automação completa, rastreabilidade
- RC: Nomenclatura padronizada SemVer

---

## Camadas de Abstração

```mermaid
flowchart TB
    subgraph PROJ["📁 REPOSITÓRIOS CODEBASE"]
        P1["Chamam workflows<br/>com inputs customizados"]
    end

    subgraph MA["📦 MENURA-ACTIONS"]
        W1["Define fluxos<br/>e políticas"]
    end

    subgraph GHA["⚙️ GITHUB ACTIONS"]
        R["Runners, secrets,<br/>environments"]
    end

    subgraph RELEASES["📦 GITHUB RELEASES"]
        A["Artefatos .zip<br/>versionados"]
    end

    PROJ --> MA
    MA --> GHA
    GHA --> RELEASES

    style PROJ fill:#a8dadc,stroke:#264653
    style MA fill:#f4a261,stroke:#264653
    style GHA fill:#457b9d,stroke:#264653,color:#fff
    style RELEASES fill:#2a9d8f,stroke:#264653,color:#fff
```

---

## Pontos de Extensão

### 1. Inputs Customizáveis

Cada projeto pode customizar via inputs:

```yaml
with:
  node-version: '18'
  artifact-path: 'build'
  skip-tests: false
```

### 2. Secrets por Environment

Diferentes credenciais para sandbox/production:

```yaml
environment: production  # Usa secrets de produção
```

### 3. Workflows Locais

Projetos podem ter workflows adicionais:

```
projeto/
├── .github/workflows/
│   ├── ci.yml           # Chama menura-actions
│   ├── release.yml      # Chama menura-actions
│   └── custom.yml       # Workflow local adicional
```

---

## Trade-offs

### Centralização vs. Flexibilidade

| Centralizado | Distribuído |
|--------------|-------------|
| Consistência | Flexibilidade total |
| Governança | Autonomia do time |
| Menos flexível | Inconsistência |
| Single point of change | Manutenção duplicada |

**Nossa escolha:** Centralização com pontos de extensão.

### Simplicidade vs. Completude

| Simples | Completo |
|---------|----------|
| Fácil de usar | Cobre todos os casos |
| Rápido onboarding | Menos workarounds |
| Pode faltar features | Complexidade |

**Nossa escolha:** Começar simples, adicionar conforme necessidade.

---

## Evolução Futura

### Implementado

- [x] Workflows Codebase completos
- [x] Release Candidates automatizados
- [x] GitHub Releases com artefatos

### Planejado

- [ ] Suporte a múltiplas linguagens (Python, Go)
- [ ] Métricas e observabilidade
- [ ] Integração com ferramentas de segurança
- [ ] Multi-stack (além de Node.js)
