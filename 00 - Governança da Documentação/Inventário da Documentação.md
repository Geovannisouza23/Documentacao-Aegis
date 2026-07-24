---
title: Inventário da Documentação
aliases:
  - Inventário Aegis
  - Catálogo da Documentação
tags:
  - aegis
  - documentacao/inventario
projeto: "[[Aegis]]"
tipo: indice
status: ativo
created: 2026-07-23
updated: 2026-07-24
---

# Inventário da Documentação

| Nota | Pasta | Tipo | Fonte primária |
| --- | --- | --- | --- |
| [[Aegis]] | raiz | `indice` | Vault |
| [[Convenções da Documentação]] | Governança | `governanca` | Referências externas |
| [[Inventário da Documentação]] | Governança | `indice` | Vault |
| [[01 - Visão Geral]] | Produto | `nota` | `../trading-core/README.md` |
| [[02 - Arquitetura]] | HLD | `hld` | `../trading-core/docs/architecture.md` |
| [[03 - Fluxo de Trading]] | HLD | `hld` | `../trading-core/docs/architecture.md` |
| [[06 - Segurança e Risco]] | HLD | `nota-critica` | Domínio/config/broker |
| [[05 - Configuração]] | DLS | `referencia` | `../trading-core/internal/config` |
| [[07 - API e Dashboard]] | DLS | `referencia` | `../trading-core/docs/openapi.yaml` |
| [[Contratos e Eventos]] | DLS | `dls` | `../trading-core/internal/contracts` |
| [[Banco de Dados]] | DLS | `dls` | `../trading-core/internal/infrastructure/database` |
| [[Fontes de Dados]] | Fontes de Dados | `fonte-dados` | Adapters externos |
| [[Market Data Binance]] | Fontes de Dados | `fonte-dados` | `marketdata/{websocket,rest}` |
| [[Quant Engine]] | Fontes de Dados | `fonte-dados` | `ports/output/quant_engine.go` (Go) + `../quant-engine` (Rust, serviço completo) |
| [[Eventos de Mercado e LLM]] | Fontes de Dados | `fonte-dados` | `news`, `llm`, `schemas` |
| [[04 - Execução Local]] | Operação | `guia` | `../trading-core/README.md` |
| [[Modos Operacionais]] | Operação | `referencia` | `domain/operation` |
| [[08 - Testes e Qualidade]] | Qualidade | `referencia` | `../trading-core/tests` |
| [[09 - Decisões Arquiteturais]] | Decisões | `indice` | `../trading-core/docs/decisions` + `../quant-engine/docs/decisions` |
| [[10 - Backlog e Lacunas]] | Planejamento | `backlog` | README + análise do código + `../quant-engine/docs/architecture.md` |

## Manutenção

- Atualize este inventário ao criar, mover ou arquivar nota.
- Preserve uma linha por nota de conhecimento; templates não entram no inventário.
