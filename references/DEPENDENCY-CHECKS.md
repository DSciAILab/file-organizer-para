# Dependency Checks Reference

## Status possiveis

- OK
- FLAGGED
- NOT_CHECKED
- ERROR

## Symlinks (18.1)

- Detectar links simbolicos no escopo.
- Qualquer symlink = FLAGGED.
- Fora do escopo = risco adicional.
- Link relativo = risco maior ao mover.
- Symlinks relativos dentro da mesma pasta pai = risco menor se mover
  a pasta inteira.
- REGRA DURA: nunca seguir symlinks recursivamente. Registrar como
  item, nunca expandir conteudo.

Opcoes: (1) Pular, (2) Mover como link sem alterar target,
(3) Resolver manualmente depois.

## Hard links (18.2)

- Detectar link count > 1.
- Cross-filesystem = FLAGGED.
- Mesmo filesystem = risco menor.

Opcoes: (1) Mover se mesmo filesystem, (2) Copiar e aceitar quebra,
(3) Pular.

## Projetos de software (18.3)

Marcadores fortes incondicionais:
.git/, package.json, Cargo.toml, go.mod, Pipfile, pyproject.toml,
Gemfile, composer.json, pom.xml, build.gradle, Makefile, CMakeLists.txt,
*.sln, *.xcodeproj, *.xcworkspace

Marcadores fortes condicionais (precisam de outro marcador):
Dockerfile, docker-compose.yml

Marcadores fracos:
node_modules/, .venv/, venv/, env/, __pycache__/, .gradle/, .maven/,
.cargo/, .npm/, .yarn/, bower_components/, vendor/, Pods/, .terraform/,
.serverless/, .cache/, .config/

Estrutura de codigo fonte (para confirmar fracos):
src/, app/, lib/, cmd/, tests/ ou test/, main.py, main.go, index.js,
index.ts, app.py, manage.py, Program.cs, Main.java

Classificacao:
- 1 forte incondicional = projeto de software
- 1 forte condicional + 1 outro marcador qualquer = projeto
- 2+ fracos + estrutura de codigo = projeto
- Fracos isolados ou condicionais isolados = nao basta

REGRA DURA: nunca mover arquivos internos separadamente. Pasta inteira
ou nada.

Opcoes: (1) Mover pasta inteira, (2) Pular, (3) Criar representacao
sem mover, (4) Revisar manualmente.

## Configs com caminhos absolutos (18.4)

Inspecionar (com redacao de segredos):
.env, .ini, .cfg, .conf, .toml, .yaml, .yml, .json em contexto de
config, .plist, .reg, .desktop

Heuristica para .json como config:
- Nome contem: config, settings, preferences, options, rc, tsconfig,
  jsconfig, launch, workspace, manifest
- Tamanho < 1 MB
- Em raiz de projeto de software
- Contem chaves: path, dir, directory, root, home, output, input, src,
  dest, target, include, exclude
Excluir: .json > 10 MB, dentro de node_modules/vendor/.venv, datasets.

Mostrar apenas: arquivo, chave/contexto, tipo de risco, caminho
mascarado. NUNCA mostrar tokens, senhas, DSN completo ou valor bruto.

Path relativo: NOT_CHECKED por padrao. Move atomico da unidade inteira
reduz risco. Nunca prometer reescrita automatica sem verificacao.

Opcoes: (1) Mover e atualizar auto, (2) Mover e usuario atualiza,
(3) Pular.

## PATH e shell configs (18.5) - BEST_EFFORT

~/.bashrc, ~/.bash_profile, ~/.zshrc, ~/.profile,
~/.config/fish/config.fish, PATH atual.

## Shortcuts e aliases (18.6) - BEST_EFFORT

.lnk (Windows), .desktop (Linux), aliases macOS.
Finder aliases: formato proprietario Apple, diferente de symlinks POSIX.
Podem sobreviver a algumas movimentacoes, mas nao assumir imunidade.

## Scheduled tasks (18.7) - BEST_EFFORT

crontab, /etc/cron.d/, ~/.config/systemd/user/, launchd, schtasks.

## Cloud sync boundary (18.8)

Dropbox, OneDrive, iCloud Drive, Google Drive.
Sair ou entrar em pasta sincronizada = FLAGGED.

## Cloud placeholders e online-only (18.9) - BEST_EFFORT

Detectar placeholders, stubs nao hidratados, online-only.
Riscos: move pode disparar download, tamanho/hash pode falhar.
FLAGGED com aviso.

Opcoes: (1) Hidratar e mover, (2) Pular, (3) Revisar manualmente.

## Arquivos grandes, bloqueados, read-only (18.10)

> 1 GB = FLAGGED. Read-only ou em uso = FLAGGED.

## Cross-filesystem (18.11)

Estrategia obrigatoria: copy -> verify -> remove source.

## Espaco livre (18.12)

Margem = max(diskSafetyMarginPercent, diskSafetyMarginMinBytes).
Insuficiente = FLAGGED ou ERROR.

## Case-collision, path length, nomes reservados (18.13)

Aviso em windowsPathWarnThreshold (240). Critico em
windowsPathHardThreshold (260). Validar CON, PRN, etc.

## Permissoes (18.14)

Leitura no source, escrita no source (se remocao), escrita no destino.
Insuficiente = FLAGGED. Oferecer: ajustar, pular, copy_only.

## Filesystem de rede (18.15) - BEST_EFFORT

NFS, SMB/CIFS, AFP, SSHFS, FUSE remoto.
NFS: rename nao e atomico para outros clientes.
SMB: depende de versao/implementacao.
FLAGGED. Recomendar copy -> verify -> remove.

## Metadata (18.16)

mtime, ctime, permissoes POSIX, ACLs, xattrs, resource forks,
Finder tags, ADS, quarantine flags.
Rename local = preserved. Copy = best_effort. Incerto = not_checked.

## Hash obrigatorio (18.17)

Quando hashing for suportado, obrigatorio para:
- >= 1 GB
- Filesystem de rede
- Databases (.sqlite, .db, .mdb)
- Executaveis, binarios, app bundles
- Compactados e imagens (.zip, .tar, .gz, .dmg, .iso, .qcow2, .vmdk)
- Configs sensiveis movidas individualmente
- Arquivos com falha previa de copy ou timeout

Se hash nao suportado nesses casos: NOT_CHECKED, exigir aprovacao do
usuario para seguir com verificacao apenas por tamanho.

## Formato do dependency report

DEPENDENCY_REPORT
Escopo: <dir>, Modo: Quick|Safe|Deep

Resumo: OK: N, FLAGGED: N, NOT_CHECKED: N, ERROR: N

Itens FLAGGED:
1. <caminho>
   Tipo: symlink | hard-link | software-project | config-path |
   path-ref | shortcut | scheduled-task | cloud-boundary |
   cloud-placeholder | large-file | read-only | cross-filesystem |
   disk-space | case-collision | path-length | reserved-name |
   permission | network-filesystem | metadata | unicode-name
   Detalhe: <descricao>
   Acao sugerida: <opcoes>

Itens NOT_CHECKED:
1. <tipo> - Motivo: <razao>

Perguntar: (a) Revisar flagged individualmente, (b) Pular flagged e
organizar o resto, (c) Abortar e revisar manualmente.