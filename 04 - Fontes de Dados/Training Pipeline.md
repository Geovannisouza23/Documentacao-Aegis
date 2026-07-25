---
title: Training Pipeline
aliases:
  - Pipeline de Treinamento
  - Training Pipeline Aegis
tags:
  - aegis
  - dados/ml
  - contratos
projeto: "[[Aegis]]"
tipo: fonte-dados
status: ativo
created: 2026-07-24
updated: 2026-07-24
---

# Training Pipeline

Training Pipeline é a origem dos modelos de ML do Aegis: um serviço Python separado (`../training-pipeline`) que lê datasets Parquet já exportados pelo `quant-engine`, treina/valida/avalia modelos, detecta drift e exporta modelos ONNX + metadata para o Model Registry em filesystem que o `quant-engine` já sabe **ler**. Ele nunca opera trading, nunca fala com corretora, nunca recalcula indicadores de produção (RSI/EMA/ATR continuam domínio exclusivo do Rust) e **nunca promove automaticamente um modelo para produção** — só produz artefatos em disco.

> [!info] Status
> As 13 etapas planejadas estão completas: 21 casos de uso, 10 comandos de CLI, exportação ONNX com prova de extração cross-language contra o algoritmo real do Rust, Model Registry writer validado por um espelho Python do algoritmo de seleção do `quant-engine`, detecção de drift, presets Hydra de hiperparâmetros, e um teste de integração ponta a ponta cobrindo fixture → validar → split → treinar → avaliar → exportar → registrar → promover → verificar seleção pelo Rust. 268 testes, ruff/black/mypy/import-linter limpos, `docker build` e `docker compose` verificados rodando de fato (não só buildando). Repositório próprio publicado em `https://github.com/Geovannisouza23/Training-pipeline`.

## Por que existe separado do `quant-engine`

Treinamento de ML precisa de um ecossistema (Pandas/Polars/DuckDB, scikit-learn, LightGBM, XGBoost, Optuna) que não faz sentido embutir no binário Rust de baixa latência que serve inferência em produção. A fronteira é a mesma família de decisão do ADR 0001 do `quant-engine` (que já separou trading-core de quant-engine): cada linguagem/runtime fica só com a responsabilidade que exige seu ecossistema.

## Arquitetura

Clean Architecture / Ports & Adapters / light DDD, mesmo estilo do `trading-core` (Go) e do `quant-engine` (Rust) — camadas enforced mecanicamente por `tests/architecture` (contrato `import-linter` + varredura de confinamento de env), não só por convenção. `domain` → `application` (21 casos de uso) → `interfaces` (CLI Typer) e `infrastructure` (adapters concretos), compostos em `src/app` (composition root). Ver `../training-pipeline/docs/architecture.md`.

Um único módulo (`src/domain/feature/feature_vector.py`) monta o vetor `[34]` float32 na ordem exata que o `quant-engine` espera — tanto o treino quanto a exportação ONNX importam esse mesmo módulo, para as duas pontas nunca divergirem silenciosamente na ordem das features.

## Contratos com o `quant-engine` (byte a byte, não só por convenção)

| Contrato | Onde | Risco se divergir |
| --- | --- | --- |
| Schema Parquet, 26 colunas, decimal-como-string | `../training-pipeline/docs/validation.md` | Leitura falha ou parseia lixo |
| 34 features, ordem fixa (`FeatureName::ALL`) | `../training-pipeline/docs/training.md` | Vetor ONNX com valores nas posições erradas |
| Tensor ONNX `[1, 34]` → saída única `success_probability` clampada `[0,1]` | `../training-pipeline/docs/onnx.md` | Modelo carrega mas retorna lixo silenciosamente — a superfície de maior risco do projeto |
| `metadata.json` (`model_name`, `version`, `state`, `feature_schema_version`, `artifact_filename`, `artifact_sha256`, `retired`) | `../training-pipeline/docs/model-registry.md` | `quant-engine` rejeita ou nunca carrega a versão |
| Versão lexicográfica (não semver), zero-padded | `../training-pipeline/docs/model-registry.md` | `find_active` do Rust escolhe a versão errada |

## Ciclo de vida do modelo — por que nunca promove sozinho

