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

