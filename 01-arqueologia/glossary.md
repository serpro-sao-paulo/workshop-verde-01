<!-- markdownlint-disable MD013 MD025 MD026 MD028 MD029 MD034 MD040 MD051 MD060 -->

# Glossário do SIFAP Legado

![ESTÁGIO 01 Arqueologia](https://img.shields.io/badge/ESTÁGIO-01%20Arqueologia-F25022?style=for-the-badge) ![TIPO Worksheet](https://img.shields.io/badge/TIPO-Worksheet-1A1A1A?style=for-the-badge) ![PREENCHA Durante S1](https://img.shields.io/badge/PREENCHA-Durante%20S1-737373?style=for-the-badge)

> 🗺 **Você está aqui:** [Kit PT-BR](../README.md) → [Estágio 1](README.md) → **glossary**

> **Para quem é isto?** Este é um **artefato preenchido pelo time** durante o Estágio 1 (Arqueologia).
>
> **O que você terá ao final do estágio:**
>
> 1. Este documento totalmente preenchido com os dados reais do legado SIFAP
> 2. Rastreabilidade para `01-arqueologia/legado-sifap/` (programas `.NSN` e DDMs)
> 3. Base de evidência usada nas EARS do Estágio 2 (`source_legacy:`)
>
> 📘 **Guia passo a passo:** [`GUIDE.md`](GUIDE.md).


> Preencha esta tabela com todos os termos, abreviações e siglas encontrados no código Natural/Adabas.
> **Meta: no mínimo 30 termos.**

## Por que isso importa

Sistemas legados têm vocabulário próprio que ninguém documenta em lugar nenhum — só está no nome das variáveis. Se o time do Estágio 2 não souber o que `DSCT`, `BENF`, `PE` ou `CTC` significam, vai escrever uma spec sobre o que ele _acha_ que isso significa. Glossário é o que evita esse desencontro.

## Como preencher

- **Termo**: a abreviação ou sigla exatamente como aparece no código
- **Expansão**: o significado completo do termo
- **Programa**: em qual arquivo `.NSN` ou `.ddm` o termo foi encontrado
- **Contexto**: breve explicação de como/onde o termo é usado

## Dica de extração

Prompt útil no Copilot Chat (cole o conteúdo de 2–3 arquivos `.NSN` no chat antes):

> _"Liste todas as abreviações e siglas usadas neste código Natural. Para cada uma, sugira a expansão e marque com 'CONFIRMADO' ou 'HIPÓTESE'."_

## Termos encontrados

> Extraídos da leitura dos 15 programas `.NSN` em [`legado-sifap/natural-programs/`](legado-sifap/natural-programs/) (comentários + constantes). Marcação: **CONFIRMADO** (expansão evidente no código/comentário) · **HIPÓTESE** (inferida, validar com especialista).

### Acrônimo do sistema e domínio de negócio

| #   | Termo            | Expansão                                                  | Programa                          | Contexto                                                                                                            |
| --- | ---------------- | -------------------------------------------------------- | --------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| 1   | `SIFAP`          | Sistema de Fiscalização e Administração de Pagamentos    | Todos (cabeçalho)                 | **CONFIRMADO**. Nome do sistema legado, presente no comentário `SISTEMA:` de todos os programas.                    |
| 2   | `BENF` / `BENEF` | Beneficiário                                              | `CALCBENF`, `CADBENEF`, `VALBENEF`| **CONFIRMADO**. Pessoa que recebe o pagamento de um programa social. Tabela `BENEFICIARIO`.                         |
| 3   | `PGT` / `PGTO`   | Pagamento                                                 | `BATCHPGT`, `RELPGT`, `NUM-PAGTO` | **CONFIRMADO**. Registro de pagamento mensal gerado por beneficiário. Tabela `PAGAMENTO`.                           |
| 4   | `DSCT`           | Desconto                                                  | `CALCDSCT`                        | **CONFIRMADO**. Dedução aplicada sobre o valor bruto (contribuição, imposto, judicial, sindical, pensão, admin).    |
| 5   | `PROG`           | Programa (social)                                         | `CADPROG`, `COD-PROGRAMA`         | **CONFIRMADO**. Programa social ao qual o beneficiário está vinculado. Tabela `PROGRAMA-SOCIAL`.                    |
| 6   | `ELEG`           | Elegibilidade                                             | `VALELEG`, `COD-ELEGIBILIDADE`    | **CONFIRMADO**. Verificação de requisitos (idade, renda, status, docs) do beneficiário para um programa.            |
| 7   | `CORR`           | Correção (monetária retroativa)                          | `CALCCORR`, `VLR-CORRECAO`        | **CONFIRMADO**. Recálculo de pagamentos passados pela variação do IPCA.                                             |
| 8   | `AUDIT`          | Auditoria (trilha de eventos)                            | `RELAUDIT`, `SEQ-AUDIT`           | **CONFIRMADO**. Registro de eventos do sistema (inclusão, alteração, conciliação...). Tabela `AUDITORIA`.           |
| 9   | `CON`            | Conciliação (bancária)                                    | `BATCHCON`, `ACAO='CO'`           | **CONFIRMADO**. Confronto entre pagamentos do SIFAP e o retorno bancário CNAB 240.                                  |
| 10  | `DEPEND` / `DEP` | Dependente                                                | `CADDEPEND`, `NUM-DEPENDENTES`    | **CONFIRMADO**. Pessoa vinculada ao beneficiário titular (filho, cônjuge, irmão, outro).                            |

### Valores, fatores e cálculo

| #   | Termo          | Expansão                                          | Programa                       | Contexto                                                                                                        |
| --- | -------------- | ------------------------------------------------- | ------------------------------ | --------------------------------------------------------------------------------------------------------------- |
| 11  | `VLR`          | Valor                                             | Todos os de cálculo            | **CONFIRMADO**. Prefixo de campos monetários: `VLR-BRUTO`, `VLR-LIQUIDO`, `VLR-DESCONTO`, `VLR-ABONO`.           |
| 12  | `FATOR-REG`    | Fator Regional                                    | `CALCBENF`, `BATCHPGT`         | **CONFIRMADO**. Multiplicador por região (tabela de 27 posições, 1,00 a 1,40). Comentários mapeiam para UFs.    |
| 13  | `FATOR-FAM`    | Fator Familiar                                    | `CALCBENF`, `BATCHPGT`         | **CONFIRMADO**. Multiplicador conforme número de dependentes.                                                   |
| 14  | `FATOR-RND`    | Fator Renda                                       | `CALCBENF`, `BATCHPGT`         | **CONFIRMADO**. Multiplicador conforme faixa de renda familiar (5 faixas: 300 a 9999,99).                       |
| 15  | `FATOR-REAJ`   | Fator de Reajuste                                 | `CALCBENF`, `CADPROG`          | **CONFIRMADO**. Percentual de reajuste do programa aplicado ao valor do benefício.                              |
| 16  | `FATOR-K`      | Fator de Correção "K" (constante 0,347215)        | `CADPROG`                      | **HIPÓTESE**. Ajusta o valor-base na inclusão de programa: `1.00 + (FATOR-REAJ * 0.347215)`. Origem do `K` obscura. |
| 17  | `ABONO`        | Abono Natalino                                    | `CALCBENF`, `BATCHPGT`         | **CONFIRMADO**. Adicional de 15% pago em dezembro para programas tipo `A` (assistencial).                       |
| 18  | `13O`          | Décimo Terceiro Salário                           | `CALCBENF`, `BATCHPGT`         | **CONFIRMADO**. Gratificação paga em dezembro (`#MES = 12`), fórmula diferenciada.                              |
| 19  | `COMPETENCIA`  | Competência (mês/ano de referência — `AAAAMM`)    | Todos de pagamento             | **CONFIRMADO**. Período do pagamento no formato numérico ano+mês.                                               |
| 20  | `ALIQ`         | Alíquota                                          | `CALCDSCT`, `#ALIQ-CONTRIB`    | **CONFIRMADO**. Percentual de contribuição social por faixa de valor bruto (3% a 9%).                           |
| 21  | `CONTRIB`      | Contribuição (social)                             | `CALCDSCT`, `CALCBENF`         | **CONFIRMADO**. Desconto compulsório calculado por faixa de valor bruto.                                        |
| 22  | `IPCA`         | Índice de Preços ao Consumidor Amplo              | `CALCCORR`, `#IPCA-ANO`        | **CONFIRMADO**. Índice mensal usado para correção retroativa. Tabela carregada até 2014.                        |
| 23  | `IND`          | Índice / Indicador                                | `CALCCORR`                     | **CONFIRMADO**. `IND-ACUM` (índice acumulado) e `IND-CORRIGIDO` (flag S/N se o pagamento já foi corrigido).     |

### Identificadores e documentos

| #   | Termo  | Expansão                                              | Programa                       | Contexto                                                                                                   |
| --- | ------ | ----------------------------------------------------- | ------------------------------ | ---------------------------------------------------------------------------------------------------------- |
| 24  | `CPF`  | Cadastro de Pessoa Física                             | Todos                          | **CONFIRMADO**. Chave do beneficiário (N11). Validado por algoritmo Módulo 11.                             |
| 25  | `NIS`  | Número de Identificação Social                        | `CADBENEF`, `CONSBENF`         | **CONFIRMADO**. Identificador social alternativo (N11), usado em buscas e em elegibilidade tipo `R`.       |
| 26  | `RG`   | Registro Geral (carteira de identidade)               | `CADBENEF`, `VALDOCS`          | **CONFIRMADO**. Documento de identidade; validação exige mínimo de 5 caracteres.                           |
| 27  | `CTPS` | Carteira de Trabalho e Previdência Social             | `VALDOCS`                      | **CONFIRMADO**. Documento complementar capturado na validação de documentos.                               |
| 28  | `CEP`  | Código de Endereçamento Postal                        | `CADBENEF`, `CONSBENF`         | **CONFIRMADO**. Campo de endereço (N8).                                                                    |
| 29  | `UF`   | Unidade Federativa (estado)                           | `VALBENEF`, vários             | **CONFIRMADO**. Sigla do estado (A2); validada contra tabela de 27 UFs.                                    |
| 30  | `DV1` / `DV2` | Dígito Verificador (1º e 2º)                   | `CADBENEF`, `VALBENEF`, `VALDOCS` | **CONFIRMADO**. Dígitos de controle do CPF calculados por Módulo 11.                                    |
| 31  | `MOD 11` | Módulo 11 (algoritmo de dígito verificador)         | `CADBENEF`, `VALBENEF`         | **CONFIRMADO**. Algoritmo de validação de CPF (soma ponderada, resto da divisão por 11).                   |

### Códigos de status e tipo (valores de domínio)

| #   | Termo         | Expansão                                                                 | Programa             | Contexto                                                                                              |
| --- | ------------- | ------------------------------------------------------------------------ | -------------------- | ----------------------------------------------------------------------------------------------------- |
| 32  | `STATUS` (benef) | A=Ativo · S=Suspenso · C=Cancelado · I=Inativo · D=Desligado          | `CONSBENF`, `VALBENEF` | **CONFIRMADO**. Situação cadastral do beneficiário. `S` também é setado para idade > 75 (idoso) em `CADBENEF`. |
| 33  | `STATUS-PGTO` | G=Gerado · P=Pago · C=Cancelado · D=Devolvido · E=Estornado              | `RELPGT`, `BATCHREL` | **CONFIRMADO**. Situação do pagamento no ciclo de vida.                                                |
| 34  | `TIPO-PGTO`   | N=Normal · D=Décimo · T=Terceiro                                         | `CALCBENF`, `RELPGT` | **CONFIRMADO**. Natureza do pagamento. (Comentário cita "TERCEIRO" — ambíguo vs. décimo terceiro.)    |
| 35  | `TIPO` (prog) | A=Assistencial · P=Previdenciário · T=Trabalho                          | `CADPROG`, `VALELEG` | **CONFIRMADO**. Categoria do programa social; define regras de elegibilidade e abono.                 |
| 36  | `TIPO-DSCT`   | C=Contrib · I=Imposto · J=Judicial · S=Sindical · P=Pensão · A=Admin     | `CALCDSCT`           | **CONFIRMADO**. Natureza do desconto. Judicial não respeita o teto de 30%.                            |
| 37  | `ACAO` (audit)| IN=Inclusão · AL=Alteração · CO=Conciliação · CN=Consulta · DV=Divergência · EX=Exclusão | `RELAUDIT`, `BATCHCON` | **CONFIRMADO**. Tipo de evento na trilha de auditoria. `EX` (exclusão) é filtrada do relatório.   |
| 38  | `PARENTESCO`  | FI=Filho · CO=Cônjuge · IR=Irmão · OU=Outro                              | `CADDEPEND`          | **CONFIRMADO**. Vínculo do dependente com o titular.                                                   |
| 39  | `OPER`        | I=Inclusão · A=Alteração · C=Consulta                                    | `CADBENEF`, `CADPROG`| **CONFIRMADO**. Operação solicitada na tela de manutenção.                                            |
| 40  | `COD-RETORNO` | 00=OK/Pago · 01=Devolvido · 02=Estornado                                 | `BATCHCON`           | **CONFIRMADO**. Código de retorno bancário do arquivo CNAB que atualiza o status do pagamento.        |

### Infraestrutura, bancário e mainframe

| #   | Termo       | Expansão                                                        | Programa             | Contexto                                                                                                  |
| --- | ----------- | --------------------------------------------------------------- | -------------------- | --------------------------------------------------------------------------------------------------------- |
| 41  | `CNAB 240`  | Centro Nacional de Automação Bancária — layout 240 posições     | `BATCHCON`           | **CONFIRMADO**. Formato do arquivo de retorno bancário (Banco do Brasil) processado na conciliação.       |
| 42  | `BB`        | Banco do Brasil                                                 | `BATCHCON`           | **CONFIRMADO**. Banco padrão do layout CNAB 240. `Banco Real` (cód. 356) é integração descontinuada.      |
| 43  | `PE`        | Periodic Group (grupo periódico Adabas)                         | `CADDEPEND`, `CALCDSCT` | **CONFIRMADO**. Estrutura Adabas que repete um conjunto de campos (dependentes, descontos). Limite 5 dependentes. |
| 44  | `MU`        | Multiple-value field (campo multivalorado Adabas)               | (DDMs Adabas)        | **HIPÓTESE**. Conceito Adabas correlato ao `PE`; confirmar ocorrências nos `.ddm` em `adabas-ddms/`.      |
| 45  | `ARQ 150/155/160/170` | Arquivos Adabas: 150=Beneficiário · 155=Programa · 160=Pagamento · 170=Auditoria | Cabeçalhos `OBJETIVO:` | **HIPÓTESE**. Números de arquivo Adabas citados nos comentários; mapear contra `.ddm`.            |
| 46  | `3270`      | Terminal IBM 3270 (tela online)                                 | `CONSBENF`           | **CONFIRMADO**. Tela online via `MAP`; consulta de beneficiário.                                          |
| 47  | `MAP`       | Map (definição de layout de tela Natural)                       | `CONSBENF`           | **CONFIRMADO**. `INPUT USING MAP 'CONSBENF-M01'` — layout de tela 3270.                                   |
| 48  | `SEQ`       | Sequencial (número de sequência)                                | `BATCHCON`, `BATCHPGT` | **CONFIRMADO**. `SEQ-AUDIT`, `SEQ-PGTO` — chaves incrementais.                                           |
| 49  | `QTD`       | Quantidade (contador)                                           | Todos os batch/rel   | **CONFIRMADO**. Prefixo de contadores: `QTD-LIDOS`, `QTD-GERADOS`, `QTD-ERROS`...                          |
| 50  | `DT` / `HR` | Data / Hora                                                     | Todos                | **CONFIRMADO**. Prefixos de campos temporais: `DT-NASCIMENTO`, `DT-GERACAO`, `HR-EVENTO`. Datas em `AAAAMMDD`. |
| 51  | `TOT` / `SUB` | Total / Subtotal                                              | `RELPGT`, `BATCHREL` | **CONFIRMADO**. Acumuladores de totalização em relatórios.                                                 |
| 52  | `*DATN` / `*TIMN` | Variáveis de sistema Natural: data/hora numéricas        | `BATCHCON`, vários   | **CONFIRMADO**. `*DATN` = data atual (AAAAMMDD); `*TIMN` = hora atual.                                     |

> Adicione mais linhas conforme necessário. Não se limite a 30!

## Exemplo de linha bem preenchida

| #   | Termo  | Expansão | Programa                        | Contexto                                                                                                         |
| --- | ------ | -------- | ------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| 1   | `DSCT` | Desconto | `CALCDSCT.NSN`, `PAGAMENTO.ddm` | Tipo de dedução aplicada sobre valor bruto do pagamento. Tipos: 'J' (judicial), 'I' (imposto), 'T' (trabalhista) |

## Observações

- Anote aqui qualquer padrão de nomenclatura que o time identificou:
  - **Prefixos de programa por verbo de domínio:** `CAD*` = cadastro/manutenção · `CALC*` = cálculo · `VAL*` = validação · `CONS*` = consulta online · `REL*` = relatório · `BATCH*` = processamento em lote.
  - **Variáveis locais** começam com `#`; **views Adabas** terminam em `-V`; **subrotinas** usam nomes-verbo com hífen (`VALIDA-CPF`, `DET-FAIXA-RENDA`).
  - **Campos monetários:** prefixo `VLR-`; **datas:** prefixo `DT-` (formato `AAAAMMDD`); **horas:** prefixo `HR-` (`HHMMSS`); **contadores:** prefixo `QTD-`; **fatores de cálculo:** prefixo `FATOR-`.
  - **Códigos de domínio** são letras únicas comentadas inline no `DEFINE DATA` (ex.: `STATUS A1`, `TIPO A1`, `TIPO-DSCT A1`).
- Convenções de prefixo/sufixo encontradas:
  - Sufixo `-V` → view de tabela Adabas (`BENEFICIARIO-V`, `PAGAMENTO-V`, `PROGRAMA-V`, `AUDITORIA-V`).
  - Grupos periódicos Adabas (`PE`) para coleções: `DEPENDENTES` (máx. 5) e `DESCONTOS`.
- Termos ambíguos que precisam de validação com especialista:
  - `FATOR-K` / constante `0.347215` em `CADPROG` — origem e justificativa do fator desconhecidas (**HIPÓTESE**).
  - `TIPO-PGTO` valor `T=TERCEIRO` (`RELPGT`) vs. `D=DÉCIMO` em `CALCBENF` — possível redundância/inconsistência de domínio.
  - `ARQ 150/155/160/170` — números de arquivo Adabas citados nos comentários; mapear contra os `.ddm` em [`adabas-ddms/`](legado-sifap/adabas-ddms/).
  - `MU` (campo multivalorado Adabas) citado por convenção, mas não localizado nos `.NSN`; verificar nos DDMs.
  - **Inconsistência conhecida** na máscara de CPF (`CONSBENF`, subrotina `MASCARA-CPF`): às vezes mostra os 3 primeiros dígitos em vez dos últimos — comentário pede não corrigir sem aprovação da auditoria.
  - **Regra opaca:** região `99` em `VALELEG` libera elegibilidade automaticamente ("internacional/diplomático").

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
<a href="business-rules-catalog.md"><strong>business-rules-catalog.md</strong></a><br/>
<sub>Catálogo de regras.</sub>
</td>
</tr>
</table>

<sub>↑ <a href="README.md">Voltar ao Kit PT-BR</a></sub>

