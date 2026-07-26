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
updated: 2026-07-26
---

# Quant Engine

Quant Engine é a fonte de sinal quantitativo do Aegis: um serviço Rust separado (`../quant-engine`) que consome candles/snapshots de mercado e retorna uma proposta de sinal, features, análise de regime e resultados de backtest/otimização. Ele nunca aprova risco, nunca dimensiona posição e nunca envia ordem — só o `trading-core` (Go) faz isso, depois que seu próprio Risk Manager aprova.

> [!info] Status
> O serviço Rust está implementado e roda de ponta a ponta (`quant-realtime` sobe gRPC + HTTP, testado manualmente e por 245+ testes automatizados). O lado Go (`trading-core`) agora tem um client gRPC real contra o contrato `quant.v1` completo (15 RPCs), substituindo o antigo placeholder de 2 RPCs — ver ADR 0010 do `trading-core`. **Validado de ponta a ponta**: 6 testes de integração (`tests/integration/quant_grpc_test.go`, `http_quant_test.go`) sobem o `quant-engine` real via Docker (Testcontainers) e provam sinal, regime, submissão/poll de backtest e mapeamento de erro passando pelos dois lados — gRPC direto e via HTTP. Ver [[10 - Backlog e Lacunas]] para o que ainda falta (produção real, não só teste automatizado).
>
> **Mensageria e cache agora reais também** (25/07/2026): o `trading-core` tinha uma fila (`messaging/pubsub`) que nunca foi NATS — era um placeholder morto do Google Cloud Pub/Sub, removido. Ele agora fala NATS JetStream de verdade dos dois lados: publica nos 2 subjects que o `quant-engine` já esperava (`quant.decision.outcome.received`, `quant.model.approved` — resolvendo a lacuna documentada na ADR 0006 do `quant-engine`) e consome os eventos de conclusão de job (`BacktestCompleted`/`BacktestFailed`/`OptimizationCompleted`), repassando pro WebSocket do dashboard. Redis também ganhou propósito real dos dois lados: cache de candles recentes no Go (com warm-start após restart) e idempotência compartilhada Go+Rust (guarda de publish no Go, extensão do Inbox Pattern no Rust) — sempre opcional, nunca bloqueia o caminho síncrono. Ver ADR 0011 do `trading-core` e ADR 0009 do `quant-engine`.
>
> **`quant.model.approved` ganhou um segundo produtor** (26/07/2026): antes só o `trading-core` publicava nesse subject; agora o `training-pipeline` também publica, ao final do próprio `run_nightly_pipeline` (gatilho automático de treino por ociosidade, ver [[Training Pipeline]]). O consumer `model_approved` deste serviço não muda — mesmo `EventEnvelope`, mesmo handler, indiferente a qual dos dois lados publicou. Validado com um smoke test real (não simulado): o `quant-engine` recarregou um modelo publicado pelo `training-pipeline`, confirmado pelo avanço limpo do `ack_floor` do consumer JetStream, sem restart.

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
| `fake` | Implementado, padrão | Regra determinística in-process para desenvolvimento — continua sendo o default (`QUANT_MODE=fake`) |
| `grpc` | Implementado | Client real contra o contrato `quant.v1` completo (15 RPCs), proto vendorizado do `quant-engine`. Sinal (`EvaluateSignal`/`AnalyzeMarketRegime`) alimenta o pipeline interno normalmente; os outros 13 RPCs (backtest, otimização, export de dataset, diagnóstico) ganharam endpoints HTTP novos em `/v1` — ver ADR 0010 |

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
| Compatibilidade | **Não** era compatível byte-a-byte com o placeholder antigo do `trading-core` (2 RPCs, regime de 3 estados, nunca compilado) — foi um redesenho deliberado, autorizado pela ADR 0008 do `trading-core`. Ver ADR 0001 do quant-engine. O placeholder foi removido: `trading-core` agora vendoriza uma cópia byte a byte deste proto em `internal/contracts/grpc/quant/v1/` (ADR 0010 do `trading-core`). |

