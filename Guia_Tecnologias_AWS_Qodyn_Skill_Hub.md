# Guia de Tecnologias Priorizadas --- Qodyn Skill Hub

**Versão:** 1.0\
**Objetivo:** definir a stack prioritária para implementação do Qodyn
Skill Hub, priorizando tecnologias AWS e adotando Angular no frontend.

------------------------------------------------------------------------

# 1. Diretriz arquitetural

A implementação deve priorizar:

-   **TypeScript ponta a ponta**;
-   **Angular no frontend**;
-   serviços gerenciados da AWS;
-   arquitetura modular;
-   baixo acoplamento;
-   escalabilidade progressiva;
-   segurança e observabilidade desde o início;
-   evitar microserviços prematuros;
-   evitar infraestrutura operacional desnecessária.

A stack deve permitir evolução de MVP para ambiente enterprise sem
reescrita estrutural.

------------------------------------------------------------------------

# 2. Stack resumida

  Camada                   Tecnologia prioritária
  ------------------------ -------------------------------------
  Linguagem principal      TypeScript
  Frontend                 Angular
  UI                       Angular Material
  Monorepo                 Nx
  Backend                  Node.js + Fastify
  Validação                Zod
  API                      REST + OpenAPI
  Banco relacional         Amazon RDS for PostgreSQL
  ORM                      Drizzle ORM
  Storage de pacotes       Amazon S3
  CDN                      Amazon CloudFront
  Filas                    Amazon SQS
  Eventos                  Amazon EventBridge
  Jobs                     ECS/Fargate workers ou AWS Lambda
  Cache                    Amazon ElastiCache
  Autenticação             Amazon Cognito / OIDC corporativo
  Containers               Amazon ECS + AWS Fargate
  Registry de imagens      Amazon ECR
  Secrets                  AWS Secrets Manager
  Criptografia             AWS KMS
  Observabilidade          CloudWatch + OpenTelemetry
  Auditoria AWS            AWS CloudTrail
  Segurança de borda       AWS WAF
  DNS                      Amazon Route 53
  CLI                      Commander.js + Ink
  MCP                      MCP TypeScript SDK
  CI/CD                    GitHub Actions
  Infra as Code            AWS CDK com TypeScript
  Artefatos npm privados   GitHub Packages ou AWS CodeArtifact

------------------------------------------------------------------------

# 3. Linguagem principal --- TypeScript

TypeScript deve ser adotado como linguagem principal para:

``` text
Frontend
Backend
CLI
MCP
Adapters
Compatibility Layer
Registry Client
Policy Engine
Source Manager
Dependency Resolver
```

Benefícios:

-   compartilhamento de contratos;
-   compartilhamento de schemas;
-   menos tecnologias para manter;
-   integração direta com o baseline do Tech Leads Club;
-   melhor reutilização entre CLI, API, MCP e adapters.

Estrutura recomendada:

``` text
apps/
  web/
  api/
  cli/
  mcp/

packages/
  contracts/
  agent-skills-compat/
  registry-client/
  adapter-sdk/
  source-manager/
  policy-engine/
```

------------------------------------------------------------------------

# 4. Monorepo --- Nx

Priorizar **Nx**.

Motivos:

-   alinhamento com o Tech Leads Club;
-   aplicações e bibliotecas no mesmo repositório;
-   dependency graph;
-   builds incrementais;
-   caching;
-   execução seletiva de testes;
-   compartilhamento de bibliotecas.

Estrutura:

``` text
qodyn-skill-hub/

apps/
  web/
  api/
  cli/
  mcp/

packages/
  contracts/
  core/
  compatibility/
  adapters/
  registry/
  policies/
  security/
  sources/
```

------------------------------------------------------------------------

# 5. Frontend --- Angular

O frontend oficial deve usar **Angular**.

Angular é adequado para o Skill Hub por ser orientado a aplicações
estruturadas e corporativas, com forte suporte a:

-   TypeScript;
-   componentes;
-   dependency injection;
-   forms;
-   routing;
-   interceptors;
-   guards;
-   signals;
-   testes;
-   arquitetura modular.

