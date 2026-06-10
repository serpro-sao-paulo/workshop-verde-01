# Inventário Legado — [Nome da Equipe]

> **Data:** 2026-06-10
> **Gerado por:** `@archaeologist-agent` — Estágio 1, primeira passada
> **Nota:** Este é um inventário inicial baseado apenas em estrutura de pastas e nomes de arquivos. Deve ser revisado e complementado conforme a equipe lê os arquivos individuais.

---

## Estrutura de Pastas

```
01-arqueologia/legado-sifap/
├── README.md
├── COMO-LER-NATURAL.md
├── natural-programs/
│   ├── README.md
│   ├── BATCHCON.NSN
│   ├── BATCHPGT.NSN
│   ├── BATCHREL.NSN
│   ├── CADBENEF.NSN
│   ├── CADDEPEND.NSN
│   ├── CADPROG.NSN
│   ├── CALCBENF.NSN
│   ├── CALCCORR.NSN
│   ├── CALCDSCT.NSN
│   ├── CONSBENF.NSN
│   ├── RELAUDIT.NSN
│   ├── RELPGT.NSN
│   ├── VALBENEF.NSN
│   ├── VALDOCS.NSN
│   └── VALELEG.NSN
├── adabas-ddms/
│   ├── README.md
│   ├── AUDITORIA.ddm
│   ├── BENEFICIARIO.ddm
│   ├── PAGAMENTO.ddm
│   └── PROGRAMA-SOCIAL.ddm
└── legacy-docs/
    ├── ARQUITETURA-ORIGINAL-1997.md
    ├── MANUAL-TECNICO-SIFAP-2008.md
    └── REGRAS-NEGOCIO-2012.md
```

**Total de diretórios:** 4 (raiz + `natural-programs/` + `adabas-ddms/` + `legacy-docs/`)

---

## Contagem de Arquivos por Tipo

| Extensão | Contagem | Finalidade provável |
|----------|----------|---------------------|
| `.NSN`   | 15       | Programa-fonte Natural — contém lógica de negócio executável no mainframe |
| `.ddm`   | 4        | Data Definition Module (Adabas) — define estrutura dos arquivos de dados |
| `.md`    | 7        | Documentação (READMEs e documentos legados digitalizados) |

**Total de arquivos rastreáveis:** 26

---

## Padrões de Convenção de Nomes

> Análise baseada exclusivamente nos nomes dos arquivos `.NSN`. Hipóteses fundamentadas em conhecimento genérico de convenções Natural/mainframe.

| Prefixo  | Contagem | Arquivos                              | Hipótese |
|----------|----------|---------------------------------------|----------|
| `BATCH`  | 3        | BATCHCON, BATCHPGT, BATCHREL          | Programas de processamento batch — executados via scheduler JCL, sem interação de terminal |
| `CAD`    | 3        | CADBENEF, CADDEPEND, CADPROG          | Programas de cadastro (CRUD) — provavelmente programas online com mapas de tela 3270 |
| `CALC`   | 3        | CALCBENF, CALCCORR, CALCDSCT          | Subprogramas de cálculo — chamados via CALLNAT, encapsulam lógica financeira |
| `VAL`    | 3        | VALBENEF, VALDOCS, VALELEG            | Subprogramas de validação — chamados via CALLNAT antes de gravações no Adabas |
| `REL`    | 2        | RELAUDIT, RELPGT                      | Programas de relatório — provavelmente batch ou online, produzem saída formatada |
| `CONS`   | 1        | CONSBENF                              | Programa de consulta — leitura somente; prefixo único, investigar se há outros programas de consulta não listados |

---

## Itens Incomuns (Top 3)

### 1. `CALCDSCT.NSN` — Adicionado mais de 15 anos após o sistema original

- **File path:** `01-arqueologia/legado-sifap/natural-programs/CALCDSCT.NSN`
- **Por que é incomum:** O Manual Técnico de 2008 lista explicitamente que este programa **não estava incluído** na documentação original e foi adicionado na versão 4.0 (2015). É o programa mais recente do inventário, com última alteração em 2018 — criado para atender descontos e consignações, possivelmente sob pressão regulatória.
- **Ação de investigação:** Ler `CALCDSCT.NSN` com atenção especial a regras de negócio que podem não ter equivalente documentado. Cruzar com `REGRAS-NEGOCIO-2012.md` (que é anterior à criação deste programa) para identificar lacunas de documentação.

### 2. `AUDITORIA.ddm` — DDM não previsto na arquitetura original

