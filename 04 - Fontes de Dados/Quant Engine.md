---
title: Quant Engine
aliases:
  - Motor Quantitativo
  - Quant Aegis
tags:
  - aegis
  - dados/quant
  - contratos
projeto: "[[Aegis]]"
tipo: fonte-dados
status: ativo
created: 2026-07-23
updated: 2026-07-23
---

# Quant Engine

Quant Engine é a fonte de sinal quantitativo. Ele consome candles e retorna uma proposta de sinal; nunca aprova risco nem envia ordem.

## Porta Go

| Item | Valor |
| --- | --- |
| Interface | `output.QuantEngine` |
| Caminho | `../trading-core/internal/application/ports/output/quant_engine.go` |
| Métodos | `EvaluateSignal`, `AnalyzeMarketRegime` |
| Entrada | Candle atual e candles recentes |
| Saída | `QuantSignal` com símbolo, lado, preços, confiança, estratégia e regime |

## Implementações

| Modo | Status | Observação |
| --- | --- | --- |
| `fake` | Implementado | Regra determinística in-process para desenvolvimento |
| `grpc` | Contrato pronto | Client real pendente nesta entrega |

## Contrato futuro

O `.proto` fica em `../trading-core/internal/contracts/grpc/quant/quant_engine.proto` e usa decimals como strings para evitar erro de ponto flutuante.

> [!warning] Barreira de execução
> Quant Engine só propõe. `EvaluateRisk` aprova ou rejeita. `ExecuteApprovedOrder` é o único caminho para `Broker.PlaceOrder`.

## Relacionado

- [[Market Data Binance]]
- [[03 - Fluxo de Trading]]
- [[Contratos e Eventos]]