Aplicação:

``` text
apps/web
```

------------------------------------------------------------------------

# 6. Angular Material

Priorizar **Angular Material** para o MVP e primeira versão enterprise.

Componentes necessários:

-   tables;
-   forms;
-   dialogs;
-   menus;
-   tabs;
-   badges/chips;
-   navigation;
-   autocomplete;
-   pagination;
-   stepper;
-   snackbars.

Principais telas:

``` text
Home
Skill Catalog
Skill Details
Versions
Sources
Publishers
Imports
Security Findings
Approvals
Policies
Projects
Installations
Adapters
Audit
Administration
```

Evitar construir um design system próprio no MVP.

------------------------------------------------------------------------

# 7. State Management

Priorizar inicialmente:

``` text
Angular Signals
+
RxJS
+
services
```

Não introduzir NgRx automaticamente.

NgRx deve entrar apenas quando a complexidade de estado global
justificar.

Para chamadas HTTP:

``` text
Angular HttpClient
+
Signals/RxJS
```

------------------------------------------------------------------------

# 8. Frontend Hosting --- S3 + CloudFront

Para Angular SPA:

``` text
Angular Build
     |
     v
Amazon S3
     |
     v
Amazon CloudFront
     |
     v
AWS WAF
     |
     v
Usuário
```

Benefícios:

-   baixo custo;
-   alta disponibilidade;
-   cache global;
-   deploy simples;
-   integração com WAF;
-   HTTPS;
-   escalabilidade automática.

------------------------------------------------------------------------

# 9. Backend --- Node.js + Fastify

Priorizar:

``` text
Node.js
TypeScript
Fastify
```

Fastify oferece uma base leve para APIs e evita complexidade
desnecessária.

Arquitetura inicial:

``` text
API Modular Monolith

modules/
  auth/
  skills/
  versions/
  registry/
  sources/
  packages/
  compatibility/
  adapters/
  dependencies/
  security/
  policies/
  approvals/
  projects/
  installations/
  audit/
```

Não criar microserviços no MVP.

------------------------------------------------------------------------

# 10. Runtime AWS --- ECS + Fargate

Priorizar **Amazon ECS com AWS Fargate** para a API.

Arquitetura:

``` text
CloudFront
   |
API endpoint
   |
Application Load Balancer
   |
ECS
   |
Fargate Tasks
   |
Fastify API
```

Benefícios:

-   não administrar servidores;
-   containers padronizados;
-   autoscaling;
-   rolling deployment;
-   integração com CloudWatch;
-   integração com Secrets Manager;
-   fácil evolução.

Não priorizar Kubernetes/EKS inicialmente.

------------------------------------------------------------------------

# 11. Amazon ECR

Docker images da API, MCP e workers:

``` text
GitHub Actions
     |
     v
Docker Build
     |
     v
Amazon ECR
     |
     v
ECS/Fargate
```

Usar versionamento imutável baseado em:

``` text
Git SHA
+
release version
```

------------------------------------------------------------------------

# 12. Banco de dados --- Amazon RDS for PostgreSQL

Banco principal:

**Amazon RDS for PostgreSQL**

Dados:

-   organizations;
-   teams;
-   users;
-   publishers;
-   skills;
-   skill_versions;
-   sources;
-   packages;
-   dependencies;
-   compatibility;
-   policies;
-   approvals;
-   installations;
-   security findings;
-   audit metadata.

Benefícios:

-   backups gerenciados;
-   snapshots;
-   Point-in-Time Recovery;
-   Multi-AZ;
-   read replicas;
-   SSL;
-   VPC;
-   operação simplificada.

Para o MVP, não há necessidade de Aurora.

Avaliar **Aurora PostgreSQL** posteriormente se escala, disponibilidade
ou requisitos específicos justificarem.

------------------------------------------------------------------------

# 13. ORM --- Drizzle ORM

Priorizar:

``` text
Drizzle ORM
```

Motivos:

-   TypeScript;
-   schemas explícitos;
-   proximidade com SQL;
-   migrations;
-   baixo overhead.

