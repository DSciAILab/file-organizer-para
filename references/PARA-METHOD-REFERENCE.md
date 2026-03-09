# PARA Method Reference

## Fonte

Metodologia criada por Tiago Forte.
- Site: fortelabs.com/blog/para/
- Livro: "The PARA Method: Simplify, Organize, and Master Your Digital
  Life." Atria Books, 2023. ISBN 978-1982167585.

## Principio central

Organizar por acionabilidade, nao por assunto. A informacao deve estar
onde o projeto ou objetivo que vai usa-la reside, nao em categorias
amplas de tema.

## Definicoes

Projetos: esforcos de curto prazo com resultado especifico e prazo
real ou estimado. A diferenca chave: projetos terminam.
Exemplos de Forte: completar design de pagina web, comprar computador
novo, escrever relatorio de pesquisa, renovar banheiro, terminar curso
de espanhol, montar moveis da sala.

Areas: partes importantes do trabalho e da vida que requerem atencao
continua. Na definicao de Forte, sao responsabilidades onde se busca
manter um padrao ao longo do tempo.
Exemplos de Forte: Marketing, RH, Product Management, R&D, Saude,
Financas, Filhos, Escrita, Carro, Casa.

Recursos: topicos de interesse ou aprendizado. Nao geram
responsabilidade direta.
Exemplos de Forte: design grafico, produtividade pessoal, jardinagem,
cafe, arquitetura moderna, web design, lingua japonesa, literatura
francesa, notetaking, breathwork, formacao de habitos, fotografia.

Arquivo: itens das tres categorias anteriores que nao sao mais ativos.
Projetos concluidos ou pausados, areas que nao fazem mais parte da
vida, recursos que perderam interesse.

## Nota sobre simplificacao nesta skill

Para fins de organizacao de arquivos, esta skill trata areas como pastas
de responsabilidade continua. A nuance de Forte sobre "manter um padrao"
e relevante ao decidir: se voce tem responsabilidade pessoal sobre
aquilo, e area. Se e apenas interesse ou referencia, e recurso.

## Extensoes operacionais desta skill (nao presentes no PARA original)

- Convencoes de nomenclatura com prefixo de data
- Limite de profundidade de 3 niveis
- Separacao Trabalho/Pessoal com arvores independentes
- Verificacao de dependencias e links
- Manifesto, rollback e recovery
- Modos de scan
- Automacao de classificacao
- Formatos de pasta numerados
- Manutencao periodica por data de modificacao

## Tabela de classificacao

| Item                              | Destino                          | Motivo                      |
|-----------------------------------|----------------------------------|-----------------------------|
| Declaracao de IR do ano atual     | 1-Projetos                       | Resultado especifico, prazo |
| Contrato de aluguel vigente       | 2-Areas/Casa                     | Responsabilidade continua   |
| Manual do carro                   | 2-Areas/Veiculo                  | Responsabilidade continua   |
| Apolice de seguro ativa           | 2-Areas/Seguros                  | Responsabilidade continua   |
| Template de proposta comercial    | 3-Recursos/Templates             | Referencia reutilizavel     |
| Artigo salvo sobre produtividade  | 3-Recursos/Artigos               | Interesse, sem responsab.   |
| Fotos de viagem passada           | 4-Arquivo/Projetos-concluidos    | Projeto encerrado           |
| Relatorio mensal recorrente       | 2-Areas/Gestao-equipe            | Atividade continua          |
| Proposta cliente em andamento     | 1-Projetos/2026-03_Proposta-X    | Entrega com prazo           |
| Planejamento festa aniversario    | 1-Projetos/2026-05_Aniversario   | Evento com data             |
| Arquivos antigos do Desktop       | 4-Arquivo/Legado-pre-organizacao | Sem classificacao clara     |

## Arvore de decisao

1. Existe resultado especifico com prazo? Sim = 1-Projetos
2. Existe responsabilidade continua? Sim = 2-Areas
3. E referencia ou aprendizado? Sim = 3-Recursos
4. Inativo, encerrado, antigo? Sim = 4-Arquivo

## Convencoes de nomes completas

Pastas:
- Projeto mensal: AAAA-MM_Nome-descritivo
- Projeto trimestral: AAAA-QN_Nome-descritivo
- Projeto anual: AAAA_Nome-descritivo
- Area: Nome-descritivo
- Recurso: Nome-descritivo
- Subpasta: NN_Nome-descritivo (01_Briefing, 02_Rascunhos, etc)

Arquivos:
- AAAA-MM-DD_Descricao.ext
- AAAA-QN_Descricao-vN.ext
- Tipo-Descricao.ext
- NF-AAAA-MM-NNN_Fornecedor.ext

Regras universais:
1. Sem espacos (excecao: number-dot-space para pastas de categoria)
2. Hifens entre palavras
3. ASCII safe: sem acentos, sem cedilha
4. Sem ALLCAPS exceto siglas reais (NF, IR, CV)
5. Versoes: -v1, -v2. Nunca FINAL, novo, copia
6. Preservar extensao exatamente
7. Colisao: -duplicata-1, -duplicata-2
8. Windows: validar CON, PRN, AUX, NUL, COM1-COM9, LPT1-LPT9
9. Nomes terminando em ponto ou espaco: FLAGGED
10. Unicode nao-latino: FLAGGED, opcoes: manter, prefixar com data,
    transliterar com confirmacao, renomear manualmente
11. Transliteracoes sao aproximadas e nunca aplicadas sem aprovacao
12. Legado-pre-organizacao: sem rename automatico

## Estrutura de pastas

Formato number-hyphen:
- 1-Projetos, 2-Areas, 3-Recursos, 4-Arquivo

Formato number-dot-space:
- 1. Projetos, 2. Areas, 3. Recursos, 4. Arquivo

Profundidade maxima: 3 niveis abaixo da raiz da categoria.

Exemplos validos:
- 2-Areas/Financas-pessoais/Extratos-bancarios/
- 1-Projetos/2026-03_Declaracao-IR/01_Briefing/
- Trabalho/2-Areas/Clientes/Contratos/

Exemplo invalido:
- 2-Areas/Financas-pessoais/Extratos-bancarios/2026/Janeiro/

Estrutura interna de projeto:
  01_Briefing/
  02_Rascunhos/
  03_Versao-final/
  04_Feedback/
  README.txt

Estrutura interna de arquivo:
  4-Arquivo/Projetos-concluidos/
  4-Arquivo/Areas-inativas/
  4-Arquivo/Legado-pre-organizacao/

## Politica de ambiguidade

Se varios arquivos tiverem padrao claro e mesma classificacao provavel,
propor decisao em lote.

Exemplo lote:
"Encontrei 17 PDFs de extrato bancario. Tratar todos como
Pessoal/2-Areas/Financas-pessoais/Extratos-bancarios? (yes/no/revisar)"

Exemplo individual:
"Encontrei Relatorio-Projeto-Alpha.pdf.
(a) 1-Projetos, se ativo
(b) 4-Arquivo/Projetos-concluidos, se terminou
(c) 3-Recursos, se e referencia"

Triagem rapida: itens incertos podem ir para Legado-pre-organizacao,
com aprovacao explicita, sem rename.