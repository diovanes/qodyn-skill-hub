# SPEC --- Evolução do `tech-leads-club/agent-skills` para Skill Marketplace Enterprise

**Versão:** 1.0\
**Data:** 2026-07-04\
**Fonte inicial:** `https://github.com/tech-leads-club/agent-skills`\
**Documento base:** PRD Skill Marketplace & Multi-Agent Adapter
Platform\
**Objetivo:** detalhar os ajustes necessários para transformar o
repositório `agent-skills` em uma plataforma enterprise de marketplace,
registry, governança e adapters de Skills para múltiplas plataformas de
agentes.

------------------------------------------------------------------------

## 1. Sumário Executivo

O repositório `tech-leads-club/agent-skills` já entrega uma base madura
para catálogo e distribuição local de Skills. Ele possui:

-   catálogo de Skills em estrutura organizada;
-   CLI para instalar, atualizar, remover, auditar e gerenciar cache;
-   suporte multiagente;
-   instalação global e local;
-   cache local;
-   lockfile;
-   audit log;
-   MCP server para descoberta progressiva;
-   segurança local com sanitização, isolamento de path, symlink guard,
    hash e scan.

Porém, o PRD gerado anteriormente exige uma solução enterprise maior,
com:

-   front-end de marketplace;
-   back-end com API própria;
-   banco de dados;
-   registry privado;
-   upload e publicação de Skills;
-   workflow de aprovação;
-   RBAC;
-   versionamento granular por Skill;
-   dependências formais entre Skills;
-   Adapter SDK;
-   métricas centralizadas;
-   dashboards;
-   governança por time/projeto;
-   policy engine;
-   suporte a ambientes;
-   skill packs;
-   assinatura de pacotes;
-   operação SaaS/on-premise.

Portanto, a recomendação é **não reescrever o projeto do zero**. O
melhor caminho é usar o repo como **engine inicial** e evoluí-lo em
camadas.

------------------------------------------------------------------------

## 2. Baseline do Repositório Atual

Com base na análise pública do repositório, o `agent-skills` atualmente
se comporta como:

``` text
Catálogo curado de Skills
  + CLI de instalação
  + MCP server de leitura/descoberta
  + cache local
  + segurança local
  + distribuição via pacote/CDN
```

O README descreve Skills como instruções e recursos empacotados que
estendem capacidades de agentes de IA. A estrutura principal segue o
padrão:

``` text
packages/skills-catalog/skills/
  (category-name)/
    skill/
      SKILL.md
      templates/
      references/
```

O CLI permite listar, instalar, atualizar, remover, limpar cache e
consultar audit log. Também suporta instalação em múltiplos agentes,
método copy/symlink e escopo global/local.

O MCP server expõe ferramentas como:

-   `list_skills`;
-   `search_skills`;
-   `read_skill`;
-   `fetch_skill_files`.

A política de segurança atual inclui:

-   sanitização de nomes e paths;
-   proteção contra path traversal;
-   symlink guard;
-   lockfile atômico;
-   hash SHA-256 de conteúdo;
-   audit trail local;
-   scan com Snyk Agent Scan;
-   allowlist para falsos positivos;
-   MCP read-only.

Isso é uma ótima base. Mas ainda não é um marketplace enterprise
completo.

------------------------------------------------------------------------

## 3. Escopo desta Especificação

Esta especificação detalha os ajustes para cobrir as lacunas do PRD,
usando o repo atual como ponto de partida.

A especificação cobre:

1.  arquitetura alvo;
2.  reorganização do monorepo;
3.  novo manifesto `skill.yaml`;
4.  registry back-end;
5.  front-end marketplace;
6.  API layer;
7.  Adapter SDK;
8.  versionamento por Skill;
9.  dependências entre Skills;
10. workflow de publicação;
11. governança e RBAC;
12. security scanner enterprise;
13. policy engine;
14. telemetria centralizada;
15. skill packs;
16. integração com GitHub/Jira/Figma;
17. CLI enterprise;
18. MCP enterprise;
19. plano de migração;
20. roadmap de implementação;
21. critérios de aceite.

------------------------------------------------------------------------

## 4. Matriz de Aderência ao PRD

  --------------------------------------------------------------------------------
  Área               Status no repo      Aderência estimada Ajuste necessário
                     atual                                  
  ------------------ ---------------- --------------------- ----------------------
  Catálogo de Skills Existe                            Alta Manter e enriquecer
                                                            metadata

  `SKILL.md`         Existe                            Alta Preservar como entrada
                                                            principal

  CLI                Existe                            Alta Adaptar para registry
                                                            privado/API

  Multiagente        Existe                      Média/Alta Formalizar Adapter SDK

  Cache              Existe                            Alta Manter e versionar por
                                                            registry

  Lockfile           Existe                      Média/Alta Expandir para
                                                            dependências e SemVer

  Audit local        Existe                           Média Enviar também para
                                                            back-end

  MCP server         Existe                            Alta Criar modo enterprise
                                                            autenticado/opcional

  Segurança local    Existe                            Alta Expandir com policy
                                                            engine e scan central

  Front-end          Parcial/site                     Baixa Criar app web completo
  marketplace        estático                               

  Back-end/API       Não existe como                  Baixa Criar API e serviços
                     registry                               
                     enterprise                             

  Banco de dados     Não existe                       Baixa Criar
                                                            PostgreSQL/Prisma

  Workflow de        Não existe                       Baixa Criar approval service
  aprovação          nativo                                 

  RBAC               Não existe                       Baixa Criar autenticação e
                                                            autorização

  Versionamento por  Parcial                    Média/Baixa Criar versionamento
  Skill                                                     granular

  Dependências entre Não formal                       Baixa Criar dependency
  Skills                                                    resolver

  Skill Packs        Não consolidado                  Baixa Criar abstração de
                                                            packs

  Métricas           Não existe                       Baixa Criar telemetry
  centralizadas                                             service

  Policy engine      Não existe                       Baixa Criar políticas
                                                            corporativas

  Assinatura de      Parcial/hash                     Média Adicionar assinatura
  pacotes                                                   publisher

  Ambientes          Não existe                       Baixa Criar stages de
  dev/staging/prod                                          registry
  --------------------------------------------------------------------------------

------------------------------------------------------------------------

## 5. Arquitetura Alvo

### 5.1 Visão Geral

``` text
                Marketplace Web
                      |
                      v
                 API Gateway
                      |
        +-------------+-------------+
        |             |             |
        v             v             v
 Skill Registry   Package API   Governance API
        |             |             |
        v             v             v
 PostgreSQL      Object Storage  Approval Engine
        |
        v
 Adapter Registry + Policy Engine + Telemetry
        |
        v
 CLI / MCP / IDE Extensions / CI Pipelines
        |
        v
 Claude Code / Codex CLI / Cursor / Gemini / Outros
```

### 5.2 Princípio Arquitetural

A Skill não deve pertencer ao Claude, Codex, Cursor ou qualquer outra
ferramenta.

A Skill pertence ao Registry.

Os Adapters apenas convertem e instalam.

