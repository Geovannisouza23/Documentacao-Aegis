---
title: Execução Local
aliases:
  - Rodando Aegis
  - Quick Start
tags:
  - aegis
  - operacao/local
  - documentacao
projeto: "[[Aegis]]"
tipo: guia
status: ativo
created: 2026-07-23
updated: 2026-07-23
---

# Execução Local

O caminho recomendado para validar o projeto é Docker Compose em modo PAPER.

> [!info] Antes de rodar
> Confira [[05 - Configuração]] para variáveis e [[Modos Operacionais]] para diferenças entre PAPER, TESTNET e REAL.

## Docker

```bash
cd ../trading-core
cp .env.example .env
make docker-up
curl localhost:8080/health
curl localhost:8080/ready
```

O Compose sobe PostgreSQL, executa migrations e inicia API + worker. O modo padrão é PAPER.

## Local sem Docker

Requer PostgreSQL 16 acessível com variáveis compatíveis com `.env.example`.

```bash
cd ../trading-core
make migrate-up
make run
```

Worker em outro terminal:

```bash
cd ../trading-core
make worker
```

## Endpoints públicos de saúde

```bash
curl localhost:8080/health
curl localhost:8080/ready
curl localhost:8080/metrics
```

> [!tip] Modo de desenvolvimento
> Use PAPER até confiar no comportamento do pipeline, nos limites de risco e na estratégia conectada ao Quant Engine.

## Próximas validações

- [[08 - Testes e Qualidade]]
- [[06 - Segurança e Risco]]
- [[Fontes de Dados]]
