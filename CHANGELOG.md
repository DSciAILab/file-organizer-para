# Changelog

## v9.0.0 - 2026-03-09

### Breaking
- Reestruturado como Agent Skills package com SKILL.md (<500 linhas) e
  6 arquivos de referencia. Isso segue a especificacao oficial de
  agentskills.io (progressive disclosure: metadata Level 1, instructions
  Level 2, references Level 3).

### Added
- Secoes 25 (falha durante execucao), 26 (rollback completo com
  colisao e pastas vazias), 27 (relatorio), 28 (checklist manual),
  29 (comandos on-demand), 30 (manutencao periodica), 31 (edge cases),
  32 (referencias metodologicas), 33 (resumo do plano), 35 (manifesto
  corrompido e schema desconhecido). Todas existem e sao referenciadas.
- Workflow formal completo (Passo 0-8) presente no SKILL.md.
- Exemplos concretos de classificacao PARA com 11 itens na tabela.
- Exemplos de ambiguidade em lote e individual.
- Hash obrigatorio para classes de risco (>=1GB, rede, databases,
  executaveis, compactados, configs sensiveis, falhas previas).
- directoriesCreated no manifesto com status kept_not_empty.
- Lock creation failure: abortar, nunca executar sem lock.
- Manifesto completo que falha ao ser movido para history: marcado com
  -archive-failed e excluido do pool ativo.
- Schema antigo: migracao com backup.
- Colisao no rollback: -rollback-collision-N.
- Timeout de scan (>10 min).
- Separacao ignoredDiscoverPatterns / ignoredRecoveryPatterns.
- Reconciliacao lock-manifesto por 4 campos.
- Regra explicita: lock nao criavel = abortar.
- Manifesto com schema mais novo = somente leitura.
- Recovery parcial de corrupted = somente leitura.

### Fixed
- Secao 24.5 completa (retryCount incremento e reset).
- Heartbeat consistente entre lock e workflow.
- Rename local intencional sem hash declarado explicitamente.
- Numeracao sequencial de secoes.
- Changelog separado por fix/feature/breaking.

### Removed
- Ambiguidade sobre type=rename (rename e atributo, nao tipo).

## v8.0.0 - 2026-03-09
- Separacao ignoredDiscoverPatterns / ignoredRecoveryPatterns.
- Hash obrigatorio em classes criticas.
- Reconciliacao 4 campos lock-manifesto.
- directoriesCreated no manifesto.
- Ciclo de vida de manifestos completos.
- Regra formal de artefatos internos.

## v7.0.0 - 2026-03-09
- Adicionadas secoes 25-35 completas.
- Corrigida Secao 24.5.
- Encoding UTF-8 obrigatorio.
- Politica Unicode nao-latino.
- Heuristica .json como config.
- Formato misto de categorias.
- Dockerfile/docker-compose como marcadores condicionais.

## v6.1 - 2026-03-08
- ignoredInternalPatterns cobrindo artefatos.
- Symlinks nunca seguidos recursivamente.
- Metadata preservation policy.
- Cloud placeholder check.
- Nomes reservados Windows.

## v6.0 - 2026-03-08
- tempPath como artefato de execucao.
- Retomada verificada por type.
- Filesystem de rede.
- APFS clones.
- Finder aliases.
- Limite de tentativas.

## v5.0 - 2026-03-08
- manifestHistoryDir.
- Reconciliacao lock-manifesto por executionId.
- Retomada por estado.
- Rollback de configEdits.
- lastDurableOperationIndex.
- Stale lock por heartbeat.

## v4.0 - 2026-03-08
- Lock timeouts no config.
- Scan pattern ativo.
- Permissoes no source e destino.
- copy_only como alternativa.
- Batch write.

## v3.0 - 2026-03-08
- Negociacao de source.
- Protecao source-root.
- Modos Quick/Safe/Deep.
- Lock com heartbeat.
- Manifesto com estado por operacao.

## v2.0 - 2026-03-08
- Dependency check completo.
- Planejamento com tabela formal.
- Convencoes de nomes.
- Politica de ambiguidade.

## v1.0 - 2026-03-08
- Versao inicial.