- **File path:** `01-arqueologia/legado-sifap/adabas-ddms/AUDITORIA.ddm`
- **Por que é incomum:** O documento `ARQUITETURA-ORIGINAL-1997.md` previa apenas 3 DDMs. O `AUDITORIA.ddm` (FNR 153) foi criado em 2005 e o próprio `MANUAL-TECNICO-SIFAP-2008.md` admite que a seção de modelo de dados ficou desatualizada. Há um comentário HTML no manual alertando para consultar "Roberto Carlos" para atualização.
- **Ação de investigação:** Ler o DDM para mapear quais campos de trilha de auditoria existem. Identificar quais programas `.NSN` gravam neste DDM (procurar `STORE`/`UPDATE` referenciando `AUDITORIA`).

### 3. `CONSBENF.NSN` — Único programa com prefixo `CONS`

- **File path:** `01-arqueologia/legado-sifap/natural-programs/CONSBENF.NSN`
- **Por que é incomum:** Todos os outros prefixos têm pelo menos 2 representantes. `CONS` é o único com apenas 1 ocorrência. Isso pode indicar que programas de consulta de pagamento (`CONSPGT`) e outros foram planejados (aparecem na arquitetura de 1997) mas nunca implementados como programas separados — ou foram incorporados nos programas batch.
- **Ação de investigação:** Verificar no `ARQUITETURA-ORIGINAL-1997.md` se há referência a `CONSPGT` ou outros `CONS*`. Confirmar se `CONSBENF.NSN` é realmente o único programa de consulta interativa do sistema.

---

## Ordem de Leitura Proposta

> **Hipótese** — esta ordem será revisada assim que a equipe começar a rastrear dependências via `CALLNAT`. Prioriza: (a) DDMs para entender os dados antes do código, (b) programa batch principal como entry point, (c) subprogramas mais referenciados.

| Ordem | Arquivo                   | Justificativa |
|-------|---------------------------|---------------|
| 1     | `BENEFICIARIO.ddm`        | DDM com maior volumetria (~4,2M registros); entender os dados do beneficiário antes de qualquer programa |
| 2     | `PROGRAMA-SOCIAL.ddm`     | DDM de parâmetros — define regras de elegibilidade e faixas; necessário para entender CALCBENF |
| 3     | `PAGAMENTO.ddm`           | DDM de transações — central no processamento batch |
| 4     | `AUDITORIA.ddm`           | DDM adicionado posteriormente; ler para fechar o modelo de dados completo |
| 5     | `BATCHPGT.NSN`            | Entry point batch principal — orquestra o processamento mensal; maior complexidade de fluxo |
| 6     | `CALCBENF.NSN`            | Subprograma mais chamado (por BATCHPGT, CADBENEF e possivelmente outros); núcleo financeiro |
| 7     | `CALCDSCT.NSN`            | Desconto/consignação — chamado por BATCHPGT após CALCBENF; item incomum (v4.0/2015) |
| 8     | `CALCCORR.NSN`            | Correções e reajustes anuais — completar o trio de cálculo |
| 9     | `CADBENEF.NSN`            | Programa online principal de cadastro; ponto de entrada dos dados no sistema |
| 10    | `VALBENEF.NSN`            | Validação chamada por CADBENEF; regras de CPF/NIS |
| 11    | `VALELEG.NSN`             | Validação de elegibilidade — cruzamento com PROGRAMA-SOCIAL |
| 12    | `VALDOCS.NSN`             | Validação de documentação — regras de checklist |
| 13    | `CONSBENF.NSN`            | Consulta online — entender o caminho de leitura e filtros |
| 14    | `RELPGT.NSN`              | Relatório de pagamentos — entender saídas do sistema |
| 15    | `RELAUDIT.NSN`            | Relatório de auditoria — fecha o ciclo de rastreabilidade |
| 16    | `BATCHREL.NSN`            | Relatórios batch — provavelmente chama RELPGT |
| 17    | `BATCHCON.NSN`            | Conciliação financeira — retorno bancário x SIAFI; fluxo mais específico |
| 18    | `CADDEPEND.NSN`           | Cadastro de dependentes — entidade secundária |
| 19    | `CADPROG.NSN`             | Cadastro de programa social — parametrização; provavelmente lido pelo CALCBENF |

---

*Revisão deste inventário: após leitura dos DDMs (itens 1–4) e de BATCHPGT (item 5), atualizar com dependências reais encontradas via CALLNAT.*
