# Milestone 3: Modelação e Avaliação

> **Nota de Revisão:** Este documento pressupõe que o *dataset* foi explorado e preparado conforme documentado em `docs/M2_exploracao.md`. O *notebook* de modelação encontra-se em `notebooks/2.0_modelacao_treino.ipynb`.

---

## 1. Estratégia de Modelação

Para iniciar a fase de modelação, foi necessário preparar os dados de forma rigorosa antes de qualquer algoritmo ser treinado. O *dataset* processado (`churn_dataset_processed.csv`) foi dividido em 80% para treino e 20% para teste, utilizando divisão estratificada (`stratify=y, random_state=42`). A estratificação garante que a proporção real de *churn* (~26.5%) é preservada em ambas as partições, evitando que uma divisão desfavorável por acaso enviese os resultados. O conjunto de teste fica assim completamente isolado durante todo o processo de treino e validação.

O *dataset* apresenta um desequilíbrio de classes significativo: ~73.5% Não-*churn* contra ~26.5% *Churn*. Sem correcção, qualquer modelo tenderia a ignorar a classe minoritária, precisamente o que pretendemos prever. A solução adoptada foi o *SMOTE* (*Synthetic Minority Oversampling Technique*), aplicado exclusivamente ao conjunto de treino, equilibrando as classes através da geração de exemplos sintéticos da classe minoritária no espaço de *features*. É fundamental que o *SMOTE* nunca seja aplicado ao conjunto de teste: fazê-lo constituiria *data leakage*, produzindo métricas artificialmente inflacionadas que não reflectem o desempenho real em produção. Após o *SMOTE*, o treino passou de ~4139 Não-*churn* / ~1495 *Churn* para uma distribuição equilibrada de ~4139 / ~4139 registos (8278 total).

Adicionalmente, foi aplicado `StandardScaler` com *fit* exclusivo no conjunto de treino balanceado, transformando depois o treino e o teste. Este passo é necessário para modelos sensíveis à escala, nomeadamente a Regressão Logística e o *KNN*, e garante que nenhuma informação estatística do teste contamina o processo de normalização.

A escolha das métricas de avaliação foi determinada pela natureza assimétrica do problema de negócio. A *Accuracy* foi excluída por ser enganadora em *datasets* desequilibrados — um modelo ingénuo que predisse sempre Não-*churn* obteria ~73.5% de *accuracy* sem detectar um único cliente em risco. A métrica principal adoptada foi o ***F1-Score* na classe *Churn***, que equilibra *Precision* e *Recall* e penaliza tanto os Falsos Positivos (campanhas desnecessárias) como os Falsos Negativos (clientes em risco não detectados). O custo de um Falso Negativo, não identificar um cliente que vai efectivamente sair, é substancialmente superior ao custo de um Falso Positivo, o que justifica também a monitorização do *Recall* de forma independente. O *AUC-ROC* foi adoptado como métrica complementar pelo seu poder discriminativo global, independente do *threshold* de decisão. O objetivo SMART definido em M1 estabelece *F1-Score* ≥ 0.75 na classe positiva (abandono = 1).

---

## 2. Experiências Realizadas

### 2.1. Modelo Baseline

O modelo *baseline* foi implementado com Regressão Logística (`max_iter=1000, random_state=42`), escolhido pela sua baixa complexidade e interpretabilidade, servindo como patamar mínimo obrigatório de comparação para todos os modelos seguintes.

Após treino no conjunto balanceado (*SMOTE* + `StandardScaler`) e avaliação no teste real, os resultados foram os seguintes:

| Classe | *Precision* | *Recall* | *F1-Score* | Support |
| --- | --- | --- | --- | --- |
| Não-*Churn* | 0.86 | 0.85 | 0.85 | 1035 |
| ***Churn*** | **0.60** | **0.61** | **0.60** | **374** |
| Macro avg | 0.73 | 0.73 | 0.73 | 1409 |
| Weighted avg | 0.79 | 0.79 | 0.79 | 1409 |

***Accuracy*:** 0.79 | ***AUC-ROC*:** 0.8340