``` text
Skill Canônica
  -> Adapter Claude
  -> Adapter Codex
  -> Adapter Cursor
  -> Adapter Gemini
```

------------------------------------------------------------------------

## 6. Reorganização Recomendada do Monorepo

O repo atual já usa uma estrutura com `packages`, `libs`, `tools` e Nx.
A recomendação é evoluir sem quebrar tudo.

### 6.1 Estrutura Alvo

``` text
agent-skills-enterprise/
  apps/
    web/                         # Marketplace Web
    api/                         # API Gateway + REST API
    worker/                      # Jobs assíncronos

  packages/
    cli/                         # CLI atual evoluída
    mcp/                         # MCP atual evoluído
    skills-catalog/              # Catálogo seed atual
    registry-client/             # SDK cliente para API
    adapter-sdk/                 # Contrato dos adapters
    adapters/
      claude-code/
      codex-cli/
      cursor/
      gemini-cli/
      github-copilot/
      windsurf/
    manifest/                    # Parser/validator de skill.yaml
    validator/                   # Validador de estrutura
    security-scanner/            # Scanner de Skills
    policy-engine/               # Políticas corporativas
    dependency-resolver/         # Resolver SemVer/deps
    package-manager/             # Pack/unpack/checksum/signature
    telemetry-sdk/               # Eventos e métricas
    governance-sdk/              # RBAC/approval helpers

  libs/
    core/
    database/
    auth/
    observability/
    shared-types/

  skills/
    seed/                        # Skills herdadas do repo atual
    internal/                    # Skills corporativas
    packs/                       # Skill packs

  docs/
    architecture/
    api/
    adapters/
    security/
    governance/
    migration/
```

### 6.2 Decisão

Manter `packages/skills-catalog` como fonte seed, mas introduzir
`skill.yaml` por Skill.

------------------------------------------------------------------------

## 7. Novo Contrato de Skill

### 7.1 Problema Atual

O repo atual usa principalmente `SKILL.md` com metadados e recursos
complementares. Isso é suficiente para agentes, mas insuficiente para
marketplace enterprise.

Faltam campos como:

-   publisher;
-   owner;
-   status;
-   maturidade;
-   compatibilidade;
-   permissões;
-   dependências;
-   assinatura;
-   política de aprovação;
-   licença;
-   visibilidade;
-   score;
-   SemVer granular.

### 7.2 Solução

Adicionar `skill.yaml` como manifesto canônico do marketplace.

### 7.3 Estrutura mínima

``` text
skill-name/
  skill.yaml
  SKILL.md
```

### 7.4 Estrutura recomendada

``` text
skill-name/
  skill.yaml
  SKILL.md
  README.md
  CHANGELOG.md
  templates/
  references/
  examples/
  scripts/
  tests/
```

### 7.5 Exemplo completo de `skill.yaml`

``` yaml
apiVersion: qodyn.io/v1
kind: Skill

metadata:
  id: qodyn.angular-implementation
  slug: angular-implementation
  name: Angular Implementation
  description: Implementa telas Angular seguindo design system, testes e padrões corporativos.
  publisher: qodyn
  owner: architecture-team
  category: frontend
  tags:
    - angular
    - frontend
    - figma
    - ui
    - design-system

version:
  semver: 1.0.0
  channel: stable
  changelog: CHANGELOG.md

entrypoint:
  file: SKILL.md
  format: markdown

compatibility:
  platforms:
    claude-code:
      enabled: true
      minVersion: ">=1.0.0"
    codex-cli:
      enabled: true
      minVersion: ">=0.5.0"
    cursor:
      enabled: true
      ruleType: project
    gemini-cli:
      enabled: false

permissions:
  filesystem:
    read: true
    write: true
    allowedPaths:
      - "src/**"
      - "tests/**"
  network:
    enabled: false
  mcp:
    allowedServers:
      - github
      - figma
  scripts:
    allowExecution: false

dependencies:
  required:
    - id: qodyn.typescript-standards
      version: "^1.0.0"
    - id: qodyn.testing-standards
      version: "^1.0.0"
  optional:
    - id: qodyn.accessibility-review
      version: "^1.0.0"

governance:
  visibility: private
  maturity: beta
  lifecycle: active
  reviewRequired: true
  requiredApprovals:
    technical: 1
    security: 1
  allowedScopes:
    - user
    - project
    - team

security:
  trustLevel: internal
  scanRequired: true
  signatureRequired: true
  allowedRiskLevel: low

license:
  type: internal
  attributionRequired: false

quality:
  score: 0
  testsRequired: false
  examplesRequired: true
```

### 7.6 Ajustes no repo

Criar pacote:

``` text
packages/manifest
```

Responsabilidades:

-   parse YAML;
-   validar schema com Zod;
-   normalizar campos;
-   validar SemVer;
-   validar IDs;
-   validar compatibilidade;
-   exportar tipos TypeScript.

### 7.7 Critérios de aceite

-   Toda Skill nova deve conter `skill.yaml`.
-   Skills antigas podem ser migradas com manifesto gerado
    automaticamente.
-   O CLI deve falhar com mensagem clara se `skill.yaml` for inválido.
-   O Registry deve aceitar apenas pacotes com manifesto válido.

------------------------------------------------------------------------

## 8. Migração do Catálogo Atual

### 8.1 Objetivo

Transformar o catálogo atual do repo em seed catalog enterprise.

### 8.2 Estratégia

Criar comando:

``` bash
agent-skills migrate:manifest
```

ou no novo nome:

``` bash
qodyn skills migrate-manifest
```

### 8.3 Entrada

``` text
packages/skills-catalog/skills/**/SKILL.md
```

### 8.4 Saída

Para cada Skill:

``` text
skill.yaml
```

### 8.5 Regras de inferência

-   `id`: derivar de slug/pasta.
-   `name`: derivar do frontmatter ou título.
-   `description`: derivar da descrição existente.
-   `category`: derivar da pasta pai.
-   `publisher`: `tech-leads-club` ou publisher customizado.
-   `version`: `0.1.0` inicial.
-   `entrypoint`: `SKILL.md`.
-   `compatibility`: derivar dos agentes suportados globalmente, mas
    marcar como `unknown` quando não houver garantia.
-   `trustLevel`: `curated`.
-   `visibility`: `public` ou `private`, conforme instalação.

### 8.6 Resultado esperado

``` text
Antes:
  packages/skills-catalog/skills/development/tlc-spec-driven/SKILL.md

Depois:
  packages/skills-catalog/skills/development/tlc-spec-driven/skill.yaml
  packages/skills-catalog/skills/development/tlc-spec-driven/SKILL.md
```

------------------------------------------------------------------------

## 9. Registry Back-End

### 9.1 Lacuna Atual

O repo atual distribui catálogo/pacotes via NPM/CDN e CLI. Não há um
Registry API enterprise com banco, autenticação, publicação privada,
aprovação e métricas centralizadas.

### 9.2 Solução

Criar `apps/api` com serviços modulares.

### 9.3 Stack recomendada

-   Node.js;
-   TypeScript;
-   Fastify;
-   Prisma;
-   PostgreSQL;
-   Redis;
-   BullMQ;
-   S3/MinIO;
-   OpenTelemetry.

