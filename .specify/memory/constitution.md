<!--
SYNC IMPACT REPORT — speckit.constitution
==========================================
Version change: TEMPLATE (unratified) → 1.0.0
Rationale: Initial ratification of the SIFAP modernization constitution after
Stage 1 (Arqueologia) sign-off. First MAJOR.MINOR.PATCH baseline.

Principles defined (1.0.0):
  I.   Legado é a Fonte da Verdade (NÃO-NEGOCIÁVEL)
  II.  Rastreabilidade REQ-ID + source_legacy (NÃO-NEGOCIÁVEL)
  III. Requisitos em Notação EARS
  IV.  Modular Monolith como Arquitetura-Alvo
  V.   Testes Junto à Implementação & Quality Gates
  VI.  Segurança e Proteção de Dados (OWASP Top 10)

Added sections:
  - Restrições de Tecnologia e Arquitetura
  - Questões Abertas Obrigatórias — Mistérios Bloqueadores (6)
  - Fluxo de Desenvolvimento e Gates de Qualidade
  - Governance

Templates / artifacts status:
  ✅ .specify/templates/plan-template.md — "Constitution Check" usa gate genérico
     ("[Gates determined based on constitution file]"); compatível, sem edição.
  ✅ .specify/templates/spec-template.md — placeholders genéricos; EARS/REQ-ID e
     source_legacy aplicáveis no preenchimento; sem edição estrutural necessária.
  ✅ .specify/templates/tasks-template.md — sem referências a princípios; compatível.
  ✅ .github/copilot-instructions.md — alinhado (stack, rastreabilidade, EARS, OWASP).

Deferred TODOs:
  - Nenhum placeholder pendente. RATIFICATION_DATE definida como 2026-06-10
    (data de assinatura do Estágio 1 / adoção da constituição).
  - Os 6 mistérios bloqueadores permanecem ABERTOS por design (ver seção dedicada);
    devem ser fechados com o negócio antes de qualquer EARS financeira/elegibilidade.
==========================================
-->

# Constituição do Projeto SIFAP — Modernização de Legado

> Sistema de Fiscalização e Administração de Pagamentos (SIFAP). Modernização do
> legado Natural/Adabas (29 anos, 15 programas `.NSN` + 4 DDMs) para Java 21 +
> Next.js 15 em arquitetura Modular Monolith. Esta constituição governa todo o
> trabalho do Estágio 2 em diante e se sobrepõe a qualquer outra prática.

## Core Principles

### I. Legado é a Fonte da Verdade (NÃO-NEGOCIÁVEL)

O comportamento real do sistema é definido pelo **código legado** em
`01-arqueologia/legado-sifap/` (programas `.NSN` e DDMs `.ddm`), **nunca** pela
documentação. A pasta `legacy-docs/` está comprovadamente desatualizada de
propósito — descreve subprogramas fantasma (VALCPF, VALNISN, LOGAUDIT, CALCIDX),
fórmula aditiva inexistente e códigos de desconto numéricos que não existem no
código real.

- Toda afirmação sobre regra de negócio DEVE ser verificada contra o `.NSN`/`.ddm`
  fonte, citando arquivo e faixa de linhas (ex.: `BATCHPGT.NSN#L280-L282`).
- Quando código e documentação divergem, **o código vence** e a divergência é
  registrada como mistério/questão aberta — nunca silenciada.
- O `glossary.md` (52 termos) é o vocabulário oficial; os 3 termos marcados como
  HIPÓTESE exigem validação antes de virarem requisito normativo.

**Rationale:** O Estágio 1 encontrou 33 mistérios e divergências graves entre
código e doc. Confiar na doc reproduziria bugs e regras inexistentes.

### II. Rastreabilidade REQ-ID + source_legacy (NÃO-NEGOCIÁVEL)

Todo requisito DEVE ter um identificador único `REQ-NNN` e uma linha
`source_legacy:` apontando para evidência real:

- um programa em `01-arqueologia/legado-sifap/natural-programs/*.NSN`, ou
- um DDM em `01-arqueologia/legado-sifap/adabas-ddms/*.ddm`, ou
- `[GREENFIELD] + justificativa` quando não houver origem legada.

O job de CI `legacy-traceability` **rejeita PRs** que violem esta regra. Testes
rastreiam para os `REQ-IDs` por comentários inline. Requisitos órfãos (sem ID ou
sem `source_legacy:`) são proibidos.

