---
name: file-organizer-para
description: >-
  Organiza arquivos e pastas no filesystem local usando a metodologia PARA
  de Tiago Forte, com verificacao de dependencias, deteccao de duplicados,
  plano antes da execucao, rollback e manutencao continua.
  Triggers PT: "organizar meus arquivos", "limpar meu desktop",
  "arquivar projetos antigos", "montar estrutura PARA",
  "renomear meus arquivos", "organizar minha pasta de downloads",
  "encontrar duplicados", "buscar duplicados", "tem arquivos repetidos?".
  Triggers EN: "organize my files", "clean up my desktop", "archive old
  projects", "set up PARA folders", "rename my files", "sort my downloads",
  "find duplicates", "check for duplicates", "any repeated files?".
user-invocable: true
metadata:
  author: Fernando Caravana
  version: 10.0.0
  tags:
    - file-organization
    - para-method
    - productivity
    - filesystem
    - deduplication
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

O usuario pode:
(a) especificar o root explicitamente ("root em ~/Documents/PARA"),
(b) especificar apenas a pasta-fonte, sem indicar root,
(c) delegar a decisao ("escolhe o melhor lugar" / "you decide").

Para (a): validar existencia, permissao de escrita e espaco. Usar
o caminho exato fornecido.

Para (b): perguntar onde o root deve ficar. Sugerir valor padrao
por SO (ver abaixo). Aceitar resposta ou alternativa.

Para (c): aplicar heuristica:
1. Se ponteiro global existe e tem activeRoot valido, propor esse.
2. Caso contrario, propor por SO:
   - macOS: ~/Documents/PARA
   - Linux: ~/Documents/PARA ou ~/PARA se Documents nao existe
   - Windows: %USERPROFILE%\Documents\PARA
3. Mostrar proposta e AGUARDAR confirmacao explicita.
   Nunca criar root sem "yes" ou equivalente.

Validacao obrigatoria (todos os cenarios):
- Caminho existe ou pode ser criado.
- Permissao de escrita confirmada.
- Espaco livre >= diskSafetyMarginPercent.
- Caminho nao esta dentro de diretorio de sistema (/System, /Library,
  /usr, /bin, /sbin, C:\Windows, C:\Program Files).
- Caminho nao esta dentro de .git, node_modules ou similar.

Primeira execucao:
1. Detectar SO e diretorios comuns (home, Desktop, Documents, Downloads).
2. Resolver root conforme cenario (a), (b) ou (c).
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
Passo 2.5. Deduplication (opcional) - detectar duplicados conforme modo
  de scan ou pedido explicito. Apresentar relatorio, aguardar decisao.
  Ver [references/DUPLICATE-DETECTION.md].
Passo 3. Dependency check - rodar conforme modo (Quick/Safe/Deep),
  gerar dependency report, resolver flagged.
  Ver [references/DEPENDENCY-CHECKS.md].
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

Acoes: Move, Move+Rename, Copy+Verify+Remove, Copy_only, Skip,
  Ask user, Archive-duplicate
Confianca: Alta, Media, Baixa
Dependencia: No, Yes:<tipo>, NOT_CHECKED:<tipo>

Resumo: pastas a criar, itens a mover, renomear, copy+verify, copy_only,
dependencias resolvidas, pulados, ambiguos, NOT_CHECKED, duplicados
encontrados e acao escolhida.

## 12. Confirmacao

Mostrar antes de executar:

PLANO DE ORGANIZACAO PARA
- Source, Root, Scan mode
- Pastas a criar: N
- Itens a mover: N, a renomear: N, copy+verify: N, copy_only: N
- Pulados: N, ambiguos: N, NOT_CHECKED: N
- Cross-filesystem: N, network: N, dependencias: N
- Duplicados: N grupos, N arquivos, N bytes recuperaveis
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

Quick: inventario, tamanhos, grandes, symlinks.
  Duplicados: desligado. Agente oferece ativar.
Safe (padrao): + projetos software, hard links, configs, bloqueios,
  cross-fs, espaco, permissoes, rede, cloud, metadata.
  Duplicados: fase 1 (agrupamento por tamanho). Se candidatos,
  perguntar se quer hash para confirmar.
Deep: + PATH, shell rc, atalhos, scheduled tasks.
  Duplicados: fase 1 + fase 2 (hash) automatico.
Se origem parecer dev/automacao, recomendar Deep.

Para algoritmo completo de deteccao de duplicados, ver
[references/DUPLICATE-DETECTION.md].

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

## 17. Deteccao de duplicados

Duplicado exato: dois ou mais arquivos com hash SHA-256 identico,
independente de nome, localizacao ou data.

Duplicado provavel: mesmo tamanho + mesma extensao + nome similar
(Levenshtein <= 3), hash diferente. Somente reportado, sem acao
automatica.

Algoritmo em duas fases:
- Fase 1: agrupar por tamanho. Descartar grupos com um membro.
- Fase 2: hash dos candidatos. Arquivos >100 MB usam hash parcial
  (primeiros + ultimos 64 KB) como pre-filtro.

Integracao com scan modes: ver Secao 14.

