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

## Regras de CADBENEF.NSN

> Extração sistemática por `@archaeologist-agent` em 2026-06-10 (primeira passada).
> Arquivo analisado: `01-arqueologia/legado-sifap/natural-programs/CADBENEF.NSN` (272 linhas, lido de cima a baixo).
> Cross-reference: `01-arqueologia/legado-sifap/legacy-docs/REGRAS-NEGOCIO-2012.md` (seção 1) e Manual 2008 (3.2.1).
> **Objetivo da leitura:** caçar a "região 99 (bypass do Roberto)" (RN-005) e o `FATOR-K`. **Resultado:** nenhum dos dois está em CADBENEF — não há tratamento especial de `COD-REGIAO = 99` nem variável `FATOR-K`. O `COD-REGIAO` é gravado sem validação. Ambos os mistérios permanecem abertos.

| #  | Declaração da Regra | Candidato EARS | Fonte | Classificação | Notas |
|----|---------------------|----------------|-------|---------------|-------|
| 1  | Se a operação não for `'I'` (inclusão) nem `'A'` (alteração), o sistema deve recusar a operação. | Unwanted | `CADBENEF.NSN#L99-L103` | Inferida | Seletor de modo. Não documentado explicitamente. |
| 2  | Se o CPF for zero, o sistema deve recusar (CPF obrigatório). | Unwanted | `CADBENEF.NSN#L106-L110` | Confirmada | RN seção 1.1: CPF obrigatório e chave de identificação. Campo `BENEFICIARIO.CPF`. |
| 3  | Se o CPF não passar na validação módulo 11, o sistema deve recusar (dígito verificador incorreto). | Unwanted | `CADBENEF.NSN#L113-L118` (subrotina `#L222-L270`) | Confirmada | RN seção 1.1 (validação de CPF) e `COMO-LER-NATURAL.md`. Algoritmo mód-11 completo em `VALIDA-CPF`. |
| 4  | Se o nome estiver em branco, o sistema deve recusar (nome obrigatório). | Unwanted | `CADBENEF.NSN#L120-L124` | Confirmada | RN seção 1.1: nome é campo obrigatório. Campo `BENEFICIARIO.NOME`. |
| 5  | Se a data de nascimento for zero, o sistema deve recusar (data obrigatória). | Unwanted | `CADBENEF.NSN#L126-L130` | Confirmada | RN-006: data de nascimento obrigatória. Campo `BENEFICIARIO.DT-NASCIMENTO`. |
| 6  | Se o sexo não for `'M'` nem `'F'`, o sistema deve recusar (sexo inválido). | Unwanted | `CADBENEF.NSN#L132-L136` | Inferida | Validação de domínio. Não documentada. Campo `BENEFICIARIO.SEXO`. |
| 7  | Em inclusão, se o beneficiário (CPF) já existir, o sistema deve recusar (já cadastrado). | Unwanted | `CADBENEF.NSN#L145-L149` | Confirmada | RN seção 1.1: não permite duplicidade de CPF. Requisito de unicidade. |
| 8  | Em alteração, se o beneficiário não existir, o sistema deve recusar (não encontrado). | Unwanted | `CADBENEF.NSN#L151-L155` | Inferida | Pré-condição de alteração. Não documentada. |
| 9  | Em inclusão, o status inicial do beneficiário deve ser `'A'` (ativo). | Event-driven | `CADBENEF.NSN#L163-L166` | Confirmada (parcial) | RN seção 1.1: beneficiário entra ativo. Campo `BENEFICIARIO.STATUS`. |
| 10 | Quando a idade do beneficiário for maior que 75 anos, o sistema deve definir o status como `'S'`. | Event-driven | `CADBENEF.NSN#L168-L171` | Mistério | <!-- mystery: cabeçalho cita "AJUSTE STATUS IDOSO" (10/01/2011); doc trata 'A'=ativo, mas o significado de status 'S' não está documentado (suspenso? sênior?). Pior: 'S' SOBRESCREVE 'A' mesmo em ALTERAÇÃO, e o BATCHPGT só paga status 'A' (regra 2 de BATCHPGT) → beneficiários >75 anos podem ser EXCLUÍDOS do pagamento. Limiar 75 é magic number. --> |
| 11 | Quando a operação for inclusão, o sistema deve gravar (STORE) o novo beneficiário com data de cadastro e atualização = data corrente. | Event-driven | `CADBENEF.NSN#L177-L218` (`DECIDE` VALUE 'I') | Inferida | Persistência. `DT-CADASTRO` e `DT-ATUALIZACAO` = `*DATN`. |
| 12 | Quando a operação for alteração, o sistema deve atualizar (UPDATE) apenas um subconjunto de campos, NÃO regravando CPF, data de nascimento, sexo, programa, data de cadastro, região e NIS. | Event-driven | `CADBENEF.NSN#L177-L218` (`DECIDE` VALUE 'A') | Mistério | <!-- mystery: a alteração NÃO regrava COD-PROGRAMA, COD-REGIAO, NIS, CPF, DT-NASCIMENTO nem SEXO, embora o INPUT os colete do operador. Campos imutáveis por design ou bug de campo esquecido? A doc não detalha quais campos são alteráveis. --> |

