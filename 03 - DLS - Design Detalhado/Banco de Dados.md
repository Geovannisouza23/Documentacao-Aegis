---
title: Banco de Dados
aliases:
  - PostgreSQL Aegis
  - Persistência Aegis
tags:
  - aegis
  - arquitetura/dls
  - banco
  - postgres
projeto: "[[Aegis]]"
tipo: dls
status: ativo
created: 2026-07-23
updated: 2026-07-23
---

# Banco de Dados

O `trading-core` usa PostgreSQL e concentra pool, migrations, transações e repositories em `../trading-core/internal/infrastructure/database`.

## Migrations

| Versão | Tabela |
| --- | --- |
| `000001` | `accounts` |
| `000002` | `orders` |
| `000003` | `order_executions` |
| `000004` | `positions` |
| `000005` | `trade_signals` |
| `000006` | `risk_decisions` |
| `000007` | `market_events` |
| `000008` | `account_snapshots` |
| `000009` | `operational_modes` |
| `000010` | `system_incidents` |
| `000011` | `idempotency_keys` |
| `000012` | `outbox_events` |

## Invariantes importantes

- `accounts_single_active_idx` mantém uma conta ativa.
- `orders_client_order_id_idx` protege idempotência de ordens.
- `positions_open_symbol_idx` limita posição aberta por símbolo.
- `idempotency_keys` usa chave composta `(scope, key)`.
- `outbox_events_pending_idx` acelera claim de eventos pendentes.
- Entidades críticas usam `version` para optimistic locking.

## Operação

```bash
cd ../trading-core
make migrate-up
make migrate-version
make migrate-down N=1
```

> [!warning] Recuperação
> `make migrate-force V=<versão>` só deve ser usado para recuperação controlada de estado de migration.

## Relacionado

- [[05 - Configuração]]
- [[Contratos e Eventos]]
- [[08 - Testes e Qualidade]]
