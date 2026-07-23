---
title: Configuração
aliases:
  - Variáveis de Ambiente
  - Config Aegis
tags:
  - aegis
  - configuracao
  - operacao
projeto: "[[Aegis]]"
tipo: referencia
status: ativo
created: 2026-07-23
updated: 2026-07-23
---

# Configuração

Toda configuração do `trading-core` vem de variáveis de ambiente e é carregada uma única vez em `internal/config`.

## Fonte de verdade

| Item | Caminho |
| --- | --- |
| Config loader | `../trading-core/internal/config` |
| Defaults públicos | `../trading-core/.env.example` |
| Compose PAPER | `../trading-core/docker-compose.yml` |
| Compose TESTNET | `../trading-core/deployments/compose/docker-compose.testnet.yml` |

## Grupos principais

| Grupo | Exemplos | Uso |
| --- | --- | --- |
| Database | `DATABASE_HOST`, `DATABASE_PASSWORD` | PostgreSQL |
| Broker | `BROKER_MODE`, `BROKER_BINANCE_API_KEY` | PAPER, TESTNET ou REAL |
| Risk | `RISK_MAX_OPEN_POSITIONS`, limites de drawdown | Gestão de risco |
| Security | `SECURITY_JWT_SIGNING_SECRET` | Autenticação JWT |
| Quant | `QUANT_MODE`, `QUANT_GRPC_TARGET` | Engine quantitativo |
| LLM | `LLM_PROVIDER`, `LLM_GEMINI_API_KEY` | Inteligência de eventos |
| Observability | tracing, metrics, logs | Operação |

> [!warning] Segredos
> `.env` é local e não deve entrar em commit. Use `.env.example` como referência pública.

## Modos de broker

| Modo | Finalidade |
| --- | --- |
| PAPER | Simulação local sem capital real |
| TESTNET | Integração com exchange testnet |
| REAL | Execução com capital real, protegida por múltiplas confirmações |

## Relação com outras notas

- [[04 - Execução Local]]
- [[06 - Segurança e Risco]]
- [[Fontes de Dados]]
- [[Modos Operacionais]]
