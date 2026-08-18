# Guia de Referências de Implementação --- Qodyn Skill Hub

**Versão:** 1.0\
**Papel:** documento complementar ao PRD e à SPEC\
**Objetivo:** definir quais projetos externos devem ser usados como
referência, o que aproveitar de cada um, onde aplicar e quais limites
respeitar durante a implementação.

------------------------------------------------------------------------

# 1. Decisão de arquitetura

O Qodyn Skill Hub não deve ser implementado a partir de uma única base
externa.

A estratégia recomendada é combinar referências especializadas:

``` text
                         QODYN SKILL HUB
                                |
         +----------------------+----------------------+
         |                      |                      |
     STANDARD              PLATFORM/ENGINE       SKILL QUALITY
         |                      |                      |
 agentskills/agentskills   tech-leads-club      Anthropic + Addy
         |                  /agent-skills             |
         v                      v                      v
 Specification            CLI / Registry         Authoring
 Compatibility            Distribution           Workflows
 Conformance              Cache / MCP             Quality gates
```

Regra central:

> **Agent Skills define o padrão. Tech Leads Club fornece o melhor
> baseline operacional. Anthropic e Addy Osmani orientam a qualidade e o
> uso das Skills. Catálogos agregadores são fontes de discovery, não
> fontes de verdade.**

------------------------------------------------------------------------

# 2. Matriz executiva de referências

  ----------------------------------------------------------------------------------------
  Fonte                            Papel no Qodyn              Prioridade Tipo de uso
  -------------------------------- ---------------- --------------------- ----------------
  agentskills/agentskills          Fonte normativa                     P0 Especificação /
                                   do padrão                              Conformance

  tech-leads-club/agent-skills     Baseline técnico                    P0 Código /
                                   da plataforma                          Arquitetura /
                                                                          CLI

  anthropics/skills                Referência de                       P0 Authoring /
                                   implementação                          Runtime patterns
                                   real de Skills                         

  addyosmani/agent-skills          Referência de                       P1 Quality model /
                                   qualidade e                            Templates
                                   workflows                              

  VoltAgent/awesome-agent-skills   Discovery de                        P2 Source catalog /
                                   ecossistema                            Pesquisa
  ----------------------------------------------------------------------------------------

------------------------------------------------------------------------

# 3. Fonte normativa --- Agent Skills

## Repositório

GitHub:

https://github.com/agentskills/agentskills

Site da especificação:

https://agentskills.io

## Papel

Esta é a **fonte da verdade para o formato Agent Skills**.

O projeto define Agent Skills como um formato aberto e leve em que uma
Skill é essencialmente uma pasta contendo `SKILL.md`, podendo incluir
scripts, referências, assets e outros recursos.

## Usar para

### Agent Skills Compatibility Layer

Implementar:

-   parser de `SKILL.md`;
-   validação de metadata;
-   validação da estrutura;
-   validação de nomes;
-   regras de compatibilidade;
-   progressive disclosure;
-   conformance tests;
-   identificação de features do padrão;
-   futuras migrações entre versões da especificação.

### Skill Validator

O comando:

``` bash
qodyn skill validate
```

deve validar primeiramente contra esta especificação.

### Test fixtures

Criar fixtures:

``` text
tests/
  agent-skills-conformance/
    valid/
    invalid/
    edge-cases/
```

### Compatibility Service

``` text
Agent Skill
    |
    v
AgentSkillsCompatibilityService
    |
    +-- compatible
    +-- compatible_with_warnings
    +-- requires_extension
    +-- incompatible
```

## Não usar para

Não esperar que esse projeto forneça:

-   marketplace enterprise;
-   RBAC;
-   Registry corporativo;
-   approval workflow;
-   telemetria;
-   adapters completos;
-   Source Management;
-   governance.

Esses recursos pertencem ao Qodyn.

## Regra de implementação

Quando houver conflito entre uma decisão do Tech Leads Club e a
especificação oficial:

> **a especificação oficial prevalece.**

------------------------------------------------------------------------

# 4. Baseline operacional --- Tech Leads Club Agent Skills

## Repositório

https://github.com/tech-leads-club/agent-skills

Marketplace atual:

https://agent-skills.techleads.club

## Papel

Este deve ser o **principal baseline técnico para o MVP do Qodyn Skill
Hub**.

O projeto já implementa vários problemas que o Qodyn também precisa
resolver:

-   catálogo;
-   CLI;
-   instalação de Skills;
-   múltiplos agentes;
-   cache;
-   registry;
-   lockfile;
-   auditoria;
-   segurança de filesystem;
-   MCP;
-   marketplace;
-   build;
-   releases.