### Subrotina VALIDA-CPF (algoritmo módulo 11) — detalhamento

| #  | Declaração da Regra | Candidato EARS | Fonte | Classificação | Notas |
|----|---------------------|----------------|-------|---------------|-------|
| 13 | O dígito verificador é calculado pela soma ponderada dos dígitos; se o resto da divisão por 11 for menor que 2, o DV é 0, caso contrário DV = 11 − resto. Se não bater com o dígito informado, o CPF é inválido. | Ubiquitous | `CADBENEF.NSN#L222-L270` | Confirmada | Algoritmo padrão mód-11 de CPF (dois dígitos verificadores). Confirma RN seção 1.1. |

### Mistérios e achados transversais em CADBENEF.NSN

<!-- mystery: REGIÃO 99 AUSENTE — RN-005 cita região 99 como "bypass do Roberto" reservado para uso interno, sugerindo consultar CADBENEF. Após leitura completa, CADBENEF NÃO valida COD-REGIAO de forma alguma — grava o valor cru informado pelo operador. Não há lógica de bypass aqui. O mistério da região 99 permanece SEM resolução; pode estar em VALELEG ou ser apenas convenção de dados. -->
<!-- mystery: FATOR-K AUSENTE (5º programa sem achado) — nenhuma variável/constante FATOR-K em CADBENEF. Confirma que não está em nenhum dos programas lidos até agora (BATCHPGT, CALCBENF, CALCDSCT, CALCCORR, CADBENEF). -->
<!-- mystery: STATUS 'S' vs PAGAMENTO — beneficiários com idade > 75 recebem status 'S'. Como BATCHPGT só processa status 'A', isso pode silenciosamente excluir idosos do pagamento mensal. Verificar com VALELEG/VALBENEF se 'S' tem tratamento próprio. Possível regra de negócio crítica ou bug grave. -->
<!-- mystery: SEM AUDITORIA — CADBENEF altera dados cadastrais (incl. status) mas NÃO grava no DDM AUDITORIA nem chama LOGAUDIT (subprograma citado no README). Inclusões/alterações não deixam trilha, divergindo da existência do DDM AUDITORIA. -->
<!-- mystery: NIS SEM VALIDAÇÃO — o campo NIS é coletado e gravado mas nunca validado (existe subprograma VALNISN no README, não chamado aqui). -->
<!-- mystery: IDADE MÍNIMA AUSENTE — RN-006 menciona idade mínima de 16 anos, mas CADBENEF NÃO valida idade mínima; só verifica data de nascimento ≠ 0 e o limiar superior de 75. -->

> **Risco de migração (STATUS IDOSO):** a regra 10 (idade > 75 → status 'S') combinada com a regra 2 de BATCHPGT (só paga status 'A') é o achado mais crítico de CADBENEF. Pode estar suspendendo pagamentos de beneficiários muito idosos sem documentação. Elevar para investigação prioritária com VALELEG. Registrar como risco **CRÍTICO**.

## Regras de VALELEG.NSN

> Extração sistemática por `@archaeologist-agent` em 2026-06-10 (primeira passada).
> Arquivo analisado: `01-arqueologia/legado-sifap/natural-programs/VALELEG.NSN` (244 linhas, lido de cima a baixo).
> Autor: José Ferreira dos Santos (03/02/1999). Alterações: 28/07/2004 (novas regras eleg), 11/10/2009 (ajuste faixa etária), **05/04/2013 — Anderson Lima — "INC REGIAO 99"**.
> Cross-reference: `REGRAS-NEGOCIO-2012.md` seção 4 (Elegibilidade) e seção 6/7 (pendências, incl. "Bypass de elegibilidade por região 99").
> **🎯 DOIS MISTÉRIOS RESOLVIDOS NESTE ARQUIVO:** (1) o "bypass da região 99" — o mecanismo está aqui (não no CADBENEF, como a doc sugeria); (2) a conexão do status `'S'` (idoso, vindo do CADBENEF) com a elegibilidade — `'S'` = SUSPENSO = INELEGÍVEL aqui.

