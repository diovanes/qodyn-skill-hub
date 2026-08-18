# Qodyn Skill Hub

Plataforma para **gerenciamento, governança e distribuição de Agent
Skills** em ambientes corporativos.

O Qodyn Skill Hub utiliza o padrão aberto **Agent Skills** como formato
canônico e adiciona uma camada enterprise para permitir que organizações
criem, reutilizem, distribuam e governem Skills utilizadas por
diferentes agentes e plataformas de IA.

## Objetivo

Transformar conhecimento, padrões e processos corporativos em **Skills
reutilizáveis, versionadas, seguras e independentes de plataforma**.

``` text
Agent Skills
     │
     ▼
Qodyn Skill Hub
     │
 ┌───┼────┬──────┐
 ▼   ▼    ▼      ▼
Claude Codex Cursor Gemini ...
```

## Principais recursos

-   **Skill Registry** --- catálogo central e versionado de Skills.
-   **Marketplace** --- descoberta, pesquisa e documentação.
-   **Multi-Agent** --- distribuição para Claude Code, Codex, Cursor e
    outras plataformas.
-   **Adapter SDK** --- adaptação para plataformas com diferentes
    modelos de integração.
-   **CLI** --- instalação, atualização, sincronização e auditoria de
    Skills.
-   **Source Management** --- importação de Skills internas, GitHub e
    catálogos externos.
-   **Compatibility Layer** --- validação de conformidade com o padrão
    Agent Skills.
-   **Governança** --- ownership, aprovação, lifecycle e políticas
    corporativas.
-   **Segurança** --- provenance, hashing, assinatura, scanning e
    controle de permissões.
-   **Versionamento e Lockfile** --- instalações determinísticas e
    reproduzíveis.
-   **MCP** --- descoberta e consumo de Skills sob demanda por agentes.
-   **Observabilidade** --- métricas de utilização, versões, segurança e
    adoção.

## Princípios

-   **Open standard first**
-   **Multi-agent by design**
-   **Native integration first**
-   **Security by default**
-   **API & CLI first**
-   **Enterprise governance**
-   **Portabilidade e baixo lock-in**

## Arquitetura

``` text
              Agent Skills Standard
                       │
              Compatibility Layer
                       │
                Qodyn Skill Hub
                       │
        ┌──────────────┼──────────────┐
        │              │              │
     Registry      Governance     Marketplace
        │
  Distribution Engine
        │
     Adapter SDK
        │
 ┌──────┼──────┬───────┐
 ▼      ▼      ▼       ▼
Claude Codex Cursor  Outros
```

## Referências

O projeto utiliza como principais referências:

-   **Agent Skills** --- especificação oficial do padrão.
-   **Tech Leads Club / Agent Skills** --- referência para CLI,
    Registry, distribuição, cache e MCP.
-   **Anthropic Skills** --- padrões de estrutura e authoring.
-   **Addy Osmani / Agent Skills** --- workflows, quality gates e boas
    práticas.

## Status

🚧 **Em desenvolvimento**

O projeto está inicialmente focado na construção do **core do Skill
Hub**, Compatibility Layer, Registry, CLI, Source Management e primeiros
adapters.

------------------------------------------------------------------------

**Qodyn Skill Hub** --- Transformando conhecimento corporativo em
capacidades reutilizáveis para agentes de IA.