## Estratégia recomendada

Não realizar um fork cego.

Executar:

``` text
TCL repository
     |
     v
Technical inventory
     |
     +-- reuse
     +-- refactor
     +-- replace
     +-- extend
```

------------------------------------------------------------------------

# 5. O que aproveitar diretamente do TCL

## 5.1 CLI

Usar como referência/base para:

-   command parsing;
-   interactive wizard;
-   list;
-   search;
-   install;
-   update;
-   remove;
-   cache;
-   audit;
-   global/project scope;
-   copy/symlink;
-   seleção de agentes.

Evoluir para:

``` text
apps/cli
   |
Application Services
   |
RegistryProvider
   |
Policy
   |
Resolver
   |
Adapter SDK
```

Evitar lógica de negócio diretamente nos comandos.

------------------------------------------------------------------------

# 6. Registry do TCL

O TCL utiliza um registry gerado para distribuição do catálogo.

Aproveitar:

-   geração do índice;
-   metadata do catálogo;
-   hashing;
-   download;
-   cache;
-   lookup.

Não manter o registry estático como arquitetura definitiva.

Criar:

``` ts
interface RegistryProvider {
  search(query: SearchQuery): Promise<SkillSummary[]>;
  getSkill(id: string): Promise<Skill>;
  getVersion(id: string, version: string): Promise<SkillVersion>;
  download(id: string, version: string): Promise<SkillPackage>;
}
```

Implementações:

``` text
StaticJsonRegistryProvider
        |
        +-- compatibilidade com TCL/MVP

QodynApiRegistryProvider
        |
        +-- produção enterprise
```

Isso permite começar rápido sem acoplar o produto ao modelo estático.

------------------------------------------------------------------------

# 7. Multi-agent do TCL

O projeto já possui conhecimento sobre diferentes destinos de
instalação.

Aproveitar:

-   nomes dos agentes;
-   detecção;
-   diretórios;
-   escopos;
-   convenções;
-   comportamento local/global.

Refatorar para:

``` text
agents.ts / mappings
          |
          v
      Adapter SDK
          |
    +-----+-----+
    |     |     |
 Claude Codex Cursor
```

O mapping existente é uma excelente fonte inicial para implementar os
primeiros adapters.

------------------------------------------------------------------------

# 8. Cache

Aproveitar:

-   estratégia de cache;
-   content hashing;
-   cache de registry;
-   cache de pacotes;
-   invalidação.

Evoluir para cache baseado em conteúdo:

``` text
SHA-256
   |
   v
~/.qodyn/cache/packages/<hash>
```

O mesmo conteúdo nunca deve precisar ser baixado novamente.

------------------------------------------------------------------------

# 9. Lockfile

Aproveitar o conceito e experiência do TCL.

Não copiar necessariamente o schema atual como contrato definitivo.

Implementar Qodyn Lockfile v2:

``` text
.qodyn/skills.lock
```

Registrando:

-   skill;
-   versão;
-   source;
-   upstream;
-   hash;
-   assinatura;
-   adapter;
-   adapter version;
-   compatibility mode;
-   dependencies;
-   install path.

------------------------------------------------------------------------

# 10. Segurança do TCL

Aproveitar especialmente:

-   sanitização;
-   path traversal protection;
-   symlink guards;
-   atomic writes;
-   hashing;
-   audit trail;
-   scanning.

Estender com:

-   provenance;
-   package signing;
-   source trust;
-   permission model;
-   SBOM;
-   policy engine;
-   quarantine;
-   enterprise security findings.

------------------------------------------------------------------------

# 11. MCP do TCL

Usar como baseline para o Qodyn MCP.

Preservar o princípio de progressive disclosure.

Evoluir:

``` text
TCL MCP
  |
  v
Qodyn MCP
  |
Registry API
  |
Policy Engine
```

Tools esperadas:

``` text
search_skills
list_skills
read_skill
fetch_skill_files
get_compatibility
get_approved_version
```

O MCP deve ser consumidor do Registry; não deve virar uma segunda fonte
de verdade.

------------------------------------------------------------------------

# 12. Marketplace do TCL

Usar como referência inicial para:

-   catálogo;
-   categorias;
-   busca;
-   detail page;
-   documentação;
-   visualização das Skills.

Não usar como arquitetura final do marketplace enterprise.

O frontend Qodyn precisará adicionar:

-   autenticação;
-   organization/team;
-   publisher;
-   owner;
-   source;
-   upstream;
-   versions;
-   compatibility;
-   security;
-   approvals;
-   policies;
-   audit;
-   project inventory.

