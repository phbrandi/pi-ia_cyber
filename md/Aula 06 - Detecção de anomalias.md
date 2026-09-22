---
fonte: "Aula 06 - Detecção de anomalias.pdf"
aula: 6
nota_numeracao: "ATENÇÃO — o arquivo se chama 'Aula 06', mas todo o conteúdo do deck se identifica como 'AULA 07 · DETECÇÃO DE ANOMALIAS' (capa e rodapé de todas as páginas). A Aula 05 também antecipa este conteúdo como 'Aula 7'. Provável renumeração; ao citar, use o número do arquivo e mencione a divergência."
unidade: "3 — Machine Learning aplicado à segurança"
titulo: "Como extrair padrões e anomalias de grandes volumes de logs?"
subtitulo: "Detecção não supervisionada, Isolation Forest e clustering para caçar o que ninguém rotulou"
formato: "Aula expositiva · laboratório em duplas"
paginas: 15
---

# Aula 06 (deck rotulado "Aula 07") — Detecção de anomalias

## p01 · Capa

```text
aluno@insper:~$ python detecta.py aula7_logs_ssh.log
7.914 linhas · 143 IPs de origem · 0 rótulos
>>> IsolationForest(contamination=0.02).fit(X)
top-3 mais raros: 192.0.2.66 · 203.0.113.7 · 198.51.100.9
>>>
```

## p02 · Roteiro de hoje

| # | Bloco | Conteúdo |
|---|---|---|
| 01 | Do supervisionado ao não supervisionado | O que muda quando ninguém rotulou os dados |
| 02 | Anomalia e o problema do volume | Três tipos de anomalia e por que ler o log não escala |
| 03 | Do texto ao alerta | Parsing, agregação de eventos em entidades e engenharia de features |
| 04 | Os modelos de hoje | Isolation Forest, ranking por raridade e clustering como detector |
| 05 | Laboratório | Uma semana de logs SSH sem rótulos · em duplas · itens a) a h) |

## p03 · O que muda a partir de hoje — *PONTE COM AS AULAS 5 E 6*

| Aulas 5 e 6 · supervisionado | Hoje · não supervisionado |
|---|---|
| Alguém rotulou cada exemplo: benigno ou ataque, e qual ataque | Ninguém rotulou nada. Só temos o comportamento observado |
| O modelo aprende a fronteira entre classes conhecidas | O modelo aprende o que é normal e aponta o que foge disso |
| Avaliação direta: matriz de confusão contra o rótulo | Avaliação exige investigação humana: não há gabarito |
| Limite: só detecta o que já foi visto e rotulado | Potencial: pegar o ataque que nunca foi visto antes |

## p04 · O problema é o volume — *POR QUE NÃO DÁ PARA LER*

- **~8 mil** — linhas de autenticação em um único servidor, em uma semana. É o arquivo do lab de hoje.
- **milhões** — de eventos por dia em um SOC de médio porte, somando rede, endpoints e aplicações.
- **> 4 h** — para um analista ler o arquivo do lab a 2 s por linha, sem pausa. Um servidor, uma semana.

Ler não escala. Precisamos de **representação (features)** e de um modelo que **priorize** o que merece olhos humanos.

## p05 · O que é uma anomalia? — *DEFINIÇÕES*

**Definição**: um padrão nos dados que não se conforma com a noção estabelecida de comportamento normal (Chandola et al., 2009).

| Pontual — estranho por si só | Contextual — estranho aqui | Coletiva — estranho em conjunto |
|---|---|---|
| Um evento é anômalo em qualquer contexto. | Seria normal em outro contexto, mas não neste. | Cada evento isolado parece inofensivo; o conjunto denuncia. |
| Ex.: 1.300 falhas de senha de um único IP em 25 minutos. | Ex.: login às 3h de sábado numa conta que só opera às 2h via cron. | Ex.: 2 falhas por usuário, em 48 usuários, do mesmo IP. |

## p06 · O pipeline de hoje, de ponta a ponta — *DO TEXTO AO ALERTA*

`01 Log bruto` (linhas de texto do sshd) → `02 Parsing` (regex extrai status, usuário, IP, hora) → `03 Agregação` (eventos viram entidades: 1 linha por IP) → `04 Features` (volume, taxa de falha, usuários, horário) → `05 Modelo` (Isolation Forest ranqueia por raridade) → `06 Triagem` (analista investiga o topo da fila)

A parte inteligente **não é o modelo**: é a passagem de eventos para entidades com boas features. O modelo só ordena o que a representação permitir enxergar.

## p07 · Anatomia de uma linha do auth.log — *PARSING*

```text
Aug 25 14:02:37 srv-web01 sshd[8412]: Failed password for invalid user admin from 192.0.2.66 port 41022 ssh2
```

| Quando | O quê | Quem | De onde |
|---|---|---|---|
| timestamp (**sem ano!**) e hora do dia | `Accepted` ou `Failed` · `password` ou `publickey` | usuário alvo · a flag `invalid user` importa | IP de origem: a entidade que vamos agregar |

**Regex com grupos nomeados**: `(?P<status>Accepted|Failed) ... (?P<ip>\S+)` — o notebook traz o padrão quase pronto.

## p08 · De eventos para entidades — *AGREGAÇÃO E FEATURES*

O modelo não classifica linhas de log. Ele **compara entidades**: aqui, cada IP de origem vira uma linha com o resumo do comportamento da semana.