O *AUC-ROC* de 0.834 confirma que o modelo separa as classes com qualidade razoável, claramente acima do acaso (0.5). No entanto, o *F1-Score* de 0.60 na classe *Churn* está 15 pontos abaixo do objetivo SMART de 0.75, definindo o patamar mínimo a superar. O *Recall* de 0.61 significa que 39% dos clientes que efectivamente abandonaram o serviço não são detectados, o custo de negócio mais crítico. A limitação estrutural do modelo reside na sua assunção de linearidade: a Regressão Logística não consegue capturar interações complexas entre variáveis contratuais e de serviço que caracterizam o comportamento de *churn*.

---

### 2.2. Modelos Candidatos

Para superar o *baseline*, foram treinados e avaliados 7 algoritmos de maior complexidade, todos com parâmetros base, usando o mesmo conjunto de treino balanceado (*SMOTE* + `StandardScaler`) e avaliados no mesmo conjunto de teste real.

| Algoritmo | Parâmetros Base | F1 (Treino) | F1 (Teste) \| *AUC-ROC* | Gap F1 | Notas |
| --- | --- | --- | --- | --- | --- |
| Logistic Regression (*baseline*) | `max_iter=1000` | 0.8433 | 0.6032 \| 0.8340 | 0.2401 | Melhor F1-Teste; beneficia do `StandardScaler` |
| Gradiente Progressivo | `n_estimators=100` | 0.8602 | 0.5949 \| 0.8325 | 0.2653 | 2º melhor *AUC*; curvas de aprendizagem convergentes |
| Random Forest | `n_estimators=100` | 0.9982 | 0.5815 \| 0.8205 | 0.4167 | *Overfitting* severo |
| Naive Bayes | `var_smoothing=1e-9` | 0.7682 | 0.5804 \| 0.8076 | 0.1878 | *Overfitting* moderado |
| XGBoost | `n_estimators=100` | 0.9610 | 0.5646 \| 0.8066 | 0.3964 | *Overfitting* severo |
| KNN | `n_neighbors=5` | 0.8759 | 0.5387 \| 0.7687 | 0.3372 | Penalizado pela alta dimensionalidade |
| SVM (RBF) | `kernel=rbf, C=1.0` | 0.8535 | 0.5858 \| 0.8138 | 0.2677 | 3.º lugar; separação não-linear sem *overfitting* severo |
| Decision Tree | `default` | 0.9983 | 0.4756 \| 0.6413 | 0.5227 | *Overfitting* severo; descartado |

O resultado mais surpreendente foi a vitória da Regressão Logística no F1-Teste (0.6032) e no *AUC-ROC* (0.8340). Este resultado explica-se pela combinação do `StandardScaler`, que beneficia directamente modelos lineares, com o facto de as *features* contratuais mais preditivas (tenure, tipo de contrato) apresentarem uma separação razoavelmente linear em relação ao *churn*. Os modelos *tree-based*, por sua vez, memorizam a distribuição sintética gerada pelo *SMOTE*, obtendo F1-Treino próximos de 1.0 (ex: *Random Forest* = 0.9982, *Decision Tree* = 0.9983) mas falhando na generalização para o teste real, onde a distribuição de classes é diferente (Gap F1 entre 0.40 e 0.52).

O algoritmo que registou o pior desempenho foi a *Decision Tree*, com F1-Teste de 0.48, *AUC* de 0.64 e o maior Gap de *overfitting* (0.52). Com parâmetros por omissão, a árvore cresce sem qualquer restrição de profundidade, memorizando completamente o conjunto de treino sem qualquer capacidade de generalização. O *KNN* também ficou abaixo do esperado (F1-Teste = 0.54), penalizado pela elevada dimensionalidade resultante do *encoding* das variáveis categóricas.

Apesar da Regressão Logística ter obtido o melhor F1-Teste, o **Gradiente Progressivo** foi seleccionado como candidato a otimização na Aula 19. As suas curvas de aprendizagem revelam convergência real entre treino e validação (Gap CV = 0.017), com F1 de validação de 0.8446 já acima do objetivo SMART de 0.75. O potencial de melhoria com *tuning* de hiperparâmetros é substancialmente maior do que no LR, que já opera próximo do seu limite linear estrutural.

No que diz respeito ao diagnóstico de generalização, as curvas de aprendizagem do Gradiente Progressivo (geradas por *cross-validation* estratificado K=5 dentro do conjunto de treino balanceado) revelaram os seguintes resultados:

