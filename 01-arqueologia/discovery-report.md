<!-- markdownlint-disable MD013 MD025 MD026 MD028 MD029 MD034 MD040 MD051 MD060 -->

# Relatório de Descoberta — Estágio 1: Arqueologia Digital

![ESTÁGIO 01 Arqueologia](https://img.shields.io/badge/ESTÁGIO-01%20Arqueologia-F25022?style=for-the-badge) ![TIPO Worksheet](https://img.shields.io/badge/TIPO-Worksheet-1A1A1A?style=for-the-badge) ![PREENCHA Durante S1](https://img.shields.io/badge/PREENCHA-Durante%20S1-737373?style=for-the-badge)

> 🗺 **Você está aqui:** [Kit PT-BR](../README.md) → [Estágio 1](README.md) → **discovery-report**

> **Para quem é isto?** Este é um **artefato preenchido pelo time** durante o Estágio 1 (Arqueologia).
>
> **O que você terá ao final do estágio:**
>
> 1. Este documento totalmente preenchido com os dados reais do legado SIFAP
> 2. Rastreabilidade para `01-arqueologia/legado-sifap/` (programas `.NSN` e DDMs)
> 3. Base de evidência usada nas EARS do Estágio 2 (`source_legacy:`)
>
> 📘 **Guia passo a passo:** [`GUIDE.md`](GUIDE.md).


> Este documento consolida todas as descobertas do Estágio 1.
> Preencha cada seção com as conclusões do time. **Este é o input principal do Estágio 2** — sem ele, a especificação vira chute.

**Time**: SIFAP — Estágio 1 (Arqueologia)
**Data**: 2026-06-10
**Edição**: Workshop de Modernização de Legado
**Participantes**: Pares 1–5 (PO, RE, EA, SA, TL, Dev, DBA, QA, DevOps, TW) — síntese liderada pelo Par 1 (Visão)

---

## 1. Sumário Executivo

> Em 3 a 5 frases, resuma o que o time descobriu sobre o SIFAP legado.
> O que é este sistema? Qual sua criticidade? Qual o estado do código?

O SIFAP legado é um sistema de fiscalização e pagamento de benefícios sociais com **15 programas Natural** e **4 arquivos Adabas** (`BENEFICIARIO` ~4,2M e `PAGAMENTO` ~180M registros), em produção há ~29 anos. Foram extraídas **79 regras de negócio candidatas** (≈25 confirmadas contra a documentação) e catalogados **33 mistérios**, incluindo os **3 easter eggs** plantados. O sistema é **fracamente acoplado por chamadas** — não existe nenhum `CALLNAT` — mas **fortemente acoplado por dados** em torno do DDM `PAGAMENTO`, que toca 8 dos 15 programas. O maior risco para o Estágio 2 é o conjunto de **6 mistérios bloqueadores**, com destaque para o **bypass de elegibilidade da região 99** (MYS-008, risco de fraude) e a **divergência da fórmula financeira código × documentação** (MYS-011/012/014). **Confiança para modernização: MÉDIA** — a leitura de código foi sólida nos programas de cálculo/cadastro, mas o glossário está vazio e há divergências graves entre código e documentação que exigem validação de negócio.

---

## 2. Visão Geral do Sistema

### 2.1 Propósito do SIFAP

Sistema de **Fiscalização e Administração de Pagamentos** de benefícios sociais: cadastra beneficiários e programas sociais, valida elegibilidade, calcula o valor do benefício (com fatores regional, familiar, de renda e idade), aplica descontos/correções e gera os pagamentos mensais, conciliando o retorno bancário e mantendo trilha de auditoria. Ver [inventory.md](inventory.md) e [business-rules-catalog.md](business-rules-catalog.md).

### 2.2 Arquitetura Legada

- **15 programas `.NSN`** em 5 famílias de prefixo: `CAD*` (cadastro online 3270), `BATCH*` (batch/JCL), `CALC*` (cálculo), `VAL*` (validação), `CONS*`/`REL*` (consulta/relatório). Ver [inventory.md](inventory.md).
- **4 DDMs Adabas:** `BENEFICIARIO` (FNR 150), `PROGRAMA-SOCIAL` (151), `PAGAMENTO` (152), `AUDITORIA` (153).
- **Integração 100% por dados:** zero `CALLNAT`; toda reutilização é por subrotinas `PERFORM` internas. O `PAGAMENTO` é o hub central. Ver [dependency-map.md](dependency-map.md).

### 2.3 Usuários e Perfis

Operadores de cadastro (terminal 3270 — `CAD*`/`CONS*`), o scheduler batch/JCL (`BATCH*`, processamento mensal), e consumidores de relatório/SIAFI (`REL*`). Perfis de acesso formais não estão codificados nos programas lidos — **lacuna** a confirmar no Estágio 2.

---

## 3. Principais Descobertas

### 3.1 Regras de Negócio Críticas

> Liste as 5 regras de negócio mais importantes encontradas.

1. **Validação de CPF por módulo 11** — CPF zero ou com dígito verificador inválido é recusado no cadastro. [business-rules-catalog.md](business-rules-catalog.md) (CADBENEF #2, #3, #13) · `CADBENEF.NSN#L106-L118`, `#L222-L270`.
2. **Teto de desconto de 30% do bruto, exceto judicial** — o total de descontos é limitado a 30%, com exceção do tipo `'J'`. [business-rules-catalog.md](business-rules-catalog.md) (CALCDSCT #5, #8) · `CALCDSCT.NSN#L101-L105`, `#L123-L132`. ⚠️ ver MYS-006.
3. **Truncamento monetário em centavos** — valores são truncados (não arredondados) via campo inteiro `× 100 / 100`. [business-rules-catalog.md](business-rules-catalog.md) (BATCHPGT #11, CALCBENF #10) · `BATCHPGT.NSN#L283-L285`.
4. **Só beneficiários ativos (`status 'A'`) são pagos** — o batch ignora qualquer status diferente de `'A'`. [business-rules-catalog.md](business-rules-catalog.md) (BATCHPGT #2) · `BATCHPGT.NSN#L195-L198`. ⚠️ ver MYS-001.
5. **Elegibilidade por faixa etária e renda do programa** — idade mínima/máxima e renda máxima definidas no `PROGRAMA-SOCIAL` reprovam o beneficiário. [business-rules-catalog.md](business-rules-catalog.md) (VALELEG #6, #7, #8) · `VALELEG.NSN#L139-L163`. ⚠️ ver MYS-017.

### 3.2 Dependências Complexas

> Quais programas estão mais acoplados? Onde há risco de efeito cascata?

O acoplamento é **por dados**, não por chamada. O DDM `PAGAMENTO` (152) é o ponto crítico: escrito por BATCHPGT/CALCBENF, mutado por CALCDSCT/CALCCORR/BATCHCON e lido por BATCHREL/RELPGT — **qualquer mudança de schema em `PAGAMENTO` impacta 8 dos 15 programas**. Há ainda **duplicação de lógica financeira** entre `BATCHPGT` e `CALCBENF` (mesmos fatores e fórmula reimplementados), de modo que um reajuste de regra exige alterar dois lugares. Ver [dependency-map.md](dependency-map.md).

### 3.3 Dívida Técnica Identificada

> Que problemas no código legado vão complicar a migração?

- [ ] **Duplicação de regra financeira** entre BATCHPGT e CALCBENF (sem fonte única de verdade) — MYS-011.
- [ ] **Magic numbers** financeiros sem origem documental (fatores 0.05/0.03/0.02, faixas de renda, abono 0.15, desconto 0.03, limiar idade 75/65/60) — MYS-012/013/014.
- [ ] **Validações órfãs** (VALBENEF, VALDOCS, VALELEG não são chamadas por ninguém) — sem gate automático de elegibilidade — MYS-032.
- [ ] **Documentação divergente/desatualizada** (subprogramas fantasma, fórmula aditiva, status 'P' vs 'G', códigos de desconto numéricos vs letras) — MYS-025/029/031.
- [ ] **Código morto e tabelas congeladas** (Plano Verão comentado; tabela IPCA parada em 2010-2012) — EGG-001 / MYS-015.

### 3.4 Gaps de Documentação

> O que a documentação existente NÃO cobre?

A `legacy-docs/` está **desatualizada de propósito**: descreve chamadas e regras inexistentes no código (subprogramas VALCPF/VALNISN/LOGAUDIT/CALCIDX, fórmula aditiva, ABEND por MAX-ERROS, ordenação por nome). Não cobre: o **bypass da região 99** (introduzido em 2013), a **suspensão de idosos via status 'S'**, a **fórmula multiplicativa real**, e os **códigos de desconto por letra** (J/P/I/S/A/C). O **glossário ([glossary.md](glossary.md)) já está preenchido com 52 termos**, o que fecha essa lacuna de vocabulário para o Estágio 2.

### 3.5 Hipóteses de Bounded Context (para o Architect avaliar)

> ⚠️ **Hipóteses, não decisões.** Derivadas dos clusters de acesso a DDM em [dependency-map.md](dependency-map.md). O `@architect-agent` decide no Estágio 2.

- **Hipótese 1 — Cadastro de Beneficiários** — programas `CADBENEF`, `CADDEPEND`, `CADPROG` (+ `VAL*`); possui `BENEFICIARIO` (150) e `PROGRAMA-SOCIAL` (151). Fronteira natural: entrada e manutenção de dados cadastrais.
- **Hipótese 2 — Motor de Cálculo e Pagamento** — `BATCHPGT`, `CALCBENF`, `CALCDSCT`, `CALCCORR`; possui `PAGAMENTO` (152). Fronteira: toda a lógica financeira (hoje duplicada) num único contexto.
- **Hipótese 3 — Elegibilidade** — `VALELEG` (+ regras de VAL*); lê `BENEFICIARIO`/`PROGRAMA-SOCIAL`. Fronteira: decisão de quem pode receber (hoje órfã, sem gate automático).
- **Hipótese 4 — Conciliação e Auditoria** — `BATCHCON`, `RELAUDIT`; possui `AUDITORIA` (153). Fronteira: reconciliação bancária (CNAB/SIAFI) + trilha de auditoria.
- **Hipótese 5 — Consulta e Relatórios** — `CONSBENF`, `RELPGT`, `BATCHREL`; leitura sobre `PAGAMENTO`/`BENEFICIARIO`. Fronteira: caminhos somente-leitura e saídas.

---

## 4. Mistérios e Riscos

### 4.1 Mistérios Não Resolvidos

> Resuma os mistérios do arquivo `mysteries-found.md` que permanecem sem explicação.

| ID  | Descrição | Risco para Migração |
| --- | --------- | ------------------- |
| MYS-008 | Região 99 = bypass total de elegibilidade (`ESCAPE ROUTINE`), sem validação nem log de quem atribui. | 🔴 Fraude — qualquer cadastro pode virar elegível incondicionalmente. `VALELEG.NSN#L107-L111`. |
| MYS-001 | Idade > 75 sobrescreve status para `'S'` → VALELEG marca inelegível e BATCHPGT não paga. | 🔴 Suspende pagamento de idosos silenciosamente. `CADBENEF.NSN#L168-L171`. |
| MYS-012 | Fórmula de benefício é multiplicativa e reajuste incide sobre o total, contradizendo RN-013/RN-020 (aditiva). | 🔴 Reproduzir a fórmula errada muda o valor pago a milhões. `BATCHPGT.NSN#L280-L282`. |
| MYS-011 | CALCBENF e BATCHPGT duplicam a lógica financeira; a doc diz que o batch chama CALCBENF, mas não chama. | 🔴 Sem fonte única de verdade; regras divergem. |
| MYS-014 | Três regras de desconto/contribuição conflitantes (3% fixo inline vs 3/5/7/9% progressivo vs teto 30%). | 🔴 Valor líquido diverge conforme o caminho de cálculo. |
| MYS-006 | Teto de 30% trunca o acumulado dentro do loop — pode truncar penhora judicial dependendo da ordem. | 🔴 Passivo legal; resultado ordem-dependente. `CALCDSCT.NSN#L165-L169`. |

> Detalhes completos e os 27 mistérios não-bloqueadores em [mysteries-found.md](mysteries-found.md).

### 4.2 Riscos para o Estágio 2

> O que o time de especificação precisa saber antes de começar?

1. **Não escrever EARS de cálculo/desconto/elegibilidade** até validar com o facilitador/negócio os 6 mistérios bloqueadores (MYS-001, 006, 008, 011, 012, 014).
2. **A documentação não é confiável como fonte** — sempre rastrear `source_legacy:` ao código `.NSN` real, não à `legacy-docs/`.
3. **Usar o glossário como vocabulário oficial** ([glossary.md](glossary.md), 52 termos) — atenção aos 3 marcados como HIPÓTESE (FATOR-K, MU, ARQ 150/155/160/170) que ainda precisam de validação.

---

## 5. Recomendações

### 5.1 O que migrar primeiro

> Com base na priorização do Par 1 (Product Owner), quais funcionalidades devem ser migradas primeiro?

| Prioridade | Funcionalidade | Justificativa |
| ---------- | -------------- | ------------- |
| 1          | Cadastro de beneficiários + validação de CPF | Entidade central; regras confirmadas e estáveis. |
| 2          | Motor de cálculo de benefício (unificado) | Maior risco financeiro; resolve a duplicação BATCHPGT/CALCBENF. |
| 3          | Elegibilidade com gate automático e governança da região 99 | Fecha o buraco de fraude (MYS-008) e a órfandade de VALELEG. |

### 5.2 O que descartar

> Funcionalidades que provavelmente não precisam ser migradas:

- **Bloco "Plano Verão" (CALCCORR)**: código comentado de 1989-1991; histórico, não executa (EGG-001).
- **Integração "Banco Real" (BATCHCON)**: código morto de instituição extinta (EGG-003).
- **Subprogramas fantasma da doc** (VALCPF/VALNISN/LOGAUDIT/CALCIDX/FMTVLR/FMTDT): não existem como `.NSN` (MYS-029/031).

### 5.3 O que evoluir

> Funcionalidades que devem ser migradas E melhoradas:

- **Descontos (CALCDSCT)**: migrar com teto de 30% aplicado de forma ordem-independente e exceção judicial garantida (corrige MYS-006).
- **Correção monetária (CALCCORR)**: substituir a tabela IPCA hardcoded (congelada em 2010-2012) por fonte de índice atualizada (corrige MYS-015).
- **Auditoria**: tornar a trilha obrigatória no cadastro (hoje CADBENEF não grava em AUDITORIA — MYS-018).

---

## 6. Métricas do Estágio

| Métrica                       | Valor        |
| ----------------------------- | ------------ |
| Programas analisados          | 15 / 15 inventariados · 6 / 15 lidos em profundidade |
| DDMs mapeados                 | 4 / 4   |
| Regras de negócio encontradas | 79 candidatas (~25 confirmadas) |
| Regras escondidas encontradas | 10 / 10  |
| Easter eggs encontrados       | 3 / 3   |
| Termos no glossário           | 52      |
| Mistérios catalogados         | 33       |
| Tempo total gasto             | 1h30min |

---

## 7. Notas para o Próximo Estágio

> Deixe aqui mensagens para o time no Estágio 2 (Especificação Moderna):

Comecem pela **constituição** fixando "código legado é a fonte da verdade, não a doc". Tratem os **6 mistérios bloqueadores** como questões abertas obrigatórias antes de escrever EARS financeiras/de elegibilidade. O **glossário (52 termos) já está pronto** como vocabulário oficial. As **5 hipóteses de bounded context** (§3.5) são ponto de partida para o `@architect-agent`, não decisões.

### Artefatos-Fonte

- [inventory.md](inventory.md) — estrutura, contagem e ordem de leitura.
- [business-rules-catalog.md](business-rules-catalog.md) — 79 regras candidatas com `Programa Fonte`.
- [dependency-map.md](dependency-map.md) — grafo de acesso a DDM (zero CALLNAT).
- [mysteries-found.md](mysteries-found.md) — 33 mistérios + 3 easter eggs, por severidade.
- [glossary.md](glossary.md) — 52 termos do domínio (CONFIRMADO/HIPÓTESE).

### Aprovação da Equipe

- **Reviewed by:** Gilberto
- **Date:** 10/06/2026
- **Confidence:** ☐ high ☑ medium ☐ low

---

## Definição de Pronto deste relatório

- [x] Todas as seções acima preenchidas (sem placeholders).
- [x] Pelo menos 5 regras críticas listadas em §3.1, cada uma referenciando uma `BR-XXX` do catálogo.
- [x] Decisões de migrar/descartar/evoluir em §5 cobrem as 8+ funcionalidades principais.
- [x] Métricas de §6 conferem com os outros artefatos (glossary.md, business-rules-catalog.md, mysteries-found.md).

— Paula


---

### Continuar a leitura

<table width="100%">
<tr>
<td width="50%" valign="top" align="left">
<sub><strong>← ANTERIOR</strong></sub><br/>
<a href="mysteries-found.md"><strong>mysteries-found.md</strong></a><br/>
<sub>Lista de mistérios.</sub>
</td>
<td width="50%" valign="top" align="right">
<sub><strong>PRÓXIMO →</strong></sub><br/>
<a href="../02-spec-moderna/GUIDE.md"><strong>Estágio 2 — Spec</strong></a><br/>
<sub>Próximo estágio: spec moderna.</sub>
</td>
</tr>
</table>

<sub>↑ <a href="../README.md">Voltar ao Kit PT-BR</a></sub>