| #  | Declaração da Regra | Candidato EARS | Fonte | Classificação | Notas |
|----|---------------------|----------------|-------|---------------|-------|
| 1  | Se o beneficiário (CPF) não for encontrado no arquivo, o sistema deve recusar (não encontrado) e encerrar. | Unwanted | `VALELEG.NSN#L81-L84` | Inferida | Pré-condição. Lê BENEFICIARIO (ARQ 150). |
| 2  | Se o programa social não for encontrado, o sistema deve recusar e encerrar. | Unwanted | `VALELEG.NSN#L94-L97` | Inferida | Lê PROGRAMA-SOCIAL (ARQ 155). |
| 3  | Se o status do programa não for `'A'` (ativo), o sistema deve recusar (programa inativo) e encerrar. | Unwanted | `VALELEG.NSN#L99-L102` | Confirmada | RN-003: vínculo obrigatório com programa social com `PS-IN-ATIVO = 'S'`. Campo `PROGRAMA-SOCIAL.STATUS-PROG`. |
| 4  | Quando a região do beneficiário for 99, o sistema deve marcar o beneficiário como ELEGÍVEL e PULAR todas as demais validações (faixa etária, renda, status, tipo de programa, elegibilidade específica). | Event-driven | `VALELEG.NSN#L107-L111` | Mistério | <!-- mystery: ESTE É O "BYPASS DO ROBERTO" (RN-005). Comentário no código diz "REGIAO 99 - INTERNACIONAL/DIPLOMATICO", mas a doc o chama de "bypass do Roberto" sem explicação. O mecanismo agora está claro (ESCAPE ROUTINE força elegível), mas: (a) NÃO há verificação de QUEM pode atribuir região 99 (CADBENEF grava COD-REGIAO cru, sem validação); (b) NÃO há log/auditoria do bypass; (c) introduzido em 2013 por Anderson Lima. Risco de FRAUDE CRÍTICO: qualquer cadastro com região 99 passa por TODAS as regras de elegibilidade. --> |
| 5  | Se o status do beneficiário não for `'A'`: status `'S'` → inelegível (suspenso); `'C'` ou `'D'` → inelegível (cancelado/desligado); `'I'` → inelegível (inativo). | Unwanted | `VALELEG.NSN#L116-L134` | Confirmada (parcial) | RN seção 4.2: situação cadastral ativa (`BN-CD-SIT = 'A'`). **`'S'` = SUSPENSO aqui** — ver achado transversal sobre status idoso. |
| 6  | Quando o programa definir idade mínima (> 0) e a idade do beneficiário for inferior, o sistema deve marcar inelegível (idade abaixo do mínimo). | Unwanted | `VALELEG.NSN#L139-L145` | Confirmada | Ajuste de faixa etária (alteração 2009). Campo `PROGRAMA-SOCIAL.IDADE-MIN`. Idade derivada de `DT-NASCIMENTO`. |
| 7  | Quando o programa definir idade máxima (> 0) e a idade do beneficiário for superior, o sistema deve marcar inelegível (idade acima do máximo). | Unwanted | `VALELEG.NSN#L146-L152` | Confirmada | Campo `PROGRAMA-SOCIAL.IDADE-MAX`. |
| 8  | Quando o programa definir renda máxima (> 0) e a renda familiar for superior, o sistema deve marcar inelegível (renda acima do teto). | Unwanted | `VALELEG.NSN#L157-L163` | Confirmada | RN seção 4.2: renda per capita dentro das faixas. Campo `PROGRAMA-SOCIAL.RENDA-MAX`, `BENEFICIARIO.RENDA-FAMILIAR`. <!-- mystery: doc fala em renda PER CAPITA (BN-VL-RENDA-PC, RN-018), mas aqui usa RENDA-FAMILIAR total, sem dividir por membros. Métricas diferentes. --> |
| 9  | Quando o tipo do programa for `'A'` (assistencial): se a renda > 600 e houver menos de 1 dependente → inelegível; e se a documentação não estiver `'S'` (OK) → inelegível. | Event-driven | `VALELEG.NSN#L169-L182` | Inferida | Magic number 600.00 sem suporte documental. `DOCUMENTOS-OK`, `NUM-DEPENDENTES`. |
| 10 | Quando o tipo do programa for `'P'` (previdenciário) e a idade for menor que 60, o sistema deve marcar inelegível. | Event-driven | `VALELEG.NSN#L183-L189` | Inferida | Magic number 60. Sem suporte documental direto. |
| 11 | Quando o tipo do programa for `'T'` (trabalho) e a idade estiver fora da faixa 16–65, o sistema deve marcar inelegível. | Event-driven | `VALELEG.NSN#L190-L196` | Confirmada (parcial) | RN-006: idade mínima 16 anos. Limite superior 65 é magic number não documentado. |
| 12 | Quando o tipo do programa não for `'A'`, `'P'` nem `'T'`, o sistema deve marcar inelegível (tipo desconhecido). | Unwanted | `VALELEG.NSN#L197-L200` | Inferida | Branch `NONE` do DECIDE. |
| 13 | Quando o programa tiver código de elegibilidade específico (≠ branco), o sistema deve executar verificações adicionais. | Optional | `VALELEG.NSN#L206-L208` | Inferida | `PROGRAMA-SOCIAL.COD-ELEGIBILIDADE` (A5). Chama subrotina `VERIF-ELEG-ESPECIFICA`. |
| 14 | Na verificação específica, se a 1ª posição do código de elegibilidade for `'R'` e o NIS for zero, o sistema deve marcar inelegível (NIS não cadastrado). | Unwanted | `VALELEG.NSN#L226-L233` | Confirmada (parcial) | RN-001: NIS/NIT ativo obrigatório. Aqui só verifica NIS ≠ 0, não chama VALNISN. |
| 15 | Na verificação específica, se a 2ª posição do código de elegibilidade for `'D'` e não houver dependentes, o sistema deve marcar inelegível (programa requer dependentes). | Unwanted | `VALELEG.NSN#L234-L241` | Inferida | `NUM-DEPENDENTES`. Código de elegibilidade posicional não documentado. |