### 9.4 Serviços internos

``` text
Skill Registry Service
Package Service
Version Service
Dependency Service
Adapter Service
Approval Service
Security Scan Service
Telemetry Service
Policy Service
User/Team Service
```

### 9.5 Endpoints principais

#### Skills

``` http
GET    /api/v1/skills
POST   /api/v1/skills
GET    /api/v1/skills/:skillId
PATCH  /api/v1/skills/:skillId
DELETE /api/v1/skills/:skillId
```

#### Skill Versions

``` http
GET    /api/v1/skills/:skillId/versions
POST   /api/v1/skills/:skillId/versions
GET    /api/v1/skills/:skillId/versions/:version
PATCH  /api/v1/skills/:skillId/versions/:version
POST   /api/v1/skills/:skillId/versions/:version/deprecate
```

#### Packages

``` http
POST   /api/v1/packages/upload
GET    /api/v1/packages/:packageId/download
POST   /api/v1/packages/:packageId/validate
POST   /api/v1/packages/:packageId/scan
```

#### Install Plan

``` http
POST   /api/v1/install/resolve
POST   /api/v1/install/plan
POST   /api/v1/install/events
```

#### Adapters

``` http
GET    /api/v1/adapters
GET    /api/v1/adapters/:adapterKey
POST   /api/v1/adapters/:adapterKey/validate
POST   /api/v1/adapters/:adapterKey/transform
```

#### Approvals

``` http
GET    /api/v1/approvals
GET    /api/v1/approvals/:approvalId
POST   /api/v1/approvals/:approvalId/approve
POST   /api/v1/approvals/:approvalId/reject
POST   /api/v1/approvals/:approvalId/request-changes
```

#### Governance

``` http
GET    /api/v1/policies
POST   /api/v1/policies
PATCH  /api/v1/policies/:policyId
POST   /api/v1/policies/evaluate
```

#### Telemetry

``` http
POST   /api/v1/telemetry/events
GET    /api/v1/metrics/skills
GET    /api/v1/metrics/projects
GET    /api/v1/metrics/adapters
```

------------------------------------------------------------------------

## 10. Modelo de Dados

### 10.1 `skills`

``` sql
CREATE TABLE skills (
  id UUID PRIMARY KEY,
  skill_key TEXT UNIQUE NOT NULL,
  slug TEXT NOT NULL,
  name TEXT NOT NULL,
  description TEXT NOT NULL,
  category_id UUID,
  publisher_id UUID NOT NULL,
  owner_team_id UUID,
  visibility TEXT NOT NULL,
  lifecycle TEXT NOT NULL,
  created_by UUID NOT NULL,
  created_at TIMESTAMP NOT NULL DEFAULT now(),
  updated_at TIMESTAMP NOT NULL DEFAULT now(),
  archived_at TIMESTAMP
);
```

### 10.2 `skill_versions`

``` sql
CREATE TABLE skill_versions (
  id UUID PRIMARY KEY,
  skill_id UUID NOT NULL REFERENCES skills(id),
  version TEXT NOT NULL,
  channel TEXT NOT NULL,
  manifest_json JSONB NOT NULL,
  package_id UUID NOT NULL,
  checksum TEXT NOT NULL,
  signature TEXT,
  status TEXT NOT NULL,
  changelog TEXT,
  created_by UUID NOT NULL,
  created_at TIMESTAMP NOT NULL DEFAULT now(),
  approved_at TIMESTAMP,
  deprecated_at TIMESTAMP,
  UNIQUE(skill_id, version)
);
```

### 10.3 `skill_dependencies`

``` sql
CREATE TABLE skill_dependencies (
  id UUID PRIMARY KEY,
  skill_version_id UUID NOT NULL REFERENCES skill_versions(id),
  dependency_key TEXT NOT NULL,
  version_range TEXT NOT NULL,
  required BOOLEAN NOT NULL DEFAULT true,
  created_at TIMESTAMP NOT NULL DEFAULT now()
);
```

### 10.4 `publishers`

``` sql
CREATE TABLE publishers (
  id UUID PRIMARY KEY,
  key TEXT UNIQUE NOT NULL,
  name TEXT NOT NULL,
  type TEXT NOT NULL,
  verified BOOLEAN NOT NULL DEFAULT false,
  created_at TIMESTAMP NOT NULL DEFAULT now()
);
```

### 10.5 `projects`

``` sql
CREATE TABLE projects (
  id UUID PRIMARY KEY,
  name TEXT NOT NULL,
  repository_url TEXT,
  team_id UUID,
  environment TEXT,
  created_at TIMESTAMP NOT NULL DEFAULT now(),
  updated_at TIMESTAMP NOT NULL DEFAULT now()
);
```

### 10.6 `installations`

``` sql
CREATE TABLE installations (
  id UUID PRIMARY KEY,
  skill_version_id UUID NOT NULL REFERENCES skill_versions(id),
  project_id UUID,
  user_id UUID,
  team_id UUID,
  platform_key TEXT NOT NULL,
  scope TEXT NOT NULL,
  install_path TEXT,
  status TEXT NOT NULL,
  installed_at TIMESTAMP NOT NULL DEFAULT now(),
  updated_at TIMESTAMP,
  removed_at TIMESTAMP
);
```

### 10.7 `approval_requests`

``` sql
CREATE TABLE approval_requests (
  id UUID PRIMARY KEY,
  skill_version_id UUID NOT NULL REFERENCES skill_versions(id),
  status TEXT NOT NULL,
  requested_by UUID NOT NULL,
  created_at TIMESTAMP NOT NULL DEFAULT now(),
  completed_at TIMESTAMP
);
```

### 10.8 `approval_steps`

``` sql
CREATE TABLE approval_steps (
  id UUID PRIMARY KEY,
  approval_request_id UUID NOT NULL REFERENCES approval_requests(id),
  type TEXT NOT NULL,
  status TEXT NOT NULL,
  reviewer_id UUID,
  comment TEXT,
  completed_at TIMESTAMP
);
```

### 10.9 `security_scans`

``` sql
CREATE TABLE security_scans (
  id UUID PRIMARY KEY,
  skill_version_id UUID NOT NULL REFERENCES skill_versions(id),
  scanner TEXT NOT NULL,
  status TEXT NOT NULL,
  risk_level TEXT NOT NULL,
  findings_json JSONB NOT NULL,
  created_at TIMESTAMP NOT NULL DEFAULT now()
);
```

### 10.10 `audit_logs`

``` sql
CREATE TABLE audit_logs (
  id UUID PRIMARY KEY,
  actor_user_id UUID,
  action TEXT NOT NULL,
  entity_type TEXT NOT NULL,
  entity_id TEXT NOT NULL,
  metadata_json JSONB,
  created_at TIMESTAMP NOT NULL DEFAULT now()
);
```

------------------------------------------------------------------------

## 11. Front-End Marketplace

### 11.1 Lacuna Atual

O repo atual possui site/catálogo, mas não um marketplace enterprise
transacional.

Faltam:

