# PRD --- Skill Marketplace & Multi-Agent Adapter Platform

## 1. Nome do Produto

**Skill Marketplace Platform**

Nome interno sugerido: **Qodyn Skill Hub**

------------------------------------------------------------------------

## 2. Resumo Executivo

O produto será uma plataforma para **criar, versionar, validar,
publicar, instalar e gerenciar Skills de IA** compatíveis com múltiplas
plataformas de agentes, como Claude Code, Codex CLI, Cursor, Gemini CLI
e futuras ferramentas.

A proposta é separar a Skill da plataforma onde ela será usada.

Em vez de criar uma Skill específica para Claude, outra para Codex e
outra para Cursor, a empresa manterá uma **Skill canônica** em um
formato próprio baseado no padrão `SKILL.md`, enriquecida com um
`skill.yaml`. A partir disso, uma camada de **Adapters** converte e
instala essa Skill no formato esperado por cada plataforma.

O produto começa como um **marketplace privado corporativo**, mas deve
nascer preparado para virar marketplace público no futuro.

------------------------------------------------------------------------

# 3. Problema

Empresas que usam agentes de IA em desenvolvimento de software enfrentam
problemas claros.

## 3.1 Conhecimento espalhado

Cada time cria seus próprios prompts, arquivos de regras, instruções e
workflows.

Resultado:

-   baixa padronização;
-   retrabalho;
-   baixa governança;
-   qualidade variável;
-   dependência de pessoas específicas.

## 3.2 Skills presas a plataformas

Uma instrução criada para Claude Code pode não funcionar bem no Codex
CLI ou Cursor.

Resultado:

-   duplicação de conteúdo;
-   manutenção difícil;
-   lock-in de fornecedor;
-   dificuldade para migrar ou testar novas plataformas.

## 3.3 Falta de governança

Sem registry central, a empresa não sabe:

-   quais Skills existem;
-   quem criou;
-   qual versão está em uso;
-   se passou por revisão;
-   se tem comandos perigosos;
-   se acessa dados sensíveis;
-   qual projeto usa qual Skill.

## 3.4 Falta de versionamento

Hoje muitos times usam "a última versão do prompt".

Isso é perigoso.

Uma alteração pequena em uma Skill pode mudar o comportamento do agente
em produção.

## 3.5 Falta de métricas

A empresa não mede:

-   quais Skills são mais usadas;
-   quais reduzem retrabalho;
-   quais geram falhas;
-   quais precisam de melhoria;
-   quais têm maior custo em tokens;
-   quais aumentam aprovação em code review.

------------------------------------------------------------------------

# 4. Objetivo do Produto

Criar uma plataforma para transformar Skills de agentes em ativos
corporativos reutilizáveis, versionados, auditáveis e instaláveis em
múltiplas plataformas.

## 4.1 Objetivos principais

-   Criar um catálogo central de Skills.
-   Permitir publicação e curadoria de Skills.
-   Suportar versionamento semântico.
-   Resolver dependências entre Skills.
-   Instalar Skills em diferentes plataformas via Adapters.
-   Permitir escopo global, por usuário, por time e por projeto.
-   Validar segurança e qualidade das Skills.
-   Medir uso, impacto e adoção.
-   Preparar base para marketplace público.

## 4.2 Objetivos secundários

-   Criar CLI para instalação local.
-   Criar API para integração com IDEs e pipelines.
-   Criar SDK para novos Adapters.
-   Criar fluxo de aprovação corporativa.
-   Criar ranking interno de qualidade.
-   Criar packs de Skills por stack ou objetivo.

------------------------------------------------------------------------

# 5. Não Objetivos da Primeira Versão

A primeira versão não deve tentar resolver tudo.

Fora do MVP:

-   execução autônoma de agentes;
-   orquestração completa multiagente;
-   billing público;
-   venda externa de Skills;
-   marketplace aberto à comunidade;
-   execução remota de scripts de terceiros;
-   avaliação automática profunda com LLM judge;
-   integração obrigatória com todos os agentes do mercado.

O MVP deve provar uma coisa:

> Uma Skill canônica pode ser instalada e usada em múltiplas plataformas
> com controle de versão, dependências e governança mínima.

------------------------------------------------------------------------

# 6. Público-Alvo

## 6.1 Usuário primário

Desenvolvedores que usam agentes de IA para implementar, revisar, testar
e documentar software.

## 6.2 Usuário secundário

Arquitetos de software, tech leads e times de plataforma que querem
padronizar práticas técnicas.

## 6.3 Usuário administrativo

Gestores de engenharia, segurança, compliance e governança técnica.

## 6.4 Usuário futuro

Criadores externos de Skills que poderão publicar no marketplace
público.

------------------------------------------------------------------------

# 7. Personas

## 7.1 Desenvolvedor

Quer instalar rapidamente Skills úteis no projeto.

Exemplo:

``` bash
qodyn skill install qodyn.angular-implementation --platform claude-code --scope project
```

