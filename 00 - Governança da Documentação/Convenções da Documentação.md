---
title: Convenções da Documentação
aliases:
  - Organização da Documentação
  - Guia de Documentação Aegis
tags:
  - aegis
  - documentacao/governanca
  - documentacao/convencoes
projeto: "[[Aegis]]"
tipo: governanca
status: ativo
created: 2026-07-23
updated: 2026-07-23
---

# Convenções da Documentação

Esta documentação combina Diátaxis com uma divisão por domínio do `trading-core`. A intenção é separar o que o leitor quer fazer ou entender do detalhe técnico que ele precisa consultar.

## Referências consultadas

- [Diátaxis](https://diataxis.fr/) para separar tutoriais, how-to, referência e explicação.
- [Read the Docs: How to structure your documentation](https://docs.readthedocs.com/platform/stable/explanation/documentation-structure.html) para escolher estrutura antes de preencher conteúdo.
- [Write the Docs: Software documentation guide](https://www.writethedocs.org/guide/) para taxonomia, práticas de escrita e documentação como produto.
- [Google Developer Documentation Style Guide](https://developers.google.com/style/) para clareza, consistência e prioridade ao estilo específico do projeto.
- [Obsidian Help](https://help.obsidian.md/) para wikilinks, propriedades YAML e navegação no vault.

## Taxonomia do vault

| Pasta | Uso |
| --- | --- |
| `00 - Governança da Documentação` | Regras, inventário e manutenção da documentação |
| `01 - Produto` | Visão geral, escopo e vocabulário do Aegis |
| `02 - HLD - Arquitetura` | High-Level Design: arquitetura, fluxo principal, risco e fronteiras |
| `03 - DLS - Design Detalhado` | Detailed-Level Specification: configuração, API, banco, contratos e eventos |
| `04 - Fontes de Dados` | Origens de dados, contratos de entrada, freshness, credenciais e limitações |
| `05 - Operação` | Execução local, modos operacionais e rotinas de operador |
| `06 - Qualidade` | Testes, gates, critérios de aceite e verificação |
| `07 - Decisões` | ADRs e decisões que mudam fronteiras técnicas |
| `08 - Planejamento` | Backlog, lacunas e próximos incrementos |

## Tipos de nota

| Tipo | Quando usar |
| --- | --- |
| `indice` | Entrada navegável de projeto, pasta ou trilha |
| `guia` | Passo a passo para executar tarefa |
| `referencia` | Fatos técnicos consultáveis |
| `hld` | Visão sistêmica, fronteiras, fluxo e decisões macro |
| `dls` | Detalhe implementável de contrato, schema, API, config ou banco |
| `fonte-dados` | Origem externa/interna de dados e suas garantias |
| `governanca` | Regra de manutenção da documentação |
| `backlog` | Lacunas e próximos trabalhos |

## Regras de manutenção

- Uma nota deve responder a uma necessidade clara do leitor.
- HLD explica sistema e decisões macro; DLS descreve contrato implementável.
- Fontes de Dados ficam fora de HLD/DLS quando o foco é origem, freshness, credenciais ou limitação da fonte.
- Use wikilinks para notas internas e links Markdown para URLs externas.
- Mantenha nomes de arquivo únicos no vault para evitar ambiguidade no Obsidian.
- Mudança que altera fronteira, persistência, risco, integração externa ou operação deve ganhar ADR em [[09 - Decisões Arquiteturais]].
- Notas críticas de segurança e risco devem linkar [[06 - Segurança e Risco]] e [[Modos Operacionais]].

## Checklist para nova nota

- [ ] Definir pasta certa pela taxonomia.
- [ ] Preencher YAML com `title`, `tags`, `projeto`, `tipo`, `status`, `created` e `updated`.
- [ ] Linkar [[Aegis]] e notas relacionadas.
- [ ] Citar fonte primária do código quando a nota descreve comportamento implementado.
- [ ] Evitar misturar tutorial, referência e explicação na mesma nota longa.
