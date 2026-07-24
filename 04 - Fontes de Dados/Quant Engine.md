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
updated: 2026-07-24
---

# Quant Engine

Quant Engine é a fonte de sinal quantitativo do Aegis: um serviço Rust separado (`../quant-engine`) que consome candles/snapshots de mercado e retorna uma proposta de sinal, features, análise de regime e resultados de backtest/otimização. Ele nunca aprova risco, nunca dimensiona posição e nunca envia ordem — só o `trading-core` (Go) faz isso, depois que seu próprio Risk Manager aprova.

> [!info] Status
> O serviço Rust está implementado e roda de ponta a ponta (`quant-realtime` sobe gRPC + HTTP, testado manualmente e por 245+ testes automatizados). O lado Go (`trading-core`) ainda fala com o modo `fake` local — a migração do client Go para o gRPC real contra o novo contrato `quant.v1` é o item pendente, não o serviço Rust em si. Ver [[10 - Backlog e Lacunas]].

## Porta Go (consumidor)

| Item | Valor |
| --- | --- |
| Interface | `output.QuantEngine` |
| Caminho | `../trading-core/internal/application/ports/output/quant_engine.go` |
| Métodos usados hoje | `EvaluateSignal`, `AnalyzeMarketRegime` |
| Entrada | Candle atual e candles recentes |
| Saída | `QuantSignal` com símbolo, lado, preços, confiança, estratégia e regime |

## Implementações do lado Go

| Modo | Status | Observação |
| --- | --- | --- |
| `fake` | Implementado, em uso | Regra determinística in-process para desenvolvimento |
| `grpc` | Client pendente | O serviço real (Rust) já existe; falta o client Go migrar do placeholder de 2 RPCs para o contrato `quant.v1` completo |

## Serviço Rust (`../quant-engine`)

Arquitetura em camadas (Clean Architecture/Ports & Adapters, crate único): `domain` (indicadores, features, regime, estratégias, backtest, otimização — zero dependência de framework/transporte) → `application` (casos de uso, DTOs, mappers) → `interfaces` (gRPC + HTTP) e `infrastructure` (NATS JetStream, Postgres, ONNX Runtime, Parquet) implementando as portas de saída da `application`. Composição em `src/app`.

Quatro binários compartilham a mesma composition root: `quant-realtime` (serve gRPC + HTTP + consumers NATS — hoje o único que executa qualquer coisa) e `quant-backtest-worker` / `quant-feature-worker` / `quant-optimizer-worker` (hoje processos genéricos com HTTP health/ready/metrics + os mesmos dois consumers NATS, ainda não especializados por tipo de job — ver ADR 0002 do quant-engine).

### Contrato gRPC — `quant.v1`

| Item | Valor |
| --- | --- |
| Proto | `../quant-engine/proto/quant/v1/quant_engine.proto` |
| Serviço | `QuantEngineService`, 15 RPCs |
| `MarketRegime` | 10 estados (`TrendingUp`, `TrendingDown`, `Ranging`, `HighVolatility`, `LowVolatility`, `Breakout`, `Panic`, `Illiquid`, `NewsEvent`, `Unknown`) |
| Convenção de valores monetários | decimal-as-string em todo o wire (nunca float) |
| Compatibilidade | **Não** é compatível byte-a-byte com o placeholder antigo em `../trading-core/internal/contracts/grpc/quant/quant_engine.proto` (2 RPCs, regime de 3 estados) — é um redesenho deliberado, autorizado pela ADR 0008 do `trading-core`. Ver ADR 0001 do quant-engine. |

RPCs: `EvaluateSignal`, `CalculateFeatures`, `AnalyzeMarketRegime`, `EvaluateModel`, `RegisterDecisionOutcome`, `ValidateStrategy`, `RunBacktest`, `GetBacktestResult`, `StreamBacktestProgress`, `RunOptimization`, `GetOptimizationResult`, `ExportDataset`, `GetFeatureSchema`, `GetModelMetadata`, `ReloadApprovedModel`.

### Escopo de responsabilidade

O serviço recebe candles/snapshots, valida, calcula indicadores/features (34 features versionadas em `v1`), detecta regime, roda estratégias (`momentum`, `breakout`, `trend_following`, `mean_reversion`, mais um `ensemble` combinando várias sob demanda), consulta modelo **já aprovado** para inferência, roda backtest/otimização, gera registros de treino e exporta datasets Parquet.

> [!warning] Barreira de execução
> Quant Engine só propõe. Nunca decide tamanho de posição, alavancagem ou aprovação de risco, e não tem SDK de corretora nem capacidade de enviar ordem — isso é verificado em código por testes de arquitetura (`tests/architecture` no repo Rust), não só por convenção.

### Inferência de modelo

Quatro providers (`disabled` por padrão, `mock`, `onnx` via ONNX Runtime sempre compilado, `remote_grpc`), todos atrás de um `ModelRegistry` somente-leitura — o serviço nunca treina nem promove modelo, só lê estados `APPROVED`/`CANARY`/`PRODUCTION` já definidos externamente. Ver `../quant-engine/docs/model-inference.md`.

> [!note] Gap conhecido
> `EvaluateSignal` ainda não consulta o modelo de inferência — toda resposta hoje volta com `inference_mode: DISABLED`, mesmo que o caller peça outro modo. `EvaluateModel` (chamada direta) já funciona.

### Persistência e eventos

- Backtests/otimizações: Postgres se `database.required=true`, senão em memória.
- Decisões/`TrainingRecord` (usado por `RegisterDecisionOutcome` e `ExportDataset`): **só em memória hoje** — não sobrevive a restart. Gap conhecido, ver ADR 0005 do quant-engine.
- Eventos quantitativos (`SignalEvaluated`, `BacktestCompleted`, etc.) publicados best-effort em NATS JetStream; nunca falha uma chamada síncrona.

## Documentação própria do serviço

O repo `../quant-engine` mantém sua própria documentação técnica (não duplicada aqui):

| Assunto | Nota |
| --- | --- |
| Arquitetura completa, fluxo, gaps conhecidos | `../quant-engine/docs/architecture.md` |
| Detalhe de cada RPC, idempotência, mapeamento de erro | `../quant-engine/docs/grpc-contract.md` |
| Eventos publicados/consumidos | `../quant-engine/docs/event-contracts.md` |
| As 34 features do schema `v1` | `../quant-engine/docs/feature-contracts.md` |
| Providers e algoritmo de load do modelo | `../quant-engine/docs/model-inference.md` |
| Layout de partição e schema Parquet | `../quant-engine/docs/dataset-generation.md` |
| Estratégias e ensemble | `../quant-engine/docs/strategies.md` |
| Engine de backtest, execução, otimização | `../quant-engine/docs/backtesting.md` |
| ADRs (8 decisões, incluindo por que 5 dos 7 consumers NATS planejados não foram construídos) | `../quant-engine/docs/decisions/` |

## Relacionado

- [[Market Data Binance]]
- [[03 - Fluxo de Trading]]
- [[Contratos e Eventos]]
- [[10 - Backlog e Lacunas]]
- [[09 - Decisões Arquiteturais]]