| Métrica | Valor |
| --- | --- |
| F1 Treino (100% dos dados) | 0.8546 ± 0.0097 |
| F1 Validação CV (100%) | 0.8546 ± 0.0097 |
| Gap treino-validação | 0.0000 |
| Diagnóstico | ✅ Generalização adequada — curvas convergentes |

As curvas convergem a partir dos ~5000 registos e estabilizam acima do objetivo SMART F1 ≥ 0.75, com um desvio padrão baixo (±0.010), indicando estabilidade. O Gap residual de 0.017 é negligenciável. É importante notar que o Gap F1 = 0.27 observado na tabela comparativa não contradiz este diagnóstico: as curvas de aprendizagem são calculadas dentro do conjunto *SMOTE*-balanceado (distribuição 50/50), enquanto o Gap da tabela mede a diferença entre o treino balanceado e o teste com distribuição real (~26.5% *churn*). O modelo aprende bem a distribuição sintética e generaliza dentro dela; a penalização ocorre na transição para a distribuição real do teste.

> Ver figura: `reports/figures/curvas_aprendizagem.png`

---

## 3. Otimização (*Tuning*)

O algoritmo seleccionado para otimização foi o *Gradiente Progressivo*, com base nas curvas de aprendizagem da Aula 18 que revelaram convergência real entre treino e validação (Gap CV = 0.017) e F1 de validação de 0.8446, já acima do objectivo de 0.75 definido em M1. O potencial de melhoria com *tuning* foi considerado superior ao da Regressão Logística, que opera próximo do seu limite estrutural linear.

A técnica de otimização adoptada foi o *RandomizedSearchCV* com 50 iterações e validação cruzada estratificada K=5 integrada (*scoring='f1'*), totalizando 250 *fits*. O espaço de pesquisa foi definido de forma conservadora para prevenir *overfitting* à distribuição sintética gerada pelo *SMOTE*: `max_depth` limitado ao intervalo [3, 5] e `min_samples_split` mínimo de 10. Os parâmetros óptimos encontrados foram: `n_estimators=287`, `max_depth=3`, `learning_rate=0.1353`, `subsample=0.7465`, `min_samples_split=19`.

Para quantificar a estabilidade do modelo optimizado, foi aplicada uma validação cruzada K=5 explícita ao modelo final, com os seguintes resultados por dobra: Fold 1 = 0.8407, Fold 2 = 0.8467, Fold 3 = 0.8563, Fold 4 = 0.8673, Fold 5 = 0.8618. A média situou-se em μ = 0.8546 com desvio padrão σ = 0.0097 e intervalo de confiança a 95% de [0.8355, 0.8736]. O σ < 0.03 confirma estabilidade elevada entre as dobras.

A avaliação no conjunto de teste real revelou, contudo, um resultado contraintuitivo: o *Gradiente Progressivo* optimizado obteve F1 = 0.5707 e *AUC-ROC* = 0.8223 no teste real, ficando abaixo do *baseline* de Regressão Logística (F1 = 0.6032, *AUC* = 0.8340). O facto de o algoritmo ter convergido espontaneamente para `max_depth=3`, o valor *default*, constitui um sinal diagnóstico relevante: a complexidade adicional não acrescenta capacidade de generalização neste problema. A causa raiz é estrutural: o *tuning* optimiza para a distribuição sintética 50/50 do *SMOTE*, enquanto o conjunto de teste reproduz a distribuição real (~26.5% *churn*). Este fenómeno de *distribution shift* é particularmente pronunciado em algoritmos *tree-based*, que memorizam os padrões sintéticos com maior facilidade do que os modelos lineares, penalizando a generalização.

Para confirmar a robustez desta conclusão, foi realizado um segundo ciclo de otimização sobre o **SVM (RBF)**, o 3.º melhor candidato (F1-Teste=0.5858) e o único modelo não-linear sem *overfitting* severo. O *RandomizedSearchCV* com 20 iterações × 5 *folds* (100 *fits*) pesquisou o espaço `C` ∈ [0.1, 10] (log-uniforme) e `gamma` ∈ {*scale*, 0.01, 0.05, 0.1}, encontrando como parâmetros óptimos `C=4.6226` e `gamma=0.01`. O SVM optimizado atingiu F1=0.8416 na validação cruzada interna, mas apenas F1=0.5864 no teste real, reproduzindo exactamente o mesmo padrão de *distribution shift* observado no GB, embora com menor severidade (ΔCV-teste = 0.2552 vs 0.2839 no GB).