| Grupo | Features | O que captura |
|---|---|---|
| Volume | `total_eventos` · `pico_10min` | Padrão de força bruta. |
| Resultado | `n_falhas` · `taxa_falha` | Assinatura de scanner vs. usuário. |
| Alvos | `n_usuarios` · `n_invalidos` | Varredura de muitas contas. |
| Tempo | `frac_madrugada` · `n_dias` | Anomalia contextual no horário. |

Toda feature embute uma hipótese sobre o que é normal neste servidor. **Não supervisionado não significa livre de hipóteses.**

## p09 · Isolation Forest: a intuição — *O MODELO DE HOJE*

**A pergunta invertida**: em vez de modelar o que é normal, o algoritmo pergunta *quão fácil é isolar este ponto dos demais?*

- Sorteia uma feature e um valor de corte, repetidamente, construindo uma **árvore aleatória**
- Pontos comuns, cercados de vizinhos, exigem **muitos cortes** até ficarem sozinhos
- Pontos raros ficam sozinhos com **poucos cortes**: caminho curto na árvore
- **Score = profundidade média** do ponto em centenas de árvores. Curto = anômalo

Exemplo lado a lado:
- **Ponto raro**: `corte 1 · corte 2 → isolado` ⇒ **profundidade 2**
- **Ponto comum**: `corte 1 · 2 · 3 · … · 11 → separa` ⇒ **profundidade 11**

## p10 · Score, ranking e o parâmetro `contamination` — *USANDO O MODELO*

**O que o modelo entrega**
- Um score contínuo por entidade: quanto menor, mais anômalo
- O produto real é um **ranking**: uma fila de investigação ordenada por raridade, não um veredito binário

**O que `contamination` decide**
- Apenas onde cortar a fila: com 2%, os 2% piores scores recebem a flag de anômalo
- **Não muda o ranking.** É uma decisão operacional: quantos alertas por dia o time consegue investigar?

*Em produção*: quem escolhe esse número é quem paga o custo da triagem, não o algoritmo.

## p11 · Clustering como detector de anomalias — *A ALTERNATIVA DO LAB*

| K-Means — longe do centro | DBSCAN — ruído é suspeito | A pegadinha — escondido no cluster |
|---|---|---|
| Agrupa em k grupos; pontos longe do centro do próprio grupo são candidatos a anomalia. | Agrupa por densidade e marca como ruído (`label -1`) o que não pertence a grupo nenhum. | Um atacante cujo perfil agregado se parece com um grupo grande e inofensivo se esconde dentro do cluster. |
| Exige escolher k e sofre com grupos de formas irregulares. | Ruído é uma lista natural de suspeitos, sem escolher k. | O que o diferencia pode pesar pouco na distância. |

Ruído: fácil de listar. Escondido no cluster: difícil de ver. **Nenhum método é gabarito.**

## p12 · Anomalia não é ataque.

Anomalia é **raridade estatística**. Ataque é um **julgamento sobre intenção**. O modelo entrega uma fila; quem condena ou absolve é a investigação.

- **Corolário 1** · Todo detector de anomalias produz falsos positivos: o usuário que digita mal a senha é raro e inocente.
- **Corolário 2** · Ataques deliberadamente comuns (*low and slow*) tentam não ser raros. A corrida armamentista continua.

## p13 · O laboratório de hoje — *MÃOS À OBRA*

**O cenário** — Uma semana de logs de autenticação SSH do servidor `srv-web01`, sem nenhum rótulo. Nessa semana aconteceram coisas que não deveriam ter acontecido. Encontrem.

**Arquivos**
- `aula7_logs_ssh.log` — o dataset
- `aula7_lab_deteccao_anomalias.ipynb` — o notebook guiado
- Mesma pasta, rodar célula a célula.

**Formato** — Em duplas, em sala. Três TODOs curtos de código e perguntas de interpretação ao longo do caminho. Itens a) até h).

**Entrega da aula** — Ao final: a lista de IPs classificados como ataque, com uma justificativa de uma frase para cada. Comparamos as listas na plenária.

## p14 · Como investigar um IP suspeito — *MÉTODO, NÃO RESPOSTA*

| 1 · Perfil | 2 · Linha do tempo | 3 · Contexto | 4 · Veredito |
|---|---|---|---|
| O que as features dizem? Volume, taxa de falha, quantos usuários, que horário? | Volte ao log bruto: como os eventos se distribuem? Rajada ou gotejamento? | Isso é normal para esta entidade? Um cron às 2h é normal; o mesmo login às 3h de sábado, não. | Ataque (qual?), benigno estranho, ou **não sei**. "Não sei" é resposta profissional: vira caso de monitoramento. |

O método é o entregável, não a resposta. Duas duplas podem divergir e ambas estarem certas, se a justificativa estiver.

## p15 · O que levar desta aula — *FECHAMENTO*

1. Sem rótulos, a abordagem se inverte: modelamos a normalidade e ranqueamos desvios.
2. A qualidade vem da representação: parsing e features carregam as hipóteses.
3. Isolation Forest e clustering enxergam anomalias diferentes; nenhum é gabarito.
4. Anomalia não é ataque: o modelo prioriza, o analista julga.

**Próxima aula**: e quando a classe que interessa é raríssima? Detecção de fraude e dados desbalanceados. · **Leitura**: Chio & Freeman, cap. 3 (Anomaly Detection), no acervo.
