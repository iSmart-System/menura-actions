# Decisões de Design (ADRs)

Este documento registra as decisões arquiteturais (Architecture Decision Records) tomadas no menura-actions.

---

## ADR-001: Workflows Reutilizáveis

**Data:** 2024-01

**Status:** Aceita

### Contexto

Precisávamos de uma forma de padronizar pipelines em múltiplos projetos sem duplicar código.

### Decisão

Usar `workflow_call` do GitHub Actions para criar workflows reutilizáveis.

### Alternativas Consideradas

| Alternativa | Prós | Contras |
|-------------|------|---------|
| **Composite Actions** | Mais granular | Não orquestra jobs |
| **Template repos** | Simples | Duplicação, não sincroniza |
| **workflow_call** ✅ | Centralizado, flexível | Menos granular |

### Consequências

- ✅ Um lugar para manter workflows
- ✅ Projetos herdam atualizações
- ✅ Customização via inputs
- ⚠️ Dependência do repo central

---

## ADR-002: Duas Branches Principais

**Data:** 2024-01

**Status:** Aceita

### Contexto

Precisávamos definir quantas e quais branches seriam padrão.

### Decisão

Usar duas branches: `sandbox` (staging) e `main` (production).

### Alternativas Consideradas

| Alternativa | Prós | Contras |
|-------------|------|---------|
| **Trunk-based (1)** | Simples | Requer feature flags |
| **Duas branches** ✅ | Buffer seguro | Mais processo |
| **GitFlow (5+)** | Estruturado | Complexo demais |

### Consequências

- ✅ Ambiente de validação antes de prod
- ✅ Simples de entender
- ⚠️ Não é trunk-based puro

---

## ADR-003: Release Candidates com Tags

**Data:** 2024-01

**Status:** Aceita

### Contexto

Precisávamos de um mecanismo para validar versões antes de ir para produção.

### Decisão

Usar tags `vX.Y.Z-rc.N` como Release Candidates.

### Alternativas Consideradas

| Alternativa | Prós | Contras |
|-------------|------|---------|
| **Branches release/** | GitFlow padrão | Mais branches |
| **Tags RC** ✅ | Imutável, rastreável | Nomenclatura específica |
| **Sem RC** | Mais simples | Menos controle |

### Consequências

- ✅ Ponto de checkpoint claro
- ✅ Múltiplas iterações possíveis
- ✅ Histórico preservado
- ⚠️ Mais tags no repositório

---

## ADR-004: Automação de Versionamento

**Data:** 2024-01

**Status:** Aceita

### Contexto

Tags manuais causavam inconsistências e erros.

### Decisão

Automatizar criação de tags via workflows.

### Alternativas Consideradas

| Alternativa | Prós | Contras |
|-------------|------|---------|
| **Manual** | Flexível | Erros, inconsistência |
| **Conventional commits** | Automático | Requer disciplina commits |
| **Workflow manual** ✅ | Controlado, consistente | Menos automático |

### Consequências

- ✅ Nomenclatura sempre correta
- ✅ Versão calculada automaticamente
- ✅ Metadata padronizada
- ⚠️ Requer execução manual do workflow

---

## ADR-005: Inputs com Defaults

**Data:** 2024-01

**Status:** Aceita

### Contexto

Precisávamos garantir retrocompatibilidade ao adicionar novos inputs.

### Decisão

Todos os novos inputs devem ter `required: false` e `default` definido.

### Regra

```yaml
# ✅ Correto
inputs:
  node-version:
    required: false
    default: '20'

# ❌ Incorreto
inputs:
  node-version:
    required: true
```

### Consequências

- ✅ Projetos existentes não quebram
- ✅ Novos projetos usam defaults sensatos
- ⚠️ Todos os inputs são opcionais

---

## ADR-006: Português nos Steps

**Data:** 2024-01

**Status:** Aceita

### Contexto

Precisávamos definir idioma para nomes de steps e mensagens.

### Decisão

Usar português nos nomes de steps e mensagens de log.

### Alternativas Consideradas

| Alternativa | Prós | Contras |
|-------------|------|---------|
| **Inglês** | Padrão internacional | Time fala português |
| **Português** ✅ | Natural para o time | Menos universal |
| **Misto** | Flexível | Inconsistente |

### Consequências

- ✅ Mais acessível para o time
- ✅ Mensagens de erro claras
- ⚠️ Menos universal

---

## ADR-007: Validação de Branch em Deploys

**Data:** 2024-01

**Status:** Aceita

### Contexto

Precisávamos garantir que deploys só aconteçam das branches corretas.

### Decisão

Validar branch de origem antes de executar deploy.

### Implementação

```yaml
- name: Verificar branch de origem
  run: |
    if [[ "${{ github.ref_name }}" != "sandbox" ]]; then
      echo "::error::Deploy só permitido de sandbox"
      exit 1
    fi
```

### Consequências

- ✅ Impossível fazer deploy da branch errada
- ✅ Segurança adicional
- ⚠️ Menos flexível para casos especiais

---

## ADR-008: PR Automático para Promoção

**Data:** 2024-01

**Status:** Aceita

### Contexto

Ao promover RC para produção, precisamos garantir revisão.

### Decisão

Criar PR automaticamente de `sandbox` para `main`.

### Alternativas Consideradas

| Alternativa | Prós | Contras |
|-------------|------|---------|
| **Merge direto** | Rápido | Sem revisão |
| **PR manual** | Controlado | Mais trabalho |
| **PR automático** ✅ | Controlado + automatizado | Depende de aprovações |

### Consequências

- ✅ Histórico de aprovações
- ✅ Review obrigatório
- ✅ Automação do processo
- ⚠️ Depende de branch protection

---

## ADR-009: GitHub Releases para RCs

**Data:** 2024-02

**Status:** Aceita

### Contexto

Precisávamos de visibilidade para Release Candidates.

### Decisão

Criar GitHub Release como pre-release para cada RC.

### Implementação

```yaml
- name: Criar Release
  uses: actions/github-script@v7
  with:
    script: |
      github.rest.repos.createRelease({
        prerelease: true,  // Marca como pre-release
        ...
      });
```

### Consequências

- ✅ Visibilidade na UI do GitHub
- ✅ Notificações para watchers
- ✅ Histórico centralizado
- ⚠️ Muitas releases (incluindo RCs)

---

## ADR-010: secrets: inherit

**Data:** 2024-02

**Status:** Aceita

### Contexto

Precisávamos passar secrets para workflows reutilizáveis.

### Decisão

Usar `secrets: inherit` como padrão.

### Alternativas Consideradas

| Alternativa | Prós | Contras |
|-------------|------|---------|
| **Explícito** | Mais seguro | Verboso |
| **inherit** ✅ | Simples | Passa todos secrets |

### Consequências

- ✅ Simples de usar
- ✅ Novos secrets funcionam automaticamente
- ⚠️ Todos os secrets são passados

---

## Template para Novas ADRs

```markdown
## ADR-XXX: Título

**Data:** YYYY-MM

**Status:** Proposta | Aceita | Depreciada | Substituída por ADR-XXX

### Contexto

[Qual problema estávamos resolvendo?]

### Decisão

[O que decidimos fazer?]

### Alternativas Consideradas

| Alternativa | Prós | Contras |
|-------------|------|---------|
| Opção A | ... | ... |
| **Opção B** ✅ | ... | ... |

### Consequências

- ✅ Benefício
- ⚠️ Trade-off
```