### Mistérios e achados transversais em VALELEG.NSN

<!-- mystery: REGIÃO 99 = BYPASS TOTAL (RESOLVIDO o mecanismo, ABERTA a governança) — L107-L111 fazem ESCAPE ROUTINE marcando ELEGIVEL=TRUE assim que COD-REGIAO=99, PULANDO todas as 12 validações seguintes. Comentário no código: "INTERNACIONAL/DIPLOMATICO". Doc: "bypass do Roberto". Como CADBENEF grava COD-REGIAO sem validação alguma, qualquer operador pode marcar um beneficiário como região 99 e torná-lo elegível incondicionalmente, sem log. Introduzido 05/04/2013 (Anderson Lima). Risco de FRAUDE CRÍTICO. -->
<!-- mystery: CONEXÃO STATUS 'S' (IDOSO) — CADBENEF (regra 10) atribui status 'S' a beneficiários com idade > 75. VALELEG (regra 5) trata 'S' como SUSPENSO → INELEGÍVEL, e BATCHPGT (regra 2) só paga status 'A'. Cadeia confirmada: beneficiário > 75 anos vira 'S' no cadastro → fica INELEGÍVEL e SEM PAGAMENTO. O cabeçalho do CADBENEF chamava isso de "AJUSTE STATUS IDOSO" (2011). Intencional (suspensão de idosos?) ou bug grave? Achado CRÍTICO que precisa de validação de negócio urgente. -->
<!-- mystery: DISCREPÂNCIA DE TAMANHO/ESCOPO — a doc seção 4.2 afirma que VALELEG tem "aproximadamente 1.200 linhas de código com lógica condicional complexa" e lista regras que NÃO existem neste arquivo de 244 linhas: validação de dados bancários, limite de no máximo 2 programas simultâneos (BN-QT-PROG), data de última atualização < 24 meses (BN-DT-ULT-ATUAL), e bloqueio por ocorrência de auditoria tipo 'B' no DDM AUDITORIA. Nenhuma dessas está presente. Ou a doc descreve uma versão diferente/antiga, ou houve remoção de regras, ou o levantamento de 2012 estava incorreto. -->
<!-- mystery: RENDA FAMILIAR vs PER CAPITA — regra 8 usa RENDA-FAMILIAR total contra RENDA-MAX, mas a doc (RN-018) determina atribuição por renda PER CAPITA (BN-VL-RENDA-PC). Pode estar reprovando/aprovando beneficiários pela métrica errada. -->
<!-- mystery: SEM VALIDAÇÃO DE NIS REAL — regra 14 só checa NIS ≠ 0; não invoca o subprograma VALNISN (citado no README e RN-001) para validar o dígito do NIS. -->

> **Achado de maior impacto do Estágio 1 até aqui:** a cadeia **CADBENEF → VALELEG → BATCHPGT** revela duas regras de altíssimo risco operando juntas e sem documentação clara: (1) o **bypass da região 99** (fraude de elegibilidade) e (2) a **suspensão automática de beneficiários > 75 anos** via status 'S'. Ambas devem entrar no `mysteries-found.md` com prioridade máxima e ser questões abertas obrigatórias para o Estágio 2.

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

