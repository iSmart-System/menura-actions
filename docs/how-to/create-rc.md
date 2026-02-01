# Como Criar uma Release Candidate

## Objetivo

Criar uma tag de Release Candidate para validação antes de ir para produção.

## Pré-requisitos

- Código validado na branch `sandbox`
- Código testado no ambiente sandbox
- Validação de QA completa

## Passos

### 1. Acessar Actions

1. Vá ao repositório no GitHub
2. Clique em **Actions**
3. Selecione **Release Management**

### 2. Executar workflow

1. Clique **Run workflow**
2. Configure:
   - **Branch:** `sandbox`
   - **Action:** `create-rc`
   - **Version type:**
     - `patch` - Bug fixes (v1.0.0 → v1.0.1)
     - `minor` - Novas features (v1.0.0 → v1.1.0)
     - `major` - Breaking changes (v1.0.0 → v2.0.0)

### 3. Aguardar execução

O workflow irá:

```
✅ Validar branch (deve ser sandbox)
✅ Calcular próxima versão
✅ Criar tag (ex: v1.2.0-rc.1)
✅ Criar GitHub Release (pre-release)
```

### 4. Verificar resultado

1. Vá em **Releases**
2. A nova RC deve aparecer como pre-release
3. A tag deve estar visível em **Tags**

## Verificação

```bash
# Verificar tag criada
git fetch --tags
git tag -l "v*-rc*" | tail -5
```

## Criar RC subsequente

Se precisar criar nova RC (após correções):

1. Faça as correções em sandbox
2. Execute o workflow novamente
3. Uma nova RC será criada (rc.2, rc.3, etc.)

```
v1.2.0-rc.1  → Primeira RC
v1.2.0-rc.2  → Após correções
v1.2.0-rc.3  → RC final
```

## Troubleshooting

| Problema | Solução |
|----------|---------|
| "Branch inválida" | Execute a partir de `sandbox` |
| Tag já existe | O workflow incrementa automaticamente |
| Permissão negada | Verifique permissões de Actions |

---

*Próximo: [Como Promover para Produção](promote-to-production.md)*
