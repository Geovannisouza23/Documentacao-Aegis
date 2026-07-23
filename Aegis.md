---
title: Aegis
aliases:
  - Projeto Aegis
  - Aegis Trading Core
tags:
  - aegis
  - projeto
  - documentacao
tipo: indice
status: ativo
created: 2026-07-23
updated: 2026-07-23
---

# Aegis

Aegis é o projeto de automação operacional para trading. O componente atual do projeto é o `trading-core`, um núcleo em Go responsável por ingestão de mercado, avaliação de sinais, gestão determinística de risco, execução de ordens, reconciliação com broker e APIs para dashboard.

> [!info] Escopo desta documentação
> Este vault documenta o projeto em português e fica fora do repositório `trading-core`. O código-fonte permanece em `../trading-core`.

## Mapa por seção

| Seção | Uso | Entrada |
| --- | --- | --- |
| Governança | Regras, taxonomia e inventário | [[Convenções da Documentação]] |
| Produto | Escopo, visão geral e vocabulário | [[Índice - Produto]] |
| HLD | Arquitetura macro, fluxo e risco | [[Índice - HLD]] |
| DLS | Configuração, API, contratos e banco | [[Índice - DLS]] |
| Fontes de Dados | Origens, freshness, credenciais e limitações | [[Índice - Fontes de Dados]] |
| Operação | Execução local e modos operacionais | [[Índice - Operação]] |
| Qualidade | Testes e critérios de qualidade | [[Índice - Qualidade]] |
| Decisões | ADRs e decisões arquiteturais | [[Índice - Decisões]] |
| Planejamento | Backlog e lacunas | [[Índice - Planejamento]] |

## Ponto de partida rápido

1. Leia [[01 - Visão Geral]] para entender objetivo e fronteiras.
2. Leia [[02 - Arquitetura]] para entender camadas e dependências.
3. Consulte [[Fontes de Dados]] para entender entradas do pipeline.
4. Use [[04 - Execução Local]] para subir ambiente em modo PAPER.
5. Consulte [[06 - Segurança e Risco]] antes de qualquer uso fora de simulação.

## Trilhas rápidas

| Trilha | Sequência |
| --- | --- |
| Novo no projeto | [[01 - Visão Geral]] → [[02 - Arquitetura]] → [[03 - Fluxo de Trading]] |
| Implementar integração | [[Fontes de Dados]] → [[Contratos e Eventos]] → [[05 - Configuração]] |
| Operar localmente | [[04 - Execução Local]] → [[Modos Operacionais]] → [[08 - Testes e Qualidade]] |
| Revisar risco | [[06 - Segurança e Risco]] → [[Modos Operacionais]] → [[09 - Decisões Arquiteturais]] |

## Convenções do vault

- [[Convenções da Documentação]] define taxonomia e critérios de manutenção.
- [[Inventário da Documentação]] lista notas, tipos e fontes primárias.
- Wikilinks conectam notas internas.
- Callouts destacam decisões, riscos e próximos passos.
- Propriedades YAML no topo ajudam busca, filtros e futuras bases do Obsidian.
