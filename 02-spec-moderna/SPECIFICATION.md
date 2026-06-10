<!-- markdownlint-disable MD013 MD025 MD026 MD028 MD029 MD033 MD034 MD040 MD051 MD060 -->

# SPECIFICATION — SIFAP Moderno

![ESTÁGIO 02 Spec Moderna](https://img.shields.io/badge/ESTÁGIO-02%20Spec%20Moderna-7FBA00?style=for-the-badge) ![TIPO Especificação](https://img.shields.io/badge/TIPO-Especificação%20EARS-1A1A1A?style=for-the-badge) ![NOTAÇÃO EARS](https://img.shields.io/badge/NOTAÇÃO-EARS-8661C5?style=for-the-badge)

> **Autor:** `@architect-agent` (Requirements Engineer protagonista) · **Data:** 2026-06-10
> **Entrada:** [business-rules-catalog.md](../01-arqueologia/business-rules-catalog.md) · [bounded-contexts.md](bounded-contexts.md) · [mysteries-found.md](../01-arqueologia/mysteries-found.md)
> **Notação:** EARS (Easy Approach to Requirements Syntax) — 6 padrões.
> **Regra dura de rastreabilidade:** todo `REQ-NNN` carrega uma linha `source_legacy:` apontando para `01-arqueologia/legado-sifap/natural-programs/*.NSN` ou `[GREENFIELD] + justificativa`. O job de CI `legacy-traceability` rejeita PRs que violam isso.

---

## Escopo desta especificação

Esta spec contém **somente requisitos derivados de regras `Confirmadas`** do catálogo do Estágio 1 (regras que correspondem à documentação legada), mais um pequeno conjunto de capacidades `[GREENFIELD]` explicitamente novas exigidas pela arquitetura-alvo (REST API, auditoria obrigatória orientada a eventos).

**Regras bloqueadas (HARD GATE).** As regras financeiras e de elegibilidade afetadas pelos **6 mistérios Critical (`blocks-stage-2`)** — fórmula de cálculo (MYS-011/012), regra de desconto/contribuição (MYS-014), teto judicial ordem-dependente (MYS-006), bypass região 99 (MYS-008) e suspensão de idosos via status `'S'` (MYS-001) — **não foram convertidas em requisitos**. Elas aparecem em [Open Questions](#open-questions-ainda-não-são-requisitos) e devem ser validadas com o facilitador/negócio antes de ganharem um REQ-ID.

**Convenção de padrões EARS:**

| Padrão | Forma | Uso |
| --- | --- | --- |
| Ubiquitous | "O sistema deverá [ação]." | Sempre se aplica, sem trigger. |
| Event-driven | "Quando [evento], o sistema deverá [ação]." | Acionado por evento. |
| State-driven | "Enquanto [estado], o sistema deverá [ação]." | Ativo durante um estado. |
| Optional | "Onde [condição], o sistema deverá [ação]." | Só quando a condição vale. |
| Unwanted | "Se [condição], então o sistema deverá [ação]." | Tratamento de erro/rejeição. |
| Complex | "Enquanto [estado], quando [evento], o sistema deverá [ação]." | Combinação. |

---

## Bounded Context: Gestão de Beneficiários e Programas

> DDMs: `BENEFICIARIO` (FNR 150) · `PROGRAMA-SOCIAL` (FNR 151). Inclui o subdomínio de Elegibilidade ([bounded-contexts.md](bounded-contexts.md) §1).

### REQ-001: Rejeição de CPF ausente no cadastro

- **EARS Pattern:** Unwanted
- **Declaração:** Se o CPF informado no cadastro de beneficiário for zero ou ausente, então o sistema deverá rejeitar a operação e retornar erro de campo obrigatório.
- **source_legacy:** `01-arqueologia/legado-sifap/natural-programs/CADBENEF.NSN#L106-L110`
- **Source Rule:** Regra #2 (CADBENEF), Confirmada — RN seção 1.1 (CPF obrigatório e chave de identificação)
- **Bounded Context:** Gestão de Beneficiários e Programas
- **Critérios de Aceite:**
  - [ ] Given um payload de cadastro com `cpf = 0` ou vazio, when a inclusão é submetida, then o sistema retorna `400 Bad Request` com mensagem "CPF obrigatório" e nenhum beneficiário é persistido.
  - [ ] Given um payload de cadastro com `cpf` válido e não-zero, when a inclusão é submetida, then a validação de CPF ausente não é acionada.

### REQ-002: Validação de dígito verificador do CPF (módulo 11)

- **EARS Pattern:** Unwanted
- **Declaração:** Se o CPF informado não passar na validação de dígito verificador pelo algoritmo módulo 11, então o sistema deverá rejeitar a operação e retornar erro de CPF inválido.
- **source_legacy:** `01-arqueologia/legado-sifap/natural-programs/CADBENEF.NSN#L113-L118`, `01-arqueologia/legado-sifap/natural-programs/CADBENEF.NSN#L222-L270`
- **Source Rule:** Regras #3 e #13 (CADBENEF), Confirmadas — RN seção 1.1 (validação de CPF), algoritmo mód-11 completo na subrotina `VALIDA-CPF`
- **Bounded Context:** Gestão de Beneficiários e Programas
- **Critérios de Aceite:**
  - [ ] Given um CPF com dígitos verificadores incorretos, when submetido, then o sistema retorna erro "CPF inválido" e não persiste.
  - [ ] Given um CPF cujo resto da soma ponderada por 11 é menor que 2 (DV esperado = 0), when o DV informado é 0, then o CPF é aceito pela validação.
  - [ ] Given um CPF cujo resto é maior ou igual a 2 (DV esperado = 11 − resto), when o DV informado difere, then o CPF é rejeitado.

### REQ-003: Nome do beneficiário obrigatório

- **EARS Pattern:** Unwanted
- **Declaração:** Se o nome do beneficiário estiver em branco no cadastro, então o sistema deverá rejeitar a operação e retornar erro de campo obrigatório.
- **source_legacy:** `01-arqueologia/legado-sifap/natural-programs/CADBENEF.NSN#L120-L124`
- **Source Rule:** Regra #4 (CADBENEF), Confirmada — RN seção 1.1 (nome obrigatório). Campo `BENEFICIARIO.NOME`
- **Bounded Context:** Gestão de Beneficiários e Programas
- **Critérios de Aceite:**
  - [ ] Given um payload de cadastro com `nome` vazio ou só espaços, when submetido, then o sistema retorna `400` com "Nome obrigatório" e não persiste.
  - [ ] Given um payload com `nome` não-vazio, when submetido, then a validação de nome obrigatório não é acionada.

### REQ-004: Data de nascimento obrigatória

- **EARS Pattern:** Unwanted
- **Declaração:** Se a data de nascimento do beneficiário for zero ou ausente no cadastro, então o sistema deverá rejeitar a operação e retornar erro de campo obrigatório.
- **source_legacy:** `01-arqueologia/legado-sifap/natural-programs/CADBENEF.NSN#L126-L130`
- **Source Rule:** Regra #5 (CADBENEF), Confirmada — RN-006 (data de nascimento obrigatória). Campo `BENEFICIARIO.DT-NASCIMENTO`
- **Bounded Context:** Gestão de Beneficiários e Programas
- **Critérios de Aceite:**
  - [ ] Given um payload com `dataNascimento` ausente ou igual a zero, when submetido, then o sistema retorna `400` com "Data de nascimento obrigatória" e não persiste.
  - [ ] Given um payload com `dataNascimento` válida, when submetido, then a validação de data obrigatória não é acionada.

### REQ-005: Unicidade de CPF na inclusão

- **EARS Pattern:** Unwanted
- **Declaração:** Quando a operação for inclusão e já existir um beneficiário com o mesmo CPF, então o sistema deverá rejeitar a operação e retornar conflito de cadastro duplicado.
- **source_legacy:** `01-arqueologia/legado-sifap/natural-programs/CADBENEF.NSN#L145-L149`
- **Source Rule:** Regra #7 (CADBENEF), Confirmada — RN seção 1.1 (não permite duplicidade de CPF; requisito de unicidade)
- **Bounded Context:** Gestão de Beneficiários e Programas
- **Critérios de Aceite:**
  - [ ] Given um CPF já cadastrado, when uma nova inclusão com o mesmo CPF é submetida, then o sistema retorna `409 Conflict` com "Beneficiário já cadastrado" e não cria duplicata.
  - [ ] Given um CPF inexistente, when a inclusão é submetida, then o beneficiário é criado.

### REQ-006: Status inicial ativo na inclusão

- **EARS Pattern:** Event-driven
- **Declaração:** Quando um beneficiário for incluído, o sistema deverá definir seu status inicial como `'A'` (ativo).
- **source_legacy:** `01-arqueologia/legado-sifap/natural-programs/CADBENEF.NSN#L163-L166`
- **Source Rule:** Regra #9 (CADBENEF), Confirmada (parcial) — RN seção 1.1 (beneficiário entra ativo). Campo `BENEFICIARIO.STATUS`
- **Bounded Context:** Gestão de Beneficiários e Programas
- **Critérios de Aceite:**
  - [ ] Given uma inclusão válida, when o beneficiário é persistido, then `status = 'A'`.
  - [ ] Given uma inclusão válida, when o beneficiário é persistido, then a data de cadastro e a data de atualização são definidas como a data corrente.

> ⚠️ **Nota de gate:** a sobrescrita de status para `'S'` quando idade > 75 (CADBENEF #10) **não é requisito** — está bloqueada em MYS-001 (ver [Open Questions](#open-questions-ainda-não-são-requisitos)). REQ-006 cobre apenas o status inicial confirmado.

### REQ-007: Vínculo obrigatório com programa social ativo

- **EARS Pattern:** Unwanted
- **Declaração:** Se o programa social vinculado ao beneficiário não estiver com status ativo (`'A'`), então o sistema deverá marcar o beneficiário como inelegível e recusar a avaliação de elegibilidade.
- **source_legacy:** `01-arqueologia/legado-sifap/natural-programs/VALELEG.NSN#L99-L102`
- **Source Rule:** Regra #3 (VALELEG), Confirmada — RN-003 (vínculo obrigatório com programa social com `PS-IN-ATIVO = 'S'`). Campo `PROGRAMA-SOCIAL.STATUS-PROG`
- **Bounded Context:** Gestão de Beneficiários e Programas (subdomínio Elegibilidade)
- **Critérios de Aceite:**
  - [ ] Given um programa social com `status = 'A'`, when a elegibilidade é avaliada, then a verificação de programa ativo passa.
  - [ ] Given um programa social com `status ≠ 'A'`, when a elegibilidade é avaliada, then o resultado é inelegível com motivo "programa inativo".

### REQ-008: Faixa etária mínima e máxima por programa

- **EARS Pattern:** Optional
- **Declaração:** Onde o programa social definir idade mínima ou máxima (> 0), o sistema deverá marcar o beneficiário como inelegível quando sua idade estiver fora da faixa definida.
- **source_legacy:** `01-arqueologia/legado-sifap/natural-programs/VALELEG.NSN#L139-L145`, `01-arqueologia/legado-sifap/natural-programs/VALELEG.NSN#L146-L152`
- **Source Rule:** Regras #6 e #7 (VALELEG), Confirmadas — ajuste de faixa etária (alteração 2009). Campos `PROGRAMA-SOCIAL.IDADE-MIN`, `PROGRAMA-SOCIAL.IDADE-MAX`; idade derivada de `BENEFICIARIO.DT-NASCIMENTO`
- **Bounded Context:** Gestão de Beneficiários e Programas (subdomínio Elegibilidade)
- **Critérios de Aceite:**
  - [ ] Given um programa com `idadeMin = 18` e um beneficiário de 16 anos, when a elegibilidade é avaliada, then o resultado é inelegível com motivo "idade abaixo do mínimo".
  - [ ] Given um programa com `idadeMax = 65` e um beneficiário de 70 anos, when a elegibilidade é avaliada, then o resultado é inelegível com motivo "idade acima do máximo".
  - [ ] Given um programa com `idadeMin = 0` e `idadeMax = 0`, when a elegibilidade é avaliada, then nenhuma restrição etária é aplicada.

### REQ-009: Verificação de NIS para programas que exigem NIS

- **EARS Pattern:** Unwanted
- **Declaração:** Quando o código de elegibilidade do programa exigir NIS (1ª posição igual a `'R'`) e o NIS do beneficiário for zero, então o sistema deverá marcar o beneficiário como inelegível por NIS não cadastrado.
- **source_legacy:** `01-arqueologia/legado-sifap/natural-programs/VALELEG.NSN#L226-L233`
- **Source Rule:** Regra #14 (VALELEG), Confirmada (parcial) — RN-001 (NIS/NIT ativo obrigatório). Campo `PROGRAMA-SOCIAL.COD-ELEGIBILIDADE` posição 1
- **Bounded Context:** Gestão de Beneficiários e Programas (subdomínio Elegibilidade)
- **Critérios de Aceite:**
  - [ ] Given um programa cujo código de elegibilidade começa com `'R'` e um beneficiário com `nis = 0`, when a elegibilidade é avaliada, then o resultado é inelegível com motivo "NIS não cadastrado".
  - [ ] Given o mesmo programa e um beneficiário com `nis ≠ 0`, when a elegibilidade é avaliada, then a verificação de NIS passa.

> ⚠️ **Nota de gate:** REQ-009 reproduz apenas a checagem `NIS ≠ 0` confirmada no código. A validação completa do dígito do NIS (subprograma `VALNISN`) permanece em aberto (MYS-019, severidade Medium) e **não** é requisito aqui.

---

## Bounded Context: Cálculo e Pagamento

> DDM: `PAGAMENTO` (FNR 152). Único contexto com escrita em `PAGAMENTO` ([bounded-contexts.md](bounded-contexts.md) §2).
>
> ⚠️ **HARD GATE — fórmula financeira bloqueada.** A fórmula de cálculo do benefício bruto (fatores regional/familiar/renda/idade e ordem de aplicação do reajuste) e a regra de desconto/contribuição estão sob os mistérios Critical **MYS-011, MYS-012 e MYS-014** e **não** são requisitos. Os requisitos abaixo cobrem apenas regras de cálculo/pagamento **Confirmadas** e independentes da fórmula em disputa (pré-condições de pagamento, seleção de faixa de renda, arredondamento, teto de desconto e exceção judicial, correção monetária).

### REQ-010: Pagamento somente para beneficiários ativos

- **EARS Pattern:** State-driven
- **Declaração:** Enquanto o beneficiário não estiver com status `'A'` (ativo), o sistema deverá ignorá-lo no ciclo de pagamento mensal e não gerar pagamento para ele.
- **source_legacy:** `01-arqueologia/legado-sifap/natural-programs/BATCHPGT.NSN#L195-L198`, `01-arqueologia/legado-sifap/natural-programs/CALCBENF.NSN#L160-L163`
- **Source Rule:** Regra #2 (BATCHPGT) e Regra #3 (CALCBENF), Confirmadas — RN seção 5.1 (somente beneficiários ativos `BN-CD-SIT = 'A'` são processados). Campo `BENEFICIARIO.STATUS`
- **Bounded Context:** Cálculo e Pagamento
- **Critérios de Aceite:**
  - [ ] Given um beneficiário com `status = 'A'`, when o ciclo mensal roda, then um pagamento é gerado para ele (sujeito às demais regras).
  - [ ] Given um beneficiário com `status ≠ 'A'`, when o ciclo mensal roda, then nenhum pagamento é gerado e o beneficiário é contabilizado como ignorado.

### REQ-011: Seleção de faixa de renda

- **EARS Pattern:** Event-driven
- **Declaração:** Quando o sistema determinar o fator de renda de um beneficiário, ele deverá selecionar a primeira faixa cujo limite superior seja maior ou igual à renda familiar declarada.
- **source_legacy:** `01-arqueologia/legado-sifap/natural-programs/CALCBENF.NSN#L304-L308`, `01-arqueologia/legado-sifap/natural-programs/BATCHPGT.NSN#L367-L375`
- **Source Rule:** Regra #7 (CALCBENF) e Regra #8 (BATCHPGT), Confirmadas — RN-018 ("primeira faixa cujo limite superior seja maior ou igual à renda declarada")
- **Bounded Context:** Cálculo e Pagamento
- **Critérios de Aceite:**
  - [ ] Given as faixas ordenadas por limite superior crescente, when a renda familiar é informada, then o fator retornado é o da primeira faixa cujo limite superior ≥ renda.
  - [ ] Given uma renda igual exatamente ao limite superior de uma faixa, when o fator é determinado, then essa faixa (limite ≥ renda) é a selecionada.

> ⚠️ **Nota de gate:** os **valores** das faixas de renda (300/600/1000/1500/9999.99 → fatores) e seu uso na fórmula final dependem da resolução de MYS-012/MYS-013 e **não** são fixados aqui. REQ-011 especifica apenas a **regra de seleção** confirmada (primeira faixa com limite ≥ renda). A métrica renda total vs. per capita (MYS-017) está em aberto.

### REQ-012: Arredondamento monetário por truncamento em centavos

- **EARS Pattern:** Ubiquitous
- **Declaração:** O sistema deverá truncar (arredondar para baixo) todos os valores monetários em duas casas decimais (centavos), nunca arredondando para cima.
- **source_legacy:** `01-arqueologia/legado-sifap/natural-programs/BATCHPGT.NSN#L283-L285`, `01-arqueologia/legado-sifap/natural-programs/CALCBENF.NSN#L231-L232`
- **Source Rule:** Regra #11 (BATCHPGT) e Regra #10 (CALCBENF), Confirmadas — RN-014 ("sempre arredondado para baixo em centavos, truncamento")
- **Bounded Context:** Cálculo e Pagamento
- **Critérios de Aceite:**
  - [ ] Given um valor calculado de `123.4567`, when persistido, then o valor armazenado é `123.45` (truncado, não `123.46`).
  - [ ] Given um valor calculado de `99.999`, when persistido, then o valor armazenado é `99.99`.
  - [ ] Given dois caminhos de cálculo distintos para o mesmo valor, when comparados, then ambos aplicam truncamento idêntico (nenhum usa `ROUND`).

### REQ-013: Teto de desconto de 30% do valor bruto

- **EARS Pattern:** Unwanted
- **Declaração:** Se o total de descontos não-judiciais acumulados exceder 30% do valor bruto do pagamento, então o sistema deverá limitar o total de descontos a esse teto de 30% (truncado em centavos).
- **source_legacy:** `01-arqueologia/legado-sifap/natural-programs/CALCDSCT.NSN#L101-L105`
- **Source Rule:** Regra #5 (CALCDSCT), Confirmada — RN-021 ("total de descontos não pode exceder 30% do valor bruto")
- **Bounded Context:** Cálculo e Pagamento
- **Critérios de Aceite:**
  - [ ] Given descontos não-judiciais que somam 35% do bruto, when o teto é aplicado, then o total de descontos é limitado a 30% do bruto.
  - [ ] Given descontos não-judiciais que somam 20% do bruto, when o teto é avaliado, then nenhuma limitação é aplicada.
  - [ ] Given o teto aplicado, when o valor do teto é calculado, then ele é truncado em centavos (consistente com REQ-012).

> ⚠️ **Nota de gate:** a **interação** entre o teto de 30% e o desconto judicial (a ordem-dependência que pode truncar a parcela judicial — MYS-006) **não** é especificada aqui e bloqueia qualquer requisito sobre composição de descontos. REQ-013 cobre apenas o teto confirmado para descontos não-judiciais; REQ-014 cobre a isenção judicial isoladamente. A regra de composição correta é uma [Open Question](#open-questions-ainda-não-são-requisitos).

### REQ-014: Desconto judicial isento do teto de 30%

- **EARS Pattern:** Event-driven
- **Declaração:** Quando um desconto for do tipo `'J'` (judicial), o sistema deverá computá-lo pelo valor fixo cadastrado — ou, na ausência deste, pelo percentual sobre o bruto — sem sujeitá-lo ao teto de 30%.
- **source_legacy:** `01-arqueologia/legado-sifap/natural-programs/CALCDSCT.NSN#L123-L132`
- **Source Rule:** Regra #8 (CALCDSCT), Confirmada — RN-021 nota (exceção judicial ao teto). Campos `TIPO-DSCT='J'`, `NUM-PROCESSO`
- **Bounded Context:** Cálculo e Pagamento
- **Critérios de Aceite:**
  - [ ] Given um desconto tipo `'J'` com valor fixo cadastrado, when o desconto é computado, then o valor fixo é usado integralmente.
  - [ ] Given um desconto tipo `'J'` sem valor fixo, when o desconto é computado, then o percentual sobre o bruto é aplicado.
  - [ ] Given um desconto tipo `'J'`, when o teto de 30% é avaliado, then a parcela judicial não é limitada por esse teto.

### REQ-015: Idempotência mensal do pagamento

- **EARS Pattern:** Unwanted
- **Declaração:** Se já existir um pagamento gerado para o mesmo CPF na mesma competência (mês de referência), então o sistema deverá ignorar o beneficiário no ciclo e não gerar pagamento duplicado.
- **source_legacy:** `01-arqueologia/legado-sifap/natural-programs/BATCHPGT.NSN#L202-L210`
- **Source Rule:** Regra #3 (BATCHPGT), Confirmada (promovida de Inferida — decisão da equipe: controle de idempotência é requisito explícito da migração para evitar pagamento em duplicidade; evidência clara no código via `FIND PAGAMENTO WITH CPF-BENEF`)
- **Bounded Context:** Cálculo e Pagamento
- **Critérios de Aceite:**
  - [ ] Given um pagamento já existente para `cpf` + `competencia`, when o ciclo roda novamente para a mesma competência, then nenhum segundo pagamento é gerado.
  - [ ] Given nenhum pagamento prévio para `cpf` + `competencia`, when o ciclo roda, then um pagamento é gerado.

### REQ-016: Correção monetária por índice acumulado do período

- **EARS Pattern:** Ubiquitous
- **Declaração:** O sistema deverá corrigir o valor bruto de um pagamento multiplicando-o pelo índice acumulado do período de correção, truncando o resultado em centavos.
- **source_legacy:** `01-arqueologia/legado-sifap/natural-programs/CALCCORR.NSN#L150-L156`, `01-arqueologia/legado-sifap/natural-programs/CALCCORR.NSN#L177-L190`
- **Source Rule:** Regra #6 (CALCCORR), Confirmada (parcial) — RN-019 (reajuste por índice). O índice acumulado é o produto de `(1 + índice mensal)` ao longo do período
- **Bounded Context:** Cálculo e Pagamento
- **Critérios de Aceite:**
  - [ ] Given um pagamento e um índice acumulado de `1.05`, when a correção é aplicada, then o valor corrigido = valor bruto × 1.05, truncado em centavos.
  - [ ] Given um período com múltiplos meses, when o índice acumulado é calculado, then ele é o produto de `(1 + índiceMensal)` de cada mês do período.

> ⚠️ **Nota de gate:** REQ-016 especifica a **mecânica** confirmada da correção. A **fonte e a cobertura temporal** do índice (tabela IPCA congelada em 2010-2012 — MYS-015), o descarte de deflação (MYS-028) e o relacionamento com o subprograma fantasma `CALCIDX` (MYS-029) permanecem em aberto e não são fixados aqui.

### REQ-017: Idempotência da correção monetária

- **EARS Pattern:** Unwanted
- **Declaração:** Se um pagamento já estiver marcado como corrigido (`IND-CORRIGIDO = 'S'`), então o sistema deverá ignorá-lo no processo de correção, evitando dupla correção.
- **source_legacy:** `01-arqueologia/legado-sifap/natural-programs/CALCCORR.NSN#L141-L143`
- **Source Rule:** Regra #5 (CALCCORR), Confirmada (promovida de Inferida — decisão da equipe: idempotência da correção é requisito explícito para evitar correção repetida sobre o mesmo pagamento; evidência clara via flag `PAGAMENTO.IND-CORRIGIDO`)
- **Bounded Context:** Cálculo e Pagamento
- **Critérios de Aceite:**
  - [ ] Given um pagamento com `indCorrigido = 'S'`, when o processo de correção roda, then o pagamento é ignorado e seu valor não é alterado.
  - [ ] Given um pagamento com `indCorrigido ≠ 'S'`, when a correção é aplicada com diferença positiva, then `indCorrigido` passa a `'S'`.

---

## Bounded Context: Conciliação e Auditoria

> DDM: `AUDITORIA` (FNR 153). Dono exclusivo da trilha de auditoria ([bounded-contexts.md](bounded-contexts.md) §3).

### REQ-018: Reconciliação do retorno bancário

- **EARS Pattern:** Event-driven
- **Declaração:** Quando um arquivo de retorno bancário (CNAB 240 / Banco do Brasil) for processado, o sistema deverá conciliar cada registro de retorno contra o pagamento emitido correspondente e atualizar o status do pagamento (confirmado ou rejeitado) por meio do comando publicado pelo contexto de Cálculo e Pagamento.
- **source_legacy:** `[GREENFIELD]` — capacidade derivada do programa legado `BATCHCON.NSN` (reconciliação CNAB), porém **reformulada**: o legado escreve `PAGAMENTO` diretamente; a arquitetura-alvo exige atualização via comando publicado (anti-corruption layer — ver [bounded-contexts.md](bounded-contexts.md) §3 e ADR de hub `PAGAMENTO`). Justificativa: preserva a capacidade de negócio existente eliminando a escrita cross-context.
- **Bounded Context:** Conciliação e Auditoria
- **Critérios de Aceite:**
  - [ ] Given um arquivo CNAB com um registro de pagamento confirmado, when o arquivo é processado, then o contexto de Cálculo e Pagamento recebe `confirmPayment(paymentId, bankReturn)` e o status reflete a confirmação.
  - [ ] Given um arquivo CNAB com um registro de pagamento rejeitado, when o arquivo é processado, then `rejectPayment(paymentId, reason)` é invocado com o motivo do retorno.
  - [ ] Given a Conciliação, when um pagamento é atualizado, then a Conciliação nunca escreve diretamente na tabela `PAGAMENTO`.

### REQ-019: Trilha de auditoria obrigatória orientada a eventos

- **EARS Pattern:** Event-driven
- **Declaração:** Quando um evento de domínio relevante for emitido por qualquer contexto (cadastro/alteração de beneficiário, cálculo, emissão, confirmação ou rejeição de pagamento, ou aplicação de override de elegibilidade), o sistema deverá registrar uma entrada imutável na trilha de auditoria contendo a referência da entidade, o ator e o carimbo de tempo.
- **source_legacy:** `[GREENFIELD]` — a arquitetura-alvo torna a auditoria **obrigatória e orientada a eventos**, fechando a lacuna confirmada de que `CADBENEF.NSN` altera dados (incl. status) **sem** gravar em `AUDITORIA` (MYS-018). Justificativa: o DDM `AUDITORIA` existe no legado, mas a trilha não é garantida; o requisito a promove a invariante do sistema.
- **Bounded Context:** Conciliação e Auditoria
- **Critérios de Aceite:**
  - [ ] Given um evento `BeneficiaryRegistered`, when emitido, then uma entrada de auditoria é criada com `entityRef`, `actor` e `timestamp`.
  - [ ] Given um evento `PaymentIssued`, when emitido, then uma entrada de auditoria é criada e não pode ser posteriormente alterada nem removida.
  - [ ] Given qualquer alteração de status de beneficiário, when persistida, then existe uma entrada de auditoria correspondente (nenhuma alteração sem trilha).

---

## Bounded Context: Consulta e Relatórios

> Read side (CQRS-lite), sem dados próprios ([bounded-contexts.md](bounded-contexts.md) §4).

### REQ-020: Relatórios reutilizam valores já calculados (sem recálculo)

- **EARS Pattern:** Ubiquitous
- **Declaração:** O sistema deverá produzir consultas e relatórios de pagamento exclusivamente a partir dos valores já calculados e publicados pelo contexto de Cálculo e Pagamento, nunca reimplementando a matemática monetária do benefício.
- **source_legacy:** `[GREENFIELD]` — corrige a divergência confirmada de arredondamento do legado, em que `BATCHREL.NSN` usa `ROUND` enquanto os cálculos usam `TRUNCATE`, produzindo relatórios que divergem do valor pago (MYS-005 / INC-004). Justificativa: a arquitetura-alvo proíbe recálculo no read side ([bounded-contexts.md](bounded-contexts.md) §4).
- **Bounded Context:** Consulta e Relatórios
- **Critérios de Aceite:**
  - [ ] Given um pagamento com valor líquido `X` publicado por Cálculo e Pagamento, when um relatório o exibe, then o valor exibido é exatamente `X` (sem reprocessamento).
  - [ ] Given a geração de qualquer relatório de pagamento, when auditado, then nenhum código de cálculo de benefício/desconto é executado no contexto de Consulta e Relatórios.

### REQ-021: Endpoint REST de consulta de beneficiário

- **EARS Pattern:** Event-driven
- **Declaração:** Quando uma requisição `GET /api/v1/beneficiarios/{id}` autenticada for recebida, o sistema deverá retornar os dados cadastrais do beneficiário em formato JSON, mascarando o CPF em qualquer log.
- **source_legacy:** `[GREENFIELD]` — capacidade nova de consulta online via REST API (a arquitetura-alvo expõe REST; o legado é Natural/Adabas 3270). Deriva da capacidade de consulta de `CONSBENF.NSN`, modernizada para HTTP/JSON. Justificativa: nova superfície de integração exigida pela stack-alvo.
- **Bounded Context:** Consulta e Relatórios
- **Critérios de Aceite:**
  - [ ] Given um `id` de beneficiário existente e requisição autenticada, when `GET /api/v1/beneficiarios/{id}` é chamado, then o sistema retorna `200 OK` com o DTO do beneficiário.
  - [ ] Given um `id` inexistente, when chamado, then o sistema retorna `404 Not Found`.
  - [ ] Given qualquer resposta logada, when o CPF aparece em log, then ele é mascarado (ex.: `***.***.***-**`).

---

## Open Questions (ainda NÃO são requisitos)

> Itens classificados como **Critical `blocks-stage-2`** em [mysteries-found.md](../01-arqueologia/mysteries-found.md). Nenhum vira requisito até ser validado com o facilitador/negócio. Cada um lista a informação necessária para resolvê-lo.

| Mistério | Questão aberta | Bloqueia | Informação necessária para resolver |
| --- | --- | --- | --- |
| **MYS-001** | A suspensão automática de beneficiários com idade > 75 (status `'S'`) é regra de negócio intencional ou bug? `'S'` exclui o idoso do pagamento. | Requisito de status idoso; REQ de elegibilidade por status; regra de sobrescrita de status no cadastro. | Decisão de negócio: idosos > 75 devem ser suspensos? Significado canônico de `'S'`. Tratamento alternativo em `VALBENEF.NSN`. |
| **MYS-006** | O teto de 30% deve realmente truncar a parcela judicial quando processada antes de um desconto não-judicial? A proteção do judicial é ordem-dependente. | Requisito de composição/ordem de descontos (além do REQ-013/REQ-014 isolados). | Decisão de negócio: regra correta de composição quando há judicial + não-judicial e o teto é atingido. |
| **MYS-008** | Quem pode atribuir região 99 (bypass total de elegibilidade)? Há controle e auditoria? Hoje qualquer operador grava `COD-REGIAO = 99` sem validação nem log. | Requisito de governança da região 99 / override de elegibilidade (`EligibilityOverrideApplied`). | Definição de autorização (quem aprova), exigência de auditoria do override, e se o bypass deve sequer existir no sistema novo. **Risco de fraude.** |
| **MYS-011** | Existe uma única fonte de verdade para o cálculo, ou a duplicação BATCHPGT/CALCBENF deve ser preservada? A doc diz que o batch invoca o subprograma, mas o código duplica. | Todo requisito de fórmula de cálculo do benefício bruto. | Decisão arquitetural (ADR de unificação financeira) + confirmação de que a fórmula unificada é a correta. |
| **MYS-012** | A fórmula é multiplicativa (código) ou aditiva (RN-013)? O reajuste incide sobre o total (código) ou só sobre o valor-base (RN-020)? | Todo requisito de cálculo do benefício bruto, fatores e reajuste. | **Decisão de negócio obrigatória:** qual fórmula é a verdadeira. Afeta o valor pago a todos os beneficiários. |
| **MYS-014** | Qual é a regra canônica de desconto/contribuição: 3% fixo acima de R$500 (CALCBENF/BATCHPGT) ou contribuição progressiva 3/5/7/9% (CALCDSCT)? | Todo requisito de contribuição social e desconto que não seja o teto/judicial já confirmados. | Decisão de negócio: regra única de contribuição/desconto, com as faixas e percentuais oficiais. |

> **Questões abertas relacionadas (não-Critical) que também afetam requisitos financeiros/de elegibilidade:** métrica de renda total vs. per capita (MYS-017), valores de faixa de renda e fator familiar (MYS-013), fórmula do 13º (MYS-004), cobertura temporal do índice IPCA (MYS-015), validação real do NIS (MYS-019), idade mínima de 16 anos (MYS-020). Documentadas em [mysteries-found.md](../01-arqueologia/mysteries-found.md); resolver antes de ampliar os contextos de Cálculo e de Elegibilidade.

---

## Matriz de Rastreabilidade

| REQ-ID | EARS Pattern | source_legacy | Source Rule # | Source File | Bounded Context |
| --- | --- | --- | --- | --- | --- |
| REQ-001 | Unwanted | `CADBENEF.NSN#L106-L110` | #2 (CADBENEF) | CADBENEF.NSN | Gestão de Beneficiários e Programas |
| REQ-002 | Unwanted | `CADBENEF.NSN#L113-L118`, `#L222-L270` | #3, #13 (CADBENEF) | CADBENEF.NSN | Gestão de Beneficiários e Programas |
| REQ-003 | Unwanted | `CADBENEF.NSN#L120-L124` | #4 (CADBENEF) | CADBENEF.NSN | Gestão de Beneficiários e Programas |
| REQ-004 | Unwanted | `CADBENEF.NSN#L126-L130` | #5 (CADBENEF) | CADBENEF.NSN | Gestão de Beneficiários e Programas |
| REQ-005 | Unwanted | `CADBENEF.NSN#L145-L149` | #7 (CADBENEF) | CADBENEF.NSN | Gestão de Beneficiários e Programas |
| REQ-006 | Event-driven | `CADBENEF.NSN#L163-L166` | #9 (CADBENEF) | CADBENEF.NSN | Gestão de Beneficiários e Programas |
| REQ-007 | Unwanted | `VALELEG.NSN#L99-L102` | #3 (VALELEG) | VALELEG.NSN | Gestão de Beneficiários e Programas (Elegibilidade) |
| REQ-008 | Optional | `VALELEG.NSN#L139-L145`, `#L146-L152` | #6, #7 (VALELEG) | VALELEG.NSN | Gestão de Beneficiários e Programas (Elegibilidade) |
| REQ-009 | Unwanted | `VALELEG.NSN#L226-L233` | #14 (VALELEG) | VALELEG.NSN | Gestão de Beneficiários e Programas (Elegibilidade) |
| REQ-010 | State-driven | `BATCHPGT.NSN#L195-L198`, `CALCBENF.NSN#L160-L163` | #2 (BATCHPGT), #3 (CALCBENF) | BATCHPGT.NSN, CALCBENF.NSN | Cálculo e Pagamento |
| REQ-011 | Event-driven | `CALCBENF.NSN#L304-L308`, `BATCHPGT.NSN#L367-L375` | #7 (CALCBENF), #8 (BATCHPGT) | CALCBENF.NSN, BATCHPGT.NSN | Cálculo e Pagamento |
| REQ-012 | Ubiquitous | `BATCHPGT.NSN#L283-L285`, `CALCBENF.NSN#L231-L232` | #11 (BATCHPGT), #10 (CALCBENF) | BATCHPGT.NSN, CALCBENF.NSN | Cálculo e Pagamento |
| REQ-013 | Unwanted | `CALCDSCT.NSN#L101-L105` | #5 (CALCDSCT) | CALCDSCT.NSN | Cálculo e Pagamento |
| REQ-014 | Event-driven | `CALCDSCT.NSN#L123-L132` | #8 (CALCDSCT) | CALCDSCT.NSN | Cálculo e Pagamento |
| REQ-015 | Unwanted | `BATCHPGT.NSN#L202-L210` | #3 (BATCHPGT) | BATCHPGT.NSN | Cálculo e Pagamento |
| REQ-016 | Ubiquitous | `CALCCORR.NSN#L150-L156`, `#L177-L190` | #6 (CALCCORR) | CALCCORR.NSN | Cálculo e Pagamento |
| REQ-017 | Unwanted | `CALCCORR.NSN#L141-L143` | #5 (CALCCORR) | CALCCORR.NSN | Cálculo e Pagamento |
| REQ-018 | Event-driven | `[GREENFIELD]` (deriva de BATCHCON.NSN, reformulado) | — (ACL) | BATCHCON.NSN (ref.) | Conciliação e Auditoria |
| REQ-019 | Event-driven | `[GREENFIELD]` (fecha MYS-018) | — (invariante) | CADBENEF.NSN / AUDITORIA.ddm (ref.) | Conciliação e Auditoria |
| REQ-020 | Ubiquitous | `[GREENFIELD]` (corrige MYS-005 / INC-004) | — (CQRS-lite) | BATCHREL.NSN (ref.) | Consulta e Relatórios |
| REQ-021 | Event-driven | `[GREENFIELD]` (REST API nova) | — (nova superfície) | CONSBENF.NSN (ref.) | Consulta e Relatórios |

---

## Definição de Pronto — checklist

- [x] Pelo menos 10 requisitos EARS com REQ-IDs únicos (21 requisitos).
- [x] Todo requisito tem uma linha `source_legacy:` apontando para `.NSN` ou `[GREENFIELD] + justificativa`.
- [x] Todo requisito cita sua regra-fonte e arquivo legado (ou justificativa GREENFIELD).
- [x] Todo requisito tem pelo menos 2 critérios de aceitação testáveis.
- [x] Requisitos agrupados por bounded context (4 contextos).
- [x] Mistérios aparecem somente em "Open Questions", nunca como requisitos.
- [x] Matriz de rastreabilidade conecta todo REQ à sua regra-fonte.
- [x] Os 6 mistérios Critical (`blocks-stage-2`) permanecem como gates não convertidos em requisitos.

---

### Continuar a leitura

<table width="100%">
<tr>
<td width="50%" valign="top" align="left">
<sub><strong>← ANTERIOR</strong></sub><br/>
<a href="bounded-contexts.md"><strong>bounded-contexts.md</strong></a><br/>
<sub>Mapa de bounded contexts.</sub>
</td>
<td width="50%" valign="top" align="right">
<sub><strong>PRÓXIMO →</strong></sub><br/>
<a href="../09-cheat-sheets/spec-kit-workflow.md"><strong>spec-kit-workflow.md</strong></a><br/>
<sub>/speckit.clarify → /speckit.plan.</sub>
</td>
</tr>
</table>

<sub>↑ <a href="bounded-contexts.md">Voltar ao mapa de contextos</a></sub>