-   login;
-   upload;
-   publicação;
-   aprovação;
-   painel administrativo;
-   dashboard de métricas;
-   gestão de times;
-   permissões;
-   páginas de projeto;
-   visualização de instalações.

### 11.2 Solução

Criar `apps/web`.

### 11.3 Stack

-   Next.js;
-   React;
-   TypeScript;
-   Tailwind;
-   shadcn/ui;
-   TanStack Query;
-   Zod;
-   Auth.js ou SSO corporativo.

### 11.4 Páginas

#### Home

Conteúdo:

-   busca principal;
-   Skills em destaque;
-   categorias;
-   Skills recém-atualizadas;
-   Skills mais instaladas;
-   packs recomendados;
-   CTA para publicar Skill.

#### Catálogo

Filtros:

-   categoria;
-   plataforma;
-   publisher;
-   status;
-   lifecycle;
-   trust level;
-   score;
-   tags;
-   data de atualização.

#### Detalhe da Skill

Deve exibir:

-   nome;
-   descrição;
-   publisher;
-   owner;
-   versão atual;
-   seletor de versão;
-   status;
-   trust level;
-   plataformas suportadas;
-   comando de instalação;
-   conteúdo renderizado do `SKILL.md`;
-   manifesto `skill.yaml`;
-   dependências;
-   permissões;
-   resultados de scan;
-   changelog;
-   histórico de aprovação;
-   projetos usando;
-   botão instalar/copiar comando.

#### Publicar Skill

Fluxo:

1.  upload `.zip`/`.tgz`;
2.  validação estrutural;
3.  validação de manifesto;
4.  prévia dos dados;
5.  scan de segurança;
6.  submissão para aprovação.

#### Aprovações

Para reviewers:

-   fila de aprovação;
-   diff entre versões;
-   permissões alteradas;
-   findings de segurança;
-   botão aprovar;
-   botão rejeitar;
-   botão solicitar ajustes.

#### Dashboard Admin

Indicadores:

-   total de Skills;
-   Skills pendentes;
-   Skills bloqueadas;
-   versões deprecated em uso;
-   instalações por plataforma;
-   instalações por time;
-   erros de instalação;
-   Skills stale.

#### Projetos

Deve exibir:

-   repositório;
-   time;
-   ambiente;
-   Skills instaladas;
-   versões;
-   lockfile;
-   conflitos;
-   recomendações.

------------------------------------------------------------------------

## 12. Adapter SDK

### 12.1 Lacuna Atual

O repo atual suporta múltiplos agentes, mas a lógica tende a estar
orientada a destinos de instalação. Para o PRD, é necessário um contrato
formal de Adapter plugável.

### 12.2 Objetivo

Permitir adicionar novas plataformas sem alterar o core.

### 12.3 Pacote

``` text
packages/adapter-sdk
```

### 12.4 Interface

``` ts
export interface SkillAdapter {
  key: string;
  name: string;
  description: string;

  supports(skill: SkillManifest): boolean;

  validate(input: AdapterValidateInput): Promise<AdapterValidationResult>;

  transform(input: AdapterTransformInput): Promise<AdapterTransformResult>;

  install(input: AdapterInstallInput): Promise<AdapterInstallResult>;

  uninstall(input: AdapterUninstallInput): Promise<AdapterUninstallResult>;

  listInstalled(input: AdapterListInstalledInput): Promise<InstalledSkill[]>;

  detect?(projectPath: string): Promise<AdapterDetectionResult>;
}
```

### 12.5 Tipos

``` ts
export type AdapterValidateInput = {
  skill: SkillPackage;
  target: AdapterTarget;
};

export type AdapterTarget = {
  platform: string;
  scope: 'user' | 'project' | 'team';
  projectPath?: string;
  userHome?: string;
  config?: Record<string, unknown>;
};

export type AdapterTransformResult = {
  files: Array<{
    path: string;
    content: string | Buffer;
  }>;
  warnings: string[];
};

export type AdapterInstallResult = {
  success: boolean;
  installedPaths: string[];
  warnings: string[];
  errors: string[];
};
```

### 12.6 Adapters iniciais

Criar pacotes:

``` text
packages/adapters/claude-code
packages/adapters/codex-cli
packages/adapters/cursor
packages/adapters/gemini-cli
```

### 12.7 Adapter Claude Code

Instalação projeto:

``` text
{project}/.claude/skills/{skill-slug}/SKILL.md
```

Instalação usuário:

``` text
~/.claude/skills/{skill-slug}/SKILL.md
```

### 12.8 Adapter Codex CLI

Instalação projeto:

``` text
{project}/.codex/skills/{skill-slug}/SKILL.md
```

Instalação usuário:

``` text
~/.codex/skills/{skill-slug}/SKILL.md
```

### 12.9 Adapter Cursor

Converter `SKILL.md` para:

``` text
{project}/.cursor/rules/{skill-slug}.mdc
```

Com frontmatter:

``` yaml
---
description: Descrição da Skill
globs:
  - "**/*.ts"
  - "**/*.tsx"
alwaysApply: false
---
```

### 12.10 Critérios de aceite

-   Novo Adapter pode ser adicionado sem alterar CLI core.
-   CLI lista adapters disponíveis.
-   CLI valida se a Skill suporta a plataforma.
-   Adapter pode transformar antes de instalar.
-   Adapter pode emitir warnings específicos.

------------------------------------------------------------------------

## 13. Dependency Resolver

### 13.1 Lacuna Atual

O repo atual instala Skills escolhidas, mas não possui resolver formal
de dependências com SemVer, conflitos e lockfile completo.

### 13.2 Pacote

``` text
packages/dependency-resolver
```

### 13.3 Entrada

``` ts
{
  skillId: 'qodyn.angular-implementation',
  versionRange: '^1.0.0',
  registry: RegistryClient,
  platform: 'claude-code'
}
```

### 13.4 Saída

``` ts
{
  root: 'qodyn.angular-implementation@1.2.0',
  installOrder: [
    'qodyn.typescript-standards@1.0.3',
    'qodyn.testing-standards@1.1.0',
    'qodyn.angular-implementation@1.2.0'
  ],
  conflicts: [],
  warnings: []
}
```

### 13.5 Regras

-   Resolver dependências obrigatórias.
-   Sugerir dependências opcionais.
-   Bloquear dependências não aprovadas.
-   Detectar conflitos de versão.
-   Detectar ciclos.
-   Respeitar lockfile existente.
-   Respeitar políticas do projeto.

### 13.6 Lockfile alvo

``` json
{
  "version": 1,
  "registry": "https://skills.company.com",
  "platforms": ["claude-code", "cursor"],
  "skills": {
    "qodyn.angular-implementation": {
      "version": "1.2.0",
      "checksum": "sha256:abc123",
      "dependencies": [
        "qodyn.typescript-standards",
        "qodyn.testing-standards"
      ]
    }
  }
}
```

------------------------------------------------------------------------

## 14. CLI Enterprise

### 14.1 Lacuna Atual

O CLI atual é bom, mas precisa operar contra registry privado,
autenticação, dependências, policy engine, telemetria e adapters
formais.

### 14.2 Novos comandos

