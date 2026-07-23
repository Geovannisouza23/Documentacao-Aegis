---
title: Modos Operacionais
aliases:
  - Operational Modes
  - Modo PAPER TESTNET REAL
tags:
  - aegis
  - operacao/modos
  - risco
projeto: "[[Aegis]]"
tipo: referencia
status: ativo
created: 2026-07-23
updated: 2026-07-23
---

# Modos Operacionais

Modos operacionais controlam onde o sistema pode executar e quais ações são permitidas.

```mermaid
stateDiagram-v2
    PAPER --> TESTNET
    TESTNET --> REAL
    REAL --> PAUSED
    REAL --> CLOSE_ONLY
    PAUSED --> REAL
    CLOSE_ONLY --> KILL_SWITCH
    REAL --> KILL_SWITCH
```

## Modos

| Modo | Finalidade |
| --- | --- |
| `PAPER` | Simulação local sem capital real |
| `TESTNET` | Integração com Binance Futures testnet |
| `REAL` | Execução com capital real e múltiplas travas |
| `PAUSED` | Pausa ações operacionais |
| `CLOSE_ONLY` | Permite apenas redução/fechamento |
| `KILL_SWITCH` | Estado terminal de bloqueio operacional |

## Gatilhos de segurança

- `REAL` não tem endpoint HTTP direto.
- `BROKER_REAL_TRADING_ENABLED=true` é obrigatório para REAL.
- `BROKER_REAL_TRADING_CONFIRMATION=I_UNDERSTAND_THE_RISK` é obrigatório para REAL.
- Adapter real recusa inicialização sem confirmação.
- Reconciliação roda em todos os modos.

## Relacionado

- [[06 - Segurança e Risco]]
- [[05 - Configuração]]
- [[07 - API e Dashboard]]