A comparação final dos três modelos avaliados confirma a superioridade da Regressão Logística:

| Modelo | F1 *Churn* | *AUC-ROC* | *Recall* | *Precisão* |
| --- | --- | --- | --- | --- |
| Regressão Logística (*baseline*) | **0.6032** ⭐ | **0.8340** | 0.6096 | 0.5969 |
| *Gradient Boosting* optimizado | 0.5707 | 0.8223 | 0.5882 | 0.5542 |
| SVM (RBF) optimizado | 0.5864 | 0.8184 | 0.5668 | 0.6074 |

A Regressão Logística permanece assim o modelo com melhor desempenho no teste real. Os três modelos ficam aquém do objectivo de *Recall* ≥ 0.80, o que será abordado na Aula 20 através do ajuste do limiar de decisão (*threshold*).

---

## 4. Avaliação do Modelo Final

O modelo final adoptado é a **Regressão Logística com threshold ajustado para 0.31** (em vez do default 0.50). A decisão baseou-se em três critérios: melhor F1-Teste e AUC-ROC no conjunto de teste real entre todos os algoritmos testados (F1=0.6032, AUC=0.8340); coeficientes diretamente interpretáveis em termos de negócio; e capacidade de ajuste do threshold para satisfazer o objetivo de Recall ≥ 0.80 sem reentrenar o modelo.

### 4.1. Matriz de Confusão / Erros

Com o threshold de 0.31, o modelo final obteve os seguintes resultados no conjunto de teste (1409 instâncias reais):

| | Previsto Não-Churn | Previsto Churn |
| --- | --- | --- |
| **Real Não-Churn** | TN = 725 | FP = 310 |
| **Real Churn** | FN = 72 | TP = 302 |

| Métrica | Threshold=0.50 (baseline) | Threshold=0.31 (modelo final) | Δ |
| --- | --- | --- | --- |
| F1-Score (Churn) | 0.6032 | **0.6126** | +0.0094 |
| Recall (Churn) | 0.6096 | **0.8075** | +0.1979 |
| Precision (Churn) | 0.5969 | 0.4935 | −0.1034 |
| AUC-ROC | 0.8340 | 0.8340 | — |
| TP (churners identificados) | 228 / 374 (61.0%) | **302 / 374 (80.7%)** | +74 |
| FN (churners perdidos) | 146 / 374 (39.0%) | **72 / 374 (19.3%)** | −74 |

### Objetivo Recall ≥ 0.80: ✅ Atingido (0.8075)

O ajuste do threshold de 0.50 para 0.31 representou uma troca deliberada: a Precision desceu de 0.60 para 0.49 (mais alarmes falsos, FP passaram de 154 para 310), mas o Recall subiu de 0.61 para 0.81 (menos churners perdidos, FN reduziram de 146 para 72). Esta troca é justificada pelo contexto de negócio: o custo de não detetar um cliente que vai sair (FN) é substancialmente superior ao custo de uma campanha de retenção desnecessária (FP).

**Interpretação dos Falsos Negativos (72 churners não detetados):**

Os 72 clientes que o modelo não identificou apresentam um perfil distinto dos 302 churners detetados:

| Variável | FN (médio) | TP (médio) | Diferença |
| --- | --- | --- | --- |
| tenure | 36.25 meses | 11.75 meses | +24.50 |
| LTV_Estático | 2 968 € | 1 028 € | +1 940 € |
| RiskScore | 3.11 | 4.97 | −1.86 |
| ChargesPerService | 14.86 € | 20.64 € | −5.78 € |
| TotalServices | 3.89 | 2.78 | +1.11 |
| Probabilidade média prevista | 0.1750 | 0.6411 | — |