``` bash
qodyn login
qodyn logout
qodyn registry set https://skills.company.com
qodyn skill search angular
qodyn skill info qodyn.angular-implementation
qodyn skill install qodyn.angular-implementation --platform claude-code --scope project
qodyn skill update qodyn.angular-implementation
qodyn skill remove qodyn.angular-implementation
qodyn skill validate ./my-skill
qodyn skill pack ./my-skill
qodyn skill publish ./my-skill
qodyn skill deprecate qodyn.angular-implementation@1.0.0
qodyn skill rollback qodyn.angular-implementation --to 1.0.3
qodyn project sync
qodyn project audit
qodyn adapters list
qodyn policies check
```

### 14.3 Compatibilidade

Manter compatibilidade com os comandos atuais do `agent-skills` quando
possível.

Exemplo:

``` bash
agent-skills install -s aws-advisor -a cursor
```

Pode continuar funcionando, mas internamente deve usar o novo core.

### 14.4 Config local

``` text
~/.config/qodyn-skills/config.json
```

Exemplo:

``` json
{
  "registryUrl": "https://skills.company.com",
  "token": "***",
  "defaultPlatforms": ["claude-code"],
  "telemetry": true
}
```

------------------------------------------------------------------------

## 15. Workflow de Publicação

### 15.1 Lacuna Atual

Hoje o modelo é contribuição via repo/PR/release. Para enterprise,
precisa haver publicação via CLI/API/UI.

### 15.2 Fluxo alvo

``` text
Criador cria Skill
  -> valida localmente
  -> empacota
  -> publica no Registry
  -> scan automático
  -> workflow de aprovação
  -> Skill aprovada
  -> disponível para instalação
```

### 15.3 Estados

``` text
draft
uploaded
validation_failed
security_failed
pending_review
changes_requested
approved
published
deprecated
blocked
archived
```

### 15.4 Comando de publicação

``` bash
qodyn skill publish ./skills/angular-implementation --channel beta
```

### 15.5 Resposta esperada

``` text
Validating manifest... ok
Validating structure... ok
Packing skill... ok
Uploading package... ok
Security scan queued... ok
Approval request created: APR-1024
Status: pending_review
```

------------------------------------------------------------------------

## 16. Workflow de Aprovação

### 16.1 Requisitos

Cada Skill pode exigir:

-   aprovação técnica;
-   aprovação de segurança;
-   aprovação de owner;
-   aprovação de compliance;
-   aprovação automática para patch seguro.

### 16.2 Regras sugeridas

  Tipo de mudança                Aprovação técnica     Aprovação segurança
  ---------------------------- ------------------- -----------------------
  PATCH sem permissões novas              Opcional                     Não
  MINOR                                        Sim   Se alterar permissões
  MAJOR                                        Sim                     Sim
  Nova Skill                                   Sim                     Sim
  Script novo                                  Sim         Sim obrigatório
  Network enabled                              Sim         Sim obrigatório

### 16.3 Diff de aprovação

O sistema deve mostrar:

-   diff do `SKILL.md`;
-   diff do `skill.yaml`;
-   permissões alteradas;
-   dependências novas;
-   scripts adicionados;
-   findings de segurança;
-   projetos afetados.

------------------------------------------------------------------------

## 17. RBAC e Governança

### 17.1 Papéis

``` text
viewer
consumer
developer
publisher
technical_reviewer
security_reviewer
admin
```

### 17.2 Permissões

  Ação                    Viewer   Consumer   Publisher   Reviewer   Admin
  --------------------- -------- ---------- ----------- ---------- -------
  Ver Skills públicas        Sim        Sim         Sim        Sim     Sim
  Instalar Skill             Não        Sim         Sim        Sim     Sim
  Publicar Skill             Não        Não         Sim        Sim     Sim
  Aprovar Skill              Não        Não         Não        Sim     Sim
  Bloquear Skill             Não        Não         Não        Não     Sim
  Gerenciar policies         Não        Não         Não        Não     Sim

### 17.3 Ownership

Toda Skill deve ter:

-   owner team;
-   maintainer principal;
-   canal de suporte;
-   SLA opcional;
-   data de última revisão.

------------------------------------------------------------------------

## 18. Security Scanner Enterprise

### 18.1 Lacuna Atual

O repo já tem uma base de segurança forte, mas o PRD exige segurança
centralizada, política por ambiente, assinatura e bloqueio via registry.

### 18.2 Manter do repo atual

Preservar:

-   sanitização;
-   path isolation;
-   symlink guard;
-   lockfile atômico;
-   audit local;
-   hashing;
-   Snyk Agent Scan;
-   allowlist com expiração.

### 18.3 Expandir

Criar:

``` text
packages/security-scanner
```

Com scanners:

-   estrutura;
-   manifesto;
-   prompt injection;
-   secret access;
-   network calls;
-   command risk;
-   dependency risk;
-   license risk;
-   file size risk;
-   binary detection;
-   MCP permission risk.

### 18.4 Risk levels

``` text
none
low
medium
high
critical
```

### 18.5 Bloqueios automáticos

Bloquear se:

-   houver binário não permitido;
-   script destrutivo;
-   tentativa de exfiltrar `.env`;
-   instrução para ignorar políticas;
-   network enabled sem permissão;
-   dependência bloqueada;
-   assinatura inválida;
-   checksum divergente.

### 18.6 Relatório de scan

``` json
{
  "status": "failed",
  "riskLevel": "high",
  "findings": [
    {
      "code": "CMD_001",
      "severity": "high",
      "message": "Command pattern 'curl | bash' is not allowed",
      "file": "scripts/install.sh",
      "line": 12
    }
  ]
}
```

------------------------------------------------------------------------

## 19. Policy Engine

### 19.1 Objetivo

Permitir que empresas controlem quais Skills podem ser usadas em quais
contextos.

### 19.2 Exemplos de políticas

``` yaml
id: only-certified-in-production
name: Only certified skills in production
condition:
  environment: production
effect: deny
unless:
  skill.security.trustLevel: certified
```

``` yaml
id: no-network-for-external-skills
name: External skills cannot use network
effect: deny
when:
  skill.publisher.type: external
  skill.permissions.network.enabled: true
```

``` yaml
id: security-approval-required-for-scripts
name: Script execution requires security approval
effect: require_approval
when:
  skill.permissions.scripts.allowExecution: true
approval:
  security: 1
```

### 19.3 Pacote

``` text
packages/policy-engine
```

### 19.4 Integrações

Policy engine deve ser usado em:

-   publicação;
-   aprovação;
-   instalação;
-   update;
-   rollback;
-   CI. 

------------------------------------------------------------------------

## 20. Telemetria Centralizada

### 20.1 Lacuna Atual

O repo tem audit local, mas o PRD exige métricas centralizadas.

### 20.2 Eventos

``` text
skill.searched
skill.viewed
skill.installed
skill.install_failed
skill.updated
skill.removed
skill.rollback
skill.published
skill.approved
skill.rejected
skill.scan_failed
adapter.transform_failed
policy.denied
```

### 20.3 Payload