------------------------------------------------------------------------

# 13. CI/CD do TCL

Usar como referência para:

-   build do monorepo;
-   validação;
-   scan;
-   geração de registry;
-   releases;
-   publicação de CLI;
-   publicação do catálogo.

Separar no Qodyn:

``` text
Product CI/CD
     +
Skill Supply Chain CI/CD
```

Não tratar release do produto e publicação de Skill como a mesma
operação.

------------------------------------------------------------------------

# 14. Referência de implementação --- Anthropic Skills

## Repositório

https://github.com/anthropics/skills

## Papel

É uma referência importante de **como Skills reais são estruturadas e
utilizadas no ecossistema Claude**.

O próprio repositório diferencia sua implementação da especificação
Agent Skills.

## Usar para

### Skill authoring

Analisar:

-   estrutura de `SKILL.md`;
-   descrição;
-   instruções;
-   referências;
-   scripts;
-   assets;
-   decomposição de conteúdo.

### Progressive disclosure

Usar como referência para evitar Skills monolíticas.

Estrutura recomendada:

``` text
SKILL.md
    |
    +-- references/
    +-- scripts/
    +-- templates/
```

Carregar recursos adicionais apenas quando necessário.

### Skills complexas

Analisar Skills de produção para entender:

-   separação de responsabilidades;
-   instruções operacionais;
-   uso de scripts;
-   integração de ferramentas;
-   estrutura de documentação.

### Templates

Criar templates Qodyn inspirados nesses padrões.

## Não usar para

Não usar como base principal de:

-   Registry;
-   marketplace;
-   multi-tenant;
-   RBAC;
-   CLI enterprise.

O objetivo do repo é diferente.

------------------------------------------------------------------------

# 15. Referência de qualidade --- Addy Osmani Agent Skills

## Repositório

https://github.com/addyosmani/agent-skills

## Papel

Usar como referência para **qualidade de workflows de engenharia**.

O projeto trata Skills como processos executáveis por agentes, e não
apenas documentação.

## Conceitos que devem ser aproveitados

### Process, not prose

Skill deve definir procedimento.

Evitar:

``` text
"Considere boas práticas de testes."
```

Preferir:

``` text
1. Identifique os comportamentos.
2. Crie testes.
3. Execute a suíte.
4. Analise falhas.
5. Não finalize enquanto houver falhas.
```

### Verification gates

Toda Skill crítica deve possuir critérios objetivos de conclusão.

### Red flags

Documentar sinais de execução incorreta.

### Anti-rationalization

Para workflows importantes, documentar atalhos que o agente não deve
tomar.

### Exit criteria

Definir quando a Skill pode considerar o trabalho concluído.

------------------------------------------------------------------------

# 16. Onde aplicar Addy no Qodyn

Criar o:

``` text
Qodyn Skill Quality Model
```

Dimensões:

``` text
Structure
Instructions
Workflow
Verification
Safety
Portability
Maintainability
Documentation
```

Esse modelo pode alimentar futuramente:

``` text
Quality Score
```

Também utilizar como referência para:

-   Skill templates;
-   reviewer checklist;
-   certification;
-   authoring guide;
-   quality pipeline.

------------------------------------------------------------------------

# 17. Anthropic + Addy

As duas referências são complementares.

``` text
Anthropic
    |
    +-- Como estruturar Skills reais

Addy
    |
    +-- Como transformar instruções em workflows robustos
```

Resultado:

``` text
Qodyn Skill Authoring Guide
```

------------------------------------------------------------------------

# 18. Fonte de discovery --- VoltAgent Awesome Agent Skills

## Repositório

https://github.com/VoltAgent/awesome-agent-skills

## Papel

Não utilizar como base arquitetural.

Utilizar como:

-   catálogo de discovery;
-   pesquisa de mercado;
-   identificação de publishers;
-   análise de categorias;
-   benchmarking;
-   identificação de Skills interessantes;
-   futura Source externa.

O projeto agrega Skills de diversos times e ferramentas.

------------------------------------------------------------------------

# 19. Uso futuro do VoltAgent no Source Manager

Exemplo:

``` text
Qodyn Source Manager
       |
       +-- Tech Leads Club
       +-- GitHub
       +-- Anthropic
       +-- Company repos
       +-- VoltAgent catalog
```

Importante:

> Uma entrada em catálogo agregado não significa automaticamente que a
> Skill está aprovada.

Toda Skill externa deve passar pelo pipeline Qodyn.

------------------------------------------------------------------------

# 20. Pipeline de importação de fontes externas

