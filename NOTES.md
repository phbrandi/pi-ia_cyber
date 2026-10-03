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
- Os dados do R2 são lidos de R2_025bc8.csv.gz na raiz do repo; a célula de imports ainda aceita ../roteiro2/ e ../Roteiro 02/ como alternativas, então o notebook roda nas duas máquinas sem edição.

## Convenções de código
- "Corte t" significa `p >= t` em todos os itens. O `rf.predict()` equivale a `p > 0,5`: em 0,5 a diferença é de 8 fluxos com pontuação exata 0,5 (recall 0,9332 contra 0,9327).
- As pontuações do RF são múltiplos de 0,01 (100 árvores). O histograma usa um bin por valor.
- "Entre 0,2 e 0,8" é intervalo fechado.
- Imports ficam na célula do topo; as variáveis `rf`, `X_te`, `y_te`, `label_te`, `p_te` e `flx_limpo` são reaproveitadas pelos itens seguintes.
- Execução: `PYTHONUTF8=1 python -X utf8 -m jupyter execute --inplace avaliacao_025bc8.ipynb`. Em 22/09/2026 esse CLI passou a recusar o `--inplace` ("expected one argument") e, com `--inplace=True`, executou sem gravar as saídas. O caminho confiável é a API: `NotebookClient(nb, kernel_name='pi-ia-cyber').execute()` seguido de `nbformat.write`. Sempre conferir se as saídas novas entraram no arquivo. Não há nbconvert, e sem `-X utf8` o notebook é relido como cp1252 e os acentos corrompem. O notebook é editado com nbformat, também com `-X utf8`.

## Ambiente
- Venv do repo em .venv/ (Python 3.12.10), ignorado pelo git; dependências em requirements.txt.
- scikit-learn fixado em 1.7.2, que é a versão que treinou o RF salvo em saida/. Com 1.9.0 o mesmo
  random_state produz outra floresta: recall global 0,9369 em vez de 0,9332 e 3.630 fluxos na faixa
  [0,2; 0,8] em vez de 3.548. Trocar de versão invalida os números já escritos no notebook.
- Kernel registrado como pi-ia-cyber. O kernelspec do notebook continua sendo o python3 genérico,
  para não quebrar na máquina da dupla; na linha de comando basta passar --kernel_name pi-ia-cyber.
- xgboost fixado em 3.1.1 (versão que gerou os números de g e h). Em 01/10/2026, numa máquina sem .venv,
  o Python 3.13 global (scikit-learn 1.7.2, pandas 2.3.3) reproduziu a Parte A idêntica e o joblib byte
  a byte, rodando via NotebookClient com kernel_name='python3'. Cada execução regrava o teste.csv.gz com
  outro carimbo de data no gzip (conteúdo igual); restaurar com git checkout se não houver mudança real.
- Conferido em 22/09/2026: reexecução de ponta a ponta reproduz todos os números dos itens a), b) e c),
  e o joblib regerado é byte a byte igual ao que estava commitado.

## Estado (2026-09-22)
- [x] a) limpeza em 105.097 x 66 (62 features); treino 73.556 (prevalência 0,4019) e teste 31.541 (prevalência 0,4021); arquivos salvos em saida/.
- [x] b) p_te = predict_proba do RF. Em [0,2; 0,8] há 3.548 fluxos (2.825 BENIGN e 723 ataques); em [0,45; 0,55], 261; abaixo de 0,2, 550 ataques. O corte 0,5 passa por região povoada, num vale.
- [x] c) ROC-AUC 0,9721 e PR-AUC 0,9731, com prevalência de 0,4021 (por isso as duas concordam). Recall global em 0,5 = 0,9332 (12.684 ataques); recall do Bot = 0,1554 (592 fluxos). 75,8% do Bot tem pontuação 0; no corte 0,05, o recall do Bot é 0,1605.
- Figuras para o relatório (b e c) ficam nas saídas do notebook; extrair o PNG na hora de montar o PDF.
- [x] d) `varrer_limiares(y, p, limiares, custo_fp=None, custo_fn=None)` conta TP, FP e FN por máscara e
  soma os custos das linhas, então aceita custo escalar ou um valor por linha (conferido: escalar, vetor
  numpy e Series dão o mesmo total). As colunas de custo só aparecem quando algum custo é informado.
  Grade de 0,05 a 0,95; as pontuações são múltiplos exatos de 0,01, então `p >= t` não precisa de tolerância.
  F1 máximo em 0,80 (0,9477) contra 0,9427 em 0,5: 510 falsos positivos a menos (82 contra 592) e 339
  falsos negativos a mais (1.186 contra 847). O F1 não é monótono, com máximo local em 0,45 (0,9453).
