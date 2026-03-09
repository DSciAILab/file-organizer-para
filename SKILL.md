---
name: file-organizer-para
description: >-
  Organiza arquivos e pastas no filesystem local usando a metodologia PARA
  de Tiago Forte, com verificacao de dependencias, plano antes da execucao,
  rollback e manutencao continua. Triggers PT: "organizar meus arquivos",
  "limpar meu desktop", "arquivar projetos antigos", "montar estrutura
  PARA", "renomear meus arquivos", "organizar minha pasta de downloads".
  Triggers EN: "organize my files", "clean up my desktop", "archive old
  projects", "set up PARA folders", "rename my files", "sort my downloads".
user-invocable: true
metadata:
  author: Fernando Caravana
  version: 9.0.0
  tags:
    - file-organization
    - para-method
    - productivity
    - filesystem
---

# PARA File Organizer

Organiza arquivos locais com a metodologia PARA de Tiago Forte.
Nunca quebrar referencias, nunca mover sem verificacao, nunca executar
sem aprovacao explicita.

Para definicoes completas do PARA, exemplos de classificacao e
referencias metodologicas, ver [references/PARA-METHOD-REFERENCE.md].

## 1. Principios operacionais

1. Preservar dados vem antes de organizar.
2. Nunca mover, renomear ou arquivar sem plano e aprovacao.
3. Nunca deletar arquivos do usuario.
4. Em moves cross-filesystem, origem so e removida apos copy + verify.
5. Aplicar PARA: Projetos, Areas, Recursos, Arquivo.
6. Check impossivel = NOT_CHECKED. Nunca reportar OK sem verificar.
7. O usuario decide. A skill recomenda.
8. Usar o idioma do usuario.
9. Toda execucao gera plano, manifesto, relatorio e rollback.
10. Resolver inconsistencias entre disco, lock e manifesto antes de
    qualquer escrita.

## 2. Precedencia de regras

1. Evitar perda de dados e quebra de dependencias
2. Respeitar decisao explicita do usuario
3. Preservar atomicidade de projetos de software
4. Resolver riscos de dependencias e links
5. Manter coerencia PARA
6. Aplicar convencoes de nomes
7. Limpeza e manutencao

## 3. Escopo

- Apenas filesystem local.
- Apenas capacidades disponiveis no runtime.
- Ferramenta ausente = NOT_CHECKED.
- Persistir estado em: ponteiro global, config da raiz, lock, manifesto.
- Produzir: saida legivel, manifesto estruturado, rollback.
- Artefatos internos (.para-config.json, .para-lock.json, .para-temp/,
  .para-manifest-history/, para-manifest-*.json, para-rollback-*) nunca
  entram em classificacao ou plano PARA.
- Todos os artefatos de texto usam UTF-8 sem BOM.

## 4. Capacidades

Classificar como SUPPORTED, BEST_EFFORT ou NOT_CHECKED.

macOS/Linux: inventario, pastas, symlinks, hard links, configs com path
absoluto = SUPPORTED. PATH, cron, launchd, systemd, Finder aliases,
filesystem de rede, cloud placeholders, metadata avancada = BEST_EFFORT.

Windows: inventario, pastas = SUPPORTED. Symlinks, hard links, .lnk,
schtasks, registry PATH, cloud placeholders, ADS, ACLs = BEST_EFFORT.

WSL: filesystem Linux e /mnt = SUPPORTED. Registry, .lnk, schtasks =
NOT_CHECKED salvo bridge disponivel.

Regra dura: nunca afirmar que Finder, registry, Dock, sidebar, schtasks
ou atalhos foram checados se a ferramenta real nao confirmou.

## 5. Estado e persistencia

Ponteiro global:
- macOS/Linux: ~/.config/openclaw/para-root.json
- Windows: %APPDATA%/OpenClaw/para-root.json

Config da raiz: <root>/.para-config.json
- mode: "single" (trees=[]) ou "separate" (trees com >= 2 nomes unicos)
- categoryFolderFormat: "number-hyphen" ou "number-dot-space"
- maxDepthBelowCategoryRoot: 3
- Alertar se trees > 5 (estrutura dispersa)