``` text
External Repository
       |
       v
Source Manager
       |
       v
Discovery
       |
       v
License / Provenance
       |
       v
Agent Skills Compatibility
       |
       v
Security Scan
       |
       v
Quality Analysis
       |
       v
Policy Engine
       |
       v
Human Approval
       |
       v
Qodyn Registry
```

Nenhuma fonte externa pula esse pipeline.

------------------------------------------------------------------------

# 21. Licenciamento

Antes de reutilizar código ou conteúdo:

1.  identificar licença do repo;
2.  identificar licença específica das Skills;
3.  preservar atribuição;
4.  registrar licença no Source Manager;
5.  separar licença do software da licença do conteúdo;
6.  validar possibilidade de redistribuição;
7.  manter THIRD_PARTY_NOTICES quando necessário.

Especialmente no TCL, software e Skills podem possuir regimes de licença
distintos.

Licença deve ser tratada como metadata obrigatória de provenance.

------------------------------------------------------------------------

# 22. Plano de implementação usando as referências

## Fase 0 --- Research baseline

Clonar em área de referência:

``` text
references/
  agentskills-spec/
  tech-leads-club-agent-skills/
  anthropic-skills/
  addy-agent-skills/
```

Não misturar diretamente esses repos com código de produção.

Produzir:

-   architecture inventory;
-   license inventory;
-   component map;
-   reuse decision log.

------------------------------------------------------------------------

# 23. Fase 1 --- Compatibility

Fonte principal:

``` text
agentskills/agentskills
```

Implementar:

``` text
packages/agent-skills-compat
```

Entregáveis:

-   parser;
-   validator;
-   compatibility report;
-   fixtures;
-   conformance tests;
-   CLI validate.

------------------------------------------------------------------------

# 24. Fase 2 --- CLI Core

Fonte principal:

``` text
tech-leads-club/agent-skills
```

Aproveitar:

-   command model;
-   UX;
-   install/update/remove;
-   cache;
-   filesystem safety;
-   agent detection.

Refatorar para:

``` text
CLI
 |
Application Layer
 |
RegistryProvider
 |
Adapter SDK
```

------------------------------------------------------------------------

# 25. Fase 3 --- Registry

Começar com:

``` text
StaticJsonRegistryProvider
```

inspirado no TCL.

Depois implementar:

``` text
QodynApiRegistryProvider
```

O CLI não deve perceber a diferença.

------------------------------------------------------------------------

# 26. Fase 4 --- Adapter SDK

Fonte inicial:

``` text
TCL agent mappings
+
Agent Skills standard
+
documentação oficial das plataformas
```

Implementar primeiro:

1.  Claude Code;
2.  Codex;
3.  Cursor.

Regras:

``` text
native > pass-through > transformed
```

Nunca transformar se não for necessário.

------------------------------------------------------------------------

# 27. Fase 5 --- Skill Authoring

Fontes:

``` text
Anthropic Skills
+
Addy Osmani Agent Skills
```

Criar:

-   Qodyn Skill Template;
-   Authoring Guide;
-   Quality Checklist;
-   Verification Guide;
-   Review Checklist.

------------------------------------------------------------------------

# 28. Fase 6 --- Source Management

Usar os próprios repos como primeiras fontes de teste:

``` text
Source 1: Tech Leads Club
Source 2: Anthropic
Source 3: Addy Osmani
```

Validar:

-   discovery;
-   sync;
-   provenance;
-   license;
-   hash;
-   upstream updates.

------------------------------------------------------------------------

# 29. Fase 7 --- Enterprise Control Plane

Esta camada deve ser predominantemente implementação própria.

Não procurar encaixar à força nos repos externos:

``` text
Organizations
Teams
SSO
RBAC
Policies
Approvals
Security Findings
Audit
Telemetry
Installation Inventory
```

Esses recursos são diferenciais do produto.

------------------------------------------------------------------------

# 30. Fase 8 --- Marketplace Enterprise

Utilizar o TCL apenas como referência de catálogo.

Construir sobre Qodyn API.

``` text
Web
 |
Qodyn API
 |
Registry
```

Nunca:

``` text
Web
 |
skills-registry.json
```

como arquitetura enterprise definitiva.

------------------------------------------------------------------------

# 31. Ownership das referências

Criar no projeto:

``` text
docs/references/
```

Documentos:

``` text
AGENT_SKILLS_STANDARD.md
TCL_BASELINE.md
ANTHROPIC_PATTERNS.md
ADDY_QUALITY_PATTERNS.md
THIRD_PARTY_LICENSES.md
REFERENCE_DECISIONS.md
```

Cada decisão de reutilização deve indicar:

