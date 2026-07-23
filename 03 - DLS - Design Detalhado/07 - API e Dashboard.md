---
title: API e Dashboard
aliases:
  - API Aegis
  - Dashboard
tags:
  - aegis
  - api
  - dashboard
projeto: "[[Aegis]]"
tipo: referencia
status: ativo
created: 2026-07-23
updated: 2026-07-23
---

# API e Dashboard

O `trading-core` expõe HTTP API e WebSocket para consumo por dashboard futuro.

## Autenticação

Todos endpoints `/v1/*` exigem JWT HS256 com claim `roles`.

| Papel | Uso |
| --- | --- |
| `viewer` | Consultas GET |
| `operator` | Ações operacionais |
| `admin` | Ações operacionais e administrativas |

## Endpoints sem autenticação

- `/health`
- `/ready`
- `/metrics`
- `/ws`

> [!info] WebSocket público
> O WebSocket de dashboard é intencionalmente sem JWT nesta entrega. CORS e desenho do dashboard devem ser revisados quando o frontend real existir.

## Principais superfícies

- Dashboard query.
- Consulta de conta, posições, ordens, sinais, decisões e incidentes.
- Ações de sistema: pause, resume, close-only e kill-switch.
- Reset de estado PAPER.
- Solicitação de backtest retorna `501 Not Implemented`.

## Contrato

O contrato OpenAPI fica em `../trading-core/docs/openapi.yaml`.

## Documentação técnica relacionada

| Área | Nota |
| --- | --- |
| Eventos enviados ao dashboard | [[Contratos e Eventos]] |
| Estado persistido para consultas | [[Banco de Dados]] |
| Modos operacionais acionáveis | [[Modos Operacionais]] |
| Riscos do WebSocket público | [[06 - Segurança e Risco]] |

## Relacionado

- [[03 - Fluxo de Trading]]
- [[10 - Backlog e Lacunas]]
