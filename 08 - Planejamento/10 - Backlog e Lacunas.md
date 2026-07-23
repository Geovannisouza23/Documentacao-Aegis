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
updated: 2026-07-23
---

# Backlog e Lacunas

Esta nota lista lacunas conhecidas da entrega atual e próximos passos naturais.

## Lacunas conhecidas

### Produto e operação

- [ ] Implementar engine real de backtest para `POST /v1/backtest/request`.
- [ ] Criar checklist operacional para PAPER, TESTNET e REAL.
- [ ] Validar integração TESTNET end to end com chaves reais de testnet.

### Fontes de dados

- [ ] Agendar ingestão de notícias RSS.
- [ ] Substituir Quant fake por client gRPC real.
- [ ] Adicionar health probe real para Quant Engine e LLM.

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