Necessidades:

-   facilidade;
-   rapidez;
-   documentação clara;
-   compatibilidade com ferramenta atual;
-   rollback se algo der errado.

## 7.2 Arquiteto

Quer criar e aprovar Skills para padronizar práticas da empresa.

Necessidades:

-   revisão;
-   versionamento;
-   dependências;
-   critérios de qualidade;
-   aprovação antes de liberar.

## 7.3 Gestor de Engenharia

Quer saber se a adoção está funcionando.

Necessidades:

-   métricas de uso;
-   projetos com Skills instaladas;
-   times mais ativos;
-   Skills mais adotadas;
-   indicadores de impacto.

## 7.4 Segurança

Quer evitar Skills maliciosas ou perigosas.

Necessidades:

-   scanner;
-   permissões declarativas;
-   bloqueio de comandos;
-   auditoria;
-   aprovação obrigatória.

------------------------------------------------------------------------

# 8. Conceito Central

## 8.1 Skill Canônica

A Skill será criada uma única vez em formato universal.

Estrutura:

``` text
skills/
  angular-implementation/
    skill.yaml
    SKILL.md
    README.md
    examples/
    templates/
    scripts/
    tests/
```

O `SKILL.md` segue o padrão aberto de Agent Skills: uma pasta contendo
instruções para o agente executar uma tarefa específica.

O `skill.yaml` será específico da plataforma e conterá dados de
marketplace, governança, compatibilidade, dependências e permissões.

## 8.2 Adapter

O Adapter converte a Skill canônica para o formato esperado por cada
plataforma.

Exemplo:

``` text
Skill canônica
    ↓
ClaudeCodeAdapter
    ↓
.claude/skills/angular-implementation/SKILL.md
```

``` text
Skill canônica
    ↓
CursorAdapter
    ↓
.cursor/rules/angular-implementation.mdc
```

``` text
Skill canônica
    ↓
CodexAdapter
    ↓
.codex/skills/angular-implementation/SKILL.md
```

------------------------------------------------------------------------

# 9. Visão do Produto

## 9.1 Em uma frase

Um marketplace corporativo para distribuir Skills de agentes com
versionamento, segurança, governança e instalação multiplataforma.

## 9.2 Analogia

O produto deve funcionar como:

-   **npm** para Skills;
-   **Docker Hub** para pacotes de capacidades;
-   **GitHub Marketplace** para agentes;
-   **App Store corporativa** para workflows de IA.

------------------------------------------------------------------------

# 10. Escopo Funcional

## 10.1 Catálogo de Skills

O usuário deve conseguir:

-   listar Skills disponíveis;
-   buscar por nome;
-   filtrar por categoria;
-   filtrar por plataforma;
-   filtrar por stack;
-   filtrar por status;
-   visualizar detalhes;
-   visualizar versões;
-   visualizar dependências;
-   visualizar permissões;
-   visualizar exemplos de uso.

## 10.2 Publicação de Skill

O usuário autorizado deve conseguir:

-   criar Skill;
-   subir pacote;
-   validar manifesto;
-   validar estrutura;
-   publicar versão;
-   enviar para aprovação;
-   despublicar versão;
-   arquivar Skill.

## 10.3 Instalação de Skill

Via CLI ou API, o usuário deve conseguir:

-   instalar Skill por plataforma;
-   instalar dependências automaticamente;
-   instalar em escopo global;
-   instalar em escopo de projeto;
-   gerar lockfile;
-   atualizar Skill;
-   remover Skill;
-   fazer rollback.

## 10.4 Gestão de dependências

A plataforma deve:

-   ler dependências do `skill.yaml`;
-   resolver árvore de dependências;
-   detectar conflito de versão;
-   instalar dependências antes da Skill principal;
-   bloquear dependências não aprovadas;
-   gerar `skills.lock`.

## 10.5 Adapters

A plataforma deve suportar Adapters plugáveis.

Adapters iniciais:

-   Claude Code;
-   Codex CLI;
-   Cursor.

Adapters futuros:

-   Gemini CLI;
-   GitHub Copilot;
-   Windsurf;
-   OpenClaw;
-   agentes internos.

## 10.6 Governança

A plataforma deve controlar:

-   status da Skill;
-   proprietário;
-   aprovadores;
-   revisão técnica;
-   revisão de segurança;
-   changelog;
-   histórico de publicação;
-   compatibilidade;
-   depreciação.

## 10.7 Segurança

A plataforma deve validar:

-   comandos perigosos;
-   scripts não declarados;
-   tentativa de ler arquivos sensíveis;
-   instruções de bypass;
-   prompt injection;
-   permissões excessivas;
-   dependências desconhecidas;
-   checksum do pacote.

## 10.8 Métricas

A plataforma deve medir:

-   número de instalações;
-   projetos usando cada Skill;
-   versões mais usadas;
-   falhas de instalação;
-   downloads;
-   avaliações;
-   tempo desde última atualização;
-   Skills obsoletas;
-   dependências mais usadas.