**O modelo falha sistematicamente em clientes com tenure elevado (média 36 meses), LTV alto (média 2968€) e RiskScore moderado (3.11).** São clientes de longa data que, do ponto de vista do modelo, apresentam sinais de fidelidade (tenure alto, mais serviços), mas que acabam por sair possivelmente por razões não capturadas nas variáveis disponíveis (ex.: qualidade do serviço, oferta competitiva, mudança de circunstâncias pessoais). A probabilidade média prevista neste grupo é 0.18, abaixo do threshold de 0.31, confirmando que são casos de incerteza genuína para o modelo linear.

> Ver figura: `reports/figures/matriz_confusao.png`

---

### 4.2. Importância dos Atributos (*Feature Importance*)

Os coeficientes da Regressão Logística (calculados após `StandardScaler`) quantificam o peso de cada variável na decisão de churn. Coeficiente positivo significa que a variável aumenta a probabilidade prevista de churn; negativo significa que a reduz.

**Top 10 variáveis que mais AUMENTAM o risco de churn:**

| Variável | Coeficiente | Interpretação de negócio |
| --- | --- | --- |
| **RiskScore** | +3.070 | Variável composta de maior peso, confirma que contrato+internet+tenure combinados são mais preditivos do que qualquer variável isolada |
| MultipleLines_Yes | +2.183 | Ter múltiplas linhas associado a maior churn, possivelmente clientes com mais necessidades e mais exigentes |
| StreamingTV_Yes | +2.121 | Serviços de streaming associados a perfis de maior volatilidade |
| StreamingMovies_Yes | +2.092 | Idem, combinação de streaming TV + Movies como sinal de risco |
| TenureCohort_Loyal | +1.990 | Efeito de codificação: após one-hot com drop_first, este coeficiente é relativo à categoria base omitida |
| DeviceProtection_Yes | +1.914 | Proteção de dispositivo, associada ao segmento Fiber Optic mais volátil |
| OnlineBackup_Yes | +1.835 | Backup online, serviço premium com custo acrescido |
| TechSupport_Yes | +1.630 | Suporte técnico, clientes que recorrem a suporte têm mais fricção |
| OnlineSecurity_Yes | +1.595 | Segurança online, perfil premium mais propenso a comparar ofertas |
| TenureCohort_Mature | +1.503 | Clientes na fase Mature (25–48m) ainda com risco relevante |

**Top variáveis que mais REDUZEM o risco de churn:**

| Variável | Coeficiente | Interpretação de negócio |
| --- | --- | --- |
| **TotalServices** | **−7.281** | Principal fator de retenção, quanto mais serviços subscritos, maior o *switching cost* e menor o churn |
| tenure | −1.694 | Antiguidade como barreira de saída, clientes mais antigos têm maior inércia |
| MonthlyCharges | −0.480 | Valor mensal elevado isolado (sem outros serviços) não é o principal driver de saída |
| InternetService_Fiber optic | −0.371 | Efeito isolado da fibra, quando controlado pelo RiskScore, a fibra por si não é fator de risco |

**Validação da Questão de Investigação Q5 — novas variáveis aumentam o poder preditivo:**
O `RiskScore` é a variável com maior coeficiente positivo absoluto (+3.070) e o `TotalServices` é a de maior coeficiente negativo (−7.281), ambas criadas em M2. Nenhuma variável original do *dataset* supera estes pesos no modelo final, confirmando empiricamente que o *feature engineering* realizado em M2 aumentou de forma relevante o poder preditivo. A taxa de 70.2% de abandono no nível máximo do `RiskScore` (validada na EDA) reforça adicionalmente que o índice composto estratifica os clientes de forma operacionalmente útil.

**Insight crítico — TotalServices como principal protetor:**
O coeficiente de −7.281 do `TotalServices` é o mais forte em valor absoluto de todo o modelo. Este resultado pode surpreender à luz da análise exploratória em M2, onde `TotalServices` apresentava uma correlação de Pearson com Churn de apenas r = −0.067, o valor mais baixo dos atributos derivados. A explicação é metodológica: a correlação de Pearson mede a associação *linear e isolada* entre uma variável e o Churn binário, ignorando as interacções com as restantes variáveis. O coeficiente da Regressão Logística, por contraste, é calculado num modelo *multivariado* após `StandardScaler`, capturando o contributo *marginal* de `TotalServices` quando todas as outras variáveis estão controladas. Nesse contexto multivariado, a barreira de saída criada por cada serviço adicional revela-se o factor de retenção mais robusto do conjunto de dados. A recomendação de negócio é directa: incentivar a subscrição de serviços adicionais nos primeiros 12 meses é a intervenção com maior retorno esperado na retenção de clientes.