- [x] e) custos C_FP = 12 e C_FN = 2.500, razão de 208 para 1. Custo mínimo da grade em 0,05,
  R$ 1.369.652, contra R$ 2.124.604 em 0,5 (economia de R$ 754.952) e R$ 2.965.984 no F1 máximo
  (economia de R$ 1.596.332). O custo é monótono crescente na grade, então o mínimo está na borda;
  fora da grade o ótimo é 0,00, alertar em tudo, R$ 226.284 com recall 1,0.
  Capacidade de 3.154 alertas (10% de 31.541) é inatingível: o único limiar de 0,00 a 1,01 que cabe
  é 1,01, que não alerta em nada (recall 0, R$ 31.710.000), e o menor volume não nulo é 10.230 em 1,00.
  A causa é a prevalência: 12.684 ataques no teste, quatro vezes a capacidade, mesmo com detector perfeito.
  Sensibilidade com C_FN x5 (12.500): o mínimo continua em 0,05, R$ 6.669.652, razão de 1.041 para 1.
  A recomendação de agrupar alertas é sustentada por 9.910 dos 15.875 alertas (62,4%) serem PortScan e DoS.
- [x] f) validação cruzada estratificada de 5 folds (shuffle, RS) sobre os 105.097 fluxos limpos:
  PR-AUC 0,9980 com desvio de 0,0004 (folds 0,9985 / 0,9984 / 0,9976 / 0,9978 / 0,9979), contra 0,9731
  do split temporal, diferença de 0,0250. O contraste real está na classe difícil: com as previsões
  fora do fold, o recall do Bot é 0,9731 (1.971 fluxos no dataset limpo) contra 0,1554 no temporal, e
  0,0% do Bot recebe pontuação 0 na CV contra 75,8% no temporal. A PR-AUC global cai 0,0250 enquanto
  o recall do Bot cai 0,8177, ou seja a média esconde o colapso. Reproduz o que o R2 viu (0,96 contra 0,16).
  StratifiedKFold entrou na célula de imports do topo.
- Parte A fechada (a a f).
- Revisão de 01/10/2026: E.2 passou a dar o menor custo dentro da capacidade (R$ 31.710.000 em 1,01, sem
  alerta nenhum), e F.1 deixou de dizer que o desvio de 0,0004 confirma o otimismo (desvio mede variância
  entre folds, não vazamento; a evidência é a distância para o temporal).
- [x] g) creditcard.csv.gz na raiz do repo (284.807 linhas, 492 fraudes). Split por linhas: ordena por Time
  (sort estável) e os primeiros 70% das linhas treinam. Treino 199.364 (384 fraudes, 0,00193), teste 85.443
  (108 fraudes, 0,00126). O split por 70% do intervalo de Time daria 172.090/112.717; o por linhas reproduz
  os números da Aula 09 (RF 0,937/0,814 na aula), por isso foi o escolhido. Features: as 30 colunas menos
  Class, com Time e Amount, como na aula. scale_pos_weight 518,18.
  RF ROC 0,9462 e PR 0,8193; XGB ROC 0,9756 e PR 0,7948: as áreas ordenam ao contrário e a PR decide (RF).
  Em 0,5: RF 1 FP e 33 FN (precisão 0,9868, recall 0,6944); XGB 11 FP e 28 FN (0,8791, 0,7407).
  XGB nos extremos: 78 de 108 fraudes com p >= 0,95 (RF 0) e 26 legítimas com p >= 0,05 (RF 84).