RPCs: `EvaluateSignal`, `CalculateFeatures`, `AnalyzeMarketRegime`, `EvaluateModel`, `RegisterDecisionOutcome`, `ValidateStrategy`, `RunBacktest`, `GetBacktestResult`, `StreamBacktestProgress`, `RunOptimization`, `GetOptimizationResult`, `ExportDataset`, `GetFeatureSchema`, `GetModelMetadata`, `ReloadApprovedModel`. Todos os 15 têm client Go real e caso de uso próprio hoje — os dois primeiros alimentam o pipeline de sinal em tempo real, os outros 13 ficam atrás de endpoints HTTP novos em `/v1` no `trading-core` (`backtest`, `optimization`, `dataset`, `strategy`, `quant/*`, `decisions/{id}/outcome`), todos exigindo papel `operator`/`admin`.

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
- Dedup do lado consumidor (`decision_outcome_received`/`model_approved`): Inbox Pattern em memória (`moka`) por padrão, estendido com Redis opcional para sobreviver a restart e ser compartilhado entre os quatro binários — ver ADR 0009.
- **Labeling real (26/07/2026)**: `LabelPendingDecisionsUseCase` roda em background (poll por ticker, `app::lifecycle`) e calcula `OutcomeLabel` para cada `TrainingRecord` assim que seu horizonte forward-looking tem candle history suficiente (`CandleHistory`, nunca `Clock::now()` como critério de prontidão). Antes desta correção, `domain::services::label_calculator` existia e era testado, mas não estava ligado a nenhum caso de uso de produção — todo dataset exportado saía com colunas de label `NULL`. Ver [[Training Pipeline]] para o lado que consome.

### Volume compartilhado com o `training-pipeline` (26/07/2026)

`docker-compose.yml` monta `../shared-data/{datasets,model-registry}` via bind mount — a mesma convenção que o `training-pipeline` já usava, corrigindo um gap real: antes desta data o `quant-engine` usava **volume nomeado** (isolado ao próprio projeto compose), então nada era de fato compartilhado mesmo com os dois serviços configurados "corretamente". Ver [[Training Pipeline]].

## Bugs reais encontrados durante a integração Go↔Rust (25/07/2026)

Construir e rodar o `quant-engine` de verdade pela primeira vez (para os testes de integração do `trading-core`) achou dois bugs reais e pré-existentes no `Dockerfile`, nunca exercitado ponta a ponta antes:

- **Faltava `g++`** no estágio de build — `ort` (bindings ONNX Runtime) precisa linkar contra `libstdc++`, e o builder só instalava `ca-certificates`/`pkg-config`.
- **`bookworm` (Debian 12, glibc 2.36) é incompatível com o binário pré-compilado do ONNX Runtime** que o `ort` baixa — ele exige `__isoc23_strtoll`/`__isoc23_strtoull` (glibc 2.38+). Corrigido trocando builder e runtime para `trixie` (Debian 13).
- **Sem `.dockerignore`** — a pasta `target/` (15GB local) ia inteira pro contexto de build. Adicionado espelhando o `.gitignore`.

Esses três, juntos, explicam por que `docs/architecture.md`/CI do `quant-engine` nunca tinham pego isso: o job `docker build` do CI provavelmente nunca chegou a rodar até o fim, ou rodou num ambiente com glibc já compatível. Vale conferir o workflow de CI do `quant-engine` à luz disso.

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
| ADRs (9 decisões, incluindo por que 5 dos 7 consumers NATS planejados não foram construídos e a extensão Redis do Inbox Pattern) | `../quant-engine/docs/decisions/` |

## Relacionado

- [[Market Data Binance]]
- [[03 - Fluxo de Trading]]
- [[Contratos e Eventos]]
- [[10 - Backlog e Lacunas]]
- [[09 - Decisões Arquiteturais]]
