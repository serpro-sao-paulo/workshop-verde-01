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

| #   | Termo | Expansão | Programa | Contexto |
| --- | ----- | -------- | -------- | -------- |
| 1   | DSCT | Desconto | CALCDSCT.NSN | Campo TIPO-DSCT e fluxo de cálculo de descontos; confirmado por constantes J/P/I/S/A. |
| 2   | BENF | Beneficiário | CALCBENF.NSN, CONSBENF.NSN | Prefixo recorrente em consultas e cálculo de benefício; forma abreviada de beneficiário. |
| 3   | PGTO | Pagamento | CALCBENF.NSN, BATCHPGT.NSN, RELPGT.NSN | Código de tipo de pagamento N/D/T e status G/P/C/D/E. |
| 4   | COMP | Competência | CALCBENF.NSN, RELPGT.NSN | Competência no formato AAAAMM para cálculo e relatórios. |
| 5   | OPER | Operação | CADBENEF.NSN, CADPROG.NSN | Chaves de operação I/A/C para inclusão, alteração e consulta/cadastro. |
| 6   | UF | Unidade da Federação | VALBENEF.NSN, CONSBENF.NSN | Siglas estaduais (AC, AL, PE etc.) usadas em validação e impressão de município/UF. |
| 7   | NIS | Número de Identificação Social | CONSBENF.NSN, VALELEG.NSN | Tipo de busca C=CPF N=NIS; também em motivo de inelegibilidade (NIS não cadastrado). |
| 8   | CPF | Cadastro de Pessoas Físicas | CONSBENF.NSN, VALBENEF.NSN, VALDOCS.NSN | Máscara e validação de dígito verificador do documento. |
| 9   | RG | Registro Geral | VALDOCS.NSN | Validação de formato de RG com mensagens de erro dedicadas. |
| 10  | CNAB | Centro Nacional de Automação Bancária (layout de arquivo) | BATCHCON.NSN | Conciliação de retorno bancário com tipo de registro e códigos de ocorrência. |
| 11  | CO | Conciliação | BATCHCON.NSN, RELAUDIT.NSN, CADDEPEND.NSN | Código de ação de auditoria CO=CONCILIACAO; também aparece em parentesco CO=CONJUGE. |
| 12  | DV | Divergência | BATCHCON.NSN, RELAUDIT.NSN | Código de auditoria DV para divergências de conciliação. |
| 13  | STS | Status | BATCHREL.NSN | Vetor #NOME-STS traduz G/P/C/D/E em descrições de status. |
| 14  | REG | Região | BATCHREL.NSN, CALCBENF.NSN | Tabela de nome de região e fator regional de cálculo (#TAB-REG). |
| 15  | VLR | Valor | CONSBENF.NSN, RELPGT.NSN, CALCBENF.NSN | Campos VLR-BRUTO, VLR-DESCONTO e VLR-LIQUIDO em cálculo e relatório. |
| 16  | MSG | Mensagem | CADBENEF.NSN, VALBENEF.NSN, VALDOCS.NSN | Variáveis #MSG/#MSG-ERRO armazenam mensagens de validação. |
| 17  | QTD | Quantidade | VALBENEF.NSN, VALDOCS.NSN, VALELEG.NSN | Contadores de erros e motivos (#QTD-ERROS, #QTD-MOT). |
| 18  | MOT | Motivo | VALELEG.NSN | Array #MOTIVO recebe causas de inelegibilidade. |
| 19  | ELEG | Elegibilidade | VALELEG.NSN | Código de elegibilidade (#COD-ELEG) e validações associadas. |
| 20  | DOCS | Documentos | VALDOCS.NSN, VALELEG.NSN | Validação documental e indicador #DOCS-OK para regra de elegibilidade. |
| 21  | IPCA | Índice Nacional de Preços ao Consumidor Amplo | CALCCORR.NSN | Série mensal #IPCA-ANO (JAN..DEZ) para correção retroativa. |
| 22  | IND | Indicador | CALCCORR.NSN | Campo IND-CORRIGIDO marcado com S para pagamentos já corrigidos. |
| 23  | DT | Data | RELAUDIT.NSN | Filtros e cabeçalho de período com #DT-INI e #DT-FIM. |
| 24  | HR | Hora | RELAUDIT.NSN | Campo #HR-FORMAT exibido no relatório de trilha de auditoria. |
| 25  | ACAO | Ação | RELAUDIT.NSN | Códigos IN/AL/CO/CN/DV traduzidos para descrição textual. |
| 26  | FI | Filho | CADDEPEND.NSN | Código de parentesco FI=FILHO. |
| 27  | IR | Irmão | CADDEPEND.NSN | Código de parentesco IR=IRMAO. |
| 28  | OU | Outro | CADDEPEND.NSN | Código de parentesco OU aceito na validação de dependente. |
| 29  | PROG | Programa | CADPROG.NSN, VALELEG.NSN | STATUS-PROG e validações de atividade do programa social. |
| 30  | PREF-ESP | Prefixo especial | VALDOCS.NSN | Tabela #PREF-ESP com códigos de prefixo de RG considerados especiais. |
| 31  | DIAS-MES | Dias do mês | VALBENEF.NSN | Ajuste de fevereiro para 29 dias (ano bissexto). |
| 32  | TIPO-PGTO | Tipo de pagamento | CALCBENF.NSN, RELPGT.NSN | Domínio N=normal, D=décimo, T=terceiro. |
| 33  | PE | Periodic Group (Adabas) | [a validar em DDM] | Abreviação críptica comum em Adabas; incluir para validação cruzando arquivos .ddm. |
| 34  | MU | Multiple Value (Adabas) | [a validar em DDM] | Abreviação críptica comum em Adabas; incluir para validação cruzando arquivos .ddm. |
| 35  | CTC | [hipótese] Centro de Tratamento/Controle de Conciliação | [a confirmar no legado] | Termo solicitado para rastreio; não apareceu explicitamente nos comentários/constantes dos 15 .NSN. |

> Adicione mais linhas conforme necessário. Não se limite a 30!

## Exemplo de linha bem preenchida

| #   | Termo  | Expansão | Programa                        | Contexto                                                                                                         |
| --- | ------ | -------- | ------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| 1   | `DSCT` | Desconto | `CALCDSCT.NSN`, `PAGAMENTO.ddm` | Tipo de dedução aplicada sobre valor bruto do pagamento. Tipos: 'J' (judicial), 'I' (imposto), 'T' (trabalhista) |

## Observações

- Anote aqui qualquer padrão de nomenclatura que o time identificou:
- Convenções de prefixo/sufixo encontradas:
- Termos ambíguos que precisam de validação com especialista:

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

