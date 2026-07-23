---
title: Fluxo de Trading
aliases:
  - Pipeline de Trading
  - Fluxo PAPER
tags:
  - aegis
  - trading
  - arquitetura/fluxo
  - arquitetura/hld
projeto: "[[Aegis]]"
tipo: hld
status: ativo
created: 2026-07-23
updated: 2026-07-23
---

# Fluxo de Trading

O pipeline do modo PAPER começa com candles públicos da Binance Futures e termina em ordem simulada, posição aberta e eventos enviados ao dashboard.

```mermaid
sequenceDiagram
    participant Feed as Market Data
    participant Consumer as Market Consumer
    participant Signal as EvaluateMarketSignal
    participant Risk as EvaluateRisk
    participant Exec as ExecuteApprovedOrder
    participant Broker as PaperBroker
    participant Outbox as Outbox
    participant WS as Dashboard WebSocket

    Feed->>Consumer: Candle
    Consumer->>Signal: Avaliar sinal
    Signal->>Risk: TradeSignal
    Risk->>Exec: RiskDecision aprovado
    Exec->>Broker: PlaceOrder
    Broker-->>Exec: Fill simulado
    Exec->>Outbox: OrderFilled / PositionOpened
    Outbox->>WS: Evento para dashboard
```

## Sequência principal

1. `market.Consumer.HandleCandle` recebe candle.
2. `EvaluateMarketSignal` chama porta `QuantEngine`.
3. `TradeSignal` é persistido e publicado.
4. `EvaluateRisk` aplica política composta de risco.
5. `RiskDecision` é persistida.
6. `ExecuteApprovedOrder` cria ordem pendente, chama broker e registra fill.
7. Outbox persiste eventos críticos e worker republica eventos para consumidores.

> [!warning] Barreira de execução
> Nem LLM nem Quant Engine podem enviar ordem. Só `ExecuteApprovedOrder` chama `Broker.PlaceOrder`.

## Idempotência

- `ClientOrderID` deriva do `RiskDecisionID`.
- `IdempotencyRepository.Reserve` evita corrida de execução.
- Repositórios usam optimistic locking por `version`.
- Estado `Unknown` é usado quando broker pode ter aceitado ordem, mas resposta falhou.

## Relacionado

- [[Market Data Binance]]
- [[Quant Engine]]
- [[Contratos e Eventos]]
- [[06 - Segurança e Risco]]
- [[07 - API e Dashboard]]
