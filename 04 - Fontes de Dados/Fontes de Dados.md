---
title: Fontes de Dados
aliases:
  - Fonte de Dados
  - Data Sources
tags:
  - aegis
  - dados
  - fonte-dados
projeto: "[[Aegis]]"
tipo: fonte-dados
status: ativo
created: 2026-07-23
updated: 2026-07-23
---

# Fontes de Dados

Fontes de dados são entradas externas ou internas que alimentam decisões, visualizações ou reconciliação.

## Matriz

| Fonte | Tipo | Credencial | Status | Nota |
| --- | --- | --- | --- | --- |
| Binance Futures WebSocket | Candles fechados | Não | Implementado | [[Market Data Binance]] |
| Binance Futures REST klines | Candles históricos/backfill | Não | Implementado como client | [[Market Data Binance]] |
| Quant Engine fake | Sinal quantitativo | Não | Implementado | [[Quant Engine]] |
| Quant Engine gRPC | Sinal quantitativo externo | Depende do serviço | Contrato pronto, client real pendente | [[Quant Engine]] |
| RSS | Notícias/eventos | Não | Provider implementado, scheduler pendente | [[Eventos de Mercado e LLM]] |
| LLM noop/Gemini | Classificação de eventos | Gemini exige API key | Implementado | [[Eventos de Mercado e LLM]] |
| Broker Paper/Binance | Estado de ordens, posições e conta | Binance exige API key | Implementado | [[06 - Segurança e Risco]] |

## Critérios por fonte

- Origem e endpoint.
- Credenciais exigidas.
- Freshness e relógio usado.
- Retry/backoff.
- Contrato de entrada e saída.
- Falhas conhecidas e comportamento seguro.

> [!tip] Regra de separação
> Se uma nota descreve de onde vem dado e qual garantia ele possui, fica aqui. Se descreve como o sistema usa esse dado em uma decisão, fica em HLD ou DLS.

## Relacionado

- [[03 - Fluxo de Trading]]
- [[05 - Configuração]]
- [[Contratos e Eventos]]