Arquitetura:

``` text
Fastify
  |
Repository Layer
  |
Drizzle
  |
RDS PostgreSQL
```

Prisma continua sendo alternativa aceitável se houver maior domínio
interno.

------------------------------------------------------------------------

# 14. Busca

MVP:

``` text
PostgreSQL
+
Full Text Search
+
pg_trgm
```

Evitar OpenSearch inicialmente.

Adicionar Amazon OpenSearch somente quando houver necessidade comprovada
de:

-   busca de alto volume;
-   analytics;
-   full-text complexo;
-   faceting avançado;
-   ranking sofisticado.

------------------------------------------------------------------------

# 15. Storage de Skills --- Amazon S3

O conteúdo publicado das Skills deve ser armazenado em **Amazon S3**.

Exemplo:

``` text
s3://qodyn-skill-packages/

organizations/
  qodyn/
    angular-implementation/
      2.1.0/
        skill.tgz
        manifest.json
        signature
```

S3 deve armazenar:

-   packages;
-   manifests;
-   signatures;
-   assets;
-   scan reports exportáveis.

Configurar:

-   versioning;
-   encryption;
-   lifecycle;
-   object lock quando necessário;
-   bucket policies.

------------------------------------------------------------------------

# 16. Distribuição --- CloudFront

Pacotes públicos ou autorizados podem ser distribuídos via:

``` text
S3
 |
CloudFront
 |
CLI
```

CloudFront reduz carga sobre APIs e melhora download.

A API deve emitir URLs assinadas quando necessário.

------------------------------------------------------------------------

# 17. Filas --- Amazon SQS

Priorizar **Amazon SQS** para processamento assíncrono.

Casos:

``` text
Source synchronization
Security scanning
Skill imports
Package processing
Compatibility validation
Notification processing
Index updates
Telemetry aggregation
```

Arquitetura:

``` text
API
 |
SQS
 |
Worker
 |
ECS Fargate / Lambda
```

Use:

-   Standard Queues para maioria dos casos;
-   FIFO somente quando ordenação for requisito real.

------------------------------------------------------------------------

# 18. Dead Letter Queues

Toda fila crítica deve possuir DLQ.

Exemplo:

``` text
skill-import
     |
     v
skill-import-dlq
```

Definir:

-   maxReceiveCount;
-   alarmes;
-   processo de redrive;
-   dashboard de erros.

------------------------------------------------------------------------

# 19. Event Bus --- Amazon EventBridge

SQS e EventBridge têm responsabilidades diferentes.

Use EventBridge para eventos de domínio:

``` text
SkillPublished
SkillDeprecated
SourceUpdated
SecurityFindingCreated
ApprovalCompleted
InstallationRecorded
```

Arquitetura:

``` text
Domain Event
    |
EventBridge
    |
 +-- SQS
 +-- Lambda
 +-- Notifications
 +-- Audit integration
```

Isso reduz acoplamento entre módulos.

------------------------------------------------------------------------

# 20. Jobs e workers

Priorizar duas opções.

## AWS Lambda

Para:

-   tarefas pequenas;
-   execução curta;
-   transformações simples;
-   EventBridge handlers;
-   notificações.

## ECS Fargate workers

Para:

-   scans demorados;
-   processamento pesado;
-   Git clone;
-   análise de pacotes;
-   ferramentas externas;
-   sandbox.

Regra:

``` text
curto e stateless → Lambda

pesado, containerizado ou demorado → Fargate
```

------------------------------------------------------------------------

# 21. Cache --- Amazon ElastiCache

Não introduzir Redis no primeiro sprint.

Adicionar **Amazon ElastiCache** quando necessário para:

-   cache de consultas;
-   rate limiting;
-   locks distribuídos;
-   sessões;
-   resultados temporários;
-   deduplicação.

Não usar Redis como banco primário.

------------------------------------------------------------------------

# 22. Autenticação --- Amazon Cognito

Priorizar **Amazon Cognito** quando não existir SSO corporativo central.

Fluxo:

``` text
Angular
 |
Cognito
 |
JWT
 |
API
```

Suportar:

-   OIDC;
-   OAuth2;
-   Authorization Code + PKCE;
-   MFA quando necessário.

Se a empresa já utilizar Entra ID, Okta ou outro IdP, Cognito pode atuar
como broker ou a API pode integrar diretamente via OIDC.

------------------------------------------------------------------------

# 23. RBAC

RBAC deve permanecer na aplicação.

Papéis:

``` text
viewer
developer
publisher
reviewer
security_reviewer
source_admin
admin
```

JWT fornece identidade.

API resolve autorização considerando:

``` text
organization
team
resource
role
policy
```

------------------------------------------------------------------------

# 24. Secrets --- AWS Secrets Manager

Armazenar:

-   database credentials;
-   GitHub tokens;
-   source credentials;
-   signing credentials;
-   third-party API secrets.

Nunca:

``` text
.env em produção
GitHub repository
Docker image
```

ECS tasks devem receber secrets via integração AWS.

------------------------------------------------------------------------

# 25. Criptografia --- AWS KMS

Usar KMS para:

-   S3 encryption;
-   RDS encryption;
-   Secrets Manager;
-   signing/encryption workflows;
-   chaves específicas do tenant quando exigido.

------------------------------------------------------------------------

# 26. Segurança de rede

Arquitetura:

``` text
VPC

Public subnets
  |
ALB

Private subnets
  |
ECS
  |
RDS
```

RDS nunca deve ficar publicamente acessível.

Aplicar:

-   security groups;
-   private subnets;
-   VPC endpoints onde fizer sentido.

------------------------------------------------------------------------

# 27. AWS WAF

Colocar WAF na frente dos endpoints públicos.

Regras:

-   managed rules;
-   rate limiting;
-   IP reputation;
-   malicious payloads;
-   bot protection conforme necessidade.

------------------------------------------------------------------------

# 28. Observabilidade --- CloudWatch

Usar:

-   CloudWatch Logs;
-   CloudWatch Metrics;
-   CloudWatch Alarms;
-   CloudWatch Dashboards.

Monitorar:

``` text
API latency
5xx
4xx
ECS CPU/memory
queue depth
DLQ messages
RDS connections
RDS storage
source sync failures
security scan failures
```

------------------------------------------------------------------------

# 29. OpenTelemetry

A aplicação deve instrumentar:

``` text
HTTP
database
queues
workers
source sync
adapter execution
```

Usar OpenTelemetry para manter portabilidade de observabilidade.

CloudWatch pode continuar como backend inicial.

------------------------------------------------------------------------

# 30. CloudTrail

CloudTrail deve auditar operações AWS.

Separar:

``` text
CloudTrail
   = auditoria da infraestrutura AWS

Qodyn Audit Log
   = auditoria funcional do produto
```

Os dois são necessários.

------------------------------------------------------------------------

# 31. MCP Server

Stack:

``` text
Node.js
TypeScript
MCP TypeScript SDK
Zod
```

Deploy:

``` text
Amazon ECS + Fargate
```

O MCP deve consumir:

``` text
Qodyn Registry API
```

Não acessar PostgreSQL diretamente.

------------------------------------------------------------------------

# 32. CLI

Stack:

``` text
TypeScript
Commander.js
Ink
Zod
```

Aproveitar fortemente os conceitos do Tech Leads Club.

Comandos:

``` bash
qodyn skill search
qodyn skill install
qodyn skill update
qodyn skill sync
qodyn skill audit
qodyn source add
qodyn source sync
qodyn policy check
```

------------------------------------------------------------------------

# 33. Distribuição da CLI

Opções:

### Prioridade inicial

``` text
GitHub Packages
```

### Alternativa AWS

``` text
AWS CodeArtifact
```

Se a estratégia for centralizar artefatos corporativos na AWS,
CodeArtifact é a melhor evolução.

------------------------------------------------------------------------

# 34. Adapters

Implementados como packages TypeScript:

``` text
packages/adapters/
  claude-code/
  codex/
  cursor/
  gemini/
```

