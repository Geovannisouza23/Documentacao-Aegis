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
updated: 2026-07-26
---

# Visão Geral

[[Aegis]] é um sistema operacional para automação de trading, hoje composto por três repositórios independentes: `trading-core` (Go — a única autoridade operacional: risco, sizing e execução), `quant-engine` (Rust — indicadores, features, regime, estratégias, backtest/otimização, inferência ONNX) e `training-pipeline` (Python — treino, validação, avaliação e exportação dos modelos que o `quant-engine` consome). `trading-core` continua concentrando as decisões sensíveis: avaliação de sinais, risco, execução, estado de conta/posição, reconciliação e API.

## O que existe hoje

- Núcleo operacional em Go (`trading-core`).
- Modo PAPER com conta simulada de US$ 10.000.
- Broker paper e adapters para Binance testnet/real.
- Porta para Quant Engine externo — `fake` (default) ou `grpc` (client real contra o `quant-engine`, `QUANT_MODE=grpc`); ver abaixo.
- Porta para LLM/event intelligence.
- PostgreSQL com migrations embutidas.
- HTTP API, WebSocket de dashboard e worker de outbox.
- Testes unitários, arquitetura, contrato, integração e falha.
- **Quant Engine (Rust)** — serviço separado completo e testado, rodando de ponta a ponta (`quant-realtime` serve gRPC + HTTP). Ver [[Quant Engine]].
- **Training Pipeline (Python)** — serviço separado completo e testado, produzindo artefatos ONNX + Model Registry que o `quant-engine` sabe ler. Ver [[Training Pipeline]].
- **Client gRPC real do `trading-core`** contra os 15 RPCs do `quant-engine` — sinal em tempo real (`EvaluateSignal`/`AnalyzeMarketRegime`) mais 13 endpoints HTTP novos (`backtest`, `optimization`, `dataset`, `strategy`, `quant/*`, `decisions/{id}/outcome`). `QUANT_MODE` continua `fake` por padrão. Ver ADR 0010 do `trading-core`.
- **Gatilho automático de treino** (26/07/2026) — `trading-core` detecta ociosidade (zero posições/ordens abertas) e publica `quant.activity.changed`; `training-pipeline` treina, avalia e promove um modelo até `APPROVED` sozinho, publicando `quant.model.approved` pro `quant-engine` recarregar, sem intervenção manual. Armazenamento híbrido: volume local compartilhado no caminho crítico, S3 só como backup de durabilidade. Validado por um smoke test real dos três serviços juntos. Ver [[Training Pipeline]].

## Fora de escopo nesta entrega

- Integração real ponta a ponta entre `training-pipeline` e um dataset de verdade exportado pelo `quant-engine` — o gatilho automático (acima) já roda ponta a ponta, mas ainda só contra fixtures sintéticos.
- `trading-core` buscar candles históricos por conta própria para os novos endpoints de backtest/otimização/features — o caller precisa fornecer o array de candles, espelhando o próprio contrato gRPC do `quant-engine`.
- `RegisterDecisionOutcome` disparado automaticamente pela reconciliação de ordens (hoje é caller-supplied via HTTP).
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