``` json
{
  "event": "skill.installed",
  "timestamp": "2026-07-04T12:00:00Z",
  "skillId": "qodyn.angular-implementation",
  "version": "1.2.0",
  "platform": "claude-code",
  "scope": "project",
  "projectId": "proj_123",
  "userId": "usr_123",
  "success": true
}
```

### 20.4 Privacidade

-   Não enviar conteúdo de código-fonte.
-   Não enviar prompts do usuário.
-   Permitir opt-out local para ambientes sensíveis.
-   Mas manter auditoria obrigatória em enterprise, conforme política.

------------------------------------------------------------------------

## 21. Skill Packs

### 21.1 Lacuna Atual

Não há abstração madura de Skill Pack.

### 21.2 Solução

Criar `kind: SkillPack`.

### 21.3 Exemplo

``` yaml
apiVersion: qodyn.io/v1
kind: SkillPack
metadata:
  id: qodyn.angular-enterprise-pack
  name: Angular Enterprise Pack
  description: Pack completo para implementação Angular enterprise.
version:
  semver: 1.0.0
skills:
  - id: qodyn.angular-implementation
    version: "^1.0.0"
  - id: qodyn.typescript-standards
    version: "^1.0.0"
  - id: qodyn.testing-standards
    version: "^1.0.0"
  - id: qodyn.accessibility-review
    version: "^1.0.0"
```

### 21.4 CLI

``` bash
qodyn pack install qodyn.angular-enterprise-pack --platform claude-code
```

------------------------------------------------------------------------

## 22. MCP Enterprise

### 22.1 Lacuna Atual

O MCP atual é read-only e usa catálogo público/CDN. Isso é bom, mas em
ambiente enterprise será necessário suportar registry privado,
autenticação e políticas.

### 22.2 Novos modos

``` text
public-cdn
private-registry
hybrid
offline-cache
```

### 22.3 Ferramentas MCP alvo

Manter:

-   `list_skills`;
-   `search_skills`;
-   `read_skill`;
-   `fetch_skill_files`.

Adicionar:

-   `recommend_skills`;
-   `get_skill_manifest`;
-   `get_skill_versions`;
-   `check_skill_policy`;
-   `get_project_skills`;
-   `fetch_skill_pack`.

### 22.4 Segurança

-   MCP não deve escrever no filesystem.
-   MCP deve respeitar policy engine.
-   MCP deve usar token quando conectado a registry privado.
-   MCP deve ocultar Skills não autorizadas ao usuário.

------------------------------------------------------------------------

## 23. Integrações

### 23.1 GitHub

Funcionalidades:

-   associar projeto a repositório;
-   detectar stack;
-   abrir PR com instalação de Skills;
-   validar `skills.lock` em CI;
-   bloquear PR com Skill não aprovada;
-   publicar Skill por GitHub Action.

### 23.2 Jira

Funcionalidades:

-   sugerir Skills por tipo de tarefa;
-   anexar Skills recomendadas na issue;
-   criar rastreabilidade task -\> Skill -\> PR.

### 23.3 Figma

Funcionalidades:

-   validar Skills que usam MCP Figma;
-   mapear componentes de design system;
-   sugerir Skill `figma-to-angular` ou equivalente.

### 23.4 SSO

Suportar:

-   Google Workspace;
-   Azure AD;
-   Okta;
-   SAML/OIDC.

------------------------------------------------------------------------

## 24. API Client / SDK

### 24.1 Pacote

``` text
packages/registry-client
```

### 24.2 Métodos

``` ts
client.skills.search(query)
client.skills.get(skillId)
client.skills.getVersions(skillId)
client.skills.publish(packagePath)
client.skills.download(skillId, version)
client.install.resolve(planInput)
client.telemetry.send(event)
client.policies.evaluate(input)
```

### 24.3 Uso pelo CLI

O CLI não deve acessar endpoints manualmente. Deve usar
`registry-client`.

------------------------------------------------------------------------

## 25. Observabilidade

### 25.1 Logs estruturados

Usar JSON logs.

Campos mínimos:

``` text
traceId
requestId
userId
action
entityType
entityId
status
durationMs
```

### 25.2 Métricas

``` text
skills_total
skill_versions_total
skills_installed_total
skill_install_failures_total
adapter_transform_failures_total
security_scan_failures_total
approval_pending_total
policy_denials_total
deprecated_versions_in_use_total
```

### 25.3 Traces

Instrumentar:

-   upload;
-   scan;
-   approval;
-   install plan;
-   download;
-   adapter transform.

------------------------------------------------------------------------

## 26. CI/CD

### 26.1 Pipelines

Criar workflows:

``` text
ci.yml
release.yml
security-scan.yml
publish-skill.yml
publish-api.yml
publish-web.yml
```

### 26.2 Gates obrigatórios

-   lint;
-   typecheck;
-   unit tests;
-   integration tests;
-   manifest validation;
-   security scan;
-   adapter tests;
-   package integrity tests.

### 26.3 Ambientes

``` text
dev
staging
production
```

------------------------------------------------------------------------

## 27. Plano de Migração em Fases

### Fase 0 --- Fork e baseline

Objetivo:

-   fork do repo;
-   renomear branding;
-   manter CLI funcionando;
-   mapear arquitetura atual;
-   travar versão base.

Entregas:

-   repo interno;
-   build passando;
-   testes atuais passando;
-   documentação de baseline.

### Fase 1 --- Manifesto Enterprise

Objetivo:

-   adicionar `skill.yaml`;
-   criar parser/validator;
-   migrar catálogo existente.

Entregas:

-   pacote `manifest`;
-   comando de migração;
-   schemas Zod;
-   docs de formato.

### Fase 2 --- Adapter SDK

Objetivo:

-   formalizar adapters;
-   refatorar instalação atual para usar Adapter SDK.

Entregas:

-   `adapter-sdk`;
-   adapters Claude, Codex, Cursor;
-   testes por adapter;
-   CLI usando adapters.

### Fase 3 --- Registry API MVP

Objetivo:

-   criar API;
-   criar banco;
-   expor busca e download.

Entregas:

-   `apps/api`;
-   PostgreSQL;
-   Prisma;
-   endpoints de Skills;
-   upload/download.

### Fase 4 --- CLI conectado ao Registry

Objetivo:

-   CLI usa API;
-   login;
-   instalação via registry;
-   lockfile novo.

Entregas:

-   `registry-client`;
-   auth token;
-   install plan;
-   dependency resolver.

### Fase 5 --- Marketplace Web

Objetivo:

-   criar UI de catálogo, detalhe e publicação.

Entregas:

-   `apps/web`;
-   login;
-   catálogo;
-   detalhe;
-   upload;
-   dashboard inicial.

### Fase 6 --- Governança

Objetivo:

-   aprovação;
-   RBAC;
-   policies;
-   scan central.

Entregas:

-   approval service;
-   policy engine;
-   security scanner enterprise;
-   dashboard admin.

### Fase 7 --- Enterprise Scale

Objetivo:

-   métricas;
-   integrações;
-   skill packs;
-   ambientes;
-   SSO.

Entregas:

-   telemetry service;
-   GitHub/Jira/Figma;
-   skill packs;
-   SSO.