------------------------------------------------------------------------

# 11. Requisitos do Front-End

## 11.1 Aplicação Web

A aplicação web será o marketplace visual.

Stack sugerida:

-   Next.js;
-   React;
-   TypeScript;
-   Tailwind CSS;
-   shadcn/ui;
-   TanStack Query;
-   Zod;
-   Auth.js ou integração com SSO corporativo.

## 11.2 Páginas principais

### 11.2.1 Home

Deve exibir:

-   busca principal;
-   Skills em destaque;
-   categorias;
-   packs recomendados;
-   últimas atualizações;
-   Skills mais usadas;
-   botão "Publicar Skill".

### 11.2.2 Listagem de Skills

Filtros:

-   categoria;
-   plataforma;
-   publisher;
-   status;
-   maturidade;
-   stack;
-   compatibilidade;
-   licença;
-   avaliação;
-   data de atualização.

Cards devem exibir:

-   nome;
-   descrição curta;
-   categoria;
-   publisher;
-   versão atual;
-   plataformas suportadas;
-   status;
-   número de instalações;
-   score de qualidade.

### 11.2.3 Detalhe da Skill

Deve exibir:

-   nome;
-   descrição;
-   versão;
-   publisher;
-   owner;
-   status;
-   compatibilidade;
-   comando de instalação;
-   documentação;
-   conteúdo do `SKILL.md`;
-   manifesto;
-   dependências;
-   permissões;
-   changelog;
-   versões anteriores;
-   exemplos;
-   avaliações;
-   histórico de aprovação.

### 11.2.4 Publicação de Skill

Fluxo:

1.  Upload de pacote `.zip` ou `.tgz`.
2.  Leitura do `skill.yaml`.
3.  Validação estrutural.
4.  Validação de segurança.
5.  Pré-visualização.
6.  Submissão para aprovação.

### 11.2.5 Dashboard Administrativo

Deve exibir:

-   Skills pendentes de aprovação;
-   Skills bloqueadas;
-   Skills obsoletas;
-   versões em uso;
-   instalações por time;
-   alertas de segurança;
-   dependências vulneráveis.

### 11.2.6 Página de Projeto

Deve permitir ver:

-   Skills instaladas por projeto;
-   versões;
-   plataformas;
-   lockfile;
-   última sincronização;
-   conflitos;
-   recomendações.

## 11.3 Componentes de UI

Componentes necessários:

-   SkillCard;
-   SkillSearch;
-   PlatformBadge;
-   VersionSelector;
-   InstallCommandBox;
-   ManifestViewer;
-   SkillMarkdownViewer;
-   DependencyGraph;
-   PermissionPanel;
-   ApprovalTimeline;
-   SecurityScanResult;
-   ChangelogPanel;
-   RatingPanel;
-   AuditLogTable.

------------------------------------------------------------------------

# 12. Requisitos do Back-End

## 12.1 Stack sugerida

Para MVP leve:

-   Node.js;
-   TypeScript;
-   Fastify;
-   PostgreSQL;
-   Prisma;
-   Redis;
-   S3-compatible storage;
-   BullMQ para jobs;
-   OpenTelemetry;
-   Docker.

Alternativa:

-   NestJS apenas se a equipe já dominar e quiser estrutura mais
    opinativa.

Recomendação: **Fastify + Prisma** para MVP limpo, rápido e com menos
cerimônia.

## 12.2 Serviços principais

### 12.2.1 API Gateway

Responsável por:

-   autenticação;
-   autorização;
-   rate limit;
-   roteamento;
-   logging;
-   validação de payload.

### 12.2.2 Skill Registry Service

Responsável por:

-   cadastro de Skills;
-   versões;
-   metadados;
-   busca;
-   status;
-   publicação.

### 12.2.3 Package Service

Responsável por:

-   upload;
-   download;
-   armazenamento;
-   checksum;
-   assinatura;
-   compactação;
-   extração temporária para validação.

### 12.2.4 Validation Service

Responsável por:

-   validar `skill.yaml`;
-   validar `SKILL.md`;
-   validar estrutura;
-   validar dependências;
-   validar compatibilidade;
-   validar comandos/scripts.

### 12.2.5 Adapter Service

Responsável por:

-   listar adapters disponíveis;
-   validar compatibilidade;
-   gerar pacote convertido;
-   aplicar transformações específicas;
-   expor SDK de adapters.

### 12.2.6 Install Telemetry Service

Responsável por:

-   registrar instalação;
-   registrar atualização;
-   registrar remoção;
-   registrar erro;
-   associar Skill a projeto/time.

### 12.2.7 Approval Workflow Service

Responsável por:

-   criar fluxo de aprovação;
-   atribuir revisores;
-   aprovar;
-   rejeitar;
-   solicitar alterações;
-   registrar auditoria.

------------------------------------------------------------------------

# 13. Modelo de Dados

## 13.1 Tabela `skills`

Campos:

``` text
id
slug
name
description
category_id
publisher_id
owner_user_id
visibility
status
created_at
updated_at
archived_at
```

Status possíveis:

``` text
draft
pending_review
approved
rejected
deprecated
archived
blocked
```

## 13.2 Tabela `skill_versions`

Campos:

``` text
id
skill_id
version
manifest_json
entrypoint
package_url
checksum
signature
status
changelog
created_by
created_at
approved_at
deprecated_at
```

## 13.3 Tabela `publishers`

Campos:

``` text
id
name
type
verified
created_at
```

Tipos:

``` text
internal_team
external_vendor
community
system
```

## 13.4 Tabela `platforms`

Campos:

``` text
id
key
name
description
adapter_key
enabled
created_at
```

Exemplos:

``` text
claude-code
codex-cli
cursor
gemini-cli
```

## 13.5 Tabela `skill_dependencies`

Campos:

``` text
id
skill_version_id
dependency_skill_id
version_range
required
created_at
```

## 13.6 Tabela `installations`

Campos:

``` text
id
skill_id
skill_version_id
platform_id
project_id
user_id
scope
status
installed_at
updated_at
removed_at
```

## 13.7 Tabela `projects`

Campos:

``` text
id
name
repository_url
team_id
created_at
updated_at
```

## 13.8 Tabela `security_scans`

Campos:

``` text
id
skill_version_id
status
score
findings_json
created_at
```

## 13.9 Tabela `audit_logs`

Campos:

``` text
id
actor_user_id
action
entity_type
entity_id
metadata_json
created_at
```

------------------------------------------------------------------------

# 14. Contrato da Skill

## 14.1 Estrutura mínima

``` text
my-skill/
  skill.yaml
  SKILL.md
```

## 14.2 Estrutura recomendada

``` text
my-skill/
  skill.yaml
  SKILL.md
  README.md
  CHANGELOG.md
  examples/
  templates/
  scripts/
  references/
  tests/
```

## 14.3 Exemplo de `skill.yaml`

``` yaml
id: qodyn.angular-implementation
name: Angular Implementation
version: 1.0.0
description: Implementa telas Angular seguindo padrões corporativos.
publisher: qodyn
category: frontend
entrypoint: SKILL.md

tags:
  - angular
  - frontend
  - ui
  - figma
  - design-system

platforms:
  claude-code:
    enabled: true
  codex-cli:
    enabled: true
  cursor:
    enabled: true

dependencies:
  - id: qodyn.typescript-standards
    version: "^1.0.0"
  - id: qodyn.testing-standards
    version: "^1.0.0"

permissions:
  filesystem:
    read: true
    write: true
  network:
    enabled: false
  mcp:
    - github
    - figma

quality:
  maturity: beta
  owner: architecture-team
  review_required: true
```

## 14.4 Campos obrigatórios

``` text
id
name
version
description
publisher
category
entrypoint
platforms
```

## 14.5 Campos opcionais

``` text
tags
dependencies
permissions
quality
license
examples
minimum_agent_version
runtime_requirements
```

------------------------------------------------------------------------

# 15. Adapter SDK

## 15.1 Objetivo

Permitir que qualquer plataforma seja suportada sem alterar o core do
marketplace.

## 15.2 Interface TypeScript

``` ts
export interface SkillAdapter {
  key: string;
  name: string;

  supports(skill: SkillManifest): boolean;

  validate(skill: SkillPackage): Promise<AdapterValidationResult>;

  transform(skill: SkillPackage, options: AdapterOptions): Promise<TransformedSkillPackage>;

  install(skill: SkillPackage, target: InstallTarget): Promise<InstallResult>;

  uninstall(skillId: string, target: InstallTarget): Promise<UninstallResult>;

  listInstalled(target: InstallTarget): Promise<InstalledSkill[]>;
}
```

## 15.3 Tipos principais

``` ts
export type SkillManifest = {
  id: string;
  name: string;
  version: string;
  description: string;
  publisher: string;
  category: string;
  entrypoint: string;
  platforms: Record<string, { enabled: boolean }>;
  dependencies?: Array<{ id: string; version: string }>;
  permissions?: Record<string, unknown>;
};

export type SkillPackage = {
  manifest: SkillManifest;
  rootPath: string;
  files: string[];
};

export type AdapterOptions = {
  platform: string;
  scope: "user" | "project";
  projectPath?: string;
  userHome?: string;
};

export type InstallTarget = {
  scope: "user" | "project";
  projectPath?: string;
  userHome?: string;
};

export type AdapterValidationResult = {
  valid: boolean;
  errors: string[];
  warnings: string[];
};

export type InstallResult = {
  success: boolean;
  installedPath?: string;
  errors?: string[];
};
```

------------------------------------------------------------------------

# 16. Adapter Claude Code

## 16.1 Responsabilidade

Instalar Skills no formato reconhecido pelo Claude Code.

## 16.2 Estratégia inicial

Escopo usuário:

``` text
~/.claude/skills/{skill-name}/SKILL.md
```

