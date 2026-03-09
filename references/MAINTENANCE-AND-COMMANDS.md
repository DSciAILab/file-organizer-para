# Maintenance and On-Demand Commands Reference

## Comandos on-demand (Secao 29)

### Novo projeto
Trigger: "novo projeto" / "new project"
1. Perguntar nome e prazo estimado.
2. Perguntar arvore (se mode=separate).
3. Criar: <tree>/1-Projetos/AAAA-MM_Nome-descritivo/
4. Subpastas: 01_Briefing, 02_Rascunhos, 03_Versao-final, 04_Feedback.
5. README.txt opcional.
6. Confirmar.

### Nova area
Trigger: "nova area" / "new area"
1. Perguntar nome e arvore.
2. Criar: <tree>/2-Areas/Nome-descritivo/
3. Confirmar.

### Novo recurso
Trigger: "novo recurso" / "new resource"
1. Perguntar nome e arvore.
2. Criar: <tree>/3-Recursos/Nome-descritivo/
3. Confirmar.

### Arquivar projeto
Trigger: "arquivar projeto" / "archive project"
1. Listar projetos em 1-Projetos.
2. Perguntar qual arquivar.
3. Mover para 4-Arquivo/Projetos-concluidos/
4. Gerar registro breve.

### Status
Trigger: "status PARA" / "PARA status"
1. Mostrar config atual.
2. Contar projetos ativos, areas, recursos, itens em arquivo.
3. Informar ultimo scan e ultima manutencao.
4. Listar manifestos incompletos, se houver.

## Manutencao periodica (Secao 30)

### Revisao semanal (5 minutos)
- Listar projetos em 1-Projetos.
- Verificar data de ultima modificacao.
- Projeto inativo > 30 dias: FLAGGED.
  Sugerir: (a) arquivar, (b) manter ativo, (c) revisar depois.

### Revisao mensal (15 minutos)
- Listar areas e recursos.
- Verificar data de ultima modificacao.
- Sem modificacao > 90 dias: FLAGGED.
  Sugerir: (a) arquivar, (b) manter, (c) revisar depois.
- Verificar 4-Arquivo/Legado-pre-organizacao.
  Se houver itens: perguntar se quer classificar ou manter.
- Re-rodar dependency check nos itens flagged, se relevante.

### Relatorio de manutencao

PARA MAINTENANCE REPORT
Data, Root.

Projetos ativos: N
- Inativos (>30 dias): <lista>

Areas: N
- Sem modificacao (>90 dias): <lista>

Recursos: N
- Sem modificacao (>90 dias): <lista>

Arquivo:
- Legado-pre-organizacao: N arquivos
- Projetos concluidos: N
- Areas inativas: N

Recomendacoes: <lista>
Proximo scan recomendado: <data>