------------------------------------------------------------------------

## 28. Épicos e User Stories

### EPIC 1 --- Manifesto Enterprise

**Objetivo:** criar `skill.yaml` como contrato do marketplace.

User stories:

-   Como publisher, quero declarar metadados completos da Skill.
-   Como registry, quero validar manifesto antes de aceitar pacote.
-   Como CLI, quero usar manifesto para saber compatibilidade e
    permissões.

Critérios de aceite:

-   Schema validado com Zod.
-   Erros claros de validação.
-   Documentação com exemplos.
-   Migração automática das Skills atuais.

### EPIC 2 --- Adapter SDK

**Objetivo:** permitir suporte plugável a plataformas.

User stories:

-   Como desenvolvedor, quero instalar a mesma Skill no Claude e Cursor.
-   Como mantenedor, quero adicionar Adapter novo sem alterar o core.

Critérios de aceite:

-   Interface padrão implementada.
-   Claude Adapter funcionando.
-   Cursor Adapter convertendo `.mdc`.
-   Codex Adapter funcionando.

### EPIC 3 --- Registry API

**Objetivo:** criar backend de Skills.

User stories:

-   Como usuário, quero buscar Skills no registry.
-   Como publisher, quero publicar uma versão.
-   Como CLI, quero baixar pacote validado.

Critérios de aceite:

-   API com autenticação.
-   Banco persistente.
-   Upload/download funcionando.
-   Package checksum validado.

### EPIC 4 --- Marketplace Web

**Objetivo:** criar interface visual.

User stories:

-   Como consumidor, quero explorar Skills por categoria.
-   Como arquiteto, quero ver detalhes, permissões e dependências.
-   Como publisher, quero publicar Skill pela UI.

Critérios de aceite:

-   Catálogo funcional.
-   Detalhe funcional.
-   Upload funcional.
-   Aprovação visível.

### EPIC 5 --- Governança e Segurança

**Objetivo:** controlar uso seguro.

User stories:

-   Como segurança, quero bloquear Skills perigosas.
-   Como admin, quero criar políticas.
-   Como reviewer, quero aprovar ou rejeitar versões.

Critérios de aceite:

-   Scanner central.
-   Policy engine.
-   Workflow de aprovação.
-   RBAC.

### EPIC 6 --- Telemetria

**Objetivo:** medir uso e adoção.

User stories:

-   Como gestor, quero saber quais Skills são mais usadas.
-   Como arquiteto, quero saber quais projetos usam versões antigas.

Critérios de aceite:

-   Eventos de instalação enviados.
-   Dashboard com métricas.
-   Relatório de Skills deprecated em uso.

------------------------------------------------------------------------

## 29. Critérios Gerais de Aceite

A solução será considerada aderente ao PRD quando:

1.  toda Skill tiver `skill.yaml` válido;
2.  o catálogo atual estiver migrado;
3.  o CLI instalar Skills via registry privado;
4.  houver adapters formais para pelo menos Claude, Codex e Cursor;
5.  houver API com busca, download, upload e versionamento;
6.  houver UI de catálogo e detalhe;
7.  houver workflow de aprovação;
8.  houver RBAC mínimo;
9.  houver scan central;
10. houver dependency resolver;
11. houver lockfile com SemVer e checksum;
12. houver telemetria centralizada;
13. houver audit central;
14. policy engine bloquear instalações inválidas;
15. skill packs forem suportados.

------------------------------------------------------------------------

## 30. Riscos e Mitigações

### Risco 1 --- Quebrar o CLI atual

Mitigação:

-   criar camada de compatibilidade;
-   manter comandos antigos;
-   introduzir novo core gradualmente.

### Risco 2 --- Complexidade excessiva

Mitigação:

-   fases incrementais;
-   primeiro manifest/adapters;
-   front/back somente depois.

### Risco 3 --- Lock-in no registry próprio

Mitigação:

-   suportar registry local;
-   suportar Git/CDN;
-   manter export/import de pacotes.

### Risco 4 --- Segurança de Skills externas

Mitigação:

-   scan obrigatório;
-   trust levels;
-   approval;
-   policies;
-   assinatura.

### Risco 5 --- Baixa adoção interna

Mitigação:

-   manter CLI simples;
-   criar packs por stack;
-   gerar comandos de instalação na UI;
-   integrar com GitHub/Jira.

------------------------------------------------------------------------

## 31. Decisões Técnicas Recomendadas

1.  **Manter `SKILL.md` como conteúdo principal.**
2.  **Adicionar `skill.yaml` como contrato enterprise.**
3.  **Não jogar fora o CLI atual. Refatorar.**
4.  **Criar Adapter SDK antes do front-end.**
5.  **Criar registry API antes do marketplace visual completo.**
6.  **Usar PostgreSQL e storage S3-compatible.**
7.  **Usar Fastify para MVP back-end.**
8.  **Usar Next.js para marketplace web.**
9.  **Criar migration tool para catálogo atual.**
10. **Manter MCP read-only por padrão.**
11. **Fazer policy engine desde cedo, mesmo simples.**
12. **Tratar Skills como artefatos de software, não como prompts
    soltos.**

------------------------------------------------------------------------

## 32. Backlog Técnico Inicial

### Sprint 1

-   Fork interno.
-   Rodar build/testes.
-   Mapear CLI atual.
-   Criar pacote `manifest`.
-   Criar schema `skill.yaml`.

### Sprint 2

-   Criar migration command.
-   Migrar 10 Skills piloto.
-   Criar validação de manifesto.
-   Criar docs de Skill format.

### Sprint 3

-   Criar Adapter SDK.
-   Refatorar Claude install.
-   Refatorar Cursor install.
-   Criar testes de adapter.

### Sprint 4

-   Criar Registry API MVP.
-   Criar banco Prisma.
-   Criar endpoints `GET /skills` e `GET /skills/:id`.
-   Criar package upload/download.

### Sprint 5

-   CLI login.
-   CLI usando registry.
-   Download por API.
-   Lockfile novo.

### Sprint 6

-   Front catálogo.
-   Front detalhe.
-   Install command box.
-   Visualizar manifesto.

------------------------------------------------------------------------

## 33. Conclusão

O repo `tech-leads-club/agent-skills` é uma excelente base para acelerar
a implementação do PRD. Ele já resolve bem a parte de catálogo, CLI,
instalação multiagente, MCP, cache, lockfile e segurança local.

Mas o PRD pede uma plataforma enterprise completa. Para chegar lá, os
principais ajustes são:

``` text
Adicionar skill.yaml
Criar Registry API
Criar Marketplace Web
Formalizar Adapter SDK
Criar Dependency Resolver
Criar Governança/RBAC
Criar Security Scanner central
Criar Policy Engine
Criar Telemetria centralizada
Criar Skill Packs
Criar integrações corporativas
```

A melhor estratégia é evoluir em camadas, preservando o que já funciona.

O caminho correto não é substituir o repo.

É transformá-lo no **motor inicial** de uma plataforma maior.

------------------------------------------------------------------------

# Atualização v2 do SPEC --- Agent Skills como padrão canônico

## Decisão arquitetural revisada

