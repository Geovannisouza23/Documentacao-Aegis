---
title: Decisões Arquiteturais
aliases:
  - ADRs Aegis
  - Architecture Decision Records
tags:
  - aegis
  - arquitetura/decisoes
  - adr
projeto: "[[Aegis]]"
tipo: indice
status: ativo
created: 2026-07-23
updated: 2026-07-26
---

# Decisões Arquiteturais

Este vault documenta dois conjuntos de ADRs, mantidos cada um no repositório a que pertence.

## ADRs do `trading-core` (Go)

Mantidas em `../trading-core/docs/decisions`.

| ADR | Tema |
| --- | --- |
| 0001 | Clean Architecture |
| 0002 | Uber Fx para composição |
| 0003 | pgx sem ORM |
| 0004 | Outbox transacional |
| 0005 | Separação `interfaces` vs `external` |
| 0006 | Pasta única de banco |
| 0007 | PaperBroker |
| 0008 | gRPC para integração futura com Rust |
| 0009 | EventBus em memória para MVP |
| 0010 | Client gRPC real contra `quant-engine`, todos os 15 RPCs (parcialmente supera a 0008) |
| 0011 | Client NATS real + Redis (cache de market data, idempotência cross-serviço) — supera parcialmente a 0009 |

## ADRs do `quant-engine` (Rust)

Mantidas em `../quant-engine/docs/decisions`. Cobrem as decisões tomadas durante a implementação completa do serviço — ver [[Quant Engine]] para o resumo do que cada uma significa na prática.

| ADR | Tema |
| --- | --- |
| 0001 | Proto `quant.v1` redesenha, não estende, o placeholder do `trading-core` |
| 0002 | Jobs pesados (backtest/otimização/export) rodam in-process, não em worker separado |
| 0003 | Model Registry MVP em filesystem local |
| 0004 | `ort` (bindings ONNX Runtime) como dependência sempre compilada — risco de rc-only aceito |
| 0005 | Postgres é efetivamente obrigatório uma vez que `RegisterDecisionOutcome` é exercitado (gap: adapter ainda não existe) |
| 0006 | Só 2 dos 7 consumers NATS originalmente planejados foram construídos |
| 0007 | Colunas monetárias no Parquet são string, não `Decimal128` nativo |
| 0008 | Um único semáforo compartilhado entre `Backtest`/`FeatureBatch`/`Optimization` |
| 0009 | Extensão do Inbox Pattern com Redis (dedup cross-processo/cross-restart para os 2 consumers reais) |

## ADRs do `training-pipeline` (Python)

> [!warning] Sem ADRs próprias ainda
> `../training-pipeline/docs/decisions/` existe mas está vazia. As decisões estruturais do serviço (7 estados de lifecycle vs. 3 reconhecidos pelo `quant-engine`, versão lexicográfica, `registry_root` desacoplado por padrão, `CANARY` fora de escopo, e a arquitetura híbrida de armazenamento abaixo) estão documentadas em prosa em `../training-pipeline/docs/model-registry.md` e [[Training Pipeline]], mas não formalizadas como ADR. Ver [[10 - Backlog e Lacunas]].

### Gatilho automático de treino + armazenamento híbrido (26/07/2026, não formalizada como ADR)

Decisão tomada em três iterações — as duas primeiras foram descartadas explicitamente antes da terceira ser aprovada:

1. **Descartada**: S3 como canal primário de troca de artefatos entre os serviços, treino numa máquina separada, agendamento por horário fixo (cron).
2. **Descartada**: mesma máquina para os três serviços, gatilho por detecção de ociosidade em vez de horário fixo, sem S3.
3. **Aprovada**: mesma máquina + gatilho por ociosidade (mantido da v2) **+ S3 de volta**, mas com papel estritamente de durabilidade — nunca no caminho crítico do treino.

Motivo da versão final: volume local compartilhado dá zero dependência de rede durante o treino (mais rápido, mais simples, sem custo de storage recorrente por operação) — mas não sobrevive a perda/corrupção da máquina, não tem histórico imutável de dataset/modelo, e não abre caminho pra mover o treino pra outra máquina no futuro. S3 resolve exatamente essas três lacunas sem reintroduzir a dependência de rede no caminho síncrono, porque fica isolado num processo (`scripts/s3_archive_sync.py`) que nunca é chamado pelo gatilho de treino (`scripts/idle_watcher.py`) — a separação de processo é o mecanismo que garante o isolamento, não uma convenção de código.

Ver [[Training Pipeline]] para a arquitetura completa e o smoke test que a validou.

> [!tip] Manutenção
> Toda decisão nova que muda fronteiras, risco, persistência, integração externa ou operação deve ganhar uma ADR — no repositório a que pertence.

## Links internos

- [[02 - Arquitetura]]
- [[03 - Fluxo de Trading]]
- [[06 - Segurança e Risco]]
- [[Contratos e Eventos]]
- [[Banco de Dados]]
- [[Quant Engine]]
- [[Training Pipeline]]

## Quando criar nova ADR

- Mudança em direção de dependência ou fronteira de camada.
- Novo adapter externo que muda contrato operacional.
- Alteração incompatível em evento, API, schema ou banco.
- Mudança em modo REAL, risco, reconciliação ou kill switch.
