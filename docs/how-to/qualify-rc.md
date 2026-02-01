# Como Qualificar Release Candidate

## Objetivo

Qualificar uma Release Candidate validada como release de produção, criando PR e tag final.

> ⚠️ **Importante:** Este processo cria a **release com artefato** no GitHub. Deploy/provisionamento é responsabilidade dos repositórios Architecture Live (Terragrunt).

## Pré-requisitos

- Release Candidate criada (ex: `v1.2.0-rc.1`)
- RC validada no ambiente sandbox
- Aprovações de stakeholders
- Artefato da RC testado e aprovado

## Passos

### 1. Identificar RC a qualificar

```bash
# Ver RCs disponíveis
git fetch --tags
git tag -l "*-rc*" --sort=-v:refname | head -5
```

Saída esperada:
```
v1.2.0-rc.3
v1.2.0-rc.2
v1.2.0-rc.1
```

### 2. Acessar Actions

1. Vá ao repositório no GitHub
2. Clique em **Actions**
3. Selecione **Release Management**

### 3. Executar workflow

1. Clique **Run workflow**
2. Configure:
   - **Action:** `qualify-rc`
   - **RC Tag:** `v1.2.0-rc.3` (a RC validada)
   - **Auto-merge:** `false` (recomendado)

### 4. Aguardar execução

O workflow `codebase-qualify-rc.yml` irá:

```
✅ Validar tag RC existe
✅ Verificar que tag de produção não existe
✅ Criar PR: sandbox → main
✅ Criar tag de produção (v1.2.0)
```

> **Nota:** A GitHub Release com artefato será criada APÓS o merge do PR, quando o workflow workflows de release (`codebase-release-node.yml` ou `codebase-release-bun.yml`) for disparado pela tag.

### 5. Aprovar e Mergear PR

1. Vá em **Pull Requests**
2. Encontre o PR criado: "🚀 Release: v1.2.0"
3. Revise as mudanças
4. Obtenha aprovações necessárias (mínimo 2 para `main`)
5. Faça merge

### 6. Acompanhar criação da Release

Após o merge, o workflow workflows de release (`codebase-release-node.yml` ou `codebase-release-bun.yml`) é automaticamente disparado:

```
✅ Build do projeto
✅ Empacotamento em .zip
✅ Criação da GitHub Release
✅ Upload do artefato
```

## Verificação

### Verificar tag de produção

```bash
git fetch --tags
git tag -l "v*" --sort=-v:refname | grep -v rc | head -5
```

### Verificar no GitHub

1. **Releases** → Release de produção listada com artefato .zip
2. **Actions** → Workflow "Release" executado com sucesso
3. **Tags** → Tag `v1.2.0` criada

### Verificar artefato

1. Vá em **Releases**
2. Encontre a release `v1.2.0`
3. Verifique que o arquivo `.zip` está disponível para download
4. Baixe e teste o artefato se necessário

## Próximos Passos

Após a release estar disponível:

1. ✅ Artefato `.zip` pronto para consumo
2. ✅ Release documentada no GitHub
3. ➡️ **Deploy/provisionamento:** Executado via Architecture Live (Terragrunt)

> Deploy para infraestrutura de produção é gerenciado pelos repositórios de Architecture Live, **não** pelo menura-actions.

## Com Auto-Merge

Se habilitar `auto-merge: true`:

- O PR será mergeado automaticamente após obter as aprovações mínimas
- Útil para projetos com processo bem automatizado
- Requer branch protection configurado corretamente

⚠️ **Cuidado:** Use auto-merge apenas se tiver confiança total no processo e proteções adequadas.

## Troubleshooting

| Problema | Solução |
|----------|---------|
| "Tag RC não encontrada" | Verifique se a tag existe: `git tag -l "*rc*"` |
| "Tag de produção já existe" | A versão já foi qualificada. Use outra versão RC ou patch |
| "PR já existe" | O workflow atualiza o PR existente automaticamente |
| Merge falhou | Verifique conflitos ou aprovações pendentes |
| Release não foi criada | Verifique workflow workflows de release (`codebase-release-node.yml` ou `codebase-release-bun.yml`) em Actions |

---

*Veja também: [Como Criar RC](create-rc.md) | [Reference: Workflows](../reference/workflows.md)*
