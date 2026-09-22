# Resumo das aulas 05 a 09 (referência para a Avaliação Intermediária)

Detalhe slide a slide em [00-INDEX.md](00-INDEX.md). Aqui fica só o que a avaliação usa.

## Fio condutor

R1 regras à mão → R2/A05 features e classificador → A06 anomalias sem rótulo → A07 classe rara → A08 medir direito → A09 escolher o corte com custo e capacidade → avaliação.
A ideia que se repete: a métrica boa no laboratório engana na operação (tempo, raridade, escolha da métrica, dinheiro e capacidade).

## Por aula, o essencial

| Aula | Ideia central | Números de referência |
|---|---|---|
| A05 ML em cyber | Features descrevem comportamento, não identidade. Três vazamentos: pré-processar antes do split, feature que é o rótulo disfarçado (`alertas_ids`), duplicatas entre treino e teste. Split temporal dentro de cada (dia, rótulo). | Acurácia 0,989 no split aleatório contra 0,955 no temporal. Bot 0,96 → 0,16 (no fim da campanha os fluxos duram 17 vezes mais). |
| A06 Anomalias | Sem rótulo, modela-se o normal e ranqueia-se o desvio. `contamination` só corta a fila, não muda o ranking. Anomalia não é ataque. | Isolation Forest: score é a profundidade média. |
| A07 Fraude | Classe rara (578:1, 0,172%). Acurácia engana. Undersampling, oversampling, SMOTE e `class_weight` só movem a fronteira; o limiar é a alavanca mais barata. | Modelo nulo: 99,83% de acurácia e 0 de 98 fraudes. LR base: 62 de 98, com 13 FP. |
| A08 Meu modelo é bom? | Reamostragem só no treino e depois do split; teste na proporção real. F1 pune o desequilíbrio, mas trata os dois erros como se custassem igual. A ROC é cega em classe rara; a PR não. | Todas as técnicas de balanceamento: 90 de 98, com 1.383 a 1.655 FP. RF: recall 0,995 no treino e 0,816 no teste (overfitting medido). |
| A09 Custo de uma fraude | O modelo dá a pontuação e o limiar é decisão de negócio. Custo = FP·C_FP + FN·C_FN. Capacidade é uma restrição separada. O número honesto é o temporal. | Taxa-base: 99% de recall com 1% de FPR dá precisão de 1%. RF 0,937/0,814 contra XGB 0,985/0,803 (ROC/PR). Em 0,5: RF com 0 FP e 32 FN; XGB com 24 FP e 25 FN. Custo mínimo do RF: R$ 2.624 em 0,15, contra R$ 4.682 em 0,5 (+44%). XGB: R$ 2.816 em 0,80. F1 máximo em 0,30. |

## Regras que o professor cobra

- O 0,5 é o default da biblioteca, não uma decisão de segurança. Um corte em região povoada troca FP por FN a cada centésimo.
- A PR é a curva do SOC: a precisão tem os alertas no denominador. A ROC usa TN + FP, e poucos FP somem nesse denominador.
- Referências do acaso: a PR-AUC do acaso é igual à prevalência; a ROC-AUC do acaso é 0,5. Quando as duas áreas ordenam os modelos de forma diferente, quem decide é a PR.
- Comparar modelos no corte 0,5 é comparar dois limiares diferentes (o `scale_pos_weight` infla as pontuações do XGB). Compare pela varredura.
- Calibração: 0,5 não quer dizer 50%. Se a pontuação for usada como probabilidade, calibre antes (`CalibratedClassifierCV`) e recalibre a cada retreino.
- Quatro limiares lado a lado: corte 0,5, F1 máximo, custo mínimo e menor custo dentro da capacidade. Alertas = TP + FP; o que excede a capacidade vira fila, e fila vira FN não contabilizado. Se custo e capacidade conflitam, a decisão é de gestão.
- Sensibilidade: repita a conta com C_FN cinco vezes maior. Se a decisão não muda, ela é robusta; se muda, quem precisa decidir o custo é o gestor.
- Dois limiares e três faixas: resposta automática (limiar escolhido pela precisão), fila do analista (a única faixa limitada pela capacidade) e só log (limiar escolhido pelo recall).
- Validação: a validação cruzada aleatória treina com o futuro e serve para escolher hiperparâmetros. O `TimeSeriesSplit` e o split temporal dão o número que se reporta. A diferença entre os dois mede o drift.
- As cinco fontes de viés, com o exemplo de cada uma nos nossos dados:

  | fonte de viés | onde aparece |
  |---|---|
  | taxa-base | 0,17% de fraude |
  | viés de rótulo | fraude não contestada fica com `Class = 0` |
  | origem de laboratório | o CICIDS vem de laboratório |
  | drift | fim da campanha do Bot; 2 dias de 2013 |
  | erro desigual | FP concentrados por hora e por faixa de valor |

  Governança: função MAP do NIST AI RMF.
- Limites do dataset de cartão: ULB/Europa, setembro de 2013, 2 dias; V1 a V28 são PCA e não são interpretáveis; rótulo vem de contestação, com atraso e incompleto; sem contexto brasileiro (Pix, LGPD, Lei 15.397/2026).

## Item da avaliação → onde está o apoio

| Item | Aula e slide |
|---|---|
| b histograma e zona de dúvida | A09 p05 |
| c ROC contra PR | A08 p16, A09 p04 e p07 |
| d varredura de limiar | A09 p06 |
| e custo e capacidade | A09 p09, p10 e p11 (e p12) |
| f validação cruzada contra temporal | A09 p13 e p14, A05 p21 |
| g RF contra XGB | A09 p07 e p08 |
| h custo com Amount | A09 p09 e p10 |
| i viés e limitações | A09 p15, A07 p03 e p14 |
| j parecer | A09 p16 |