Acoes disponiveis por grupo (sempre com aprovacao):
- archive-duplicates: mover para 4-Arquivo/Duplicados/AAAA-MM-DD/
- keep-all: nenhuma acao, todos seguem para planejamento PARA
- swap-keep: usuario escolhe qual manter
- skip-group: ignorar grupo

Heuristica para sugerir [MANTER]:
1. Arquivo dentro de estrutura PARA > arquivo fora
2. Data de modificacao mais recente > mais antiga
3. Caminho mais curto > mais longo
4. Nome original (sem "(1)", "-copia") > nome modificado
Usuario pode alterar qualquer sugestao.

Comando on-demand: ver Secao 22.

Cross-location: quando root PARA existe e usuario roda dedup avulso,
comparar hashes do source contra historico de manifestos para detectar
arquivos ja organizados.

Regras de seguranca: nunca deletar, nunca agir em provaveis sem
aprovacao, sempre mostrar relatorio antes de qualquer acao, manifestar
cada operacao para rollback.

Para algoritmo completo, formato de relatorio, schema de manifesto,
performance e edge cases, ver [references/DUPLICATE-DETECTION.md].

## 18. Execucao

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

## 19. Manifesto

Toda execucao gera manifesto JSON (schemaVersion 7).
Operacoes: type (move/copy/copy_only/deduplicate), status (planned ->
  in_progress -> copied -> verified -> source_removed -> completed /
  failed / rolled_back).
Rename e atributo (renamed=true), nao tipo.
Move local via rename: planned -> completed (intencional, rename e atomico).
completedAt so quando todas as operacoes terminarem.
Deduplicate: registra kept (path + hash), archived (lista com source,
  destination, hash, bytes, verification), groupHash e timestamp.

Para schema JSON completo e regras de estado, ver
[references/MANIFEST-SCHEMA.md].

## 20. Falha durante execucao

Individual: marcar failed, gravar manifesto, perguntar retry/skip/pause/abort.
Sistemica (3+ consecutivas): pausar automaticamente, perguntar.
Critica (filesystem inacessivel): gravar best-effort, manter lock.
Config edit: oferecer restauracao do backup, adicionar ao checklist.

## 21. Rollback

Ordem inversa das operacoes concluidas.
move: mover de volta, restaurar nome se renamed.
copy: se source existe, remover destination. Se nao, mover de volta.
copy_only: remover destination (source intacto).
deduplicate: mover arquivos arquivados de volta para localizacao original.
configEdits: restaurar backup.
Colisao no rollback: nunca sobrescrever, usar -rollback-collision-N.
Pastas criadas: remover apenas se vazias.
Rollback parcial: registrar, continuar, marcar partial.
Script .sh/.ps1 gerado apos cada execucao.
Rollback interativo disponivel via skill.

Para procedimentos completos de recovery e rollback, ver
[references/ROLLBACK-AND-RECOVERY.md].

## 22. Comandos on-demand

- "novo projeto" / "new project": criar pasta com subpastas recomendadas.
- "nova area" / "new area": criar pasta de area.
- "arquivar projeto" / "archive project": mover para Arquivo.
- "manutencao" / "maintenance": rodar revisao periodica.
- "status PARA" / "PARA status": mostrar config e contagens.
- "encontrar duplicados" / "find duplicates": rodar deteccao de duplicados
  independente do workflow PARA. Funciona mesmo sem root configurado.
  Se root existe, compara source contra PARA para detectar arquivos ja
  organizados. Ver [references/DUPLICATE-DETECTION.md].

Para detalhes de cada comando e rotinas de manutencao, ver
[references/MAINTENANCE-AND-COMMANDS.md].

## 23. Relatorio

PARA FILE ORGANIZER - REPORT
Data, Execution ID, Source, Root, Mode, Scan Mode, Format.
STATUS HONESTO: concluidas, puladas, falhadas, NOT_CHECKED.
Metadata: preserved, best_effort, not_checked, failed.
Batch write: status e indices.
Secoes: estrutura criada, arquivos movidos/renomeados, config edits,
dependencias resolvidas, duplicatas encontradas e acao tomada,
ambiguos, pulados, NOT_CHECKED, erros, estatisticas, rollback info,
checklist manual.

Checklist manual pos-move: salvo como
<root>/para-manual-checklist-AAAA-MM-DD-HHMM.md.

## 24. Manutencao

Semanal (5 min): projetos inativos >30 dias -> sugerir arquivar.
Mensal (15 min): areas/recursos sem modificacao >90 dias -> sugerir.
Verificar Legado-pre-organizacao. Re-rodar dependency check se relevante.
Re-rodar deteccao de duplicados em pastas de alta rotatividade
(Downloads, Desktop) se usuario aceitar.

## 25. Edge cases

Pastas vazias, arquivos sem extensao, circular symlinks, encoding
ambiguo, >10GB, >10.000 itens numa pasta, arquivos em uso, manifesto
corrompido, permissoes revogadas, espaco esgotado, cloud sync conflitos,
schema de manifesto desconhecido ou antigo, duplicados com hard links
(mesmo inode = nao duplicado, reportar como hard link).

Para tratamento detalhado de cada caso, ver
[references/ROLLBACK-AND-RECOVERY.md] e [references/EXECUTION-STRATEGY.md].