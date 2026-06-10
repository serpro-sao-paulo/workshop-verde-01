<!-- markdownlint-disable MD013 MD025 MD026 MD028 MD029 MD034 MD040 MD051 MD060 -->

# Mistérios Encontrados — SIFAP Legado

![ESTÁGIO 01 Arqueologia](https://img.shields.io/badge/ESTÁGIO-01%20Arqueologia-F25022?style=for-the-badge) ![TIPO Worksheet](https://img.shields.io/badge/TIPO-Worksheet-1A1A1A?style=for-the-badge) ![PREENCHA Durante S1](https://img.shields.io/badge/PREENCHA-Durante%20S1-737373?style=for-the-badge)

> 🗺 **Você está aqui:** [Kit PT-BR](../README.md) → [Estágio 1](README.md) → **mysteries-found**

> **Para quem é isto?** Este é um **artefato preenchido pelo time** durante o Estágio 1 (Arqueologia).
>
> **O que você terá ao final do estágio:**
>
> 1. Este documento totalmente preenchido com os dados reais do legado SIFAP
> 2. Rastreabilidade para `01-arqueologia/legado-sifap/` (programas `.NSN` e DDMs)
> 3. Base de evidência usada nas EARS do Estágio 2 (`source_legacy:`)
>
> 📘 **Guia passo a passo:** [`GUIDE.md`](GUIDE.md).


> Registre aqui toda lógica, comportamento ou código que o time não conseguiu explicar.
> "Mistérios" são trechos de código sem documentação, com lógica não-óbvia ou que parecem workarounds.
>
> **Cota mínima para passar pelo portão do Estágio 2:** 5 mistérios documentados.

## O que conta como "mistério"?

- Código que faz algo inesperado sem comentário explicando por quê
- Valores hardcoded sem explicação (números mágicos)
- Lógica condicional que parece um workaround ou gambiarra
- Campos no DDM que não são usados por nenhum programa
- Programas que existem mas não são chamados por ninguém
- Comportamento diferente entre o que a documentação diz e o que o código faz
- Easter eggs deixados pelos desenvolvedores originais

## Níveis de Confiança

| Nível     | Significado                                         |
| --------- | --------------------------------------------------- |
| **ALTA**  | Temos certeza de que há algo estranho aqui          |
| **MÉDIA** | Parece suspeito, mas pode ter explicação            |
| **BAIXA** | Pode ser intencional, mas não conseguimos confirmar |

## Mistérios Catalogados

> Consolidado por `@archaeologist-agent` (`/catalog-mysteries`) em 2026-06-10 a partir de todos os marcadores `<!-- mystery: -->` e divergências sinalizadas em `business-rules-catalog.md`, `dependency-map.md` e `mysteries-checklist.md`. Ordenado por severidade (Critical primeiro).

### Resumo de contagens

| Métrica | Valor |
| --- | --- |
| **Total de mistérios** | 33 |
| 🔴 Critical (`blocks-stage-2`) | 6 |
| 🟠 High (`needs-investigation`) | 13 |
| 🟡 Medium (`needs-facilitator`) | 11 |
| ⚪ Low (`parked`) | 3 |
| Easter eggs | 3 / 3 |

Classificações: **blocks-stage-2** (deve ser resolvido antes das EARS) · **needs-investigation** (resposta provavelmente está em outro arquivo) · **needs-facilitator** (precisa de input de mentor/negócio) · **parked** (fora de escopo, documentar e seguir).

### 🔴 Critical — `blocks-stage-2`

| ID | Mistério | Fonte | Classificação | Confiança | Ação sugerida |
| --- | --- | --- | --- | --- | --- |
| MYS-001 | Beneficiário com idade > 75 recebe status `'S'` (CADBENEF), que VALELEG trata como SUSPENSO/INELEGÍVEL e BATCHPGT não paga (só status `'A'`). Idosos podem ser silenciosamente excluídos do pagamento. Limiar 75 = magic number. | `CADBENEF.NSN#L168-L171`, `VALELEG.NSN#L116-L134`, `BATCHPGT.NSN#L195-L198` | blocks-stage-2 | ALTA | Confirmar com facilitador se a suspensão de idosos é intencional; ler `VALBENEF.NSN` para tratamento alternativo de `'S'`. |
| MYS-006 | Desconto judicial `'J'` deve ignorar o teto de 30%, mas o teto é aplicado sobre o total ACUMULADO dentro do loop — se um não-judicial disparar o corte depois, a parcela judicial também é truncada. Resultado depende da ORDEM dos descontos no PE group. | `CALCDSCT.NSN#L123-L132`, `#L165-L169` | blocks-stage-2 | ALTA | Reler CALCDSCT; pode estar truncando penhoras judiciais indevidamente em produção. |
| MYS-008 | Região 99 = bypass TOTAL de elegibilidade ("bypass do Roberto"/RN-005). `ESCAPE ROUTINE` força ELEGÍVEL e pula as 12 validações. CADBENEF grava `COD-REGIAO` cru, sem validação nem log → qualquer operador pode tornar um cadastro elegível incondicionalmente. Introduzido 05/04/2013 (Anderson Lima). | `VALELEG.NSN#L107-L111`, `CADBENEF.NSN` (grava COD-REGIAO sem validar) | blocks-stage-2 | ALTA | Risco de FRAUDE. Verificar com facilitador quem pode atribuir região 99; confirmar ausência de auditoria. Mapeia para checklist MYS-008. |
| MYS-011 | Duplicação de lógica financeira: `CALCBENF` (subprograma) e `BATCHPGT` (batch) reimplementam os mesmos fatores e fórmula. A doc diz que o batch "invoca CALCBENF", mas o código NÃO chama (não há `CALLNAT`). Qualquer reajuste exige alterar os dois. | `BATCHPGT.NSN#L240-L282`, `CALCBENF.NSN#L180-L228` | blocks-stage-2 | ALTA | Fazer diff campo a campo entre os dois cálculos; registrar como ponto único de verdade para o Estágio 2. |
| MYS-012 | Fórmula de cálculo é MULTIPLICATIVA (`VLR-BASE × FATOR-REG × FATOR-FAM × FATOR-RND × FATOR-IDADE × (1+FATOR-REAJ)`), contradizendo RN-013 (aditiva) e RN-020 (reajuste só sobre VALOR-BASE; aqui incide sobre o total). | `BATCHPGT.NSN#L280-L282`, `CALCBENF.NSN#L224-L228` | blocks-stage-2 | ALTA | Decisão de negócio obrigatória: qual fórmula é a verdadeira? Confrontar código × RN-013/RN-020 com facilitador. |
| MYS-014 | Três regras de desconto conflitantes: CALCBENF/BATCHPGT aplicam 3% fixo acima de R$500 inline; CALCDSCT aplica contribuição PROGRESSIVA 3/5/7/9%; RN-021 fixa teto de 30% via CALCDSCT. Magic numbers 500.00 / 0.03 / faixas. | `BATCHPGT.NSN#L306-L312`, `CALCBENF.NSN#L300-L309`, `CALCDSCT.NSN#L99` (`#L193-L201`) | blocks-stage-2 | ALTA | Definir com negócio a regra canônica de contribuição/desconto antes de qualquer EARS financeira. |

### 🟠 High — `needs-investigation`

| ID | Mistério | Fonte | Classificação | Confiança | Ação sugerida |
| --- | --- | --- | --- | --- | --- |
| MYS-002 | Limite de dependentes hardcoded = **5** no código, mas a doc/Manual diz 3. Contradiz capacidade definida? | `CADDEPEND.NSN` (PE `DEPENDENTES`) | needs-investigation | MÉDIA | Ler `CADDEPEND.NSN` e o PE `DEPENDENTES` em `BENEFICIARIO.ddm` para confirmar a capacidade real. Checklist MYS-002. |
| MYS-003 | `FATOR-K`: variável citada pela doc (RN-014, seção 6) e não encontrada em BATCHPGT/CALCBENF/CALCDSCT/CALCCORR/CADBENEF. **Localizada em `CADPROG`** (`1.00 + FATOR-REAJ * 0.347215`), mas a origem da constante `0.347215` é desconhecida e os programas de cálculo NÃO a consomem. | `CADPROG.NSN` (def. FATOR-K), doc RN-014 | needs-investigation | MÉDIA | Ler `CADPROG.NSN` onde FATOR-K é atribuído; rastrear se `FATOR-REAJ` alimenta os CALC*. Origem de `0.347215` → facilitador. Checklist MYS-003. |
| MYS-004 | Em dezembro o cálculo muda: tipo `'D'`, soma 13º e abono natalino 15% (tipo `'A'`). O 13º ignora FATOR-FAM e FATOR-RND; comentário cita `MESES_ATIVOS/12` mas o código usa `FATOR-IDADE`. | `BATCHPGT.NSN#L292-L303`, `CALCBENF.NSN#L238-L257` | needs-investigation | ALTA | Confirmar fórmula do 13º; comentário × código divergem. Checklist MYS-004. |
| MYS-005 | Arredondamento por truncamento em centavos (`× 100` inteiro `/ 100`) causa perda sistemática. BATCHREL usa `ROUND` em vez de `TRUNCATE` para o mesmo tipo de valor (INC-004). | `BATCHPGT.NSN#L283-L285`, `CALCBENF.NSN#L231-L232`, `BATCHREL.NSN` | needs-investigation | ALTA | Reconciliar método de arredondamento entre BATCHREL e os cálculos. Checklist MYS-005 / INC-004. |
| MYS-007 | CPFs com prefixo `'099'`/`'999'` aceitos sem validação real em VALDOCS (possível backdoor de teste). | `VALDOCS.NSN` (`CHECK-DOC-ESPECIAL`) | needs-investigation | MÉDIA | Ler `VALDOCS.NSN`; avaliar risco de segurança/fraude. Checklist MYS-007 / EGG-002. |
| MYS-009 | BATCHPGT lê `READ BY CPF`, mas o cabeçalho afirma ordenação por CPF "que sistemas downstream dependem" e a doc afirma ordenação por NOME. Conflito código × cabeçalho × doc. | `BATCHPGT.NSN#L178-L191` | needs-investigation | MÉDIA | Ler L178-182; identificar consumidores downstream (RELPGT) e a ordem real esperada. Checklist MYS-009. |
| MYS-010 | Evento de auditoria tipo `'B'` aparentemente ocultado dos relatórios; doc menciona bloqueio por ocorrência de auditoria `'B'` que não está implementado. | `RELAUDIT.NSN`, `BATCHCON.NSN` (`GRAVA-AUDITORIA-*`), `AUDITORIA.ddm` | needs-investigation | BAIXA | Ler `RELAUDIT.NSN` e filtros de tipo de evento. Checklist MYS-010. |
| MYS-015 | Tabela IPCA de CALCCORR declara 10 anos mas só carrega 2010-2012. Competências de 2013+ não encontram o ano → índice 1.0 (correção zero) silenciosamente. Comentário diz "ULTIMA CARGA: 2014". | `CALCCORR.NSN#L48-L98`, `#L177-L190` | needs-investigation | ALTA | Confirmar cobertura temporal; correções recentes podem estar zeradas sem erro. |
| MYS-017 | Elegibilidade e cálculo usam `RENDA-FAMILIAR` total contra `RENDA-MAX`, mas a doc (RN-018) determina renda PER CAPITA (`BN-VL-RENDA-PC`). Métricas diferentes. | `VALELEG.NSN#L157-L163`, `CALCBENF` faixa de renda | needs-investigation | ALTA | Confirmar se aprova/reprova pela métrica errada; impacto direto na elegibilidade. |
| MYS-021 | A doc afirma que VALELEG tem ~1.200 linhas com regras (dados bancários, máx 2 programas simultâneos, atualização < 24 meses, bloqueio por auditoria `'B'`) AUSENTES no arquivo real de 244 linhas. Versão diferente, regras removidas ou levantamento incorreto. | `VALELEG.NSN` (244 linhas) vs doc seção 4.2 | needs-investigation | MÉDIA | Procurar essas regras em outros `.NSN`; perguntar ao facilitador se a doc é de versão antiga. |
| MYS-024 | Tipo de desconto `'C'` (CONTRIB) declarado no comentário do DDM mas SEM `VALUE` no `DECIDE` → cai em `NONE` e é descartado. Possível dupla contagem da contribuição (somada na subrotina antes do loop). | `CALCDSCT.NSN#L160-L162`, `#L99` | needs-investigation | MÉDIA | Reler CALCDSCT; confirmar se há dupla contagem ou descarte indevido de tipo `'C'`. |
| MYS-026 | RN-023 manda aplicar descontos por prioridade numérica e descartar os de menor prioridade ao atingir 30%. O código processa na ordem física do PE group e trunca o acumulado, sem descartar itens específicos. | `CALCDSCT.NSN#L165-L169` | needs-investigation | ALTA | Comparar comportamento real × RN-023; relacionado a MYS-006. |
| MYS-032 | VALBENEF/VALDOCS/VALELEG são ÓRFÃOS (nenhum `CALLNAT` os invoca). Não há gate automático de validação no cadastro nem no batch — a validação de elegibilidade (e o bypass região 99) só roda se VALELEG for executado MANUALMENTE. | `dependency-map.md` (órfãos), `CADBENEF.NSN`, `CADDEPEND.NSN` | needs-investigation | ALTA | Confirmar como a validação é acionada em produção; risco de cadastros sem elegibilidade verificada. |

### 🟡 Medium — `needs-facilitator`

| ID | Mistério | Fonte | Classificação | Confiança | Ação sugerida |
| --- | --- | --- | --- | --- | --- |
| MYS-013 | Fator familiar multiplicativo por faixa com magic numbers `0.05`/`0.03`/`0.02` (0 dep→1.0; 1-2; 3-4; 5+), sem origem documental. | `BATCHPGT.NSN#L247-L259`, `CALCBENF.NSN#L187-L199` | needs-facilitator | MÉDIA | Pedir ao negócio a tabela oficial de acréscimo por dependente. Relacionado a MYS-012. |
| MYS-016 | `STATUS-PGTO` gravado como `'G'` (gerado) no código, mas a doc RN 5.1 afirma `'P'` (pendente). | `BATCHPGT.NSN#L332`, `CALCBENF.NSN#L281` | needs-facilitator | MÉDIA | Confirmar valor de status esperado pelos consumidores (RELPGT/CONSBENF). |
| MYS-018 | CADBENEF altera dados cadastrais (incl. status) mas NÃO grava no DDM AUDITORIA nem chama LOGAUDIT. Inclusões/alterações não deixam trilha. | `CADBENEF.NSN`, `AUDITORIA.ddm` | needs-facilitator | ALTA | Confirmar se auditoria de cadastro é requisito; quem deveria gravar AUDITORIA. |
| MYS-019 | Campo NIS é coletado e gravado mas nunca validado (subprograma VALNISN citado no README, nunca chamado). | `CADBENEF.NSN`, `VALELEG.NSN#L226-L233` | needs-facilitator | MÉDIA | Confirmar regra de validação de NIS com negócio. |
| MYS-020 | RN-006 menciona idade mínima de 16 anos, mas CADBENEF não valida idade mínima (só `DT-NASCIMENTO ≠ 0` e o limiar superior de 75). | `CADBENEF.NSN#L126-L171` | needs-facilitator | MÉDIA | Confirmar se a idade mínima é regra ativa e onde deveria ser aplicada. |
| MYS-022 | Doc RN 5.2 prevê ABEND U4038 se erros excederem MAX-ERROS (default 100). Não há verificação desse limite no código de BATCHPGT. | `BATCHPGT.NSN` | needs-facilitator | MÉDIA | Confirmar se o limite de erros é requisito operacional. |
| MYS-023 | Doc prevê gravar registro com status `'E'` em erro de programa social; o código apenas grava log e faz `ESCAPE TOP`, sem gravar `'E'`. | `BATCHPGT.NSN#L220-L226` | needs-facilitator | MÉDIA | Confirmar tratamento de erro esperado. |
| MYS-025 | Códigos de tipo de desconto: doc RN-022 usa numéricos (01-05); o código usa letras (C/I/J/S/P/A). Sem mapeamento documentado. | `CALCDSCT.NSN`, `PAGAMENTO.ddm` | needs-facilitator | MÉDIA | Obter o mapa oficial letra↔número com especialista de domínio. |
| MYS-028 | CALCCORR só grava correção quando a diferença é positiva. Em meses de IPCA negativo (deflação) o valor nunca cai. Regra de negócio ou bug? | `CALCCORR.NSN#L158-L167` | needs-facilitator | MÉDIA | Confirmar se benefício corrigido pode reduzir. |
| MYS-030 | Doc (Manual 2008 / RN seção 6) cita cálculo pro rata para benefícios com início no meio do mês; ausente no código (variável `MESES_ATIVOS` mencionada mas não calculada). | `CALCBENF.NSN` (13º), doc | needs-facilitator | MÉDIA | Confirmar se pro rata é requisito a preservar no Estágio 2. |
| MYS-033 | CADBENEF em ALTERAÇÃO não regrava COD-PROGRAMA, COD-REGIAO, NIS, CPF, DT-NASCIMENTO, SEXO embora o INPUT os colete. Campos imutáveis por design ou bug. | `CADBENEF.NSN#L177-L218` | needs-facilitator | MÉDIA | Confirmar com negócio quais campos são alteráveis. |

### ⚪ Low — `parked`

| ID | Mistério | Fonte | Classificação | Confiança | Ação sugerida |
| --- | --- | --- | --- | --- | --- |
| MYS-027 | Campos `BENEFICIARIO.STATUS`, `UF` e `#COMP` (competência) carregados em CALCDSCT mas nenhuma regra os usa. Possível regra removida/incompleta. | `CALCDSCT.NSN` | parked | BAIXA | Documentar e seguir; investigar só se surgir requisito de isenção por UF/status. |
| MYS-029 | `CALCIDX` (RN-019) citado como fonte do índice de reajuste, mas CALCCORR não o chama (tabela IPCA hardcoded) e não existe como `.NSN`. Subprograma fantasma. | `CALCCORR.NSN`, README §5.6 | parked | BAIXA | Tratar como código/doc morto, salvo evidência contrária. |
| MYS-031 | Subprogramas VALCPF, VALNISN, LOGAUDIT, FMTVLR, FMTDT citados no README §5.6 não são invocados por programa algum e não existem como `.NSN` (fantasmas). | `dependency-map.md`, README §5.6 | parked | BAIXA | Confirmar que são doc desatualizada; não migrar. |

## Detalhamento dos Mistérios

> Detalhamento dos **6 mistérios Critical** (`blocks-stage-2`). Os demais (High/Medium/Low) estão na tabela acima com fonte e ação sugerida.

### MYS-001: Suspensão automática de beneficiários com idade > 75 (status `'S'`)

- **Arquivo**: `01-arqueologia/legado-sifap/natural-programs/CADBENEF.NSN#L168-L171` (cadeia: `VALELEG.NSN#L116-L134`, `BATCHPGT.NSN#L195-L198`)
- **O que esperávamos**: idade não deveria alterar o status cadastral; beneficiários ativos continuam ativos.
- **O que o código faz**: ao cadastrar/alterar, se idade > 75 o status é sobrescrito para `'S'`. VALELEG trata `'S'` como SUSPENSO (inelegível) e BATCHPGT só paga status `'A'` — o idoso deixa de receber.
- **Hipótese do time**: "AJUSTE STATUS IDOSO" (cabeçalho, 2011) pode ter sido uma suspensão intencional, mas não há documentação e o `'S'` sobrescreve `'A'` até em alteração.
- **Risco se ignorarmos**: migrar a regra cega suspende pagamentos de idosos; removê-la sem entender pode reativar pagamentos indevidos. Decisão de negócio obrigatória.

### MYS-006: Teto de desconto 30% trunca penhora judicial dependendo da ordem

- **Arquivo**: `01-arqueologia/legado-sifap/natural-programs/CALCDSCT.NSN#L123-L132` e `#L165-L169`
- **O que esperávamos**: desconto judicial `'J'` somado integralmente, sem sofrer o teto de 30% (RN-021).
- **O que o código faz**: o teto é aplicado sobre o total ACUMULADO dentro do loop; se um desconto não-judicial estourar o teto após o judicial, o total inteiro (incluindo a parcela judicial) é truncado a 30%.
- **Hipótese do time**: a proteção do judicial só funciona se ele for o último processado no PE group — comportamento frágil e dependente de ordem.
- **Risco se ignorarmos**: em produção pode estar truncando penhoras judiciais indevidamente (passivo legal).

### MYS-008: Região 99 — bypass total de elegibilidade ("bypass do Roberto")

- **Arquivo**: `01-arqueologia/legado-sifap/natural-programs/VALELEG.NSN#L107-L111`
- **O que esperávamos**: todo beneficiário passa pelas validações de faixa etária, renda, status e tipo de programa.
- **O que o código faz**: se `COD-REGIAO = 99`, um `ESCAPE ROUTINE` marca ELEGÍVEL e pula as 12 validações seguintes. CADBENEF grava `COD-REGIAO` cru, sem validação nem log de quem atribuiu.
- **Hipótese do time**: comentário cita "INTERNACIONAL/DIPLOMATICO"; introduzido em 05/04/2013 (Anderson Lima). Sem governança, vira vetor de fraude.
- **Risco se ignorarmos**: qualquer operador pode tornar um cadastro elegível incondicionalmente, sem rastro. **Risco de fraude crítico.**

### MYS-011: Duplicação da lógica financeira entre CALCBENF e BATCHPGT

- **Arquivo**: `01-arqueologia/legado-sifap/natural-programs/BATCHPGT.NSN#L240-L282` e `CALCBENF.NSN#L180-L228`
- **O que esperávamos**: o batch invoca o subprograma de cálculo (CALCBENF), como afirma a doc RN 5.1.
- **O que o código faz**: não existe nenhum `CALLNAT` no sistema; BATCHPGT copia tabelas de fatores e a fórmula inline. As duas implementações vivem em paralelo.
- **Hipótese do time**: duplicação histórica por performance batch; nunca foi reconciliada.
- **Risco se ignorarmos**: qualquer reajuste de regra precisa ser feito em dois lugares; divergem com facilidade. Definir fonte única de verdade no Estágio 2.

### MYS-012: Fórmula multiplicativa contradiz a doc (aditiva)

- **Arquivo**: `01-arqueologia/legado-sifap/natural-programs/BATCHPGT.NSN#L280-L282` e `CALCBENF.NSN#L224-L228`
- **O que esperávamos**: fórmula aditiva e reajuste só sobre o valor-base (RN-013, RN-020).
- **O que o código faz**: `VLR-BASE × FATOR-REG × FATOR-FAM × FATOR-RND × FATOR-IDADE`, depois `× (1 + FATOR-REAJ)` — multiplicativo, com reajuste incidindo sobre o total.
- **Hipótese do time**: a doc descreve uma versão antiga ou idealizada; o código é a regra real há anos.
- **Risco se ignorarmos**: reproduzir a fórmula errada muda o valor pago a milhões de beneficiários. Decisão de negócio obrigatória antes das EARS.

### MYS-014: Três regras de desconto/contribuição conflitantes

- **Arquivo**: `01-arqueologia/legado-sifap/natural-programs/BATCHPGT.NSN#L306-L312`, `CALCBENF.NSN#L300-L309`, `CALCDSCT.NSN#L99` (subrotina `#L193-L201`)
- **O que esperávamos**: uma única regra de desconto, com teto de 30% (RN-021) aplicada via CALCDSCT.
- **O que o código faz**: CALCBENF/BATCHPGT aplicam 3% fixo acima de R$500 inline; CALCDSCT aplica contribuição progressiva 3/5/7/9% por faixa. Três lógicas distintas coexistem.
- **Hipótese do time**: o cálculo inline é um "desconto simplificado" temporário que nunca foi substituído pela chamada a CALCDSCT.
- **Risco se ignorarmos**: valor líquido diverge conforme o caminho de cálculo usado. Definir a regra canônica antes de qualquer EARS financeira.

---

> Os mistérios High, Medium e Low estão tabelados na seção anterior. Promova qualquer um deles para um bloco detalhado quando a investigação avançar.

## Easter Eggs

> Dica: existem **3 easter eggs** escondidos no código legado. Registre aqui os que encontrar:

1. [x] Easter Egg 1 (EGG-001): bloco COMENTADO do **"Plano Verão"** em `CALCCORR.NSN#L100-L110` — correção monetária de 1989-1991 (multiplicadores 2.7500 e 1.4289, transição Cruzado→Cruzeiro), marcado "NAO REMOVER (HISTORICO)". Política econômica dos anos 90 nunca removida.
2. [x] Easter Egg 2 (EGG-002): validação especial em `VALDOCS.NSN` (`CHECK-DOC-ESPECIAL`) que aceita CPFs com prefixo `'099'`/`'999'` sem verificação — aparenta backdoor de teste (ver MYS-007).
3. [x] Easter Egg 3 (EGG-003): código morto referenciando o **"Banco Real"** (bloco comentado após `END-WORK` em `BATCHCON.NSN`) — integração bancária com instituição que não existe mais (absorvida pelo Santander em 2008-2009).

## Resumo

- Total de mistérios encontrados: **33**
- Confiança alta: **9**
- Confiança média: **17**
- Confiança baixa: **7**
- Easter eggs encontrados: **3 / 3**

### Por severidade / classificação

- 🔴 Critical (`blocks-stage-2`): 6 — MYS-001, 006, 008, 011, 012, 014
- 🟠 High (`needs-investigation`): 13 — MYS-002, 003, 004, 005, 007, 009, 010, 015, 017, 021, 024, 026, 032
- 🟡 Medium (`needs-facilitator`): 11 — MYS-013, 016, 018, 019, 020, 022, 023, 025, 028, 030, 033
- ⚪ Low (`parked`): 3 — MYS-027, 029, 031

> **Os 6 Critical são portas duras para o Estágio 2.** As EARS de cálculo de benefício, descontos e elegibilidade não devem ser escritas até que MYS-008 (bypass região 99), MYS-001 (status idoso), MYS-011/012/014 (fórmula financeira e descontos) e MYS-006 (teto judicial) sejam validados com o facilitador/negócio.

---

### Continuar a leitura

<table width="100%">
<tr>
<td width="50%" valign="top" align="left">
<sub><strong>← ANTERIOR</strong></sub><br/>
<a href="mysteries-checklist.md"><strong>mysteries-checklist.md</strong></a><br/>
<sub>Lista do que procurar.</sub>
</td>
<td width="50%" valign="top" align="right">
<sub><strong>PRÓXIMO →</strong></sub><br/>
<a href="discovery-report.md"><strong>discovery-report.md</strong></a><br/>
<sub>Síntese final.</sub>
</td>
</tr>
</table>

<sub>↑ <a href="README.md">Voltar ao Kit PT-BR</a></sub>

