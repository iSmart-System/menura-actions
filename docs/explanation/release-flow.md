# Por Que Usamos Este Fluxo de Release

Este documento explica as razões por trás das decisões do fluxo de release adotado no menura-actions.

---

## O Problema

Antes de padronizar, cada projeto tinha seu próprio processo:

- Alguns usavam GitFlow completo
- Outros usavam trunk-based
- Releases eram manuais e inconsistentes
- Tags não seguiam padrão
- Deploys eram arriscados

**Consequências:**
- Dificuldade de onboarding
- Erros em produção
- Falta de rastreabilidade
- Processos diferentes por projeto

---

## Nossa Solução

Adotamos um **modelo híbrido** que combina:

- **Duas branches principais** (simplicidade)
- **Release Candidates** (segurança)
- **Automação de tags** (consistência)

```
Feature → sandbox → RC → main → Production
```

---

## Por Que Duas Branches?

### Alternativas Consideradas

| Modelo | Branches | Complexidade |
|--------|----------|--------------|
| Trunk-based | 1 (main) | Baixa |
| **Nosso modelo** | 2 (sandbox, main) | Média |
| GitFlow | 5+ (main, develop, feature, release, hotfix) | Alta |

### Razões da Escolha

1. **Sandbox como staging**
   - Ambiente de homologação real
   - Validação antes de produção
   - Buffer de segurança

2. **Main = Produção**
   - O que está em main está em produção
   - Simplicidade conceitual
   - Fácil de entender

3. **Sem branch develop**
   - GitFlow adiciona complexidade desnecessária
   - Sandbox serve o mesmo propósito
   - Menos branches = menos confusão

---

## Por Que Release Candidates?

### O Conceito

RC (Release Candidate) é uma versão **potencialmente pronta** para produção, mas que ainda precisa de validação final.

```
v1.2.0-rc.1  →  "Candidato a ser v1.2.0"
v1.2.0-rc.2  →  "Segundo candidato (após correções)"
v1.2.0       →  "Versão aprovada"
```

### Benefícios

1. **Ponto de checkpoint**
   - Marca momento específico para validação
   - Identificável e reproduzível
   - Histórico preservado

2. **Iteração segura**
   - Múltiplas RCs se necessário
   - Correções sem afetar versão final
   - Rollback para RC anterior possível

3. **Comunicação clara**
   - Stakeholders sabem o que validar
   - QA tem versão específica
   - Todos alinhados

### Sem RCs

```
❌ Fluxo sem RC:
sandbox (vários commits) → main → ??? qual versão é essa?

✅ Fluxo com RC:
sandbox → v1.2.0-rc.1 → validação → v1.2.0 → main
```

---

## Por Que Automação de Tags?

### Problemas com Tags Manuais

- Erros de nomenclatura (`v1.0` vs `v1.0.0`)
- Esquecimento de criar tag
- Versionamento inconsistente
- Falta de metadata (mensagens)

### Nossa Solução

Automação completa via workflows:

```yaml
# Desenvolvedor só precisa:
1. Escolher tipo de versão (patch/minor/major)
2. Executar workflow
3. Tag é criada automaticamente
```

### Benefícios

1. **Consistência garantida**
   - Sempre `vX.Y.Z` ou `vX.Y.Z-rc.N`
   - Mensagens padronizadas
   - Metadata incluída

2. **Cálculo automático**
   - Próxima versão calculada
   - RC incrementado automaticamente
   - Sem erro humano

3. **Integração**
   - GitHub Release criada
   - PR automatizado
   - Histórico completo

---

## Por Que Não Trunk-Based Puro?

Trunk-based development é excelente, mas:

### Requisitos para Trunk-Based

- Feature flags robustos
- Testes automatizados extensivos
- Deploy contínuo
- Monitoramento avançado
- Rollback instantâneo

### Nossa Realidade

Nem todos os projetos têm:
- ❌ Feature flags implementados
- ⚠️ Cobertura de testes variável
- ❌ Deploy contínuo em todos
- ⚠️ Monitoramento heterogêneo

### Solução Pragmática

Branch `sandbox` como **buffer de segurança**:
- Permite validação humana
- Não exige feature flags
- Funciona com cobertura de testes variável

---

## Por Que Não GitFlow?

GitFlow é completo, mas:

### Problemas do GitFlow

1. **Muitas branches**
   - main, develop, feature/*, release/*, hotfix/*
   - Difícil de manter sincronizado
   - Confuso para novos membros

2. **Merge hell**
   - Branches de longa duração
   - Conflitos frequentes
   - PRs enormes

3. **Overhead**
   - Processo burocrático
   - Lento para entregar valor
   - Não adequado para CI/CD moderno

### O Que Aproveitamos

- ✅ Conceito de release branches (via sandbox)
- ✅ Tags versionadas
- ❌ Múltiplas branches paralelas
- ❌ Branch develop separada

---

## Fluxo vs. Alternativas

### Comparação

| Aspecto | Nosso Fluxo | Trunk-Based | GitFlow |
|---------|-------------|-------------|---------|
| Branches | 2 | 1 | 5+ |
| Buffer antes prod | ✅ sandbox | ❌ feature flags | ✅ release branch |
| Complexidade | Média | Baixa | Alta |
| Velocidade | Boa | Excelente | Lenta |
| Segurança | Alta | Média* | Alta |
| Requisitos | Poucos | Muitos | Poucos |

*Com feature flags e monitoramento adequados

### Nossa Posição

Estamos no **sweet spot**:
- Não tão simples que seja arriscado
- Não tão complexo que seja burocrático
- Adapta-se à realidade dos times

---

## Evolução do Fluxo

### Migração Futura

Conforme a maturidade aumenta, podemos migrar para trunk-based:

```
Hoje:
Feature → sandbox → RC → main

Futuro (quando pronto):
Feature → main (com feature flags)
```

### Pré-requisitos para Migração

- [ ] Feature flags em todos os projetos
- [ ] Cobertura de testes > 80%
- [ ] Deploy em < 10 minutos
- [ ] Rollback automatizado
- [ ] Monitoramento com alertas

---

## Conclusão

Nosso fluxo é uma **decisão pragmática** baseada em:

1. **Realidade atual** dos times e projetos
2. **Necessidade de governança** na organização
3. **Equilíbrio** entre segurança e velocidade
4. **Caminho evolutivo** para práticas mais avançadas

Não é o fluxo "perfeito", mas é o **fluxo certo para agora**.
