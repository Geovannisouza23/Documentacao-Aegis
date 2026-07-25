---
title: Visão Geral
aliases:
  - Overview Aegis
  - Introdução ao Aegis
tags:
  - aegis
  - projeto/visao-geral
  - documentacao
projeto: "[[Aegis]]"
tipo: nota
status: ativo
created: 2026-07-23
updated: 2026-07-24
---

# Visão Geral

[[Aegis]] é um sistema operacional para automação de trading, hoje composto por três repositórios independentes: `trading-core` (Go — a única autoridade operacional: risco, sizing e execução), `quant-engine` (Rust — indicadores, features, regime, estratégias, backtest/otimização, inferência ONNX) e `training-pipeline` (Python — treino, validação, avaliação e exportação dos modelos que o `quant-engine` consome). `trading-core` continua concentrando as decisões sensíveis: avaliação de sinais, risco, execução, estado de conta/posição, reconciliação e API.

## O que existe hoje

- Núcleo operacional em Go (`trading-core`).
- Modo PAPER com conta simulada de US$ 10.000.
- Broker paper e adapters para Binance testnet/real.
- Porta para Quant Engine externo — hoje falando com o modo `fake` local; ver abaixo.
- Porta para LLM/event intelligence.
- PostgreSQL com migrations embutidas.
- HTTP API, WebSocket de dashboard e worker de outbox.
- Testes unitários, arquitetura, contrato, integração e falha.
- **Quant Engine (Rust)** — serviço separado completo e testado, rodando de ponta a ponta (`quant-realtime` serve gRPC + HTTP). Ver [[Quant Engine]].
- **Training Pipeline (Python)** — serviço separado completo e testado, produzindo artefatos ONNX + Model Registry que o `quant-engine` sabe ler. Ver [[Training Pipeline]].

## Fora de escopo nesta entrega

- Client gRPC real do `trading-core` contra o `quant-engine` — o serviço Rust já existe e roda; falta o lado Go migrar do modo `fake` para o contrato `quant.v1` completo.
- Endpoint HTTP de backtest do `trading-core` chamando o motor real do `quant-engine` — o motor (`RunBacktest`/`GetBacktestResult`/`StreamBacktestProgress`) já existe e é testado no Rust; falta o `trading-core` expô-lo via `POST /v1/backtest/request`.
- Integração real ponta a ponta entre `training-pipeline` e um dataset de verdade exportado pelo `quant-engine` — hoje só há testes contra fixtures sintéticos.
- Dashboard TypeScript.
- Infraestrutura cloud.

> [!tip] Leitura útil
> Use esta nota como porta de entrada. Depois siga para [[Índice - HLD]], [[Índice - DLS]] e [[Índice - Fontes de Dados]].

## Nomes importantes

| Nome | Significado |
| --- | --- |
| Aegis | Nome do projeto |
| trading-core | Repositório/serviço Go — única autoridade operacional |
| quant-engine | Repositório/serviço Rust — sinal, features, backtest, inferência |
| training-pipeline | Repositório/serviço Python — treino e exportação de modelos ML |
| PAPER | Modo simulado, sem capital real |
| TESTNET | Modo de teste com exchange testnet |
| REAL | Modo com broker real e risco financeiro |

## Onde encontrar

| Preciso entender | Nota |
| --- | --- |
| Arquitetura macro | [[02 - Arquitetura]] |
| Pipeline de trading | [[03 - Fluxo de Trading]] |
| Dados de entrada | [[Fontes de Dados]] |
| Configuração e contratos | [[Índice - DLS]] |
| Operação local | [[04 - Execução Local]] |