Para schema completo do config, lock e manifesto, ver
[references/MANIFEST-SCHEMA.md].

## 6. Root directory setup (Passo 0)

Primeira execucao:
1. Detectar SO e diretorios comuns (home, Desktop, Documents, Downloads).
2. Sugerir raiz PARA por SO.
3. Perguntar: arvore unica ou separada? Quais arvores?
4. Perguntar formato: number-hyphen ou number-dot-space.
5. Verificar existencia e permissao de escrita.
6. Criar root, .para-manifest-history/, .para-temp/.
7. Salvar ponteiro global e config.

Execucoes futuras:
1. Ler ponteiro e config.
2. Checar lock (Secao 8).
3. Checar manifestos incompletos, ver [references/ROLLBACK-AND-RECOVERY.md].
4. Mostrar config atual. Perguntar: "Continuar? (yes / change)"

Formato misto detectado (ex: 1-Projetos junto com 2. Areas):
1. Listar pastas encontradas e formato de cada uma.
2. Perguntar: (a) padronizar para formato configurado, (b) adotar o mais
   comum, (c) manter.
3. Se padronizar, gerar plano de rename com confirmacao.

## 7. Negociacao do source (Passo 1)

Distinguir root PARA (destino) de source directory (origem).
- Caminho explicito: validar existencia e acessibilidade.
- "desktop"/"downloads"/"documents": detectar automaticamente.
- Generico: perguntar qual diretorio.
- Nunca assumir source sem confirmacao.
- Validar leitura no source. Sem leitura = abortar.
- Validar escrita no source quando necessario. Sem escrita = oferecer
  copy_only.

Protecao contra sobreposicao:
- Root dentro de source: excluir root do scan.
- Source = root: tratar como manutencao, nao importacao.
- Source dentro de root: tratar como reorganizacao interna.
- Impedir recursion e auto-encaixe.

## 8. Lock de execucao

Criar <root>/.para-lock.json antes de qualquer escrita.
Atualizar heartbeat: a cada operacao ou a cada 60s, o que vier primeiro.
Stale: updatedAt > heartbeatTimeoutMinutes. Duracao total longa = alerta,
nao stale.
Lock nao criavel (permissao, disco cheio) = abortar, nunca executar sem lock.
Reconciliacao: exigir coerencia de executionId, manifestPath, root e
source entre lock e manifesto.
Remover lock apenas em: sucesso, rollback concluido, abort seguro.

Para schema completo do lock, ver [references/MANIFEST-SCHEMA.md].

## 9. Workflow obrigatorio

Passo 0. Setup - ler config, checar lock, checar manifestos incompletos.
Passo 1. Source - negociar origem, validar permissoes, checar sobreposicao.
Passo 2. Discover - escanear, contar, excluir root se dentro do source,
  excluir artefatos internos, nunca seguir symlinks recursivamente.
  >10.000 itens: pausar e perguntar. >10 min: pausar.
Passo 3. Dependency check - rodar conforme modo (Quick/Safe/Deep),
  gerar dependency report, resolver flagged. Ver [references/DEPENDENCY-CHECKS.md].
Passo 4. Plan - propor destino, indicar rename, confianca, dependencia.
  Agrupar decisoes repetidas. Ver Secao 11 (Planejamento).
Passo 5. Confirm - mostrar resumo do plano. Nunca executar sem "yes".
  Ver Secao 12 (Confirmacao).
Passo 6. Execute - criar lock, criar manifesto, criar estrutura PARA,
  mover/copiar com verificacao, renomear, atualizar configs aprovados,
  registrar cada operacao no manifesto, atualizar heartbeat.
  Ver [references/EXECUTION-STRATEGY.md].
Passo 7. Verify - confirmar existencia no destino, ausencia na origem
  quando aplicavel, validar config edits, preencher completedAt.
Passo 8. Report - relatorio completo, checklist manual, rollback,
  remover lock, perguntar se quer ajustes.

## 10. Classificacao PARA

Arvore de decisao:
1. Resultado especifico com prazo? -> 1-Projetos
2. Responsabilidade continua? -> 2-Areas
3. Referencia ou aprendizado? -> 3-Recursos
4. Inativo ou encerrado? -> 4-Arquivo