Escopo projeto:

``` text
{project}/.claude/skills/{skill-name}/SKILL.md
```

## 16.3 Transformação

No MVP, copiar estrutura inteira.

Depois, adicionar:

-   geração de índice;
-   geração de `CLAUDE.md`;
-   recomendação de Skills;
-   skill discovery local.

## 16.4 Critérios de aceite

-   Instala Skill em escopo usuário.
-   Instala Skill em escopo projeto.
-   Mantém exemplos e templates.
-   Não sobrescreve versão diferente sem confirmação.
-   Remove corretamente.

------------------------------------------------------------------------

# 17. Adapter Codex CLI

## 17.1 Responsabilidade

Instalar Skills compatíveis com Codex CLI.

## 17.2 Estratégia inicial

Escopo usuário:

``` text
~/.codex/skills/{skill-name}/SKILL.md
```

Ou diretório configurável pelo usuário.

Escopo projeto:

``` text
{project}/.codex/skills/{skill-name}/SKILL.md
```

## 17.3 Transformação

-   Copiar `SKILL.md`.
-   Copiar recursos.
-   Preservar scripts opcionais.
-   Gerar arquivo de índice se necessário.

## 17.4 Critérios de aceite

-   Instala Skill.
-   Valida presença de `SKILL.md`.
-   Suporta recursos auxiliares.
-   Gera lockfile do projeto.

------------------------------------------------------------------------

# 18. Adapter Cursor

## 18.1 Responsabilidade

Converter Skills para regras do Cursor.

## 18.2 Estratégia inicial

Gerar arquivo:

``` text
{project}/.cursor/rules/{skill-name}.mdc
```

## 18.3 Transformação

Converter:

``` text
SKILL.md
```

para:

``` text
.cursor/rules/{skill-name}.mdc
```

Com cabeçalho adaptado:

``` markdown
---
description: Implementa telas Angular seguindo padrões corporativos.
globs:
  - "**/*.ts"
  - "**/*.html"
  - "**/*.scss"
alwaysApply: false
---
```

## 18.4 Critérios de aceite

-   Gera `.mdc`.
-   Preserva instruções principais.
-   Remove metadata incompatível.
-   Permite globs por Skill.

------------------------------------------------------------------------

# 19. CLI

## 19.1 Nome

``` bash
qodyn
```

## 19.2 Comandos

``` bash
qodyn skill search angular
qodyn skill info qodyn.angular-implementation
qodyn skill install qodyn.angular-implementation --platform claude-code --scope project
qodyn skill update qodyn.angular-implementation
qodyn skill remove qodyn.angular-implementation
qodyn skill list-installed
qodyn skill validate ./skills/angular-implementation
qodyn skill publish ./skills/angular-implementation
qodyn skill pack ./skills/angular-implementation
```

## 19.3 Arquivos gerados

``` text
.qodyn/
  skills.yaml
  skills.lock
```

## 19.4 `skills.yaml`

``` yaml
skills:
  - id: qodyn.angular-implementation
    version: "^1.0.0"
  - id: qodyn.code-review
    version: "^1.2.0"

platforms:
  - claude-code
  - codex-cli
```

## 19.5 `skills.lock`

``` yaml
locked:
  qodyn.angular-implementation:
    version: 1.0.3
    checksum: sha256:abc123
  qodyn.code-review:
    version: 1.2.1
    checksum: sha256:def456
```

------------------------------------------------------------------------

# 20. APIs

## 20.1 Skills

``` http
GET /api/skills
GET /api/skills/:id
POST /api/skills
PATCH /api/skills/:id
DELETE /api/skills/:id
```

## 20.2 Versões

``` http
GET /api/skills/:id/versions
GET /api/skills/:id/versions/:version
POST /api/skills/:id/versions
PATCH /api/skills/:id/versions/:version
```

## 20.3 Pacotes

``` http
POST /api/packages/upload
GET /api/packages/:id/download
POST /api/packages/:id/validate
```

## 20.4 Instalação

``` http
POST /api/install/resolve
POST /api/install/plan
POST /api/install/telemetry
```

## 20.5 Adapters

``` http
GET /api/adapters
GET /api/adapters/:key
POST /api/adapters/:key/transform
POST /api/adapters/:key/validate
```

## 20.6 Aprovação

``` http
GET /api/approvals
POST /api/approvals/:id/approve
POST /api/approvals/:id/reject
POST /api/approvals/:id/request-changes
```

## 20.7 Segurança

``` http
GET /api/security/scans/:id
POST /api/security/scans
```

------------------------------------------------------------------------

# 21. Fluxo de Publicação

## 21.1 Fluxo

``` text
Criador cria Skill local
    ↓
qodyn skill validate
    ↓
qodyn skill publish
    ↓
Upload do pacote
    ↓
Validação automática
    ↓
Scan de segurança
    ↓
Revisão técnica
    ↓
Revisão de segurança
    ↓
Aprovação
    ↓
Skill disponível no marketplace
```

