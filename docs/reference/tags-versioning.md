# Reference: Tags & Versioning

Documentação do padrão de versionamento e nomenclatura de tags.

---

## Padrão SemVer

O menura-actions segue o [Semantic Versioning 2.0.0](https://semver.org/).

### Formato

```
v<MAJOR>.<MINOR>.<PATCH>[-rc.<RC_NUMBER>]
```

### Componentes

| Componente | Descrição | Quando incrementar |
|------------|-----------|-------------------|
| `v` | Prefixo obrigatório | Sempre presente |
| `MAJOR` | Versão maior | Breaking changes |
| `MINOR` | Versão menor | Novas features (retrocompatível) |
| `PATCH` | Correção | Bug fixes (retrocompatível) |
| `rc.N` | Release Candidate | Pre-release para validação |

---

## Tipos de Tags

### Tag de Produção

**Formato:** `vX.Y.Z`

**Regex:** `^v[0-9]+\.[0-9]+\.[0-9]+$`

**Exemplos:**
```
v1.0.0
v1.2.3
v2.0.0
v10.20.30
```

**Características:**
- Representa código em produção
- Criada apenas após merge em `main`
- Imutável após criação
- Gera GitHub Release oficial

---

### Tag de Release Candidate

**Formato:** `vX.Y.Z-rc.N`

**Regex:** `^v[0-9]+\.[0-9]+\.[0-9]+-rc\.[0-9]+$`

**Exemplos:**
```
v1.0.0-rc.1
v1.2.3-rc.2
v2.0.0-rc.10
```

**Características:**
- Representa código em homologação
- Criada a partir de `sandbox`
- Pode haver múltiplas RCs para mesma versão
- Gera GitHub Release como pre-release

---

## Regras de Incremento

### MAJOR (X.0.0)

Incrementar quando houver:

- Mudanças incompatíveis na API
- Alterações no schema do banco de dados
- Remoção de funcionalidades
- Mudanças que quebram integrações existentes

```
v1.9.9 → v2.0.0
```

### MINOR (x.Y.0)

Incrementar quando houver:

- Nova funcionalidade retrocompatível
- Novos endpoints de API
- Novas opções de configuração
- Depreciação de features (sem remoção)

```
v1.2.3 → v1.3.0
```

### PATCH (x.y.Z)

Incrementar quando houver:

- Correção de bugs
- Correções de segurança
- Melhorias de performance
- Correções de documentação

```
v1.2.3 → v1.2.4
```

---

## Fluxo de Versionamento

### Cenário: Nova Feature

```
Início: v1.2.0 (última produção)

1. Desenvolver feature
2. Merge para sandbox
3. Criar RC (minor): v1.3.0-rc.1
4. Bug encontrado, corrigir
5. Criar RC: v1.3.0-rc.2
6. Validação OK
7. Promover: v1.3.0

Resultado: v1.3.0 (produção)
```

### Cenário: Bug Fix

```
Início: v1.3.0 (última produção)

1. Corrigir bug
2. Merge para sandbox
3. Criar RC (patch): v1.3.1-rc.1
4. Validação OK
5. Promover: v1.3.1

Resultado: v1.3.1 (produção)
```

### Cenário: Breaking Change

```
Início: v1.3.1 (última produção)

1. Desenvolver mudança
2. Merge para sandbox
3. Criar RC (major): v2.0.0-rc.1
4. Validação OK
5. Promover: v2.0.0

Resultado: v2.0.0 (produção)
```

---

## Cálculo Automático de Versão

O workflow `create-rc-tag.yml` calcula automaticamente:

### Algoritmo

```
1. Buscar última tag de produção (sem -rc)
2. Incrementar baseado no tipo escolhido
3. Verificar se já existe RC para essa versão
4. Incrementar número do RC se necessário
```

### Exemplos

| Estado atual | Tipo | Resultado |
|--------------|------|-----------|
| Nenhuma tag | patch | v0.0.1-rc.1 |
| v1.0.0 | patch | v1.0.1-rc.1 |
| v1.0.0 | minor | v1.1.0-rc.1 |
| v1.0.0 | major | v2.0.0-rc.1 |
| v1.0.1-rc.1 existe | patch | v1.0.1-rc.2 |

---

## Validação de Tags

### Formato Válido

```bash
# Produção
v1.0.0     ✅
v10.20.30  ✅
v1.2.3     ✅

# Release Candidate
v1.0.0-rc.1   ✅
v1.2.3-rc.10  ✅
```

### Formato Inválido

```bash
1.0.0          ❌ (falta 'v')
v1.0           ❌ (falta PATCH)
v1.0.0-alpha   ❌ (sufixo inválido)
v1.0.0-rc      ❌ (falta número RC)
v1.0.0-RC.1    ❌ (RC maiúsculo)
v1.0.0.0       ❌ (4 componentes)
```

---

## Comandos Git

### Listar tags

```bash
# Todas as tags
git tag -l "v*"

# Apenas produção
git tag -l "v*" | grep -v "\-rc"

# Apenas RCs
git tag -l "*-rc*"

# Ordenado por versão
git tag -l "v*" --sort=-v:refname
```

### Criar tag manual (não recomendado)

```bash
# Tag anotada
git tag -a v1.0.0 -m "Release v1.0.0"

# Push
git push origin v1.0.0
```

### Deletar tag (com cuidado)

```bash
# Local
git tag -d v1.0.0-rc.1

# Remoto
git push origin --delete v1.0.0-rc.1
```

---

## Boas Práticas

1. **Sempre usar workflow** - Evite criar tags manualmente
2. **Não deletar tags de produção** - São imutáveis
3. **RCs podem ser deletadas** - Se necessário limpar
4. **Uma RC por vez** - Valide antes de criar outra
