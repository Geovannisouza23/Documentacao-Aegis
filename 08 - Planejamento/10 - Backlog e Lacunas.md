---
title: Backlog e Lacunas
aliases:
  - Lacunas Aegis
  - Roadmap Aegis
tags:
  - aegis
  - backlog
  - roadmap
projeto: "[[Aegis]]"
tipo: backlog
status: ativo
created: 2026-07-23
updated: 2026-07-26
---

# Backlog e Lacunas

Esta nota lista lacunas conhecidas da entrega atual e próximos passos naturais.

## Lacunas conhecidas

### Produto e operação

- [x] ~~Implementar engine real de backtest para `POST /v1/backtest/request` no `trading-core`~~ — feito junto com o client gRPC real (ADR 0010 do `trading-core`): `RunBacktest`/`GetBacktestResult`/`StreamBacktestProgress` (SSE) agora respondem de verdade, `QUANT_MODE=grpc` requerido.
- [ ] Criar checklist operacional para PAPER, TESTNET e REAL.
- [ ] Validar integração TESTNET end to end com chaves reais de testnet.

### Fontes de dados

- [ ] Agendar ingestão de notícias RSS.
- [x] ~~Substituir Quant fake por client gRPC real no `trading-core`~~ — feito: client real contra `quant.v1` completo (15 RPCs), `QUANT_MODE=grpc` liga. `QUANT_MODE` continua `fake` por padrão (ver ADR 0010 do `trading-core`). Ver [[Quant Engine]].
- [ ] Adicionar health probe real para Quant Engine e LLM — o `quant-engine` já expõe `/health` e `/ready`; falta o `trading-core` consultar isso.
- [ ] `RegisterDecisionOutcome` não é disparado automaticamente pela reconciliação de ordens do `trading-core` — o endpoint HTTP novo (`POST /v1/decisions/{id}/outcome`) aceita o outcome fornecido pelo caller; ligar isso à reconciliação real exigiria propagar `signal_id`/`decision_id` pelo domínio e uma migração de banco (decisão registrada na ADR 0010 do `trading-core`).
- [ ] `trading-core` não busca candles históricos por conta própria para os novos endpoints de backtest/otimização/features — o caller precisa fornecer o array de candles no corpo da requisição, espelhando o próprio contrato gRPC do `quant-engine`. Um repositório de candles históricos (backfill por período arbitrário) é feature separada, não construída ainda.
- [x] ~~`trading-core` não tinha NATS real (só um placeholder morto de Google Pub/Sub) nem uso definido pra Redis; `quant-engine` não tinha Redis~~ — feito (25/07/2026): NATS JetStream real dos dois lados (Go publica nos 2 subjects que o Rust já esperava e consome os eventos de conclusão de backtest/otimização), Redis com propósito real nos dois lados (cache de market data + idempotência cross-serviço), rede Docker externa compartilhada (`aegis-net`). Ver ADR 0011 do `trading-core`, ADR 0009 do `quant-engine`, [[Quant Engine]] e [[Contratos e Eventos]].

### Quant Engine (Rust) — gaps do próprio serviço

Detalhado em `../quant-engine/docs/architecture.md` (seção "Known gaps") e nas ADRs 0002/0005/0006 do `quant-engine`; resumo aqui para não duplicar:

- [ ] `EvaluateSignal` não consulta o modelo de inferência ainda — toda resposta volta com `inference_mode: DISABLED`, mesmo quando o caller pede outro modo.
- [ ] `DecisionRepository` (árvore `TrainingRecord`, usada por `RegisterDecisionOutcome`/`ExportDataset`) só tem implementação em memória — outcomes não sobrevivem a restart do processo.
- [ ] Nenhum arquivo `.onnx` de fixture existe no repo — o caminho de carga bem-sucedida do `ModelLoader` (via ONNX Runtime real) não é coberto pela suíte automatizada, só os caminhos de rejeição. **Agora que o `training-pipeline` existe e exporta ONNX real e auto-verificado, gerar esse fixture cross-repo é um próximo incremento natural** — ver [[Training Pipeline]].
- [ ] Walk-forward e Monte Carlo existem como algoritmos de domínio testados, mas não estão conectados a `RunOptimization` — uma requisição com essas flags roda a mesma busca de passo único de sempre.
- [ ] `ObjectStorage` está implementado e testado mas não é usado por nenhum caso de uso ainda.
- [ ] `quant-backtest-worker`/`quant-feature-worker`/`quant-optimizer-worker` ainda não são especializados por tipo de job — os três rodam o mesmo processo genérico hoje (ADR 0002 do `quant-engine`).

### Training Pipeline (Python) — gaps do próprio serviço

As 13 etapas planejadas estão completas (ver [[Training Pipeline]]); o gatilho automático de treino (Fase 2, 26/07/2026) fechou boa parte dos gaps de integração cross-repo que existiam aqui até esta atualização:

- [x] ~~`quant-engine` ainda não calcula/popula labels reais~~ — feito (26/07/2026): `LabelPendingDecisionsUseCase` roda em background e popula `label_version`/`profitable_trade`/`net_return`/`forward_return_n_candles` de verdade. Ver [[Quant Engine]] e [[Training Pipeline]].
- [x] ~~`registry_root` do `training-pipeline` e `model_registry_root` do `quant-engine` apontam por padrão para caminhos locais desacoplados~~ — feito (26/07/2026) para o cenário Docker: os dois `docker-compose.yml` agora montam `../shared-data/{datasets,model-registry}` por padrão (bind mount, não volume nomeado). O cenário não-Docker (`.env` local) continua desacoplado por padrão, por design.
- [ ] O gatilho automático de treino (`idle_watcher.py` → `run_nightly_pipeline` → `quant.model.approved` → reload) está validado ponta a ponta por um smoke test real dos três serviços, mas ainda contra um dataset **sintético** — nenhuma execução aconteceu ainda contra um dataset genuinamente exportado pelo `quant-engine` a partir de mercado ao vivo. Ver [[Training Pipeline]].
- [ ] `scripts/s3_archive_sync.py` (sync de durabilidade pro S3) não tem agendamento automático — depende de o operador configurar um cron/systemd timer manualmente; não existe runbook publicado para isso ainda.
- [ ] `docs/decisions/` do `training-pipeline` está vazia — decisões estruturais (versão lexicográfica, `CANARY` fora de escopo, `registry_root` desacoplado por padrão, arquitetura híbrida de armazenamento) existem em prosa (ver [[09 - Decisões Arquiteturais]]) mas não como ADR formal.

### API e dashboard

- [ ] Revisar estratégia de autenticação do WebSocket quando dashboard existir.
- [ ] Criar dashboard TypeScript consumindo HTTP API e WebSocket.

## Próximos incrementos sugeridos

- [ ] Adicionar CI para `make test-unit`, `test-architecture` e `test-contract`.
- [ ] Criar release notes por tag.
- [ ] Documentar runbooks de incidente.
- [ ] Adicionar notas DLS para novos endpoints quando dashboard existir.

> [!question] Decisão pendente
> Definir onde o vault Obsidian será mantido no longo prazo: junto ao workspace do projeto, em repo próprio ou em vault pessoal sincronizado.

## Relacionado

- [[Fontes de Dados]]
- [[Eventos de Mercado e LLM]]
- [[Índice - Operação]]