**Rationale:** A rastreabilidade spec → code → test é o eixo do workshop e a
única garantia de que a modernização preserva o comportamento legado verificado.

### III. Requisitos em Notação EARS

Todo requisito DEVE ser escrito em **notação EARS** e classificado em um dos seis
padrões: Ubiquitous, Event-driven (`Quando…`), State-driven (`Enquanto…`),
Optional (`Onde…`), Unwanted (`Se… então…`) ou Complex. Cada requisito carrega
critérios de aceitação testáveis. Declarações que não se enquadram em um padrão
EARS são reescritas antes da aceitação.

**Rationale:** EARS elimina ambiguidade e torna os requisitos verificáveis e
auditáveis frente ao legado.

### IV. Modular Monolith como Arquitetura-Alvo

A arquitetura-alvo é um **Modular Monolith** — uma única unidade implantável com
fronteiras internas de módulos claras (package-by-feature). **Não** se decompõe
em microservices. Cada módulo possui seu domínio, repository e service;
comunicação cross-module via interfaces ou domain events; anti-corruption layers
nas fronteiras com o legado (Strangler Fig para coexistência).

- Decisões estruturais significativas (mapeamento de banco, posicionamento de
  fronteiras, autenticação) DEVEM ser registradas como ADRs (status, contexto,
  decisão, consequências).
- Novas dependências exigem justificativa em um ADR.

**Rationale:** O acoplamento legado é por dados (hub `PAGAMENTO`, zero `CALLNAT`);
um Modular Monolith preserva coesão de domínio sem o custo operacional de sistemas
distribuídos.

### V. Testes Junto à Implementação & Quality Gates

Testes são escritos **durante** a implementação, nunca depois. Lógica de negócio
(cálculo, desconto, elegibilidade, validação de CPF) exige testes unitários
obrigatórios. Backend: JUnit 5 + Testcontainers; Frontend: Vitest + Testing
Library. Cada teste referencia o `REQ-ID` que cobre.

**Rationale:** A lógica financeira do SIFAP é duplicada e ambígua; testes
caracterizando o comportamento são a rede de segurança da migração.

### VI. Segurança e Proteção de Dados (OWASP Top 10)

- Validar entradas em toda fronteira do sistema; SQL apenas via JPA/JPQL (sem
  concatenação de strings).
- **Nunca** fazer hardcode de secrets, API keys ou credenciais (commits, logs,
  PRs, `locals`/`variables`). Secrets via Key Vault / Managed Identity.
- **Mascarar CPF e valores de benefício** em logs, mensagens de erro e telemetria.
- CORS explícito (sem wildcard `*` em produção); autenticação OAuth2/JWT.

**Rationale:** O SIFAP processa dados sensíveis de ~4,2M beneficiários e ~180M
pagamentos; vazamento de CPF/valor é incidente regulatório.

## Restrições de Tecnologia e Arquitetura

Toolchain e stack são **fixos** — misturar ferramentas quebra a rastreabilidade e
as demos do workshop.

- **Backend:** Java 21 (records, sealed interfaces, pattern matching, virtual
  threads) + Spring Boot 3.3 + JPA/Hibernate + PostgreSQL 16. `@Transactional`
  apenas na camada de service; `@Valid` na camada de controller; `Optional` em
  retornos públicos (nunca `null`); nomes de classe e comentários em inglês.
- **Frontend:** Next.js 15 (App Router) + TypeScript 5 `strict` + Tailwind CSS +
  shadcn/ui. Server actions para mutations; somente named exports; `async/await`.
- **REST:** path `/api/v1/{resource}`; verbos e status codes corretos; OpenAPI/
  Swagger em todos os endpoints.
- **Mapeamento Adabas→JPA:** campos MU → `@ElementCollection`/JSONB; grupos PE →
  `@OneToMany` com entidade embedded; super-descriptors → `@Index` composto.
- **Containers:** Docker + Docker Compose. **IaC:** Terraform (Azure ~> 3.x), com
  `tags` (`project`, `environment`, `owner`), secrets só via Key Vault, e
  `terraform fmt`/`validate` verdes antes do commit. **CI/CD:** GitHub Actions.
- Ferramentas de IA, IDEs e frameworks SDD alternativos são proibidos (ver
  `.github/copilot-instructions.md`).

