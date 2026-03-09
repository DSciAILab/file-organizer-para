# Rollback and Recovery Reference

## Recovery de execucao interrompida (Secao 11)

No inicio de cada execucao, antes de qualquer scan:

1. Escanear root, excluindo ignoredRecoveryPatterns.
   Manifestos ativos (para-manifest-*.json) ficam visiveis.
2. Validar cada manifesto: JSON valido? schemaVersion conhecido?
   completedAt presente?
3. Incompleto = completedAt ausente ou null.
4. Se lock + manifesto incompleto: reconciliar por executionId,
   manifestPath, root, source. Coerentes = mesma execucao.
   Incoerentes = perguntar: (a) confiar lock, (b) confiar manifesto,
   (c) arquivar ambos, (d) abortar.
5. Multiplos incompletos: listar por data e source.
6. Apresentar opcoes: (a) retomar, (b) reverter, (c) arquivar e novo.

Retomar:
- Carregar manifesto, reconciliar por estado (ver abaixo).
- Continuar da primeira operacao nao concluida.
- Preencher completedAt ao concluir.

Reverter:
- Rollback das operacoes efetivamente concluidas.
- Gerar relatorio de rollback.

Arquivar:
- Mover para .para-manifest-history/ com sufixo -archived.

## Reconciliacao por estado (Secao 24)

Regra geral: nunca assumir que o manifesto sozinho reflete a verdade.
Comparar: existencia de origem, destino, tamanho, hash, tempPath.

planned:
- Origem existe, destino nao: executar.
- Destino existe: investigar colisao.

in_progress:
- Origem existe, destino nao, tempPath nao: resetar para planned.
- tempPath existe e parcial: perguntar se remove e reinicia.
- tempPath existe e consistente: promover para copied.
- Destino existe e completo: promover para copied/verified.

copied:
- Revalidar destino (tamanho + hash se aplicavel).
- Valido: promover para verified. Invalido: failed ou planned.

verified:
- type=move ou copy: se origem existe, seguir para source_removed.
- type=copy_only: promover para completed.
- Origem nao existe + destino valido: completed.

source_removed:
- Destino valido: completed.
- Destino invalido: ERROR critico.

completed: nao reexecutar.
skipped: nao reexecutar.

failed:
- retryCount < max: oferecer retry, pular, reverter, abortar.
- retryCount >= max: apenas pular, reverter, abortar.

rolled_back: nao reexecutar.

## Config edits na retomada

- planned: aplicar se necessario.
- completed: nao reaplicar.
- restore_planned: rollback em andamento.
- failed: perguntar se tenta restaurar.

## Limpeza de tempPath orfaos

Antes de processar operacoes na retomada:
1. Identificar operacoes com tempPath nao-null.
2. Verificar existencia fisica.
3. Se existe e operacao nao e copied/verified: perguntar remove/
   inspeciona/pula.
4. tempPath e artefato de execucao, remocao nao viola preservacao
   de dados do usuario.

## Limite de tentativas

- retryCount so incrementa para operacoes failed.
- Ao atingir maxRetryPerOperation: parar retry automatico.
- Reset pelo usuario: confirmacao explicita + registro no manifesto.

## Rollback (Secao 26)

Principios:
- Ordem inversa das operacoes concluidas.
- So atua em completed, source_removed, verified.
- planned, skipped, failed: nao revertidas.
- configEdits: revertidos apos arquivos, ordem inversa.

Estrategia por tipo:

type=move:
- Mover destination de volta para source.
- Se renamed=true, restaurar nome original.
- Se rename falhar, usar copy -> verify -> remove.
- Marcar rolled_back.

type=copy:
- Se source existe: remover destination.
- Se source nao existe: mover destination de volta.
- Marcar rolled_back.

type=copy_only:
- Remover destination (source intacto).
- Marcar rolled_back.

configEdits completed:
- Restaurar backup. Marcar restored.

Colisao no rollback:
- Nunca sobrescrever silenciosamente.
- Perguntar: (a) sobrescrever, (b) usar -rollback-collision-N,
  (c) pular.
- Padrao seguro: (b).

Pastas criadas:
- Remover durante rollback apenas se criada por esta execucao E vazia.
- Se contem conteudo novo: manter, registrar como kept_not_empty.

Rollback parcial:
- Se uma operacao falha no rollback: registrar, continuar.
- Gerar relatorio com: revertidas, falhas, estado atual.
- rollbackStatus = partial.

Script de rollback:
- .sh (Unix) ou .ps1 (Windows).
- Comandos em ordem inversa, comentados.
- Cabecalho com aviso para revisar antes de executar.

Rollback interativo:
- Reverter tudo, reverter selecionadas, cancelar.
- Respeitar dependencias (nao remover pasta com arquivos dentro).

## Manifesto corrompido (Secao 35)

JSON invalido:
1. Informar: "O manifesto <nome> nao pode ser lido."
2. Opcoes: (a) mover para history com -corrupted, (b) recovery parcial
   (ler ate onde JSON e valido, apresentar ao usuario, SOMENTE LEITURA,
   nunca dirige resume ou rollback automatico), (c) deletar (artefato
   da skill, nao dado do usuario), (d) abortar.

Schema desconhecido (schemaVersion > suportado):
1. Informar: "Schema versao N, suportado ate M."
2. SOMENTE LEITURA. Resume, rollback e escrita sao proibidos.
3. Opcoes: (a) ler como best-effort (campos desconhecidos ignorados),
   (b) mover para history com -unknown-schema, (c) abortar para
   atualizar a skill.

Schema antigo (schemaVersion < atual):
1. Informar que e schema anterior.
2. Opcoes: (a) migrar com backup, (b) ler best-effort sem migrar,
   (c) mover para history.
3. Se migrar: copia do original antes de alterar.