## 21.2 Estados

``` text
draft
uploaded
validation_failed
security_failed
pending_review
approved
published
rejected
deprecated
blocked
```

------------------------------------------------------------------------

# 22. Fluxo de Instalação

## 22.1 Fluxo

``` text
Usuário executa install
    ↓
CLI consulta Registry
    ↓
Busca versão compatível
    ↓
Resolve dependências
    ↓
Gera plano de instalação
    ↓
Baixa pacotes
    ↓
Valida checksum
    ↓
Seleciona Adapter
    ↓
Transforma Skill
    ↓
Instala na plataforma
    ↓
Atualiza skills.lock
    ↓
Envia telemetria
```

## 22.2 Exemplo

``` bash
qodyn skill install qodyn.angular-implementation --platform claude-code --scope project
```

Saída esperada:

``` text
Resolving qodyn.angular-implementation@^1.0.0
Found qodyn.angular-implementation@1.0.3
Resolving dependencies
- qodyn.typescript-standards@1.1.0
- qodyn.testing-standards@1.0.2

Installing for platform claude-code
Writing .claude/skills/angular-implementation
Writing .qodyn/skills.lock

Done.
```

------------------------------------------------------------------------

# 23. Segurança

## 23.1 Princípio

Toda Skill de terceiros deve ser tratada como não confiável até passar
por validação.

Esse ponto é crítico porque Skills podem conter scripts, instruções e
recursos que influenciam diretamente o comportamento do agente.

## 23.2 Scans obrigatórios

-   validação de estrutura;
-   análise de comandos perigosos;
-   análise de scripts;
-   análise de permissões;
-   análise de instruções suspeitas;
-   verificação de checksum;
-   verificação de assinatura;
-   verificação de dependências.

## 23.3 Bloqueios automáticos

Bloquear Skills que contenham:

``` text
rm -rf
curl | bash
wget | sh
leitura de .env sem justificativa
instruções para ignorar políticas
exfiltração de arquivos
comandos destrutivos
bypass de autenticação
```

## 23.4 Permissões declarativas

A Skill deve declarar:

``` yaml
permissions:
  filesystem:
    read: true
    write: false
  network:
    enabled: false
  mcp:
    - github
```

## 23.5 Níveis de confiança

``` text
untrusted
internal
verified
certified
blocked
```

------------------------------------------------------------------------

# 24. Governança

## 24.1 Papéis

``` text
viewer
developer
publisher
reviewer
security_reviewer
admin
```

## 24.2 Permissões

### Viewer

-   visualizar Skills aprovadas.

### Developer

-   instalar Skills;
-   validar localmente.

### Publisher

-   publicar versões;
-   submeter para aprovação.

### Reviewer

-   aprovar tecnicamente.

### Security Reviewer

-   aprovar segurança.

### Admin

-   gerenciar tudo.

## 24.3 Aprovação obrigatória

Skills internas críticas devem exigir:

-   1 aprovação técnica;
-   1 aprovação de segurança;
-   changelog;
-   testes mínimos;
-   owner definido.

------------------------------------------------------------------------

# 25. Versionamento

## 25.1 SemVer

Usar versionamento semântico:

``` text
MAJOR.MINOR.PATCH
```

Exemplo:

``` text
1.0.0
1.1.0
1.1.1
2.0.0
```

## 25.2 Regras

### PATCH

-   correções pequenas;
-   melhoria de texto;
-   ajuste de exemplo.

### MINOR

-   novo workflow;
-   nova validação;
-   nova compatibilidade.

### MAJOR

-   mudança de comportamento;
-   mudança de contrato;
-   remoção de etapa;
-   alteração incompatível.

## 25.3 Depreciação

Permitir marcar versão como:

``` text
deprecated
```

Com mensagem:

``` text
Use qodyn.angular-implementation@2.x
```

------------------------------------------------------------------------

# 26. Skill Packs

## 26.1 Conceito

Skill Pack é um conjunto de Skills instaláveis de uma vez.

Exemplos:

``` text
Angular Enterprise Pack
Java Backend Pack
AI Engineering Pack
Security Review Pack
Legacy Modernization Pack
```

## 26.2 Estrutura

``` yaml
id: qodyn.angular-enterprise-pack
name: Angular Enterprise Pack
version: 1.0.0

skills:
  - qodyn.angular-implementation@^1.0.0
  - qodyn.typescript-standards@^1.0.0
  - qodyn.testing-standards@^1.0.0
  - qodyn.accessibility-review@^1.0.0
```

------------------------------------------------------------------------

# 27. Recomendação de Skills

## 27.1 MVP

Recomendação manual por categoria.

## 27.2 Futuro

Recomendação baseada em:

-   linguagem do projeto;
-   arquivos detectados;
-   frameworks;
-   histórico de uso;
-   tarefas recorrentes;
-   falhas em PR;
-   tags do repositório.

Exemplo:

``` text
Detectado Angular + Jest + Figma
Recomendadas:
- Angular Implementation
- Figma to Angular
- Jest Testing
- Accessibility Review
```

------------------------------------------------------------------------

# 28. Integrações

## 28.1 GitHub

Usos:

-   detectar projetos;
-   associar instalação a repositório;
-   abrir PR com arquivos de Skill;
-   validar Skills em CI;
-   publicar pacote por release.

## 28.2 Jira

Usos:

-   sugerir Skills por tipo de tarefa;
-   anexar Skills recomendadas à issue;
-   criar fluxo PRD → tasks → implementation skills.

## 28.3 Figma

Usos:

-   Skills de implementação de tela;
-   leitura de design system;
-   mapeamento de componentes.

## 28.4 SSO

Usos:

-   login corporativo;
-   times;
-   permissões;
-   auditoria.

------------------------------------------------------------------------

# 29. Observabilidade

## 29.1 Logs

Registrar:

-   publicação;
-   instalação;
-   erro;
-   validação;
-   aprovação;
-   remoção;
-   update;
-   rollback.

## 29.2 Métricas

Métricas principais:

``` text
skills_published_total
skills_installed_total
skill_install_failed_total
skill_download_total
skill_validation_failed_total
adapter_transform_failed_total
approval_pending_total
deprecated_versions_in_use_total
```

## 29.3 Tracing

Usar OpenTelemetry nos serviços:

-   API;
-   Registry;
-   Package Service;
-   Validation Service;
-   Adapter Service.

------------------------------------------------------------------------

# 30. Critérios de Aceite do MVP

## 30.1 Marketplace

-   Usuário consegue listar Skills.
-   Usuário consegue abrir detalhe da Skill.
-   Usuário consegue ver versões.
-   Usuário consegue ver comando de instalação.
-   Usuário consegue filtrar por plataforma.

## 30.2 Publicação

-   Usuário consegue publicar pacote.
-   Sistema valida `skill.yaml`.
-   Sistema valida `SKILL.md`.
-   Sistema bloqueia pacote inválido.
-   Sistema cria versão.

## 30.3 CLI

-   CLI busca Skill no registry.
-   CLI instala Skill.
-   CLI resolve dependências.
-   CLI gera `skills.lock`.
-   CLI remove Skill.
-   CLI valida Skill local.

## 30.4 Adapters

-   Claude Code Adapter instala Skill.
-   Codex CLI Adapter instala Skill.
-   Cursor Adapter converte para `.mdc`.
-   Sistema detecta plataforma não suportada.

## 30.5 Segurança

-   Scanner detecta comandos proibidos.
-   Scanner gera relatório.
-   Skill bloqueada não pode ser instalada.
-   Auditoria registra ações.

------------------------------------------------------------------------

# 31. Roadmap

## Fase 1 --- MVP Local

Duração sugerida: 4 a 6 semanas.

Entregas:

-   formato `skill.yaml`;
-   CLI local;
-   validação local;
-   adapters Claude, Codex e Cursor;
-   registry em pasta local;
-   lockfile.

## Fase 2 --- Registry API

Duração sugerida: 6 a 8 semanas.

Entregas:

-   backend;
-   banco;
-   upload/download;
-   publicação;
-   versões;
-   busca;
-   autenticação.

## Fase 3 --- Marketplace Web

Duração sugerida: 6 a 8 semanas.

Entregas:

-   UI;
-   listagem;
-   detalhe;
-   upload;
-   aprovação;
-   dashboard básico.

## Fase 4 --- Governança

Duração sugerida: 4 a 6 semanas.

Entregas:

-   workflow de aprovação;
-   scan de segurança;
-   permissões;
-   auditoria;
-   status de confiança.

## Fase 5 --- Enterprise

Duração sugerida: contínua.

Entregas:

-   SSO;
-   GitHub;
-   Jira;
-   Figma;
-   métricas avançadas;
-   skill packs;
-   recomendações.

------------------------------------------------------------------------

# 32. Arquitetura Técnica

## 32.1 Visão geral

``` text
Frontend Marketplace
        ↓
API Gateway
        ↓
Registry Service
        ↓
Package Service
        ↓
Validation Service
        ↓
Adapter Service
        ↓
Storage + PostgreSQL + Redis
        ↓
CLI / IDE / CI
```

## 32.2 Fluxo com CLI

``` text
CLI
 ↓
Registry API
 ↓
Download Package
 ↓
Validate Checksum
 ↓
Resolve Dependencies
 ↓
Adapter Transform
 ↓
Local Install
 ↓
Lockfile Update
 ↓
Telemetry
```

------------------------------------------------------------------------

# 33. Estrutura do Monorepo

``` text
qodyn-skill-hub/
  apps/
    web/
    api/
    cli/

  packages/
    core/
    sdk/
    adapters/
      claude-code/
      codex-cli/
      cursor/
    validator/
    manifest/
    package-manager/
    security-scanner/

  skills/
    examples/
      angular-implementation/
      code-review/
      testing-standards/

  docs/
    architecture.md
    skill-format.md
    adapter-sdk.md
    security.md
```

