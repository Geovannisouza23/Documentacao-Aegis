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
updated: 2026-07-24
---

# Backlog e Lacunas

Esta nota lista lacunas conhecidas da entrega atual e próximos passos naturais.

## Lacunas conhecidas

### Produto e operação

- [ ] Implementar engine real de backtest para `POST /v1/backtest/request` no `trading-core` — note que o `quant-engine` (Rust) já tem `RunBacktest`/`GetBacktestResult`/`StreamBacktestProgress` implementados e testados; o que falta é o endpoint HTTP do `trading-core` chamar esse serviço em vez de retornar algo local.
- [ ] Criar checklist operacional para PAPER, TESTNET e REAL.
- [ ] Validar integração TESTNET end to end com chaves reais de testnet.

### Fontes de dados

- [ ] Agendar ingestão de notícias RSS.
- [ ] Substituir Quant fake por client gRPC real no `trading-core` — migrar de `output.QuantEngine`'s modo `fake` para um client contra `quant.v1` (15 RPCs). O serviço Rust já está pronto e rodando; este item é só o lado Go. Ver [[Quant Engine]].
- [ ] Adicionar health probe real para Quant Engine e LLM — o `quant-engine` já expõe `/health` e `/ready`; falta o `trading-core` consultar isso.

### Quant Engine (Rust) — gaps do próprio serviço

Detalhado em `../quant-engine/docs/architecture.md` (seção "Known gaps") e nas ADRs 0002/0005/0006 do `quant-engine`; resumo aqui para não duplicar:

- [ ] `EvaluateSignal` não consulta o modelo de inferência ainda — toda resposta volta com `inference_mode: DISABLED`, mesmo quando o caller pede outro modo.
- [ ] `DecisionRepository` (árvore `TrainingRecord`, usada por `RegisterDecisionOutcome`/`ExportDataset`) só tem implementação em memória — outcomes não sobrevivem a restart do processo.
- [ ] Nenhum arquivo `.onnx` de fixture existe no repo — o caminho de carga bem-sucedida do `ModelLoader` (via ONNX Runtime real) não é coberto pela suíte automatizada, só os caminhos de rejeição.
- [ ] Walk-forward e Monte Carlo existem como algoritmos de domínio testados, mas não estão conectados a `RunOptimization` — uma requisição com essas flags roda a mesma busca de passo único de sempre.
- [ ] `ObjectStorage` está implementado e testado mas não é usado por nenhum caso de uso ainda.
- [ ] `quant-backtest-worker`/`quant-feature-worker`/`quant-optimizer-worker` ainda não são especializados por tipo de job — os três rodam o mesmo processo genérico hoje (ADR 0002 do `quant-engine`).

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