Esta versão substitui a premissa de que a plataforma precisa definir um
formato canônico proprietário de Skill. **Agent Skills passa a ser o
padrão canônico de empacotamento e descrição**, enquanto a Qodyn
implementa a camada enterprise de gerenciamento, governança,
distribuição, compatibilidade e observabilidade.

A arquitetura passa a seguir:

``` text
Agent Skills Standard
        |
Compatibility Layer
        |
Qodyn Skill Hub
  |        |         |
Registry Governance Marketplace
  |
Distribution Engine
  |
Adapter SDK
  |
Claude / Codex / Cursor / Gemini / Copilot / ...
```

## Alterações obrigatórias no modelo anterior

1.  `SKILL.md` permanece a fonte portátil da Skill.
2.  `skill.yaml` deixa de ser obrigatório. Metadata enterprise reside
    prioritariamente no Registry; um `qodyn.yaml` pode existir apenas
    como extensão opcional.
3.  Adapters passam a trabalhar em modos `native`, `pass-through`,
    `transformed`, `partial` e `unsupported`.
4.  Plataformas com suporte nativo a Agent Skills não devem sofrer
    transformação desnecessária.
5.  Dependências entre Skills são extensão Qodyn e não parte obrigatória
    do padrão.
6.  O Registry deve importar Skills de múltiplas fontes, inclusive
    Git/GitHub e catálogos externos.
7.  Toda Skill importada deve manter proveniência: source, upstream,
    commit/ref, hash, publisher original e licença.
8.  O produto deve suportar `mirror`, `fork` e `overlay`, com detecção
    de novas versões upstream e diff.
9.  Deve existir uma `Agent Skills Compatibility Layer` com testes de
    conformidade e warnings de portabilidade.
10. O CLI deve utilizar uma abstração `RegistryProvider`, permitindo
    coexistência de registry JSON/CDN e Registry API.
11. A configuração simples de diretórios por agente deve evoluir para
    Adapter SDK formal.
12. O lockfile deve registrar source, upstream ref, hash, assinatura,
    adapter, versão do adapter e compatibility mode.
13. Segurança deve incluir provenance, assinatura, source trust, análise
    de permissões, scripts, dependências e quarentena.
14. O marketplace deve distinguir metadata do padrão, metadata Qodyn,
    origem, upstream, compatibilidade e security findings.
15. Policies devem poder restringir fontes, trust levels, scripts,
    network/filesystem/MCP, adapters transformed e versões deprecated.

## Source Management

Fontes suportadas: - Qodyn first-party; - organização privada; -
GitHub/Git; - `tech-leads-club/agent-skills`; - catálogos aprovados; -
upload; - futura federation.

Campos mínimos: `source_type`, `source_uri`, `upstream_id`,
`upstream_version/ref`, `commit_sha`, `content_hash`, `license`,
`publisher_original`, `imported_at`, `last_sync_at`, `sync_policy`.

## Upstream, Fork e Overlay

-   **Mirror:** cópia imutável de uma versão upstream.
-   **Fork:** derivação mantida independentemente.
-   **Overlay:** customização corporativa aplicada sobre upstream
    preservando a relação.

O sistema deve detectar updates upstream, mostrar diff, identificar
conflitos e exigir policy/aprovação antes de promover mudanças
relevantes.

## Compatibility Layer

Estados: - `compatible`; - `compatible_with_warnings`; -
`requires_extension`; - `incompatible`; - `unknown_standard_version`.

Deve validar estrutura, frontmatter, recursos, extensões e produzir
portability score.

## Adapter SDK revisado

``` ts
interface SkillAdapter {
  key: string;
  detect(target: Target): Promise<DetectionResult>;
  assess(skill: SkillPackage, target: Target): Promise<CompatibilityAssessment>;
  transform?(skill: SkillPackage, target: Target): Promise<TransformedPackage>;
  planInstall(skill: SkillPackage, target: Target): Promise<InstallPlan>;
  install(plan: InstallPlan): Promise<InstallResult>;
  verify(result: InstallResult): Promise<VerificationResult>;
  uninstall(target: Target, skillId: string): Promise<UninstallResult>;
  listInstalled(target: Target): Promise<InstalledSkill[]>;
}
```

Requisitos: idempotência, dry-run, rollback, atomicidade, paths seguros,
logs estruturados, versionamento e conformance tests.

## Registry e APIs adicionais

Adicionar recursos para: - `/sources`; - `/sources/{id}/sync`; -
`/imports`; - `/compatibility`; - `/upstream`; - `/diff`; -
`/policies`; - `/install/plan`; - `/installations`; - `/audit`.

Operações mutáveis relevantes devem suportar idempotency keys.

## Modelo de dados adicional

Adicionar: - `sources`; - `upstream_links`; - `overlays`; -
`sync_events`; - `compatibility_results`; - `policies`; -
`package_signatures`; - `installation_inventory`.

## Segurança e supply chain

Além dos controles já definidos: - pacote imutável; - SHA-256; -
assinatura; - identidade do publisher/build; - commit/ref de origem; -
source allow/deny; - license/provenance; - archive/zip-bomb
validation; - sandbox opcional; - SBOM quando aplicável; - waiver de
finding com expiração.

## CLI revisado

Adicionar:

``` bash
qodyn source add
qodyn source sync
qodyn skill import
qodyn skill diff
qodyn skill rollback
qodyn skill sync --frozen-lockfile
qodyn policy check
qodyn adapter list
qodyn adapter doctor
```

Todos os comandos de CI devem suportar `--json`, `--non-interactive`,
códigos de saída estáveis e `--dry-run` quando aplicável.

## Critérios de aceite adicionais

-   Importar uma Agent Skill externa sem modificar seu `SKILL.md`.
-   Cadastrar `tech-leads-club/agent-skills` como source.
-   Registrar hash, commit/ref, licença e publisher original.
-   Detectar atualização upstream.
-   Exibir diff upstream → versão corporativa.
-   Instalar em pelo menos duas plataformas em modo native/pass-through.
-   Instalar em uma plataforma em modo transformed.
-   Bloquear Skill por policy/security.
-   Validar assinatura antes da instalação.
-   Reproduzir instalação via lockfile.
-   Migrar progressivamente do registry JSON baseline para Registry API
    sem big-bang rewrite.

## Estratégia de migração do repo baseline

1.  Manter registry JSON e CLI existentes.
2.  Introduzir `RegistryProvider`.
3.  Adicionar Compatibility Layer.
4.  Implementar Qodyn API provider em paralelo.
5.  Migrar agent mappings para adapters gradualmente.
6.  Evoluir lockfile para v2 com migration automática.
7.  Migrar marketplace estático para API.
8.  Adicionar Source Management e upstream tracking.
9.  Adicionar governança, policy, approvals e telemetria.
10. Manter registry estático como provider read-only durante a
    transição.

## Posicionamento final

``` text
Agent Skills = padrão aberto da Skill
tech-leads-club/agent-skills = baseline/acelerador
Qodyn Skill Hub = control plane enterprise
Adapters = integração com plataformas
CLI/API/MCP = canais de consumo
```
