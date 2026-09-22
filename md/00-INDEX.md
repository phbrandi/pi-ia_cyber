# Índice — Cibersegurança Aplicada com IA · INSPER 2026-2 · Unidade 3

Conversão fiel dos 5 PDFs de slides da pasta `PI/`. Um `## pNN` por slide, rastreável ao PDF original. Figuras com dado estão descritas em texto (autossuficientes) e linkadas em `assets/`.

| Arquivo | Págs | Tema | Conceitos-chave |
|---|---|---|---|
| [Aula 05 - ML em cyber.md](Aula%2005%20-%20ML%20em%20cyber.md) | 24 | Aprendizado supervisionado em tráfego de rede | features vs. identificadores · train_test_split + stratify · 3 vazamentos + Pipeline · árvore de decisão · matriz de confusão · CICIDS2017 · 5-tupla/fluxo · engenharia de features de rede · defeitos do dataset (Infinity, NaN, duplicatas, encoding) · Random Forest · **split aleatório 0,989 vs. temporal 0,955; Bot 0,96→0,16** · Roteiro 2 (itens a–l) |
| [Aula 06 - Detecção de anomalias.md](Aula%2006%20-%20Detec%C3%A7%C3%A3o%20de%20anomalias.md) | 15 | Detecção não supervisionada em logs | ⚠️ *deck se identifica como "Aula 07"* · anomalia pontual/contextual/coletiva · pipeline log→parsing→agregação→features→modelo→triagem · auth.log + regex · eventos→entidades (1 linha por IP) · **Isolation Forest** (profundidade média) · `contamination` corta a fila, não o ranking · K-Means/DBSCAN · **anomalia ≠ ataque** · lab `aula7_logs_ssh.log` |
| [Aula 07 - Prevenção a fraudes.md](Aula%2007%20-%20Preven%C3%A7%C3%A3o%20a%20fraudes.md) | 16 | Classes raras e desbalanceamento | dataset ULB (284.807 tx, 492 fraudes, **578:1**, V1–V28 por PCA) · paradoxo da acurácia (99,83% / 0 de 98) · matriz de confusão · LR base **62/98, 13 FP** · undersampling · oversampling · **SMOTE** · `class_weight="balanced"` · limiar · panorama Brasil (Abecs/Febraban/PF) · cadeia do crime · Lei 15.397/2026, PLD/AML, KYC, LGPD · atividade de canvas |
| [Aula 08 -Meu modelo é bom.md](Aula%2008%20-Meu%20modelo%20%C3%A9%20bom.md) | 17 | Avaliação de modelos | resultados das 5 estratégias (**todas 90/98, 1.386–1.655 FP**) · **3 armadilhas**: SMOTE antes do split, teste na proporção errada, SMOTE sem escalar · TP/FN/FP/TN com nomes · precisão/recall/F1/acurácia · **dois rankings que não coincidem** · treino/validação/teste · overfitting medido (RF recall treino 0,995 → teste 0,816) · **ROC é cega, PR não** |
| [Aula 09 - Custo de uma fraude.md](Aula%2009%20-%20Custo%20de%20uma%20fraude.md) | 16 | Da medida à decisão | SOC: 10.000 alertas/dia, 8 min cada, >160 analistas · **falácia da taxa-base** (99%/1% → precisão 1%) · o limiar decide, não o modelo · varredura de limiar (F1 máx 0,30) · ROC×PR discordam (RF 0,937/0,814 vs XGB 0,985/0,803) · **calibração** · matriz de custo (`FP×C_FP + FN×C_FN`) · **custo mínimo R$ 2.624 em 0,15 vs R$ 4.682 em 0,5 (+44%)** · capacidade do time · 2 limiares/3 faixas · `TimeSeriesSplit` · viés e NIST AI RMF |

## Fio condutor entre as aulas

`regras à mão (R1)` → `features + classificador (A05/R2)` → `sem rótulo: ranquear anomalias (A06)` → `classe raríssima: rebalancear (A07)` → `dar nome e medir direito (A08)` → `escolher o corte com custo e capacidade (A09)` → *Roteiro 3 / avaliação intermediária*

Tensão que se repete e cresce em todas: **a métrica boa no laboratório é inútil na operação.** Aula 05 mostra isso pelo tempo (split aleatório × temporal), Aula 07 pela raridade (acurácia), Aula 08 pela escolha da métrica (ROC × PR), Aula 09 pelo dinheiro e pela capacidade do time.

## Figuras em `assets/`

| Arquivo | De onde | O que mostra |
|---|---|---|
| `a07-p07-fronteira-acomodada.png` | A07 p07 | esquema: fronteira empurrada para a maioria, fraudes do lado errado |
| `a07-p08-undersampling.png` | A07 p08 | esquema: legítimas "apagadas" vs. "mantidas" |
| `a07-p09-smote-antes-depois.png` | A07 p09 e A08 p04 | dados reais V14×V10, antes/depois do SMOTE — interpolação em região vazia |
| `a08-p16-roc-vs-pr.png` | A08 p16 | ROC (3 curvas coladas) vs. PR (curvas separadas) — a prova visual |
| `a09-p05-histograma-limiar.png` | A09 p05 | dois sinos sobrepostos, "zona de dúvida", corte 0,5 |
| `a09-p06-varredura-limiar.png` | A09 p06 | precisão × recall × limiar, F1 máx em 0,30 |
| `a09-p08-calibracao.png` | A09 p08 | diagrama de confiabilidade: RF acima, XGB muito abaixo da diagonal |
| `a09-p10-custo-x-limiar.png` | A09 p10 | custo total × limiar, mínimos em 0,15 (RF) e 0,80 (XGB) |
| `a09-p14-timeseriessplit.png` | A09 p14 | CV aleatória (treina com o futuro) vs. janela crescente |

## Notas de conversão

- Descartado de propósito: logo, rodapé e numeração repetidos em toda página.
- Números mantidos na formatação pt-BR original do slide (`0,964`, `1.989`).
- **Divergência de numeração**: `Aula 06 - Detecção de anomalias.pdf` se identifica internamente como "AULA 07" em capa e rodapé, e a Aula 05 p05 também a antecipa como "Aula 7". Os arquivos 07, 08 e 09 batem com o próprio conteúdo.