``` text
Source
Commit/Version
Component
Reason
License
Changes
Owner
```

------------------------------------------------------------------------

# 32. Version pinning

Nunca desenvolver contra `main` de uma referência sem registrar o
commit.

Exemplo:

``` yaml
source: tech-leads-club/agent-skills
commit: a90d9f2
reviewed_at: 2026-08-17
components:
  - cli
  - cache
  - registry
```

Isso garante reprodutibilidade.

------------------------------------------------------------------------

# 33. Processo de atualização das referências

Sugestão mensal ou por release relevante:

``` text
Check upstream
    |
    v
Diff
    |
    +-- Standard changed?
    +-- Security fix?
    +-- CLI improvement?
    +-- Adapter change?
    +-- Authoring pattern?
    |
    v
Architecture Review
```

Atualização de referência nunca deve alterar automaticamente o produto.

------------------------------------------------------------------------

# 34. ADRs obrigatórios

Criar ADRs para decisões como:

``` text
ADR-001 Agent Skills as canonical standard
ADR-002 TCL as operational baseline
ADR-003 RegistryProvider abstraction
ADR-004 Adapter SDK
ADR-005 Enterprise metadata outside SKILL.md
ADR-006 Source provenance model
ADR-007 Native-first adapter strategy
```

------------------------------------------------------------------------

# 35. Ordem de autoridade

Quando fontes divergirem:

``` text
1. Agent Skills official specification
2. Qodyn ADRs / SPEC
3. Platform official documentation
4. Tech Leads Club implementation
5. Anthropic examples
6. Addy patterns
7. Aggregator/community repositories
```

Isso evita decisões arquiteturais contraditórias.

------------------------------------------------------------------------

# 36. Mapa final --- o que usar de cada fonte

## agentskills/agentskills

**Use para:** padrão, parser, validator, compatibility, conformance.

**Não use como:** marketplace enterprise.

------------------------------------------------------------------------

## tech-leads-club/agent-skills

**Use para:** CLI, registry baseline, cache, install, multi-agent,
filesystem security, MCP, lockfile concepts, marketplace baseline,
CI/CD.

**Refatore:** registry, agent mappings, lockfile e marketplace.

**Não use como fonte normativa do padrão.**

------------------------------------------------------------------------

## anthropics/skills

**Use para:** estrutura de Skills reais, progressive disclosure,
scripts/references/assets, authoring patterns.

**Não use como:** backend/registry enterprise.

------------------------------------------------------------------------

## addyosmani/agent-skills

**Use para:** workflow design, quality gates, verification, red flags,
review model, templates.

**Não use como:** plataforma de distribuição.

------------------------------------------------------------------------

## VoltAgent/awesome-agent-skills

**Use para:** discovery, benchmarking, análise de ecossistema e futura
source.

**Não use como:** fonte de confiança automática.

------------------------------------------------------------------------

# 37. Estratégia recomendada de código

``` text
                    External References
                           |
       +-------------------+-------------------+
       |                   |                   |
 Official Spec            TCL            Anthropic/Addy
       |                   |                   |
       v                   v                   v
 Compatibility         Platform Core       Quality Model
       |                   |                   |
       +-------------------+-------------------+
                           |
                           v
                    Qodyn Skill Hub
```

O código Qodyn deve depender de **interfaces próprias**, não diretamente
das estruturas internas dos repos de referência.

Exemplo:

``` text
TCL registry implementation
         |
         v
StaticJsonRegistryProvider
         |
         v
RegistryProvider interface
         ^
         |
QodynApiRegistryProvider
```

Essa abordagem reduz lock-in arquitetural.

------------------------------------------------------------------------

# 38. Resultado esperado

Ao seguir este plano, o projeto ganha:

-   aderência ao padrão aberto;
-   redução do tempo de implementação;
-   aproveitamento de código e padrões já testados;
-   menor risco de reinventar soluções;
-   independência do TCL;
-   compatibilidade futura;
-   melhor qualidade das Skills;
-   supply chain controlada;
-   evolução incremental do MVP para enterprise.

------------------------------------------------------------------------

# 39. Regra final para o time

Antes de implementar qualquer recurso relacionado a Skills, perguntar:

``` text
É regra do padrão?
    -> consultar agentskills.io

É distribuição/CLI/install?
    -> analisar TCL

É estrutura e uso real da Skill?
    -> analisar Anthropic

É qualidade/workflow?
    -> analisar Addy

É recurso enterprise?
    -> seguir PRD + SPEC Qodyn
```

Este documento, o **PRD** e a **SPEC** devem formar o conjunto de
referência obrigatório para implementação do Qodyn Skill Hub.