Se baixa confianca para Trabalho vs Pessoal, perguntar.
Se ambiguo, propor lote ou triagem para Legado-pre-organizacao.

Para definicoes detalhadas, exemplos concretos e referencia a Tiago
Forte, ver [references/PARA-METHOD-REFERENCE.md].

## 11. Planejamento

Tabela obrigatoria:

| # | Origem | Destino | Acao | Escopo | Confianca | Dependencia |
|---|--------|---------|------|--------|-----------|-------------|

Acoes: Move, Move+Rename, Copy+Verify+Remove, Copy_only, Skip, Ask user
Confianca: Alta, Media, Baixa
Dependencia: No, Yes:<tipo>, NOT_CHECKED:<tipo>

Resumo: pastas a criar, itens a mover, renomear, copy+verify, copy_only,
dependencias resolvidas, pulados, ambiguos, NOT_CHECKED.

## 12. Confirmacao

Mostrar antes de executar:

PLANO DE ORGANIZACAO PARA
- Source, Root, Scan mode
- Pastas a criar: N
- Itens a mover: N, a renomear: N, copy+verify: N, copy_only: N
- Pulados: N, ambiguos: N, NOT_CHECKED: N
- Cross-filesystem: N, network: N, dependencias: N
- Espaco necessario vs disponivel
- Batch write: sim/nao

"Proceed? (yes / no / show details / edit)"

## 13. Convencoes de nomes

Pastas: AAAA-MM_Nome (projetos), Nome-descritivo (areas/recursos),
NN_Nome (subpastas).
Arquivos: AAAA-MM-DD_Descricao.ext, versao -v1/-v2, nunca FINAL.
Hifens entre palavras, ASCII safe (sem acentos), preservar extensao.
Colisao: -duplicata-N. Windows: validar nomes reservados (CON, PRN, etc).
Unicode nao-latino: FLAGGED, nunca renomear automaticamente.
Legado-pre-organizacao: sem rename automatico.

Para convencoes completas, ver [references/PARA-METHOD-REFERENCE.md].

## 14. Modos de scan

Quick: inventario, tamanhos, grandes, duplicatas, symlinks.
Safe (padrao): + projetos software, hard links, configs, bloqueios,
  cross-fs, espaco, permissoes, rede, cloud, metadata.
Deep: + PATH, shell rc, atalhos, scheduled tasks.
Se origem parecer dev/automacao, recomendar Deep.

## 15. Ignore

Standalone: .DS_Store, thumbs.db, desktop.ini, .Spotlight-V100, etc.
Internos: .para-config.json, .para-lock.json, .para-manifest-history/,
  .para-temp/, para-manifest-*.json, para-rollback-*.
Ocultos: nao mover isoladamente, inspecionar em dependency check.
Projetos software: .git, node_modules, .venv, etc = mover so com pai.
.env: nunca ignore para dependencia.

## 16. Dependency check

Obrigatorio antes do plano. Status: OK, FLAGGED, NOT_CHECKED, ERROR.

Checks principais:
- Symlinks: FLAGGED, nunca seguir recursivamente
- Hard links: FLAGGED se cross-filesystem
- Projetos software: mover atomicamente, nunca extrair internos
- Configs com path absoluto: FLAGGED, mascarar segredos
- Cloud boundary e placeholders: FLAGGED
- Arquivos >1GB, read-only, em uso: FLAGGED
- Cross-filesystem: obriga copy+verify+remove
- Espaco: margem = max(percent, minBytes)
- Case-collision, path length, nomes reservados: FLAGGED
- Permissoes: FLAGGED se insuficientes
- Filesystem de rede: FLAGGED, nao confiar em rename atomico
- Hash obrigatorio quando suportado para: >=1GB, rede, databases,
  executaveis, compactados, configs sensiveis, falhas previas

Para lista completa de checks, heuristicas e opcoes por tipo de flag,
ver [references/DEPENDENCY-CHECKS.md].

## 17. Execucao

Same-filesystem local: rename nativo (exceto: rede, cloud boundary,
  hard links preservando count, pedido explicito de copy+verify).
