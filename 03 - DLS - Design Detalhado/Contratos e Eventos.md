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
updated: 2026-07-26
---

# Contratos e Eventos

Esta nota centraliza contratos que outros componentes ou adapters devem respeitar.

## Contratos atuais

| Contrato | Caminho | Consumidor |
| --- | --- | --- |
| OpenAPI HTTP | `../trading-core/docs/openapi.yaml` | Dashboard e clientes HTTP |
| Eventos de outbox | `../trading-core/internal/contracts/events/README.md` | Worker de outbox e WebSocket |
| Quant gRPC (placeholder, Go) | `../trading-core/internal/contracts/grpc/quant/quant_engine.proto` | Ainda usado pelo client Go (2 RPCs, não compilado) |
| Quant gRPC `quant.v1` (canônico, Rust) | `../quant-engine/proto/quant/v1/quant_engine.proto` | Client Go real (pendente) — 15 RPCs, substitui o placeholder acima, não é compatível byte-a-byte |
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

> [!info] Contrato `quant.v1` é um redesenho, não uma extensão
> O `trading-core` (ADR 0008) autorizou o lado Rust a evoluir livremente o contrato de integração. O resultado (`quant.v1`, 15 RPCs, `MarketRegime` de 10 estados) substitui o placeholder de 2 RPCs — não convive com ele nem é aditivo. Ver [[Quant Engine]] e o ADR 0001 em `../quant-engine/docs/decisions/`.

## Eventos NATS entre os serviços (atualizado 26/07/2026)

Os três lados agora falam NATS JetStream de verdade sobre o mesmo stream compartilhado (`QUANT_EVENTS`, subject filter `quant.>`, prefixo `quant` por padrão nos três repos) — a antiga fila `messaging/pubsub` do `trading-core` nunca foi NATS (placeholder morto do Google Cloud Pub/Sub) e foi removida. Envelope compartilhado: `EventEnvelope` (`event_id`, `event_type`, `event_version`, `aggregate_id`, `aggregate_type`, `aggregate_version`, `correlation_id`, `causation_id`, `idempotency_key`, `occurred_at`, `producer`, `payload`), mesmo shape nos três lados.

| Subject | Direção | Produtor | Consumidor |
| --- | --- | --- | --- |
| `quant.decision.outcome.received` | Go → Rust | `trading-core` (`POST /v1/decisions/{id}/outcome/publish`) | `quant-engine` (mesmo use case da RPC síncrona `RegisterDecisionOutcome`) |
| `quant.model.approved` | Go → Rust **e** Python → Rust | `trading-core` (`POST /v1/quant/model/approved/publish`) **ou** `training-pipeline` (`run_nightly_pipeline`, ao promover um modelo até `APPROVED`) | `quant-engine` (mesmo use case da RPC síncrona `ReloadApprovedModel`, indiferente a qual dos dois produziu) |
| `quant.backtest.*.events` (`BacktestCompleted`/`BacktestFailed`) | Rust → Go | `quant-engine` | `trading-core` (repassa pro WebSocket do dashboard) |
| `quant.optimization.*.events` (`OptimizationCompleted`) | Rust → Go | `quant-engine` | `trading-core` (repassa pro WebSocket do dashboard) |
| `quant.activity.changed` | Go → Python | `trading-core` (`startActivityWatcher`, ticker 30s, publica só na transição idle↔ativo — payload `{idle, changed_at}`) | `training-pipeline` (`idle_watcher.py`, consumer JetStream durável) |

> [!warning] `quant.activity.changed` não é um subject "livre" — precisa caber no stream existente
> O nome do subject foi escolhido inicialmente como `trading.activity.changed`, o que pareceria mais correto semanticamente (é sobre o `trading-core`), mas **não batia com o filtro do stream compartilhado** (`quant.>`) — o publish falhava silenciosamente com "no response from stream" porque não existe (por padrão) um stream separado escutando `trading.>`. Achado só num smoke test real com os três serviços rodando juntos, não por nenhum teste automatizado isolado. Todo subject novo de integração cross-serviço tem que usar o prefixo `quant.` pelo mesmo motivo, mesmo quando o assunto não é sobre o `quant-engine` — ver [[Training Pipeline]].

Redis complementa como cache/dedup opcional em Go+Rust (não usado pelo `training-pipeline`): candles recentes no Go (`output.MarketDataCache`) e idempotência cross-serviço no caminho de publish do Go + extensão do Inbox Pattern no consumo do Rust. Nenhum dos três lados exige NATS ou Redis para funcionar — sem eles, o Go cai para o event bus em memória (ADR 0009 do `trading-core`) e o Rust para o Inbox só-`moka` (ADR 0006 do `quant-engine`). Topologia: rede Docker externa compartilhada `aegis-net` (`docker network create aegis-net` uma vez), documentada nos `docker-compose.yml`/README de cada repo — o `training-pipeline` só entra nessa rede pelo serviço `watcher` (o `trainer`, de CLI ad-hoc, não precisa). Ver ADR 0011 do `trading-core` e ADR 0009 do `quant-engine`.

## Barreiras de contrato

- [[Quant Engine]] propõe sinal, mas não aprova nem envia ordem.
- [[Eventos de Mercado e LLM]] classifica notícia em schema estruturado, mas não executa ação direta.
- [[07 - API e Dashboard]] expõe ações operacionais; modo REAL não é habilitado por endpoint HTTP.

## Relacionado

- [[03 - Fluxo de Trading]]
- [[Banco de Dados]]
- [[09 - Decisões Arquiteturais]]
