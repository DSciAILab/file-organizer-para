# Manifest Schema Reference

## Ponteiro global

Localizacao:
- macOS/Linux: ~/.config/openclaw/para-root.json
- Windows: %APPDATA%/OpenClaw/para-root.json

{
  "version": 1,
  "activeRoot": "/caminho/absoluto/para/PARA",
  "knownRoots": ["/caminho/absoluto/para/PARA"],
  "lastUsed": "2026-03-09T10:30:00Z"
}

## Config da raiz (.para-config.json)

{
  "version": 7,
  "root": "/caminho/absoluto/para/PARA",
  "mode": "single | separate",
  "trees": [],
  "os": "macOS | Linux | Windows | WSL",
  "created": "2026-03-09T10:30:00Z",
  "lastOrganized": null,
  "lastMaintenance": null,
  "scanModeDefault": "safe",
  "categoryFolderFormat": "number-hyphen | number-dot-space",
  "maxDepthBelowCategoryRoot": 3,
  "manifestActiveScanPattern": "para-manifest-*.json",
  "manifestHistoryDir": ".para-manifest-history",
  "tempDirName": ".para-temp",
  "ignoredStandalonePatterns": [
    ".DS_Store", "thumbs.db", "desktop.ini", ".Spotlight-V100",
    ".Trashes", ".fseventsd", "$RECYCLE.BIN",
    "System Volume Information"
  ],
  "ignoredDiscoverPatterns": [
    ".para-config.json", ".para-lock.json",
    ".para-manifest-history", ".para-manifest-history/",
    ".para-temp", ".para-temp/",
    "para-manifest-*.json", "para-rollback-*"
  ],
  "ignoredRecoveryPatterns": [
    ".para-manifest-history", ".para-manifest-history/",
    ".para-temp", ".para-temp/",
    "para-rollback-*"
  ],
  "dependencyLog": [],
  "lastIncompleteManifest": null,
  "lockTimeoutMinutes": 120,
  "heartbeatTimeoutMinutes": 15,
  "diskSafetyMarginPercent": 10,
  "diskSafetyMarginMinBytes": 1073741824,
  "windowsPathWarnThreshold": 240,
  "windowsPathHardThreshold": 260,
  "batchWriteThreshold": 1000,
  "batchWriteSize": 10,
  "maxRetryPerOperation": 3
}

Validacao do campo trees:
- mode=single: trees deve ser []
- mode=separate: trees deve ter >= 2 nomes unicos, ASCII safe, hifens ok
- Se trees > 5: alertar (estrutura dispersa)

## Lock (.para-lock.json)

{
  "version": 3,
  "executionId": "uuid",
  "manifestPath": "/abs/path/para-manifest-AAAA-MM-DD-HHMM.json",
  "startedAt": "2026-03-09T11:00:00Z",
  "updatedAt": "2026-03-09T11:05:00Z",
  "source": "/abs/source",
  "root": "/abs/root",
  "mode": "organize | maintenance | archive | resume | rollback",
  "status": "active"
}

Regras do lock:
- Criar antes de qualquer escrita.
- Atualizar updatedAt: a cada operacao ou a cada 60s.
- Stale: updatedAt > heartbeatTimeoutMinutes.
- Duracao total longa: alertar, nao stale.
- Lock nao criavel (permissao, disco) = abortar.
- Reconciliacao: executionId + manifestPath + root + source coerentes.
- Divergencia = perguntar antes de continuar.
- Remover apenas em: sucesso, rollback concluido, abort seguro.
- Falha grave: manter lock.

## Manifesto (para-manifest-*.json)

{
  "schemaVersion": 6,
  "date": "2026-03-09T11:00:00Z",
  "completedAt": null,
  "executionId": "id-unico",
  "root": "/abs/path/PARA",
  "source": "/abs/path/source",
  "mode": "organize",
  "scanMode": "safe",
  "categoryFolderFormat": "number-hyphen",
  "lastDurableOperationIndex": 0,
  "directoriesCreated": [
    {
      "path": "/abs/path/criada",
      "status": "created | kept_not_empty | removed_on_rollback"
    }
  ],
  "operations": [
    {
      "index": 1,
      "type": "move | copy | copy_only",
      "status": "planned | in_progress | copied | verified |
                 source_removed | completed | skipped | failed |
                 rolled_back",
      "source": "/abs/origem",
      "destination": "/abs/destino",
      "tempPath": null,
      "sameFilesystem": true,
      "networkFilesystem": false,
      "sourceExistedBefore": true,
      "destinationExistedBefore": false,
      "bytesExpected": 12345,
      "bytesCopied": 12345,
      "verified": true,
      "renamed": false,
      "retryCount": 0,
      "metadataStatus": "preserved | best_effort | not_checked | failed",
      "placeholderState": "none | hydrated | online_only | unknown",
      "timestampStarted": "2026-03-09T11:00:01Z",
      "timestampEnded": "2026-03-09T11:00:02Z",
      "error": null
    }
  ],
  "configEdits": [
    {
      "index": 1,
      "targetFile": "/abs/.zshrc",
      "backupFile": "/abs/.zshrc.bak-2026-03-09-1100",
      "status": "planned | completed | restore_planned | restored |
                 failed",
      "timestampStarted": null,
      "timestampEnded": null,
      "error": null
    }
  ],
  "skipped": [],
  "notChecked": [],
  "errors": [],
  "rollbackStatus": null
}

## Mapeamento plano -> manifesto

| Acao no plano                  | type      | renamed |
|-------------------------------|-----------|---------|
| Move                          | move      | false   |
| Move + Rename                 | move      | true    |
| Copy + Verify + Remove source | copy      | *       |
| Copy_only                     | copy_only | *       |
| Skip                          | sem op    |         |

## Estados da operacao

1. planned: estado inicial
2. in_progress: antes da acao real
3. copied: copia concluida (apenas cross-fs/rede)
4. verified: integridade confirmada
5. source_removed: origem removida (type=move ou copy)
6. completed: operacao finalizada

Move local via rename nativo: planned -> completed (intencional,
rename e atomico, nao separa copia de verificacao).

7. failed: erro (registrar no campo error)
8. rolled_back: revertida com sucesso
9. skipped: pulada por decisao

completedAt do manifesto: so preencher quando todas as operacoes
estiverem em estado terminal coerente.

## directoriesCreated status values

- created: pasta foi criada por esta execucao
- kept_not_empty: rollback tentou remover mas pasta tem conteudo novo
- removed_on_rollback: pasta removida no rollback (estava vazia)

## retryCount

- Incrementar antes de cada nova tentativa de operacao failed.
- Ao atingir maxRetryPerOperation: nao oferecer retry automatico.
- Reset pelo usuario: exigir confirmacao, registrar no manifesto com
  timestamp.
- Nunca incrementar para operacoes que nao estejam em failed.

## Ciclo de vida de manifestos

- Incompletos (completedAt null): ficam no root. Visiveis para recovery.
- Completos: movidos para .para-manifest-history/ no inicio da proxima
  execucao.
  Se o move falhar: marcar com sufixo -archive-failed e excluir do pool
  de scan ativo.
- Corrompidos: ver ROLLBACK-AND-RECOVERY.md.

## Informacao sensivel

O manifesto NUNCA contem: conteudo de arquivos, segredos, tokens, senhas,
valores de variaveis de ambiente.
O manifesto PODE conter: caminhos, tipos, nomes de chaves, estados, erros.