Cross-filesystem: copy -> verify (tamanho + hash quando exigido) ->
  remove origem. Falha na verificacao = manter origem.
Colisao: -duplicata-N, nunca sobrescrever.
Config edits: backup .bak-AAAA-MM-DD-HHMM antes, validar sintaxe depois.
Projetos software: pasta inteira como unidade.
Escrita atomica: temp file -> flush/fsync -> rename.
Batch write: apos batchWriteThreshold, gravar em lotes de batchWriteSize.
Rede: nunca confiar em rename, sempre copy+verify+remove.
Metadata: preserved (rename local), best_effort (copy), not_checked.

Para estrategia completa, precaucoes e edge cases, ver
[references/EXECUTION-STRATEGY.md].

## 18. Manifesto

Toda execucao gera manifesto JSON (schemaVersion 6).
Operacoes: type (move/copy/copy_only), status (planned -> in_progress ->
  copied -> verified -> source_removed -> completed / failed / rolled_back).
Rename e atributo (renamed=true), nao tipo.
Move local via rename: planned -> completed (intencional, rename e atomico).
completedAt so quando todas as operacoes terminarem.

Para schema JSON completo e regras de estado, ver
[references/MANIFEST-SCHEMA.md].

## 19. Falha durante execucao

Individual: marcar failed, gravar manifesto, perguntar retry/skip/pause/abort.
Sistemica (3+ consecutivas): pausar automaticamente, perguntar.
Critica (filesystem inacessivel): gravar best-effort, manter lock.
Config edit: oferecer restauracao do backup, adicionar ao checklist.

## 20. Rollback

Ordem inversa das operacoes concluidas.
move: mover de volta, restaurar nome se renamed.
copy: se source existe, remover destination. Se nao, mover de volta.
copy_only: remover destination (source intacto).
configEdits: restaurar backup.
Colisao no rollback: nunca sobrescrever, usar -rollback-collision-N.
Pastas criadas: remover apenas se vazias.
Rollback parcial: registrar, continuar, marcar partial.
Script .sh/.ps1 gerado apos cada execucao.
Rollback interativo disponivel via skill.

Para procedimentos completos de recovery e rollback, ver
[references/ROLLBACK-AND-RECOVERY.md].

## 21. Relatorio

PARA FILE ORGANIZER - REPORT
Data, Execution ID, Source, Root, Mode, Scan Mode, Format.
STATUS HONESTO: concluidas, puladas, falhadas, NOT_CHECKED.
Metadata: preserved, best_effort, not_checked, failed.
Batch write: status e indices.
Secoes: estrutura criada, arquivos movidos/renomeados, config edits,
dependencias resolvidas, duplicatas, ambiguos, pulados, NOT_CHECKED,
erros, estatisticas, rollback info, checklist manual.

Checklist manual pos-move: salvo como
<root>/para-manual-checklist-AAAA-MM-DD-HHMM.md.

## 22. Comandos on-demand

- "novo projeto" / "new project": criar pasta com subpastas recomendadas.
- "nova area" / "new area": criar pasta de area.
- "arquivar projeto" / "archive project": mover para Arquivo.
- "manutencao" / "maintenance": rodar revisao periodica.
- "status PARA" / "PARA status": mostrar config e contagens.

Para detalhes de cada comando e rotinas de manutencao, ver
[references/MAINTENANCE-AND-COMMANDS.md].

## 23. Manutencao

Semanal (5 min): projetos inativos >30 dias -> sugerir arquivar.
Mensal (15 min): areas/recursos sem modificacao >90 dias -> sugerir.
Verificar Legado-pre-organizacao. Re-rodar dependency check se relevante.

## 24. Edge cases

Pastas vazias, arquivos sem extensao, circular symlinks, encoding
ambiguo, >10GB, >10.000 itens numa pasta, arquivos em uso, manifesto
corrompido, permissoes revogadas, espaco esgotado, cloud sync conflitos,
schema de manifesto desconhecido ou antigo.

Para tratamento detalhado de cada caso, ver
[references/ROLLBACK-AND-RECOVERY.md] e [references/EXECUTION-STRATEGY.md].