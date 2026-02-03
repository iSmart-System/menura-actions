# Documentação Menura Actions

Bem-vindo à documentação oficial do **menura-actions**, o repositório central de governança de pipelines CI/CD da organização Menura.

## Estrutura da Documentação

Esta documentação segue o framework [Diátaxis](https://diataxis.fr/), organizando o conteúdo em quatro categorias distintas baseadas nas necessidades do usuário:

```
docs/
├── tutorials/      → Aprendizado guiado passo-a-passo
├── how-to/         → Guias práticos para tarefas específicas
├── reference/      → Documentação técnica detalhada
└── explanation/    → Conceitos e arquitetura
```

## Como Usar Esta Documentação

### 🎓 Tutorials (Aprendizado)

**Para quem:** Novos usuários que querem aprender a usar os workflows.

| Tutorial | Descrição |
|----------|-----------|
| [Setup Completo de Novo Repositório](tutorials/setup-novo-repositorio.md) | Configure um novo repo com todas as melhores práticas |
| [Primeiros Passos](tutorials/getting-started.md) | Configure seu primeiro projeto com menura-actions |
| [Criando uma Release](tutorials/first-release.md) | Aprenda o fluxo completo de release |

### 📋 How-To Guides (Tarefas)

**Para quem:** Usuários que precisam realizar uma tarefa específica.

| Guia | Descrição |
|------|-----------|
| [Configurar CI](how-to/setup-ci.md) | Configure CI em um novo projeto |
| [Criar Release Candidate](how-to/create-rc.md) | Crie uma nova RC para validação |
| [Qualificar RC](how-to/qualify-rc.md) | Qualifique uma RC validada |
| [Preview Deploy em PRs](how-to/preview-deploy.md) | Configure preview deploy manual de PRs |
| [Configurar Secrets](how-to/configure-secrets.md) | Configure secrets necessários |

### 📖 Reference (Consulta)

**Para quem:** Usuários que precisam de informações técnicas detalhadas.

| Referência | Descrição |
|------------|-----------|
| [Workflows](reference/workflows.md) | Todos os workflows disponíveis |
| [Inputs & Outputs](reference/inputs-outputs.md) | Parâmetros de cada workflow |
| [Tags & Versioning](reference/tags-versioning.md) | Padrões de versionamento |
| [Secrets](reference/secrets.md) | Secrets necessários |

### 💡 Explanation (Conceitos)

**Para quem:** Usuários que querem entender o "porquê" das decisões.

| Conceito | Descrição |
|----------|-----------|
| [Arquitetura](explanation/architecture.md) | Visão geral da arquitetura |
| [Fluxo de Release](explanation/release-flow.md) | Por que usamos esse fluxo |
| [Decisões de Design](explanation/design-decisions.md) | ADRs e decisões técnicas |

## Links Rápidos

- [README Principal](../README.md) - Visão geral e início rápido
- [AGENTS.md](../AGENTS.md) - Instruções para agentes de IA
- [Exemplos](../examples/) - Workflows prontos para copiar

## Contribuindo

Para contribuir com a documentação:

1. Identifique a categoria correta (tutorial, how-to, reference, explanation)
2. Siga o template da categoria
3. Mantenha o texto conciso e prático
4. Inclua exemplos de código quando aplicável
5. Abra um PR com suas alterações

---

*Documentação estruturada seguindo o framework [Diátaxis](https://diataxis.fr/).*
