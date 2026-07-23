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
updated: 2026-07-23
---

# Visão Geral

[[Aegis]] é um sistema operacional para automação de trading. Nesta fase, ele entrega o núcleo Go chamado `trading-core`, que concentra decisões sensíveis: avaliação de sinais, risco, execução, estado de conta/posição, reconciliação e API.

## O que existe hoje

- Núcleo operacional em Go.
- Modo PAPER com conta simulada de US$ 10.000.
- Broker paper e adapters para Binance testnet/real.
- Porta para Quant Engine externo.
- Porta para LLM/event intelligence.
- PostgreSQL com migrations embutidas.
- HTTP API, WebSocket de dashboard e worker de outbox.
- Testes unitários, arquitetura, contrato, integração e falha.

## Fora de escopo nesta entrega

- Engine quantitativo real em Rust.
- Dashboard TypeScript.
- Infraestrutura cloud.
- Motor de backtest completo.

> [!tip] Leitura útil
> Use esta nota como porta de entrada. Depois siga para [[Índice - HLD]], [[Índice - DLS]] e [[Índice - Fontes de Dados]].

## Nomes importantes

| Nome | Significado |
| --- | --- |
| Aegis | Nome do projeto |
| trading-core | Repositório/serviço Go atual |
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