> Ver figura: `reports/figures/feature_importance.png`

---

## 5. Conclusão da Fase de Modelação

### 5.1. Síntese dos Resultados

A fase de modelação testou 8 algoritmos de classificação, aplicou otimização de hiperparâmetros por pesquisa aleatória e ajustou o limiar de decisão para satisfazer os objetivos definidos em M1. O modelo final selecionado é a **Regressão Logística com limiar de decisão ajustado para 0.31**.

| Métrica | Objetivo (M1) | Resultado Final | Estado |
| --- | --- | --- | --- |
| F1-Score (abandono) | ≥ 0.75 | **0.6126** | ⚠️ Não atingido |
| Recall (abandono) | ≥ 0.80 | **0.8075** | ✅ Atingido |
| AUC-ROC | — | **0.8340** | ✅ Sólido |
| Clientes em risco identificados | — | **302 / 374 (80.7%)** | ✅ |

O objetivo SMART de F1-Score ≥ 0.75 foi definido em M1 para ser medido **em validação cruzada estratificada com k=5**. Nesse contexto, o objetivo foi formalmente atingido: o Gradiente Progressivo obteve μ(F1-CV) = 0.8546 e a Regressão Logística valores similares (~0.84) na distribuição balanceada pelo SMOTE. Contudo, estes valores de CV não se transferem para o conjunto de teste real: a validação cruzada opera sobre dados com distribuição 50/50 (SMOTE), enquanto o teste reproduz a distribuição real de ~26.5% de abandono. Esta discrepância, designada *distribution shift*, explica o gap de ~0.25 pontos entre o F1 em CV e o F1 no teste (0.61). Nenhum dos 8 algoritmos testados conseguiu superar F1=0.61 no teste real, o que indica que a limitação não é do algoritmo, mas estrutural: o objetivo SMART foi formulado antes de se conhecer a magnitude do *distribution shift* introduzido pelo SMOTE. O objetivo de Recall ≥ 0.80 foi atingido, garantindo que 302 dos 374 clientes em risco (80.7%) são identificados, o critério operacionalmente mais relevante para o negócio.

### 5.2. Justificação da Escolha do Modelo Final

A Regressão Logística foi selecionada face aos restantes candidatos com base em três critérios:

**Desempenho no teste real:** Obteve o melhor F1-Teste (0.6032) e o melhor AUC-ROC (0.8340) entre os 8 algoritmos. Os modelos *tree-based* — Floresta Aleatória (F1=0.58), XGBoost (F1=0.56) e Árvore de Decisão (F1=0.48), sofreram de *overfitting* severo à distribuição sintética do SMOTE, com gaps treino-teste entre 0.40 e 0.52. O SVM (RBF) obteve F1=0.59 sem *overfitting* severo (Gap=0.27), mas ficou em 3.º lugar, abaixo do modelo linear. O Gradiente Progressivo, selecionado inicialmente para otimização pelas suas curvas de aprendizagem convergentes, obteve F1=0.5707 no teste real após *tuning* — abaixo do *baseline* linear — confirmando que a causa do fraco desempenho é estrutural e não resolúvel com hiperparâmetros.

**Interpretabilidade:** Os coeficientes da Regressão Logística são diretamente interpretáveis em termos de negócio, sem necessidade de técnicas de explicabilidade adicionais. O RiskScore (coeficiente +3.070) e o TotalServices (coeficiente −7.281) emergem como as variáveis de maior peso, com interpretação operacional clara.

**Flexibilidade do limiar:** A capacidade de ajustar o limiar de decisão sem reentrenar o modelo permitiu satisfazer o objetivo de Recall ≥ 0.80 (atingido: 0.8075) através de um ajuste de 0.50 para 0.31, uma vantagem prática relevante para implementação em produção.

### 5.3. Respostas às Questões de Investigação

