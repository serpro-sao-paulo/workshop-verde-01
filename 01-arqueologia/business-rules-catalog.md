<!-- markdownlint-disable MD013 MD025 MD026 MD028 MD029 MD034 MD040 MD051 MD060 -->

# Catálogo de Regras de Negócio — SIFAP Legado

![ESTÁGIO 01 Arqueologia](https://img.shields.io/badge/ESTÁGIO-01%20Arqueologia-F25022?style=for-the-badge) ![TIPO Worksheet](https://img.shields.io/badge/TIPO-Worksheet-1A1A1A?style=for-the-badge) ![PREENCHA Durante S1](https://img.shields.io/badge/PREENCHA-Durante%20S1-737373?style=for-the-badge)

> 🗺 **Você está aqui:** [Kit PT-BR](../README.md) → [Estágio 1](README.md) → **business-rules-catalog**

> **Para quem é isto?** Este é um **artefato preenchido pelo time** durante o Estágio 1 (Arqueologia).
>
> **O que você terá ao final do estágio:**
>
> 1. Este documento totalmente preenchido com os dados reais do legado SIFAP
> 2. Rastreabilidade para `01-arqueologia/legado-sifap/` (programas `.NSN` e DDMs)
> 3. Base de evidência usada nas EARS do Estágio 2 (`source_legacy:`)
>
> 📘 **Guia passo a passo:** [`GUIDE.md`](GUIDE.md).


> Registre aqui todas as regras de negócio extraídas do código Natural/Adabas.
> Cada regra precisa ter rastreabilidade até o código-fonte.
>
> **REGRA DURA:** linhas com `Programa Fonte` vazio são **inválidas** e não contam para o gate do Estágio 2. Use o formato `01-arqueologia/legado-sifap/natural-programs/ARQUIVO.NSN#L<inicio>-L<fim>` sempre que possível. Mínimo aceito: nome do arquivo .NSN.

## Como pensar em "regra de negócio"

O que conta:

- Um `IF` que decide algo no domínio (ex.: _"se a UF é do Nordeste e o programa é Seca, valor base × 1.2"_)
- Uma constante numérica sem explicação (ex.: `0.075` num cálculo de imposto)
- Uma transição de status com regra (ex.: _"só de A para S, nunca de I para A"_)
- Um tratamento especial para um caso (ex.: _"se o CPF começa com 999, é teste"_)

O que NÃO conta: paginação de relatório, formatação de saída, manipulação de cursor Adabas, abertura de arquivo. Ignore esses detalhes de implementação.

## Níveis de Risco

| Nível       | Descrição                                                     |
| ----------- | ------------------------------------------------------------- |
| **CRÍTICO** | Regra financeira ou de segurança — erro causa prejuízo direto |
| **ALTO**    | Regra de negócio central — afeta fluxo principal              |
| **MÉDIO**   | Regra de validação ou formatação — afeta qualidade dos dados  |
| **BAIXO**   | Regra de apresentação ou conveniência — impacto limitado      |

## Regras Encontradas

| ID     | Regra de Negócio | Programa Fonte | Campos DDM | Nível de Risco | Notas |
| ------ | ---------------- | -------------- | ---------- | -------------- | ----- |
| BR-001 |                  |                |            |                |       |
| BR-002 |                  |                |            |                |       |
| BR-003 |                  |                |            |                |       |
| BR-004 |                  |                |            |                |       |
| BR-005 |                  |                |            |                |       |
| BR-006 |                  |                |            |                |       |
| BR-007 |                  |                |            |                |       |
| BR-008 |                  |                |            |                |       |
| BR-009 |                  |                |            |                |       |
| BR-010 |                  |                |            |                |       |
| BR-011 |                  |                |            |                |       |
| BR-012 |                  |                |            |                |       |
| BR-013 |                  |                |            |                |       |
| BR-014 |                  |                |            |                |       |
| BR-015 |                  |                |            |                |       |

> Adicione mais linhas conforme necessário. Lembre-se: existem **10 regras escondidas** no código!

## Regras de BATCHPGT.NSN

> Extração sistemática por `@archaeologist-agent` em 2026-06-10 (primeira passada).
> Arquivo analisado: `01-arqueologia/legado-sifap/natural-programs/BATCHPGT.NSN` (377 linhas, lido de cima a baixo).
> Cross-reference: `01-arqueologia/legado-sifap/legacy-docs/REGRAS-NEGOCIO-2012.md`.
> **Convenção:** Confirmada = corresponde à documentação · Inferida = só código, sem suporte documental · Mistério = lógica/valor pouco claro ou em contradição com a doc.

| #  | Declaração da Regra | Candidato EARS | Fonte | Classificação | Notas |
|----|---------------------|----------------|-------|---------------|-------|
| 1  | Se o CPF do beneficiário for igual ao do registro imediatamente anterior, o sistema deve ignorá-lo (conta como ignorado) e não gerar pagamento. | Unwanted | `BATCHPGT.NSN#L188-L191` | Inferida | Deduplicação por CPF; só detecta duplicatas **adjacentes** — depende da leitura `READ ... BY CPF` (L182). Não documentada. |
| 2  | Se o beneficiário não tiver status `'A'` (ativo), o sistema deve ignorá-lo. | Unwanted | `BATCHPGT.NSN#L195-L198` | Confirmada | RN seção 5.1: "Todos os beneficiários ativos (BN-CD-SIT = 'A') são processados". Campo `BENEFICIARIO.STATUS`. |
| 3  | Quando já existir um pagamento para o CPF na mesma competência, o sistema deve ignorar o beneficiário (evita duplicidade no mês). | Unwanted | `BATCHPGT.NSN#L202-L210` | Inferida | Controle de idempotência mensal via `FIND PAGAMENTO WITH CPF-BENEF`. Não documentado explicitamente. |
| 4  | Se o programa social do beneficiário não for encontrado, o sistema deve registrar erro em log, contabilizar como erro e não gerar pagamento. | Unwanted | `BATCHPGT.NSN#L220-L226` | Confirmada (parcial) | RN 5.2 trata erros, mas diz marcar status `'E'` no DDM PAGAMENTO. O código apenas grava log e faz `ESCAPE TOP` — **não grava registro 'E'**. <!-- mystery: divergência entre doc (status 'E' gravado) e código (apenas log) --> |
| 5  | Se o programa social não estiver ativo (`STATUS-PROG NE 'A'`), o sistema deve ignorar o beneficiário. | Unwanted | `BATCHPGT.NSN#L227-L230` | Inferida | Campo `PROGRAMA-SOCIAL.STATUS-PROG`. Não documentado. |
| 6  | Quando o código de região estiver entre 1 e 25, o sistema deve aplicar o fator regional da tabela interna; caso contrário, deve usar fator 1.0000. | Event-driven | `BATCHPGT.NSN#L240-L244` (tabela `#L124-L150`) | Mistério | <!-- mystery: a tabela #TAB-REG tem 27 posições mas a condição só usa 1-25; regiões 26/27 (e qualquer outra) caem no fator 1.0. Tabela duplicada do CALCBENF (L123). Relaciona-se ao "bypass região 99" pendente na doc seção 6 --> |
| 7  | O fator familiar é determinado por faixas de dependentes: 0 dep → 1.0000; 1-2 dep → 1.0000 + (dep × 0.05); 3-4 dep → 1.1000 + ((dep-2) × 0.03); 5+ dep → 1.1600 + ((dep-4) × 0.02). | Ubiquitous | `BATCHPGT.NSN#L247-L259` | Mistério | <!-- mystery: contradiz RN-013, que documenta acréscimo LINEAR (ACRESCIMO-DEPEND × QT-DEPEND). Aqui é multiplicativo por faixa, com magic numbers 0.05/0.03/0.02 sem origem documental --> |
| 8  | O fator de renda é a primeira faixa cujo limite superior seja maior ou igual à renda familiar declarada (faixas: 300→1.0; 600→0.85; 1000→0.70; 1500→0.55; 9999.99→0.40). | Event-driven | `BATCHPGT.NSN#L367-L375` (tabela `#L153-L162`) | Confirmada | RN-018: "primeira faixa cujo limite superior seja maior ou igual à renda declarada". Magic numbers das faixas só no código. |
| 9  | O fator idade é: idade ≥ 65 → 1.1500; ≥ 60 → 1.1000; < 18 → 1.0500; entre 18 e 59 → 1.0000. | Ubiquitous | `BATCHPGT.NSN#L265-L277` | Inferida | Magic numbers de fator etário sem suporte documental. Idade derivada de `DT-NASCIMENTO` (L236-237). |
| 10 | O valor bruto do benefício é `VLR-BASE × FATOR-REG × FATOR-FAM × FATOR-RND × FATOR-IDADE`, multiplicado por `(1 + FATOR-REAJ)`. | Ubiquitous | `BATCHPGT.NSN#L280-L282` | Mistério | <!-- mystery: contradiz RN-013 (fórmula ADITIVA documentada) e RN-020 (reajuste só sobre VALOR-BASE; aqui o reajuste incide sobre o total já com todos os fatores). Possível relação com o "FATOR-K" não explicado na doc seção 6 --> |
| 11 | Os valores monetários são truncados em centavos (não arredondados): `(valor × 100)` em campo inteiro, depois `/ 100`. | Ubiquitous | `BATCHPGT.NSN#L283-L285` e `#L319-L320` | Confirmada | RN-014: "sempre arredondado para baixo em centavos (truncamento)". Usa `#VLR-TEMP (N11)` como inteiro. |
| 12 | Quando o mês for dezembro, o sistema deve marcar o pagamento como tipo `'D'` e somar ao bruto um 13º calculado como `VLR-BASE × FATOR-REG × FATOR-IDADE`. | Event-driven | `BATCHPGT.NSN#L292-L297` | Inferida | <!-- mystery: doc seção 6 lista "Cálculo do 13o benefício" como PENDENTE/Alta. Não está claro por que o 13º ignora FATOR-FAM e FATOR-RND, usando só região e idade --> |
| 13 | Onde o tipo de programa for `'A'`, em dezembro, o sistema deve calcular um abono de `VLR-BENF × 0.15` e somá-lo ao valor bruto. | Optional | `BATCHPGT.NSN#L298-L303` | Inferida | Magic number 0.15 e seletor `TIPO = 'A'` sem documentação. Abono natalino mencionado como pendente na doc. |
| 14 | Se o valor bruto exceder R$ 500,00, o sistema deve aplicar um desconto de 3% sobre o bruto. | Unwanted | `BATCHPGT.NSN#L306-L312` | Mistério | <!-- mystery: a doc 5.1 afirma que os descontos são aplicados invocando CALCDSCT e RN-021 fixa teto de 30%. O código batch faz um "DESCONTO SIMPLIFICADO" inline de 3% acima de R$500, sem chamar CALCDSCT. Magic numbers 500.00 e 0.03 --> |
| 15 | Se o valor líquido resultar negativo, o sistema deve defini-lo como zero. | Unwanted | `BATCHPGT.NSN#L315-L318` | Inferida | Piso de zero no líquido (`VLR-LIQ = VLR-BRUTO - VLR-DESC`). Não documentado. |

### Mistérios transversais (não-condicionais) sinalizados em BATCHPGT.NSN

<!-- mystery: ORDENAÇÃO — o cabeçalho (L178-179) afirma leitura "em ordem alfabética por CPF" e que "sistemas downstream dependem desta ordenação", mas a doc (RN 5.1, nota) afirma que a ordenação real é alfabética por NOME (BN-NM-BENEF). O código usa READ BY CPF (L182). Conflito entre código, cabeçalho e doc. -->
<!-- mystery: STATUS DO PAGAMENTO — o código grava STATUS-PGTO = 'G' (gerado) ao STORE (L332), mas a doc RN 5.1 afirma status 'P' (pendente). Divergência de valor de status. -->
<!-- mystery: MAX-ERROS/ABEND — a doc RN 5.2 prevê interrupção com ABEND U4038 se erros excederem MAX-ERROS (default 100). Não há nenhuma verificação desse limite no código de BATCHPGT.NSN. -->

> **Observação de processo:** as regras 6, 7, 9 e a tabela regional (L123-150) aparecem como "MESMA DO CALCBENF" no comentário L123 — provável **duplicação de lógica de cálculo** entre BATCHPGT e CALCBENF. Confirmar ao ler `CALCBENF.NSN` (próximo na ordem de leitura) e registrar como risco de divergência.

## Regras de CALCBENF.NSN

> Extração sistemática por `@archaeologist-agent` em 2026-06-10 (primeira passada).
> Arquivo analisado: `01-arqueologia/legado-sifap/natural-programs/CALCBENF.NSN` (326 linhas, lido de cima a baixo).
> Cross-reference: `01-arqueologia/legado-sifap/legacy-docs/REGRAS-NEGOCIO-2012.md` e `MANUAL-TECNICO-SIFAP-2008.md`.
> **Achado central:** a lógica de cálculo deste subprograma é **quase idêntica** à de `BATCHPGT.NSN` (mesmos fatores, mesma fórmula multiplicativa, mesmo 13º, mesmo abono 15%, mesmo desconto inline 3%). Confirma o risco de **duplicação de regra financeira** registrado na seção anterior.

| #  | Declaração da Regra | Candidato EARS | Fonte | Classificação | Notas |
|----|---------------------|----------------|-------|---------------|-------|
| 1  | Se a competência informada tiver mês fora do intervalo 1-12, o sistema deve recusar o cálculo e encerrar. | Unwanted | `CALCBENF.NSN#L141-L144` | Inferida | Validação de entrada (`#MES < 1 OR #MES > 12`). Não documentada. |
| 2  | Se o beneficiário (por CPF) não for encontrado, o sistema deve recusar o cálculo. | Unwanted | `CALCBENF.NSN#L155-L158` | Inferida | `FIND BENEFICIARIO WITH CPF`; flag `#FOUND-B`. Não documentada. |
| 3  | Se o beneficiário não tiver status `'A'` (ativo), o sistema deve recusar o cálculo. | Unwanted | `CALCBENF.NSN#L160-L163` | Confirmada | RN 4.1/5.1: situação cadastral ativa (`BN-CD-SIT = 'A'`). Campo `BENEFICIARIO.STATUS`. |
| 4  | Se o programa social do beneficiário não for encontrado, o sistema deve recusar o cálculo. | Unwanted | `CALCBENF.NSN#L174-L177` | Inferida | `FIND PROGRAMA WITH COD-PROGRAMA`; flag `#FOUND-P`. Não documentada. |
| 5  | Quando a região estiver entre 1 e 25, o sistema deve aplicar o fator regional da tabela interna; caso contrário, deve usar fator 1.0000. | Event-driven | `CALCBENF.NSN#L180-L184` (tabela `#L91-L117`) | Mistério | <!-- mystery: o comentário L90 diz "99=ESPEC" e L116-117 reserva posições 26/27, mas a condição só cobre 1-25. Região 99 (o "bypass do Roberto", RN-005) cai silenciosamente no fator 1.0. Significado de 99 segue desconhecido. --> |
| 6  | O fator familiar é: 0 dep → 1.0000; 1-2 dep → 1.0000 + (dep × 0.05); 3-4 dep → 1.1000 + ((dep-2) × 0.03); 5+ dep → 1.1600 + ((dep-4) × 0.02). | Ubiquitous | `CALCBENF.NSN#L187-L199` | Mistério | <!-- mystery: contradiz RN-013, que documenta acréscimo ADITIVO (ACRESCIMO-DEPEND × QT-DEPEND). Aqui é multiplicativo por faixa, com magic numbers 0.05/0.03/0.02. Idêntico ao BATCHPGT regra 7. --> |
| 7  | O fator de renda é a primeira faixa cujo teto seja ≥ à renda familiar (300→1.0; 600→0.85; 1000→0.70; 1500→0.55; 9999.99→0.40). | Event-driven | `CALCBENF.NSN#L304-L308` (tabela `#L120-L129`) | Confirmada | RN-018: "primeira faixa cujo limite superior seja maior ou igual à renda declarada". Subrotina `DET-FAIXA-RENDA`. |
| 8  | O fator idade é: ≥ 65 → 1.1500; ≥ 60 → 1.1000; < 18 → 1.0500; entre 18 e 59 → 1.0000. | Ubiquitous | `CALCBENF.NSN#L207-L219` | Inferida | Magic numbers etários sem suporte documental. Idade derivada de `DT-NASCIMENTO`. Idêntico ao BATCHPGT regra 9. |
| 9  | O valor do benefício é `VLR-BASE × FATOR-REG × FATOR-FAM × FATOR-RND × FATOR-IDADE`, depois multiplicado por `(1 + FATOR-REAJ)`. | Ubiquitous | `CALCBENF.NSN#L224-L228` | Mistério | <!-- mystery: contradiz RN-013 (fórmula ADITIVA) e RN-020 (reajuste só sobre VALOR-BASE; aqui incide sobre o total com todos os fatores). Nenhum "FATOR-K" citado na doc (seção 6) foi encontrado no código — o FATOR-K continua sem evidência. --> |
| 10 | Os valores monetários são truncados em centavos (`× 100` em campo inteiro, depois `/ 100`). | Ubiquitous | `CALCBENF.NSN#L231-L232`, `#L270-L271` | Confirmada | RN-014: "sempre arredondado para baixo em centavos (truncamento)". |
| 11 | Quando o mês for dezembro, o sistema deve marcar o pagamento como tipo `'D'` e somar ao bruto um 13º = `VLR-BASE × FATOR-REG × FATOR-IDADE`. | Event-driven | `CALCBENF.NSN#L240-L246` | Mistério | <!-- mystery: o comentário L238 documenta a fórmula como VLR_BASE × FATOR_REG × (MESES_ATIVOS/12), mas o CÓDIGO usa FATOR_IDADE no lugar de (MESES_ATIVOS/12). Comentário e código divergem. Doc seção 6 lista o 13º como pendente. --> |
| 12 | Onde o programa for tipo `'A'`, em dezembro, o sistema deve calcular abono natalino = `VLR-BENF × 0.15` e somá-lo ao bruto; caso contrário o abono é zero. | Optional | `CALCBENF.NSN#L249-L257` | Inferida | Magic number 0.15 e seletor `TIPO = 'A'` sem documentação formal. Abono natalino citado como pendente na doc. Idêntico ao BATCHPGT regra 13. |
| 13 | Se o valor bruto exceder R$ 500,00, o sistema deve aplicar desconto de 3% ("contribuição social") sobre o bruto. | Unwanted | `CALCBENF.NSN#L300-L309` (subrotina `CALC-DESCONTOS`) | Mistério | <!-- mystery: o próprio comentário L299 diz "SIMPLIFICADO (VER CALCDSCT P/ COMPLETO)". Contradiz RN-021 (teto de 30% via CALCDSCT). Magic numbers 500.00 e 0.03. Idêntico ao BATCHPGT regra 14 — duplicação. --> |
| 14 | Se o valor líquido resultar negativo, o sistema deve defini-lo como zero. | Unwanted | `CALCBENF.NSN#L265-L267` | Inferida | Piso de zero no líquido. Não documentado. Idêntico ao BATCHPGT regra 15. |

### Mistérios e achados transversais em CALCBENF.NSN

<!-- mystery: STATUS DO PAGAMENTO — grava STATUS-PGTO = 'G' ao STORE (L281), mas a doc RN 5.1 afirma status 'P' (pendente). Mesma divergência observada em BATCHPGT. -->
<!-- mystery: FATOR-K — a doc (RN-014 nota e seção 6) cita um "FATOR-K" que ninguém soube explicar. Após leitura completa de CALCBENF, NÃO existe variável ou constante FATOR-K no código. O mistério persiste: ou foi removido, ou está em outro programa (CALCCORR?). -->
<!-- mystery: PRO RATA — a doc (Manual 2008, seção 3.3.1 e RN seção 6) cita "cálculo proporcional para benefícios com início no meio do mês". Não há lógica pro rata em CALCBENF; o comentário do 13º até menciona (MESES_ATIVOS/12), mas a variável não é calculada nem usada. -->

> **Risco de migração (DUPLICAÇÃO):** `CALCBENF.NSN` (subprograma) e `BATCHPGT.NSN` (batch) reimplementam a MESMA lógica financeira de forma independente (fatores regional/familiar/renda/idade, fórmula multiplicativa, 13º, abono 15%, desconto 3%). A doc (RN 5.1) afirma que o batch "invoca CALCBENF", mas o código do BATCHPGT **não chama CALCBENF** — copiou a lógica. Qualquer reajuste de regra exige alterar os dois. Registrar como mistério/risco CRÍTICO para o Estágio 2.

## Regras de CALCDSCT.NSN

> Extração sistemática por `@archaeologist-agent` em 2026-06-10 (primeira passada).
> Arquivo analisado: `01-arqueologia/legado-sifap/natural-programs/CALCDSCT.NSN` (204 linhas, lido de cima a baixo).
> Cross-reference: `01-arqueologia/legado-sifap/legacy-docs/REGRAS-NEGOCIO-2012.md` (seção 3) e `COMO-LER-NATURAL.md`.
> **Achado central:** este é o módulo "completo" de descontos (v4.0/2015). Confirma o teto de 30% e a exceção judicial (RN-021), mas revela divergências graves com a doc nos **códigos de tipo de desconto** e na **ordem de aplicação do teto**.

| #  | Declaração da Regra | Candidato EARS | Fonte | Classificação | Notas |
|----|---------------------|----------------|-------|---------------|-------|
| 1  | Quando o pagamento (`NUM-PAGTO`) localizado não pertencer ao CPF informado, o sistema não deve considerá-lo válido. | Unwanted | `CALCDSCT.NSN#L75-L79` | Inferida | Casa `NUM-PAGTO` com `CPF-BENEF` antes de processar. Não documentado. |
| 2  | Se o pagamento não for encontrado, o sistema deve recusar o cálculo de descontos. | Unwanted | `CALCDSCT.NSN#L82-L85` | Inferida | Flag `#FOUND`. Não documentado. |
| 3  | Se o beneficiário não for encontrado, o sistema deve recusar o cálculo de descontos. | Unwanted | `CALCDSCT.NSN#L91-L94` | Inferida | Usa `*NUMBER(BENEFICIARIO-V) = 0`. Não documentado. |
| 4  | O sistema deve sempre aplicar uma contribuição social obrigatória sobre o bruto, por faixa progressiva: ≤ R$500 → 3%; ≤ R$1000 → 5%; ≤ R$2000 → 7%; acima → 9%. | Ubiquitous | `CALCDSCT.NSN#L99` e subrotina `#L193-L201` (tabela `#L57-L65`) | Mistério | <!-- mystery: contribuição PROGRESSIVA 3/5/7/9% aqui contradiz o desconto FIXO de 3% acima de R$500 em CALCBENF/BATCHPGT. Qual é a regra verdadeira? Magic numbers das faixas sem origem documental. Tipo 'C' (CONTRIB) é tratado AQUI, não no DECIDE. --> |
| 5  | O teto máximo de desconto é 30% do valor bruto (truncado em centavos). | Ubiquitous | `CALCDSCT.NSN#L101-L105` | Confirmada | RN-021: "total de descontos não pode exceder 30% do valor bruto". Também citado em `COMO-LER-NATURAL.md` §5. |
| 6  | Se um desconto tiver data-fim diferente de zero e anterior a hoje, o sistema deve ignorá-lo (desconto expirado). | Unwanted | `CALCDSCT.NSN#L112-L115` | Inferida | Vigência: `DT-FIM-DSCT = 0` significa sem término. Não documentado. |
| 7  | Se um desconto tiver data-início posterior a hoje, o sistema deve ignorá-lo (ainda não vigente). | Unwanted | `CALCDSCT.NSN#L116-L118` | Inferida | Vigência futura. Não documentado. |
| 8  | Quando o tipo de desconto for `'J'` (judicial), o sistema deve usar o valor fixo cadastrado, ou, se ausente, calcular pelo percentual sobre o bruto, e somá-lo **sem aplicar o teto de 30%**. | Event-driven | `CALCDSCT.NSN#L123-L132` | Confirmada | RN-021 nota: exceção judicial ao teto. Resolve o mistério pendente da doc (seção 6). Campos `TIPO-DSCT='J'`, `NUM-PROCESSO`. |
| 9  | Quando o tipo for `'P'` (pensão alimentícia), o sistema deve usar valor fixo ou percentual sobre o bruto e somar ao total. | Event-driven | `CALCDSCT.NSN#L133-L141` | Inferida | Tipo 'P' não consta na tabela RN-022 (que só lista 01-05). Sujeito ao teto. |
| 10 | Quando o tipo for `'I'` (imposto retido), o sistema deve calcular o desconto pelo percentual sobre o bruto. | Event-driven | `CALCDSCT.NSN#L142-L146` | Confirmada (parcial) | RN-022 código 02 = IRRF. Alíquota vem de `PCT-DSCT`, não de tabela da Receita como a doc sugere. |
| 11 | Quando o tipo for `'S'` (sindical), o sistema deve descontar 1% fixo do bruto. | Event-driven | `CALCDSCT.NSN#L147-L150` | Inferida | Magic number 0.01. Tipo 'S' não consta em RN-022. Sujeito ao teto. |
| 12 | Quando o tipo for `'A'` (administrativo), o sistema deve usar valor fixo ou percentual sobre o bruto. | Event-driven | `CALCDSCT.NSN#L151-L159` | Inferida | Tipo 'A' não consta em RN-022. Pode ser o código 04 (ressarcimento ao erário)? Não confirmável. |
| 13 | Quando o tipo de desconto não corresponder a J/P/I/S/A, o sistema deve ignorá-lo. | Unwanted | `CALCDSCT.NSN#L160-L162` (`NONE`) | Mistério | <!-- mystery: o tipo 'C' (CONTRIB) está declarado no comentário do DDM (L26) mas NÃO tem VALUE no DECIDE — cai em NONE/IGNORE. Descontos tipo 'C' cadastrados no PE group são silenciosamente descartados (a contribuição vem da subrotina, não do PE). --> |
| 14 | Para todo desconto que não seja judicial, se o total acumulado exceder o teto de 30%, o sistema deve limitar o total ao teto. | Unwanted | `CALCDSCT.NSN#L165-L169` | Mistério | <!-- mystery: o teto é aplicado DENTRO do loop, sobre o total ACUMULADO. Se um judicial (sem teto) for somado e depois um não-judicial disparar o corte, o total inteiro — incluindo a parcela judicial — é truncado a 30%, violando "judicial não tem teto". O resultado depende da ORDEM dos descontos no PE group. --> |

### Mistérios e divergências transversais em CALCDSCT.NSN

<!-- mystery: CÓDIGOS DE TIPO — a doc RN-022 lista códigos NUMÉRICOS (01=consignação, 02=IR, 03=previdenciária, 04=ressarcimento, 05=?). O código usa códigos de UMA LETRA (C/I/J/S/P/A). Não há mapeamento documentado entre os dois esquemas. Impossível confirmar correspondência item a item. -->
<!-- mystery: ORDEM DE PRIORIDADE — RN-023 afirma que descontos são aplicados por prioridade numérica do código (01, depois 02...) e que ao atingir 30% os de MENOR prioridade são descartados. O código NÃO ordena por prioridade: processa na ordem física do PE group (#IDX 1..N) e ao estourar o teto trunca o ACUMULADO, sem descartar itens específicos. Comportamento real diverge da regra documentada. -->
<!-- mystery: CONTRIBUIÇÃO DUPLA? — a contribuição social (3/5/7/9%) é somada ao #VLR-TOTAL-DSCT na subrotina (L99) ANTES do loop. Não está claro se isso é intencional ou se há dupla contagem quando também existe um desconto tipo 'C'/'I' cadastrado. -->
<!-- mystery: STATUS/UF/COMPETENCIA lidos mas não usados — campos BENEFICIARIO.STATUS, BENEFICIARIO.UF e #COMP (competência) são carregados mas nenhuma regra os utiliza. Possível regra removida ou incompleta (ex.: isenção por UF ou por status). -->

> **Risco de migração (TETO ORDEM-DEPENDENTE):** a regra 14 + o comentário "JUDICIAL NAO TEM TETO" (L131) são contraditórios na prática. Como o teto trunca o acumulado total dentro do loop, a proteção do desconto judicial depende de ele ser processado por último. Registrar como risco **CRÍTICO** — em produção pode estar truncando penhoras judiciais indevidamente.

## Regras de CALCCORR.NSN

> Extração sistemática por `@archaeologist-agent` em 2026-06-10 (primeira passada).
> Arquivo analisado: `01-arqueologia/legado-sifap/natural-programs/CALCCORR.NSN` (192 linhas, lido de cima a baixo).
> Cross-reference: `01-arqueologia/legado-sifap/legacy-docs/REGRAS-NEGOCIO-2012.md` (seção 2.3) e Manual 2008 (3.3.2).
> **Achado central:** correção retroativa por IPCA acumulado. **NÃO contém `FATOR-K`** — após ler CALCBENF, CALCDSCT e CALCCORR, o "FATOR-K" citado pela doc continua sem evidência no código. A doc (RN-019) dizia que o índice vinha do subprograma `CALCIDX`, mas aqui ele está **hardcoded** numa tabela interna.

| #  | Declaração da Regra | Candidato EARS | Fonte | Classificação | Notas |
|----|---------------------|----------------|-------|---------------|-------|
| 1  | Se a competência inicial for maior que a final, o sistema deve recusar a correção (período inválido). | Unwanted | `CALCCORR.NSN#L119-L122` | Inferida | Validação de intervalo `#COMP-INI > #COMP-FIM`. Não documentada. |
| 2  | Ao ler pagamentos, quando o CPF do registro diferir do CPF solicitado, o sistema deve encerrar a leitura. | Event-driven | `CALCCORR.NSN#L130-L132` | Inferida | Controle de fim de range no `READ BY CPF-BENEF`. Detalhe de implementação na fronteira do loop. |
| 3  | Se a competência do pagamento for anterior à competência inicial, o sistema deve ignorar o registro. | Unwanted | `CALCCORR.NSN#L134-L136` | Inferida | Filtro de período (limite inferior). Não documentada. |
| 4  | Se a competência do pagamento for posterior à competência final, o sistema deve encerrar a leitura. | Event-driven | `CALCCORR.NSN#L137-L139` | Inferida | Filtro de período (limite superior). Não documentada. |
| 5  | Se o pagamento já estiver marcado como corrigido (`IND-CORRIGIDO = 'S'`), o sistema deve ignorá-lo (evita dupla correção). | Unwanted | `CALCCORR.NSN#L141-L143` | Inferida | Idempotência da correção. Campo `PAGAMENTO.IND-CORRIGIDO`. Não documentado. |
| 6  | O sistema deve corrigir o valor bruto multiplicando-o pelo índice IPCA acumulado do período, truncando o resultado em centavos. | Ubiquitous | `CALCCORR.NSN#L150-L156` (subrotina `#L177-L190`) | Confirmada (parcial) | RN-019: reajuste por índice. Mas RN-019 fala em reajuste ANUAL em janeiro via decreto; aqui é correção RETROATIVA mensal por IPCA acumulado. Mecanismos diferentes. |
| 7  | O sistema deve registrar a correção apenas quando a diferença for positiva (valor corrigido > valor original); caso contrário, não grava. | Unwanted | `CALCCORR.NSN#L158-L167` | Inferida | Só aplica correção "para cima". Deflação (IPCA negativo) é descartada silenciosamente. Não documentado. |
| 8  | Para cada mês do período, o índice acumulado é multiplicado por `(1 + IPCA do mês/ano)` buscado na tabela interna. | Ubiquitous | `CALCCORR.NSN#L177-L190` | Mistério | <!-- mystery: a tabela #IPCA-ANO declara 10 anos (#L48) mas só carrega 2010, 2011 e 2012 (#L55-L98). Anos 4-10 ficam ZERADOS. Para qualquer competência fora de 2010-2012, o FOR não acha o ano (nenhum match) e #IND-ACUM permanece 1.0 → correção zero, silenciosamente. Comentário diz "ULTIMA CARGA: 2014" mas dados param em 2012. --> |

### Mistérios e achados transversais em CALCCORR.NSN

<!-- mystery: FATOR-K AUSENTE (definitivo nesta passada) — a doc (RN-014 nota; seção 6) cita um "FATOR-K" como multiplicador de cálculo não explicado. Após leitura completa de CALCBENF, CALCDSCT e CALCCORR, NENHUM dos três contém variável/constante FATOR-K. Restam CALCCORR já lido; investigar se está em CADBENEF/CADPROG ou se foi removido. Hipótese: pode ser apelido informal para um dos fatores (FATOR-REG? FATOR-REAJ?) — não confirmável. -->
<!-- mystery: CALCIDX NÃO EXISTE COMO CHAMADA — RN-019 afirma que o índice de reajuste "é registrado na tabela interna do subprograma CALCIDX". CALCCORR NÃO chama CALCIDX (nenhum PERFORM/CALLNAT); a tabela IPCA está hardcoded no próprio programa. CALCIDX não está na lista de 15 programas do inventário. Subprograma fantasma ou renomeado. -->
<!-- mystery: BLOCO PLANO VERAO COMENTADO — L100-L110 contêm lógica COMENTADA de correção do "Plano Verão" (01/1989-01/1991, multiplicadores 2.7500 e 1.4289, transição Cruzado→Cruzeiro). Marcado "NAO REMOVER (HISTORICO)". Não executa, mas documenta magic numbers monetários de um período de hiperinflação. Decidir se é regra morta ou se precisa ser reativada para correções muito antigas. -->
<!-- mystery: DEFLAÇÃO DESCARTADA — a regra 7 só grava correção quando a diferença é positiva. Em meses de IPCA negativo (deflação, ex.: alguns meses de 2017) o acumulado pode reduzir o valor, mas o programa nunca registra valor menor. Não está claro se isso é regra de negócio (benefício não cai) ou bug. -->

> **Observação de cobertura temporal:** a tabela IPCA está congelada em 2010-2012 (3 anos efetivamente carregados de 10 posições declaradas), apesar do comentário "ULTIMA CARGA: 2014". Correções para competências de 2013 em diante retornam índice 1.0 (nenhuma correção) sem erro. Risco **ALTO** de correções silenciosamente zeradas.

## Exemplo de linha bem preenchida

| ID     | Regra de Negócio                                                                        | Programa Fonte                                   | Campos DDM                                                               | Nível de Risco | Notas                                      |
| ------ | --------------------------------------------------------------------------------------- | ------------------------------------------------ | ------------------------------------------------------------------------ | -------------- | ------------------------------------------ |
| BR-013 | Desconto total não pode exceder 30% do valor bruto, exceto descontos judiciais (tipo J) | `01-arqueologia/legado-sifap/natural-programs/CALCDSCT.NSN#L142-L148` | `PAGAMENTO.VLR-BRUTO`, `PAGAMENTO.VLR-TOTAL-DSCT`, `PAGAMENTO.TIPO-DSCT` | CRÍTICO        | Regra financeira. Tipo 'J' = exceção legal |

## Regras por Categoria

### Cálculos Financeiros

<!-- Liste aqui as regras relacionadas a cálculos de valores, benefícios, etc. -->

### Validações de Status

<!-- Liste aqui as regras de transição de status (A, S, C, I, D) -->

### Regras de Autorização

<!-- Liste aqui as regras de quem pode fazer o quê -->

### Regras de Negócio Temporais

<!-- Liste aqui regras com prazos, datas-limite, períodos -->

## Resumo Estatístico

- Total de regras encontradas: \_\_\_
- Regras críticas: \_\_\_
- Regras com duplicação: \_\_\_
- Regras sem documentação (escondidas): \_\_\_

---

### Continuar a leitura

<table width="100%">
<tr>
<td width="50%" valign="top" align="left">
<sub><strong>← ANTERIOR</strong></sub><br/>
<a href="GUIDE.md"><strong>GUIDE do Estágio 1</strong></a><br/>
<sub>Passo a passo do estágio.</sub>
</td>
<td width="50%" valign="top" align="right">
<sub><strong>PRÓXIMO →</strong></sub><br/>
<a href="dependency-map.md"><strong>dependency-map.md</strong></a><br/>
<sub>Mapa de quem chama quem.</sub>
</td>
</tr>
</table>

<sub>↑ <a href="README.md">Voltar ao Kit PT-BR</a></sub>