## Questões Abertas Obrigatórias — Mistérios Bloqueadores

Os **6 mistérios bloqueadores** abaixo são questões abertas **obrigatórias**. É
PROIBIDO escrever EARS financeiras, de desconto ou de elegibilidade antes de cada
um ser resolvido com o negócio/facilitador. Cada resolução vira ADR + `REQ-ID`.

| ID | Questão aberta | Risco | Evidência |
| --- | --- | --- | --- |
| **MYS-008** | Região 99 = bypass total de elegibilidade (`ESCAPE ROUTINE`), sem validação nem log de quem atribui. | 🔴 Fraude | `VALELEG.NSN#L107-L111` |
| **MYS-001** | Idade > 75 sobrescreve status para `'S'` → inelegível e não pago, silenciosamente. | 🔴 Suspensão de idosos | `CADBENEF.NSN#L168-L171` |
| **MYS-012** | Fórmula de benefício é multiplicativa e o reajuste incide sobre o total, contradizendo a doc (aditiva). | 🔴 Valor pago a milhões | `BATCHPGT.NSN#L280-L282` |
| **MYS-011** | CALCBENF e BATCHPGT **duplicam** a lógica financeira; a doc diz que o batch chama CALCBENF — não chama. | 🔴 Sem fonte única de verdade | `dependency-map.md` |
| **MYS-014** | Três regras de desconto/contribuição conflitantes (3% fixo inline vs 3/5/7/9% progressivo vs teto 30%). | 🔴 Líquido diverge por caminho | `CALCDSCT.NSN` |
| **MYS-006** | Teto de 30% trunca o acumulado dentro do loop — pode truncar penhora judicial conforme a ordem. | 🔴 Passivo legal, ordem-dependente | `CALCDSCT.NSN#L165-L169` |

Detalhes completos: `01-arqueologia/mysteries-found.md`. Os 27 mistérios
não-bloqueadores são tratados como riscos rastreados, não como gates rígidos.

## Fluxo de Desenvolvimento e Gates de Qualidade

- **Fluxo Spec-Kit:** `/speckit.constitution` → `/speckit.specify` → `@architect`
  (bounded contexts, ADRs) → `/speckit.clarify` → `/speckit.plan` → `@architect`
  (design Modular Monolith) → `/speckit.tasks` → `/speckit.analyze`.
- **Branching:** `spec/<NNN>-<feature>` → `develop` → `main` (sem `stage`).
- **Gates de PR (bloqueantes):**
  1. `legacy-traceability` verde (REQ-ID + `source_legacy:` em todo requisito).
  2. Pelo menos uma revisão entre pares antes de merge em `main`.
  3. Testes presentes e verdes para lógica de negócio alterada.
  4. Sem secrets em commits/logs/PR; CPF e valores mascarados.
  5. `terraform fmt`/`validate` (quando IaC) e build verde.
- **Definição de Pronto do Estágio 2:** ≥10 requisitos EARS com `REQ-NNN`; mapa de
  2–4 bounded contexts (Mermaid); diagramas C4 L1 e L2; ≥3 ADRs; rascunho de
  modelo de dados Adabas→JPA; esboço de ≥3 contratos REST.

## Governance

Esta constituição **se sobrepõe** a todas as outras práticas. Em caso de conflito
entre uma prática e esta constituição, a constituição prevalece.

- **Emendas** exigem: descrição da mudança, justificativa, aprovação por revisão
  entre pares e atualização do versionamento semântico abaixo.
- **Versionamento semântico da constituição:**
  - **MAJOR** — remoção/redefinição incompatível de princípio ou governança.
  - **MINOR** — novo princípio/seção ou expansão material de orientação.
  - **PATCH** — esclarecimentos, correções e refinamentos não-semânticos.
- **Conformidade:** todo PR e revisão DEVE verificar aderência aos princípios;
  complexidade fora do escopo do Modular Monolith exige justificativa em ADR.
- **Orientação em tempo de execução:** ver `.github/copilot-instructions.md` e os
  arquivos de instrução em `.github/instructions/`.
- Os 6 mistérios bloqueadores permanecem itens de governança abertos até serem
  fechados com o negócio; reabrir um mistério fechado exige novo ADR.

**Version**: 1.0.0 | **Ratified**: 2026-06-10 | **Last Amended**: 2026-06-10