Nenhum adapter deve acessar AWS diretamente.

Adapter recebe:

``` text
SkillPackage
Target
Policy
Configuration
```

e executa transformação/instalação local.

------------------------------------------------------------------------

# 35. Compatibility Layer

Package puro TypeScript:

``` text
packages/agent-skills-compat
```

Responsabilidades:

-   parse;
-   schema;
-   validation;
-   compatibility;
-   conformance.

Não depender de AWS.

Isso garante portabilidade e testes locais.

------------------------------------------------------------------------

# 36. Source Manager

O Source Manager roda no backend.

Fontes:

-   GitHub;
-   Git;
-   Tech Leads Club;
-   Anthropic;
-   Addy Osmani;
-   repos corporativos.

Arquitetura:

``` text
EventBridge Scheduler
       |
       v
SQS source-sync
       |
       v
Fargate Worker
       |
       v
Git/GitHub
       |
       v
Registry
```

------------------------------------------------------------------------

# 37. GitHub Integration

Usar GitHub App preferencialmente a PAT permanente.

Casos:

-   acessar repos;
-   sincronizar Skills;
-   receber webhooks;
-   criar PR;
-   detectar commits;
-   ler releases.

Secrets/configuração sensível em Secrets Manager.

------------------------------------------------------------------------

# 38. CI/CD --- GitHub Actions

Manter GitHub Actions como ferramenta principal.

Pipeline:

``` text
Pull Request
 |
lint
 |
typecheck
 |
unit
 |
integration
 |
compatibility
 |
adapter tests
 |
security
 |
build
```

Merge:

``` text
main
 |
Docker build
 |
ECR
 |
CDK deploy
 |
ECS deployment
 |
smoke tests
```

------------------------------------------------------------------------

# 39. Infrastructure as Code --- AWS CDK

Priorizar:

``` text
AWS CDK
+
TypeScript
```

Motivos:

-   mesma linguagem do produto;
-   constructs;
-   integração AWS;
-   facilidade para compartilhar infraestrutura como código.

Estrutura:

``` text
infra/

  stacks/
    network-stack.ts
    data-stack.ts
    api-stack.ts
    web-stack.ts
    observability-stack.ts
```

Terraform continua sendo alternativa se já for padrão da empresa.

------------------------------------------------------------------------

# 40. Ambientes

Manter:

``` text
dev
staging
prod
```

Cada ambiente deve possuir:

-   conta AWS separada ou forte isolamento;
-   banco separado;
-   buckets separados;
-   filas separadas;
-   secrets separados.

Ideal enterprise:

``` text
AWS Organizations
```

com contas por ambiente.

------------------------------------------------------------------------

# 41. Arquitetura AWS alvo

``` text
                         Route 53
                            |
                         CloudFront
                            |
              +-------------+-------------+
              |                           |
          Angular SPA                     API
              |                           |
             S3                          WAF
                                          |
                                         ALB
                                          |
                                  ECS / Fargate
                                          |
               +--------------------------+-------------------+
               |             |            |                   |
           RDS PostgreSQL    SQS      EventBridge             S3
               |             |                                |
               |         Fargate/Lambda                   Skill Packages
               |
          ElastiCache
          (quando necessário)

Supporting:
Secrets Manager
KMS
CloudWatch
CloudTrail
ECR
```

------------------------------------------------------------------------

# 42. MVP --- serviços obrigatórios

Para o primeiro ambiente funcional:

``` text
Angular
S3
CloudFront
Fastify
ECS Fargate
ALB
RDS PostgreSQL
S3 Packages
SQS
ECR
Secrets Manager
CloudWatch
GitHub Actions
CDK
```

------------------------------------------------------------------------

# 43. Serviços que podem esperar

Não colocar no MVP sem necessidade:

``` text
ElastiCache
OpenSearch
Aurora
Lambda em larga escala
Step Functions
MSK/Kafka
EKS
DynamoDB
AppSync
GraphQL
Service Mesh
```

------------------------------------------------------------------------

