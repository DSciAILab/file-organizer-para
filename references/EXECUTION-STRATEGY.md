# Execution Strategy Reference

## Mesmo filesystem local (22.1)

Rename nativo e o padrao. Excecoes explicitas:
- Filesystem de rede detectado
- Cloud sync boundary detectado
- Pedido explicito de copy + verify

Rename nativo preserva hard link count no mesmo filesystem.
metadataStatus = preserved.

## Filesystems diferentes local (22.2)

Copy -> verify (tamanho + hash quando exigido pela Secao 18.17) ->
remove origem. Falha = manter origem intacta, marcar erro.

## APFS no macOS (22.3)

Clone copy-on-write ocorre dentro do mesmo volume APFS, nao entre
volumes. Para verificacao, tamanho basta. Hash so nos casos da 18.17.

## Colisao de nomes (22.4)

Nunca sobrescrever silenciosamente. Usar -duplicata-N. Registrar.

## Config edits (22.5)

Backup .bak-AAAA-MM-DD-HHMM antes. Validar sintaxe depois.
Registrar diff resumido. Nunca exibir segredo.

## Projetos de software (22.6)

Pasta inteira como unidade. Nunca extrair internos.

## Legado-pre-organizacao (22.7)

Mover sem rename. Renomear so depois, se usuario pedir.

## Escrita atomica (22.8)

1. Escrever em temp file no mesmo diretorio
2. flush + fsync quando suportado
3. Rename atomico para nome final
4. Nunca deixar arquivo parcial como valido

## lastDurableOperationIndex (22.9)

Indice da ultima operacao incluida em gravacao duravel.
- Gravacao por operacao: indice da operacao recem concluida.
- Gravacao em lote: indice da ultima operacao do lote.

## Batch write (22.10)

Apos batchWriteThreshold (padrao 1000): gravar em lotes de
batchWriteSize (padrao 10).
- Heartbeat do lock continua obrigatorio.
- Operacoes acima de lastDurableOperationIndex devem ser
  re-verificadas na retomada.
- Informar ao usuario que a gravacao e em lotes.

## Filesystem de rede (22.11)

Nao confiar em rename atomico.
Preferir copy -> verify -> remove.
Timeout explicito quando suportado.
Verify falha por timeout = nao remover origem.

## Metadata (22.12)

- Move local: preserved (salvo evidencia contraria)
- Copy entre filesystems ou rede: best_effort
- Runtime nao suporta: not_checked
- Relatorio declara nivel aplicado.

## Temp files (22.13)

tempPath e artefato de execucao, nao dado do usuario.
Criar em <root>/.para-temp/ quando possivel.
Todo tempPath registrado no manifesto.

## Falha individual (25.1)

1. Marcar failed, registrar erro detalhado.
2. Gravar manifesto imediatamente (escrita atomica).
3. Atualizar heartbeat.
4. Perguntar: (a) retry, (b) pular e continuar, (c) pausar, (d) abort.

## Falha sistemica (25.2)

3+ consecutivas: pausar automaticamente.
Informar possivel problema sistemico.
Oferecer: (a) investigar, (b) pular restante e gerar relatorio,
(c) reverter concluidas, (d) abortar.

## Falha critica (25.3)

Filesystem inacessivel ou lock nao atualizavel:
1. Gravar manifesto best-effort.
2. Registrar erro critico.
3. Manter lock ativo.
4. Nunca tentar remover lock em erro critico.
5. Recovery na proxima execucao via lock stale + manifesto incompleto.

## Falha em config edit (25.4)

1. Oferecer restauracao do backup se existir.
2. Marcar failed.
3. Adicionar ao checklist manual.
4. Continuar salvo se for pre-requisito.

## Edge cases de execucao

Pastas vazias: FLAGGED. Perguntar: remover, manter ou mover para Arquivo.

Arquivos sem extensao: FLAGGED. Nao assumir tipo. Perguntar.

Encoding ambiguo: FLAGGED. Mostrar hex se necessario. Perguntar rename.

> 10 GB: FLAGGED com aviso especial. Informar tempo estimado.

> 10.000 itens numa pasta: FLAGGED. Sugerir subdivisao.

Circular symlinks: detectar loops. FLAGGED. Nunca seguir.

Arquivos em uso: FLAGGED. Tentar identificar processo (BEST_EFFORT).
Oferecer: pular e incluir em checklist, tentar depois, abortar.

Permissoes revogadas durante execucao: marcar failed, registrar,
continuar ou perguntar.

Espaco esgotado durante copy: parar imediatamente, nao remover origem,
informar espaco necessario vs disponivel, perguntar.

Cloud sync conflito: detectar BEST_EFFORT. FLAGGED. Nao sobrescrever.