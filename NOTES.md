# Notas da Avaliação Intermediária (dupla 025bc8, RS 39569455, classe difícil Bot)

## Regras de redação (valem para todo texto do notebook e do relatório)
- Linguagem formal, porém natural e humana.
- Sem travessões (nem — nem –) e sem negrito.
- Responder à pergunta logo na primeira frase e sustentar com números do próprio notebook.
- Frases curtas; nada de explicação que o enunciado não pediu.

## Padrão da disciplina (detalhe em md/RESUMO.md)
- Todo achado tem o código que o produziu e o número que o sustenta; os números são conferidos contra o dataset da dupla (tolerância de 0,02 nas áreas e de 0,05 nos limiares).
- Leituras que o professor espera:
  - o 0,5 é o default da biblioteca, não uma decisão;
  - a PR é a curva do SOC;
  - a PR-AUC do acaso é igual à prevalência;
  - custo e capacidade são restrições diferentes;
  - o número honesto é o do split temporal.
- Descontos: `alertas_ids` no modelo (−2,0); `saida/` ausente ou com nome errado (−1,0); falta do RS (−0,5); números fora da tolerância (até −2,0).

## Decisões
- Split temporal por (dia, Label), como no h) do R2. O split só por dia (texto literal do enunciado) deixa 0 fluxos de Bot no teste e prevalência 0,429/0,338, o que inviabiliza o item c).
- O RF usa só as 62 features do CICFlowMeter, sem as 6 derivadas do g) do R2.
- Notebook enxuto: código e células Markdown só para as perguntas do enunciado. O item a) não tem Markdown de resposta.
- Os dados são lidos de ../roteiro2/roteiro2_dados/dados/R2_025bc8.csv.gz; a pasta roteiro2/ não é alterada.

## Convenções de código
- "Corte t" significa `p >= t` em todos os itens. O `rf.predict()` equivale a `p > 0,5`: em 0,5 a diferença é de 8 fluxos com pontuação exata 0,5 (recall 0,9332 contra 0,9327).
- As pontuações do RF são múltiplos de 0,01 (100 árvores). O histograma usa um bin por valor.
- "Entre 0,2 e 0,8" é intervalo fechado.
- Imports ficam na célula do topo; as variáveis `rf`, `X_te`, `y_te`, `label_te`, `p_te` e `flx_limpo` são reaproveitadas pelos itens seguintes.
- Execução: `PYTHONUTF8=1 python -X utf8 -m jupyter execute --inplace avaliacao_025bc8.ipynb`. Não há nbconvert, e sem `-X utf8` o notebook é relido como cp1252 e os acentos corrompem. O notebook é editado com nbformat, também com `-X utf8`.

## Estado (2026-09-17)
- [x] a) limpeza em 105.097 x 66 (62 features); treino 73.556 (prevalência 0,4019) e teste 31.541 (prevalência 0,4021); arquivos salvos em saida/.
- [x] b) p_te = predict_proba do RF. Em [0,2; 0,8] há 3.548 fluxos (2.825 BENIGN e 723 ataques); em [0,45; 0,55], 261; abaixo de 0,2, 550 ataques. O corte 0,5 passa por região povoada, num vale.
- [x] c) ROC-AUC 0,9721 e PR-AUC 0,9731, com prevalência de 0,4021 (por isso as duas concordam). Recall global em 0,5 = 0,9332 (12.684 ataques); recall do Bot = 0,1554 (592 fluxos). 75,8% do Bot tem pontuação 0; no corte 0,05, o recall do Bot é 0,1605.
- Figuras para o relatório (b e c) ficam nas saídas do notebook; extrair o PNG na hora de montar o PDF.
- [ ] d), e) e f) (Parte A)
- [ ] g) a j) (Parte B, dados em PI/creditcard.csv.gz: 284.807 linhas, 492 fraudes)
- [ ] relatório PDF
