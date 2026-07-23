---
title: Contratos e Eventos
aliases:
  - Contratos Aegis
  - Event Contracts
tags:
  - aegis
  - arquitetura/dls
  - contratos
  - eventos
projeto: "[[Aegis]]"
tipo: dls
status: ativo
created: 2026-07-23
updated: 2026-07-23
---

# Contratos e Eventos

Esta nota centraliza contratos que outros componentes ou adapters devem respeitar.

## Contratos atuais

| Contrato | Caminho | Consumidor |
| --- | --- | --- |
| OpenAPI HTTP | `../trading-core/docs/openapi.yaml` | Dashboard e clientes HTTP |
| Eventos de outbox | `../trading-core/internal/contracts/events/README.md` | Worker de outbox e WebSocket |
| Quant gRPC | `../trading-core/internal/contracts/grpc/quant/quant_engine.proto` | Futuro Quant Engine Rust |
| LLM schema | `../trading-core/internal/contracts/schemas/llm/event_assessment.schema.json` | Adapter Gemini e futuros providers |

## Eventos duráveis de outbox

| Evento | Produtor |
| --- | --- |
| `OrderCreated` | `ExecuteApprovedOrder` |
| `OrderSubmitted` | `ExecuteApprovedOrder` |
| `OrderFilled` | `ExecuteApprovedOrder` |
| `PositionOpened` | `ExecuteApprovedOrder` |
| `KillSwitchActivated` | `ActivateKillSwitch` |
| `CriticalEventDetected` | `ProcessMarketEvent` |
| `ReconciliationDivergenceDetected` | `ReconcileBrokerState` |

> [!note] Versionamento
> Mudança incompatível deve criar novo `event_type`, como `OrderFilled.v2`, em vez de mudar significado de evento existente.

## Barreiras de contrato

- [[Quant Engine]] propõe sinal, mas não aprova nem envia ordem.
- [[Eventos de Mercado e LLM]] classifica notícia em schema estruturado, mas não executa ação direta.
- [[07 - API e Dashboard]] expõe ações operacionais; modo REAL não é habilitado por endpoint HTTP.

## Relacionado

- [[03 - Fluxo de Trading]]
- [[Banco de Dados]]
- [[09 - Decisões Arquiteturais]]
