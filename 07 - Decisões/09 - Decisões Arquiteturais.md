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
updated: 2026-07-23
---

# Decisões Arquiteturais

O repositório mantém ADRs em `../trading-core/docs/decisions`.

## ADRs atuais

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

> [!tip] Manutenção
> Toda decisão nova que muda fronteiras, risco, persistência, integração externa ou operação deve ganhar uma ADR.

## Links internos

- [[02 - Arquitetura]]
- [[03 - Fluxo de Trading]]
- [[06 - Segurança e Risco]]
- [[Contratos e Eventos]]
- [[Banco de Dados]]

## Quando criar nova ADR

- Mudança em direção de dependência ou fronteira de camada.
- Novo adapter externo que muda contrato operacional.
- Alteração incompatível em evento, API, schema ou banco.
- Mudança em modo REAL, risco, reconciliação ou kill switch.
