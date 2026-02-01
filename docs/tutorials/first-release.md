# Tutorial: Criando uma Release

> 🎓 Neste tutorial, você vai aprender o fluxo completo de release, incluindo versionamento semântico e gestão de tags.

## O que você vai aprender

- Como funciona o versionamento semântico
- Quando usar patch, minor ou major
- Como criar e gerenciar Release Candidates
- Como promover para produção

## Pré-requisitos

- Ter completado os tutoriais anteriores
- Código validado em sandbox pronto para release
- Entendimento básico de SemVer

## Passo 1: Entender Versionamento Semântico

### 1.1 Formato da Versão

```
v<MAJOR>.<MINOR>.<PATCH>[-rc.<N>]

Exemplos:
v1.0.0        → Release de produção
v1.0.0-rc.1   → Release Candidate
v2.0.0        → Major release
```

### 1.2 Quando Incrementar Cada Parte

| Tipo | Quando usar | Exemplo |
|------|-------------|---------|
| **PATCH** | Bug fixes, correções | `v1.0.0` → `v1.0.1` |
| **MINOR** | Nova feature, retrocompatível | `v1.0.1` → `v1.1.0` |
| **MAJOR** | Breaking changes | `v1.1.0` → `v2.0.0` |

### 1.3 Exemplos Práticos

```
Cenário: Corrigir bug no login
→ PATCH: v1.2.3 → v1.2.4

Cenário: Adicionar novo endpoint de API
→ MINOR: v1.2.4 → v1.3.0

Cenário: Mudar estrutura do banco de dados
→ MAJOR: v1.3.0 → v2.0.0
```

## Passo 2: Preparar para Release

### 2.1 Verificar estado do sandbox

Antes de criar uma release, confirme:

```bash
# Ver commits desde última release
git log --oneline $(git describe --tags --abbrev=0)..HEAD

# Ver diferenças
git diff $(git describe --tags --abbrev=0)..HEAD --stat
```

### 2.2 Checklist pré-release

- [ ] Todas as features foram testadas em sandbox
- [ ] Nenhum bug crítico aberto
- [ ] Documentação atualizada

## Passo 3: Criar Release Candidate

### 3.1 Acessar workflow

1. Vá em **Actions** no GitHub
2. Selecione **Release Management**
3. Clique **Run workflow**

### 3.2 Configurar parâmetros

```
Branch: sandbox (IMPORTANTE!)
Action: create-rc
Version type: [escolha baseado no Passo 1.2]
```

### 3.3 Executar

Clique **Run workflow** e observe:

```
🏷️ Calculando próxima versão...
📦 Última release: v1.2.0
🆕 Nova versão base: v1.3.0
🏷️ Tag a ser criada: v1.3.0-rc.1
✅ Tag criada com sucesso!
```

### 3.4 Verificar resultado

1. Vá em **Releases** no GitHub
2. Você verá uma nova pre-release:

```
🚧 Release Candidate: v1.3.0-rc.1
Pre-release

Esta é uma Release Candidate para a versão v1.3.0.
⚠️ Esta versão está em homologação no ambiente sandbox
```

## Passo 4: Validar Release Candidate

### 4.1 Ambiente de teste

A RC foi criada a partir do estado atual do sandbox. Faça validação final:

- [ ] Smoke tests passando
- [ ] Regressão OK
- [ ] Performance aceitável
- [ ] Stakeholders aprovaram

### 4.2 Se encontrar problemas

Se encontrar bugs na RC:

```bash
# 1. Corrigir na feature branch
git checkout -b fix/bug-encontrado
# ... fazer correção ...
git commit -m "fix: corrigir bug encontrado na RC"

# 2. Abrir PR para sandbox
# 3. Após merge, criar nova RC
# → v1.3.0-rc.2
```

### 4.3 Múltiplas RCs

É comum ter várias RCs antes da release final:

```
v1.3.0-rc.1  → Bug encontrado no QA
v1.3.0-rc.2  → Correção aplicada, novo bug encontrado
v1.3.0-rc.3  → Todas as correções, pronto para produção
v1.3.0       → Release final!
```

## Passo 5: Promover para Produção

### 5.1 Iniciar promoção

Após validação da RC:

1. Vá em **Actions**
2. Selecione **Release Management**
3. Configure:
   ```
   Action: promote-to-production
   RC Tag: v1.3.0-rc.3
   ```

### 5.2 O que acontece

O workflow executa:

```
✅ Validar RC Tag
   └── Tag v1.3.0-rc.3 encontrada e válida

✅ Criar PR para Main
   └── PR #42: 🚀 Release: v1.3.0
       sandbox → main

✅ Criar Tag de Produção
   └── Tag v1.3.0 criada

✅ Criar GitHub Release
   └── Release: 🚀 v1.3.0
```

### 5.3 Aprovar PR

Um PR foi criado automaticamente:

```
🚀 Release: v1.3.0

## Informações
- Release Candidate: v1.3.0-rc.3
- Versão de Produção: v1.3.0

## Checklist
- [ ] Validado no ambiente sandbox
- [ ] Testes de regressão executados
- [ ] Aprovações obtidas
```

Obtenha as aprovações necessárias e faça o merge.

## Passo 6: Verificar Release de Produção

### 6.1 Release automática

Após o merge do PR, o workflow de release é disparado automaticamente pela tag de produção.

### 6.2 Verificar release

1. Vá em **Releases**
2. A release final deve estar listada:

```
🚀 Release: v1.3.0
Latest release

## Release de Produção
Esta é a versão de produção v1.3.0.
```

## Passo 7: Pós-Release

### 7.1 Comunicar

Informe stakeholders sobre a nova release:

- Novas features
- Bugs corrigidos
- Breaking changes (se houver)

### 7.2 Monitorar

Após a release estar disponível:

- Monitorar logs de erro
- Verificar métricas de performance
- Baixar e testar o artefato .zip se necessário

## Recapitulando

Neste tutorial você aprendeu:

1. ✅ Versionamento semântico (patch/minor/major)
2. ✅ Criar Release Candidate
3. ✅ Gerenciar múltiplas RCs
4. ✅ Promover para produção
5. ✅ Verificar release de produção
6. ✅ Procedimentos pós-release

## Fluxo Completo Visual

```
sandbox (código validado)
    │
    ▼
[Create RC] ──► v1.3.0-rc.1
    │
    │ ◄── Bug encontrado
    ▼
[Create RC] ──► v1.3.0-rc.2
    │
    │ ◄── Tudo OK!
    ▼
[Promote] ────► PR: sandbox → main
    │           Tag: v1.3.0
    ▼
[Approve & Merge]
    │
    ▼
[GitHub Release] ──► 🎉 Artefato Disponível!
```

## Próximos Passos

- [Reference: Tags & Versioning](../reference/tags-versioning.md) - Detalhes técnicos
- [How-To: Hotfix](../how-to/hotfix.md) - Correções urgentes em produção
- [Explanation: Release Flow](../explanation/release-flow.md) - Por que esse fluxo

---

*Teve problemas? Consulte os [How-To Guides](../how-to/) ou abra uma issue.*