# 44. Evolução enterprise

Adicionar conforme necessidade:

``` text
Cognito / enterprise SSO
EventBridge
ElastiCache
WAF advanced policies
Security Hub
GuardDuty
AWS Config
AWS Backup
OpenSearch
Multi-region
Aurora PostgreSQL
```

------------------------------------------------------------------------

# 45. Step Functions

Avaliar posteriormente para workflows complexos:

``` text
Import
 ↓
Compatibility
 ↓
Security
 ↓
Quality
 ↓
Approval
 ↓
Publish
```

No MVP isso pode ser controlado por:

``` text
SQS
+
status no PostgreSQL
```

Step Functions entra quando o workflow exigir forte orquestração,
retries visuais e estados complexos.

------------------------------------------------------------------------

# 46. DynamoDB

Não usar como banco principal.

Pode ser considerado futuramente para:

-   alta escala de eventos;
-   idempotency keys;
-   ephemeral metadata;
-   materialized views específicas.

PostgreSQL permanece system of record.

------------------------------------------------------------------------

# 47. Amazon OpenSearch

Não usar inicialmente.

Introduzir quando PostgreSQL deixar de atender:

-   busca semântica híbrida;
-   ranking complexo;
-   alto volume;
-   analytics;
-   faceting.

------------------------------------------------------------------------

# 48. Bedrock

Amazon Bedrock pode ser integrado futuramente para:

-   classificação automática de Skills;
-   geração de metadata;
-   quality analysis;
-   semantic search;
-   documentação;
-   recomendações;
-   análise de risco assistida.

Não deve ser requisito do core.

O Hub gerencia Skills independentemente do provedor de LLM.

------------------------------------------------------------------------

# 49. AWS Bedrock AgentCore

Pode ser avaliado posteriormente para agentes corporativos que consumam
o Skill Hub.

Arquitetura possível:

``` text
Bedrock Agent / AgentCore
       |
       v
Qodyn MCP
       |
       v
Skill Registry
```

Não acoplar o Registry ao Bedrock.

------------------------------------------------------------------------

# 50. Prioridade de implementação

## P0

``` text
TypeScript
Nx
Angular
Angular Material
Fastify
Zod
OpenAPI
RDS PostgreSQL
Drizzle
S3
ECS/Fargate
ECR
SQS
Secrets Manager
CloudWatch
GitHub Actions
CDK
```

## P1

``` text
CloudFront
WAF
Cognito/OIDC
EventBridge
MCP
KMS
CloudTrail
```

## P2

``` text
ElastiCache
Step Functions
OpenSearch
Bedrock
Security Hub
GuardDuty
```

------------------------------------------------------------------------

# 51. Stack final recomendada

``` text
FRONTEND
Angular
Angular Material
TypeScript
Signals
RxJS
S3
CloudFront

BACKEND
Node.js
TypeScript
Fastify
Zod
OpenAPI
Drizzle

DATA
Amazon RDS PostgreSQL
Amazon S3

ASYNC
Amazon SQS
Amazon EventBridge
AWS Lambda / ECS Fargate Workers

CACHE
Amazon ElastiCache

SECURITY
Cognito/OIDC
AWS Secrets Manager
AWS KMS
AWS WAF
CloudTrail

RUNTIME
Amazon ECS
AWS Fargate
Amazon ECR

OBSERVABILITY
Amazon CloudWatch
OpenTelemetry

CLI
Commander.js
Ink
Zod

MCP
MCP TypeScript SDK

CI/CD
GitHub Actions

INFRA
AWS CDK + TypeScript
```

------------------------------------------------------------------------

# 52. Decisão final

A arquitetura deve permanecer simples no início:

``` text
Angular SPA
    |
Fastify API
    |
PostgreSQL + S3
    |
SQS Workers
```

executada predominantemente sobre serviços gerenciados AWS.

A prioridade não é utilizar o maior número de serviços AWS possível.

A prioridade é:

> **usar serviços AWS onde eles reduzam operação, aumentem segurança e
> permitam evolução gradual sem adicionar complexidade prematura.**