- [x] h) C_FP = 5, C_FN = Amount por linha, `varrer_limiares` sem alteração. Mínimo: RF em 0,15,
  R$ 2.606,48, contra R$ 4.656,73 em 0,5 (economia R$ 2.050,25, 44%); 99 alertas, recall 0,7870.
  XGB mínimo em 0,15, R$ 2.860,20 (0,5 custa R$ 3.836,53: no 0,5 o XGB pareceria melhor).
  F1 máximo do RF em 0,35 (0,8497, custo R$ 2.791,96). FN: 26 somando R$ 2.776,96 em 0,35 contra 23
  somando R$ 2.536,48 em 0,15; as 3 fraudes entre os dois valem R$ 1,00, 2,22 e 237,26 (R$ 240,48)
  contra 11 FP a mais (R$ 55). A maior fraude perdida, R$ 1.096,99, passa até em 0,05.
  Variáveis para o i): `m_esc` = 'RF', `t_esc` = 0,15, `p_esc`, `tab_esc`, `cc_te`, `yc_te`.
- [x] i) RF em 0,15. Quartis do Amount sobre todas as transações do teste (não só as fraudes), para a
  mesma faixa servir a recall e a FP por 1.000 legítimas: Q1 até R$ 5, Q2 até 20, Q3 até 72,17, Q4 até
  25.691,16. Recall 0,8148 / 0,4545 / 0,8000 / 0,8485 (54 / 11 / 10 / 33 fraudes). FP 10 / 0 / 0 / 4
  (0,46 / 0 / 0 / 0,19 por 1.000). Perda nos FN: R$ 2.384,86 de 2.536,48 no Q4 (94%).
  Hora = Time // 3600 % 24; o teste só cobre 12h a 23h (12h com 554 legítimas). 14 FP no total, 0,164 por
  1.000; 21h e 22h têm 9 deles (0,51 e 0,77) e recall 0,50. Sete horas sem FP. Contagens pequenas,
  dito no texto. I.2: origem e período, PCA (com LGPD art. 20), viés de rótulo.
  `resumo_erros(chave)` agrupa o DataFrame `erros` (fraudes, legítimas, TP, FP, perda) e põe linha total.
- [x] Verificado em 01/10/2026: recálculo de g) e h) fora do notebook bate; `predict()` igual a
  `p >= 0,5` nos dois modelos (nenhuma pontuação exatamente 0,5).
- [x] j) Parecer na seção j), 249 palavras: RF em 0,15, PR-AUC 0,8193 contra 0,7948, custo R$ 2.606,48
  contra R$ 4.656,73 em 0,5 (44%), 99 alertas com 14 falsos, recall 0,787 contra 0,694; validação temporal
  e revalidação com meses de dados do banco; monitorar recall por faixa (0,455 em R$ 5 a 20), perda acima
  de R$ 72,17 (94%), FP por hora (9 de 14 às 21h e 22h), volume e distribuição das pontuações.
- Revisão de 03/10/2026: o .venv estava com xgboost 3.4.1, fora do pin; reinstalado o 3.1.1. Reexecução
  completa (kernel pi-ia-cyber, 72 s) reproduziu todas as saídas de texto, o joblib byte a byte e o
  teste.csv.gz com o mesmo conteúdo (restaurado o arquivo original por causa do carimbo do gzip).
  PDF do enunciado conferido com PyMuPDF: sem texto oculto (cor do fundo, fonte minúscula, modo invisível,
  texto coberto, camadas, anotações ou anexos).
- [ ] relatório PDF
