---
title: Market Data Binance
aliases:
  - Binance Market Data
  - Candles Binance
tags:
  - aegis
  - dados/market-data
  - binance
projeto: "[[Aegis]]"
tipo: fonte-dados
status: ativo
created: 2026-07-23
updated: 2026-07-23
---

# Market Data Binance

O feed de mercado atual usa Binance Futures para candles.

## WebSocket

| Item | Valor |
| --- | --- |
| Adapter | `../trading-core/internal/infrastructure/external/marketdata/websocket` |
| Base URL padrão | `wss://fstream.binance.com` |
| Símbolo padrão | `BTCUSDT` |
| Timeframe padrão | `1m` |
| Credencial | Não exige API key |
| Evento consumido | Candle fechado (`IsClosed == true`) |

O lifecycle mantém até `50` candles recentes em memória e reconecta com backoff até `30s` quando conexão cai.

## REST klines

| Item | Valor |
| --- | --- |
| Adapter | `../trading-core/internal/infrastructure/external/marketdata/rest` |
| Endpoint | `/fapi/v1/klines` |
| Uso esperado | Backfill, histórico recente ou fallback sem WebSocket |
| Campos principais | open, high, low, close, volume, open time, close time |

## Limitações conhecidas

- Feed WebSocket usa horário de chegada como `close_time` aproximado.
- Símbolo/timeframe padrão ainda estão fixos no lifecycle.
- REST client existe, mas pipeline atual usa WebSocket.

## Relacionado

- [[03 - Fluxo de Trading]]
- [[Fontes de Dados]]
- [[Quant Engine]]