7 estados (`DRAFT → TRAINED → VALIDATING → SHADOW → APPROVED → PRODUCTION → RETIRED`), mas o loader do `quant-engine` só reconhece `APPROVED|CANARY|PRODUCTION` como carregável (case-insensitive). `CANARY` é a única exceção — o `quant-engine` reconhece, mas o spec de 7 estados deste projeto deliberadamente não implementa esse estado. Isso não é um detalhe cosmético: é o mecanismo real que garante "nunca promove para produção sozinho" — um modelo em qualquer um dos outros 5 estados é estruturalmente invisível para o `find_active` do Rust, independente de como o CLI for chamado. Ver [[Quant Engine]] para o lado que lê.

## CLI (Typer, `src/interfaces/cli/main.py`)

`train, evaluate, walk-forward, optimize, export, validate, compare, drift, promote, archive` — 10 comandos, cada um recebendo a `Application` já montada via `ctx.obj` do Typer (injetada por `src/app/main.py`), nunca construindo um adapter concreto diretamente.

## Stack

Python 3.13, Poetry, Pandas/Polars/PyArrow/DuckDB (dataset), scikit-learn/LightGBM/XGBoost (treino), ONNX/skl2onnx/onnxmltools (exportação), Optuna (busca de hiperparâmetros via walk-forward, nunca split aleatório), MLflow (adapter opcional — `noop` é o padrão, suíte não depende de `mlflow` instalado), Pydantic v2 + Hydra (config e presets de hiperparâmetro), Typer (CLI), Loguru/Structlog/Prometheus/OpenTelemetry (observabilidade), Pytest/Hypothesis (testes, incluindo propriedades como ordenação lexicográfica de versão e ausência de leakage temporal).

## Documentação própria do serviço

O repo `../training-pipeline` mantém sua própria documentação técnica (não duplicada aqui):

| Assunto | Nota |
| --- | --- |
| Camadas, os 21 casos de uso, guard rails | `../training-pipeline/docs/architecture.md` |
| Modelos treináveis, feature selection, HPO, Hydra | `../training-pipeline/docs/training.md` |
| As 3 validações (dataset/feature/label) e o gap de labels | `../training-pipeline/docs/validation.md` |
| PSI, teste KS, feature/prediction/label drift | `../training-pipeline/docs/drift.md` |
| Contrato ONNX, conversão, graph surgery, prova cross-language | `../training-pipeline/docs/onnx.md` |
| Layout do registry, versão lexicográfica, o gap do CANARY | `../training-pipeline/docs/model-registry.md` |
| Split temporal e walk-forward, por que nunca é split aleatório | `../training-pipeline/docs/walk-forward.md` |
| CI (ruff, black, mypy, import-linter, pytest, hypothesis, build, docker) | `../training-pipeline/.github/workflows/ci.yml` |

## Gaps conhecidos

> [!warning] Labels reais ainda não chegam do `quant-engine`
> Em toda exportação real de hoje, `label_version`/`profitable_trade`/`net_return`/`forward_return_n_candles` vêm `NULL` — o cálculo de label existe no lado Rust mas não está ligado a nenhum caso de uso que popule esses campos ainda. `ValidateLabelSchema` detecta e reporta isso explicitamente (`no_labeled_records`) em vez de falhar escondido dentro do treino. O pipeline é funcional ponta a ponta contra fixtures rotulados hoje e não precisa de nenhuma mudança de código quando esse gap do lado Rust fechar.

- `CANARY` fora de escopo (ver acima) — decisão deliberada, não esquecimento.
- Sem ADRs próprias ainda — diferente de `trading-core` e `quant-engine`, o `training-pipeline` não tem `docs/decisions/` preenchido (pasta existe, vazia). Ver [[09 - Decisões Arquiteturais]].
- `registry_root` do `training-pipeline` aponta por padrão para um caminho local próprio, desacoplado do volume que o `quant-engine` usa — apontar os dois para o mesmo caminho compartilhado é uma ação explícita de operador, não o padrão.
- Nenhuma integração cross-repo real ainda aconteceu: o `quant-engine` não tem fixture `.onnx` real na sua própria suíte (ver [[10 - Backlog e Lacunas]]); o `training-pipeline` agora consegue gerar um artefato real que poderia preencher esse gap, mas isso não foi feito.

## Relacionado

- [[Quant Engine]]
- [[10 - Backlog e Lacunas]]
- [[09 - Decisões Arquiteturais]]
- [[03 - Fluxo de Trading]]