------------------------------------------------------------------------

# 34. Stack Recomendada

## 34.1 Front-end

``` text
Next.js
React
TypeScript
Tailwind
shadcn/ui
TanStack Query
Zod
```

## 34.2 Back-end

``` text
Node.js
TypeScript
Fastify
Prisma
PostgreSQL
Redis
BullMQ
S3-compatible storage
OpenTelemetry
```

## 34.3 CLI

``` text
Node.js
TypeScript
Commander
Oclif ou Clipanion
Zod
Ora
Chalk
fs-extra
```

## 34.4 Infra

``` text
Docker
Docker Compose
GitHub Actions
PostgreSQL
Redis
MinIO para dev local
AWS S3 para produção
```

------------------------------------------------------------------------

# 35. Riscos

## 35.1 Risco de virar apenas repositório de prompts

Mitigação:

-   exigir manifesto;
-   exigir validação;
-   exigir versionamento;
-   exigir critérios de saída.

## 35.2 Risco de lock-in em plataforma

Mitigação:

-   Skill canônica;
-   Adapters plugáveis;
-   SDK público.

## 35.3 Risco de segurança

Mitigação:

-   scan;
-   permissões declarativas;
-   aprovação;
-   assinatura;
-   bloqueio automático.

## 35.4 Risco de baixa adoção

Mitigação:

-   CLI simples;
-   packs por stack;
-   integração com GitHub;
-   documentação clara;
-   templates prontos.

## 35.5 Risco de excesso de complexidade

Mitigação:

-   MVP local primeiro;
-   API depois;
-   marketplace visual por último.

------------------------------------------------------------------------

# 36. Pontos Ainda Não Abordados e Importantes

## 36.1 Skill Quality Score

Criar score interno calculado por:

-   documentação completa;
-   exemplos;
-   testes;
-   aprovação;
-   uso;
-   atualização recente;
-   baixa taxa de erro.

## 36.2 Compatibilidade por versão de agente

Adicionar:

``` yaml
compatibility:
  claude-code: ">=1.0.0"
  codex-cli: ">=0.5.0"
```

## 36.3 Policy Engine

Criar políticas corporativas:

``` text
Projetos financeiros só podem usar Skills certified.
Skills com network enabled exigem aprovação de segurança.
Skills externas não podem ter scripts.
```

## 36.4 Sandbox de validação

Antes de publicar, rodar scripts em ambiente isolado.

## 36.5 Skill Diff

Mostrar diferença entre versões:

``` text
1.0.0 → 1.1.0
```

Com destaque para:

-   instruções alteradas;
-   permissões alteradas;
-   scripts adicionados;
-   dependências novas.

## 36.6 Rollback

Permitir:

``` bash
qodyn skill rollback qodyn.angular-implementation --to 1.0.2
```

## 36.7 Ownership obrigatório

Toda Skill deve ter:

-   owner técnico;
-   time responsável;
-   SLA de manutenção.

## 36.8 Expiração de Skill

Skills sem atualização por muito tempo devem ser marcadas como:

``` text
stale
```

## 36.9 Assinatura de pacote

Cada pacote deve ter:

-   checksum;
-   assinatura;
-   publisher verificado.

## 36.10 Ambientes

Suportar:

``` text
dev
staging
production
```

Para testar Skills antes de liberar.

------------------------------------------------------------------------

# 37. Métricas de Sucesso

## 37.1 Produto

-   50 Skills publicadas em 90 dias.
-   10 projetos usando Skills em 90 dias.
-   3 plataformas suportadas no MVP.
-   80% das instalações sem erro.
-   100% das Skills com manifesto válido.

## 37.2 Engenharia

-   Redução de retrabalho em tarefas repetitivas.
-   Aumento de padronização entre projetos.
-   Menor tempo de onboarding.
-   Menos divergência entre agentes.

## 37.3 Governança

-   100% das Skills aprovadas antes de uso em produção.
-   0 Skills bloqueadas instaladas.
-   100% das instalações auditáveis.

------------------------------------------------------------------------

# 38. Conclusão

A solução deve nascer como um **Skill Registry com Adapters**, não como
um simples fork do Agent Skills.

O Agent Skills entra como padrão-base de conteúdo e inspiração. O
diferencial da empresa será construir a camada que falta:

``` text
Registry
Versionamento
Governança
Adapters
Instalação
Segurança
Métricas
Marketplace
```

A decisão técnica mais importante é esta:

> A Skill não pertence ao Claude, ao Codex ou ao Cursor.\
> A Skill pertence ao marketplace.\
> Os Adapters apenas traduzem e instalam.

Essa separação permite escalar o produto, evitar lock-in, criar
governança corporativa e futuramente abrir um marketplace público de
Skills.

------------------------------------------------------------------------

# Atualização v2 do PRD --- Agent Skills como padrão canônico

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
