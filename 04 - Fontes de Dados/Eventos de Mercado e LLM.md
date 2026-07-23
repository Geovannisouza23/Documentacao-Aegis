---
title: Eventos de Mercado e LLM
aliases:
  - Notícias e LLM
  - Event Intelligence
tags:
  - aegis
  - dados/eventos
  - llm
  - rss
projeto: "[[Aegis]]"
tipo: fonte-dados
status: ativo
created: 2026-07-23
updated: 2026-07-23
---

# Eventos de Mercado e LLM

Eventos de mercado entram por providers de notícia e são classificados por `EventIntelligence`.

## RSS

| Item | Valor |
| --- | --- |
| Interface | `provider.Provider` |
| Caminho | `../trading-core/internal/infrastructure/external/news/provider` |
| Implementação | `../trading-core/internal/infrastructure/external/news/rss` |
| Saída | `NewsItem` com fonte, URL, título, conteúdo e publicação |
| Status | Provider pronto, scheduler pendente |

## LLM

| Provider | Status | Uso |
| --- | --- | --- |
| `noop` | Implementado | Desenvolvimento sem credenciais |
| `gemini` | Implementado | Classificação estruturada via schema |

O schema fica em `../trading-core/internal/contracts/schemas/llm/event_assessment.schema.json`.

## Garantias

- LLM retorna classificação, não ordem.
- `action` é enum determinístico: `NORMAL`, `REDUCE`, `BLOCK_NEW_ENTRIES`, `REVIEW_REQUIRED`, `KILL_SWITCH`.
- Conteúdo de notícia é tratado como dado não confiável.

## Lacunas

- Polling RSS ainda não é agendado no lifecycle.
- Health real de LLM ainda aparece como constante no dashboard.

## Relacionado

- [[Fontes de Dados]]
- [[06 - Segurança e Risco]]
- [[10 - Backlog e Lacunas]]
