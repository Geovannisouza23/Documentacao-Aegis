---
title: Testes e Qualidade
aliases:
  - Qualidade Aegis
  - Estratégia de Testes
tags:
  - aegis
  - testes
  - qualidade
projeto: "[[Aegis]]"
tipo: referencia
status: ativo
created: 2026-07-23
updated: 2026-07-23
---

# Testes e Qualidade

O projeto inclui suites para domínio, arquitetura, contratos, integração com PostgreSQL e cenários de falha.

## Comandos

```bash
cd ../trading-core
make test-unit
make test-architecture
make test-contract
make test-integration
make test-failure
make test
```

## Suites

| Suite | Prova | Docker |
| --- | --- | --- |
| `tests/unit` | Invariantes de value objects, state machines, risco e PaperBroker | Não |
| `tests/architecture` | Direção de dependência e regras de pasta | Não |
| `tests/contract` | Assinatura Binance e contrato do Quant fake | Não |
| `tests/integration` | PostgreSQL real, locks, transações e outbox | Sim |
| `tests/failure` | Banco indisponível, timeout de broker, JSON inválido e corrida de execução | Parcial |

> [!success] Última validação local
> `make test-unit` passou após publicação inicial do projeto.

## Qualidade arquitetural

- `domain` e `application` não importam frameworks.
- Apenas `internal/config` lê variáveis de ambiente.
- HTTP handlers e consumers dependem de portas, não de adapters concretos.
- Outbox usa claim atômico com `FOR UPDATE SKIP LOCKED`.

## Áreas cobertas

| Área | Evidência |
| --- | --- |
| HLD | Testes de arquitetura e dependência |
| DLS | Testes de contrato, migrations e repositories |
| Fontes de dados | Binance signing e Quant fake contract |
| Falhas operacionais | DB indisponível, timeout de broker, JSON LLM inválido e corrida de execução |

## Relacionado

- [[Contratos e Eventos]]
- [[Banco de Dados]]
- [[Fontes de Dados]]