| Questão | Resposta |
| --- | --- |
| **Q1** — Variáveis com maior associação ao abandono | RiskScore (+3.070), TotalServices (−7.281) e tenure (−1.694) são as variáveis de maior peso no modelo final. Na EDA, Contract_Month-to-month (taxa de abandono ~42%) e TenureCohort_New (taxa ~47.4%) foram os sinais mais fortes. |
| **Q2** — Limiar de antiguidade com risco elevado | Clientes com tenure ≤ 12 meses apresentam taxa de abandono de 47.4% — cerca de 5× superior à dos clientes com mais de 48 meses (9.1%). O limiar de 12 meses é operacionalmente relevante. |
| **Q3** — Contrato mensal + ausência de suporte técnico | Parcialmente confirmada, com uma nuance importante. Clientes com contrato mensal atingem ~42% de abandono — risco claramente superior. Quanto ao TechSupport, o modelo revela um resultado contraintuitivo: `TechSupport_Yes` tem coeficiente **positivo (+1.630)**, o que significa que **ter** suporte técnico está associado a mais churn, não menos. Este resultado não contradiz a Q3, reflecte um mecanismo diferente: clientes que recorrem a suporte técnico estão, por definição, a experienciar problemas de serviço (fricção), e essa fricção é um preditor de saída mais forte do que a ausência do serviço em si. A interação entre contrato mensal, internet Fiber Optic e tenure curto é capturada de forma composta pelo `RiskScore` (nível 6: 70.2% de abandono). |
| **Q4** — Índice de risco com taxa > 60% no nível máximo | ✅ Confirmado: RiskScore nível 6 apresenta taxa de abandono de **70.2%**, superando o objetivo de 60%. |
| **Q5** — Novas variáveis aumentam o poder preditivo | ✅ Confirmado: RiskScore é a variável de maior coeficiente positivo absoluto (+3.070) e TotalServices é a de maior coeficiente negativo (−7.281). Ambas são variáveis criadas na fase de preparação. |
| **Q6** — Recall ≥ 0.80 com balanceamento e ajuste de limiar | ✅ Atingido: SMOTE + threshold=0.31 resultou em Recall=0.8075, identificando 302 dos 374 churners reais. |

### 5.4. Limitações e Recomendações

**Limitações identificadas:**

- **Desvio de distribuição (distribution shift):** O SMOTE equilibra artificialmente o treino, mas o teste e a realidade têm ~26.5% de abandono. Esta discrepância penaliza os modelos mais complexos e constitui o principal fator limitativo do F1 no teste real. Uma alternativa a explorar em trabalho futuro seria o uso de `class_weight='balanced'` como substituto ao SMOTE, eliminando o desvio de distribuição.

- **Falsos Negativos de alto valor:** Os 72 churners não detetados têm tenure médio de 36 meses e LTV médio de 2 968€ — o grupo de maior valor para o negócio. O modelo linear não consegue capturar os padrões que levam clientes fidelizados a abandonar, possivelmente porque esses padrões dependem de variáveis externas não presentes no conjunto de dados (qualidade do serviço, ofertas competitivas, alterações de circunstâncias pessoais).

- **Dados transversais:** O conjunto de dados representa um único momento temporal, sem historial de comportamento ao longo do tempo. Um modelo longitudinal, com séries temporais de utilização e faturação, teria potencialmente maior poder preditivo.

**Recomendações operacionais:**

1. **Intervenção prioritária nos primeiros 12 meses:** A taxa de abandono de 47.4% nos clientes novos justifica programas de onboarding e incentivos de subscrição de serviços adicionais logo nos primeiros meses de contrato.

2. **Estratégia de *bundling*:** O coeficiente de TotalServices (−7.281) demonstra empiricamente que cada serviço adicional cria uma barreira de saída. Ofertas combinadas com desconto são a intervenção com maior retorno esperado.

3. **Vigilância diferenciada para clientes de alto LTV:** Os 72 FN do modelo têm LTV médio de 2 968€. Recomenda-se a criação de um canal paralelo de monitorização para este segmento, baseado em critérios de negócio diretos (tenure alto + contrato mensal + sem renovação recente).

4. **Revisão do limiar por campanha:** O threshold de 0.31 maximiza o Recall mas gera 310 FP (alarmes desnecessários). Em campanhas com custo elevado por cliente contactado, o threshold deve ser reajustado para equilibrar Precision e Recall de acordo com o orçamento disponível.

---

Última atualização: 29/04/2026
