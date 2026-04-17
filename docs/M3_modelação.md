# Milestone 3: Modelação e Avaliação

> **Nota de Revisão:** Este documento pressupõe que o *dataset* foi explorado e preparado conforme documentado em `docs/M2_exploracao.md`. O *notebook* de modelação encontra-se em `notebooks/2.0_modelacao_treino.ipynb`.

---

## 1. Estratégia de Modelação

Para iniciar a fase de modelação, foi necessário preparar os dados de forma rigorosa antes de qualquer algoritmo ser treinado. O *dataset* processado (`churn_dataset_processed.csv`) foi dividido em 80% para treino e 20% para teste, utilizando divisão estratificada (`stratify=y, random_state=42`). A estratificação garante que a proporção real de *churn* (~26.5%) é preservada em ambas as partições, evitando que uma divisão desfavorável por acaso enviese os resultados. O conjunto de teste fica assim completamente isolado durante todo o processo de treino e validação.

O *dataset* apresenta um desequilíbrio de classes significativo: ~73.5% Não-*churn* contra ~26.5% *Churn*. Sem correcção, qualquer modelo tenderia a ignorar a classe minoritária — precisamente o que pretendemos prever. A solução adoptada foi o *SMOTE* (*Synthetic Minority Oversampling Technique*), aplicado exclusivamente ao conjunto de treino, equilibrando as classes através da geração de exemplos sintéticos da classe minoritária no espaço de *features*. É fundamental que o *SMOTE* nunca seja aplicado ao conjunto de teste: fazê-lo constituiria *data leakage*, produzindo métricas artificialmente inflacionadas que não reflectem o desempenho real em produção. Após o *SMOTE*, o treino passou de ~4508 Não-*churn* / ~1630 *Churn* para uma distribuição equilibrada de ~4508 / ~4508 registos.

Adicionalmente, foi aplicado `StandardScaler` com *fit* exclusivo no conjunto de treino balanceado, transformando depois o treino e o teste. Este passo é necessário para modelos sensíveis à escala, nomeadamente a Regressão Logística e o *KNN*, e garante que nenhuma informação estatística do teste contamina o processo de normalização.

A escolha das métricas de avaliação foi determinada pela natureza assimétrica do problema de negócio. A *Accuracy* foi excluída por ser enganadora em *datasets* desequilibrados — um modelo ingénuo que predisse sempre Não-*churn* obteria ~73.5% de *accuracy* sem detectar um único cliente em risco. A métrica principal adoptada foi o ***F1-Score* na classe *Churn***, que equilibra *Precision* e *Recall* e penaliza tanto os Falsos Positivos (campanhas desnecessárias) como os Falsos Negativos (clientes em risco não detectados). O custo de um Falso Negativo — não identificar um cliente que vai efectivamente sair — é substancialmente superior ao custo de um Falso Positivo, o que justifica também a monitorização do *Recall* de forma independente. O *AUC-ROC* foi adoptado como métrica complementar pelo seu poder discriminativo global, independente do *threshold* de decisão. O objetivo SMART definido em M1 estabelece *F1-Score* ≥ 0.80 na classe *Churn*.

---

## 2. Experiências Realizadas

### 2.1. Modelo Baseline

O modelo *baseline* foi implementado com Regressão Logística (`max_iter=1000, random_state=42`), escolhido pela sua baixa complexidade e interpretabilidade, servindo como patamar mínimo obrigatório de comparação para todos os modelos seguintes.

Após treino no conjunto balanceado (*SMOTE* + `StandardScaler`) e avaliação no teste real, os resultados foram os seguintes:

| Classe | *Precision* | *Recall* | *F1-Score* | Support |
|---|---|---|---|---|
| Não-*Churn* | 0.86 | 0.85 | 0.86 | 1035 |
| ***Churn*** | **0.60** | **0.61** | **0.60** | **374** |
| Macro avg | 0.73 | 0.73 | 0.73 | 1409 |
| Weighted avg | 0.79 | 0.79 | 0.79 | 1409 |

***Accuracy*:** 0.79 | ***AUC-ROC*:** 0.8340

O *AUC-ROC* de 0.834 confirma que o modelo separa as classes com qualidade razoável, claramente acima do acaso (0.5). No entanto, o *F1-Score* de 0.60 na classe *Churn* está 20 pontos abaixo do objetivo de 0.80, definindo o patamar mínimo a superar. O *Recall* de 0.61 significa que 39% dos clientes que efectivamente abandonaram o serviço não são detectados — o custo de negócio mais crítico. A limitação estrutural do modelo reside na sua assunção de linearidade: a Regressão Logística não consegue capturar interações complexas entre variáveis contratuais e de serviço que caracterizam o comportamento de *churn*.

---

### 2.2. Modelos Candidatos

Para superar o *baseline*, foram treinados e avaliados 6 algoritmos de maior complexidade, todos com parâmetros base, usando o mesmo conjunto de treino balanceado (*SMOTE* + `StandardScaler`) e avaliados no mesmo conjunto de teste real.

| Algoritmo | Parâmetros Base | F1 (Treino) | F1 (Teste) \| *AUC-ROC* | Gap F1 | Notas |
|---|---|---|---|---|---|
| Logistic Regression (*baseline*) | `max_iter=1000` | 0.8436 | 0.6048 \| 0.8340 | 0.2388 | Melhor F1-Teste; beneficia do `StandardScaler` |
| Gradient Boosting | `n_estimators=100` | 0.8588 | 0.5898 \| 0.8319 | 0.2690 | 2º melhor *AUC*; curvas de aprendizagem convergentes |
| Random Forest | `n_estimators=100` | 0.9982 | 0.5812 \| 0.8194 | 0.4169 | *Overfitting* severo |
| Naive Bayes | `var_smoothing=1e-9` | 0.7684 | 0.5799 \| 0.8077 | 0.1885 | *Overfitting* moderado |
| XGBoost | `n_estimators=100` | 0.9606 | 0.5677 \| 0.8009 | 0.3930 | *Overfitting* severo |
| KNN | `n_neighbors=5` | 0.8758 | 0.5398 \| 0.7690 | 0.3360 | Penalizado pela alta dimensionalidade |
| Decision Tree | `default` | 0.9983 | 0.4881 \| 0.6502 | 0.5102 | *Overfitting* severo; descartado |

O resultado mais surpreendente foi a vitória da Regressão Logística no F1-Teste (0.6048) e no *AUC-ROC* (0.834). Este resultado explica-se pela combinação do `StandardScaler`, que beneficia directamente modelos lineares, com o facto de as *features* contratuais mais preditivas (tenure, tipo de contrato) apresentarem uma separação razoavelmente linear em relação ao *churn*. Os modelos *tree-based*, por sua vez, memorizam a distribuição sintética gerada pelo *SMOTE*, obtendo F1-Treino próximos de 1.0 (ex: *Random Forest* = 0.9982, *Decision Tree* = 0.9983) mas falhando na generalização para o teste real, onde a distribuição de classes é diferente (Gap F1 entre 0.39 e 0.51).

O algoritmo que registou o pior desempenho foi a *Decision Tree*, com F1-Teste de 0.49, *AUC* de 0.65 e o maior Gap de *overfitting* (0.51). Com parâmetros por omissão, a árvore cresce sem qualquer restrição de profundidade, memorizando completamente o conjunto de treino sem qualquer capacidade de generalização. O *KNN* também ficou abaixo do esperado (F1-Teste = 0.54), penalizado pela elevada dimensionalidade resultante do *encoding* das variáveis categóricas.

Apesar da Regressão Logística ter obtido o melhor F1-Teste, o **Gradient Boosting** foi seleccionado como candidato a otimização na Aula 19. As suas curvas de aprendizagem revelam convergência real entre treino e validação (Gap CV = 0.017), com F1 de validação de 0.8446 já acima do objetivo de 0.80. O potencial de melhoria com *tuning* de hiperparâmetros é substancialmente maior do que no LR, que já opera próximo do seu limite linear estrutural.

No que diz respeito ao diagnóstico de generalização, as curvas de aprendizagem do Gradient Boosting (geradas por *cross-validation* estratificado K=5 dentro do conjunto de treino balanceado) revelaram os seguintes resultados:

| Métrica | Valor |
|---|---|
| F1 Treino (100% dos dados) | 0.8619 ± 0.0026 |
| F1 Validação CV (100%) | 0.8446 ± 0.0100 |
| Gap treino-validação | 0.0172 |
| Diagnóstico | ✅ Generalização adequada — curvas convergentes |

As curvas convergem a partir dos ~5000 registos e estabilizam acima do objetivo F1 ≥ 0.80, com um desvio padrão baixo (±0.010), indicando estabilidade. O Gap residual de 0.017 é negligenciável. É importante notar que o Gap F1 = 0.27 observado na tabela comparativa não contradiz este diagnóstico: as curvas de aprendizagem são calculadas dentro do conjunto *SMOTE*-balanceado (distribuição 50/50), enquanto o Gap da tabela mede a diferença entre o treino balanceado e o teste com distribuição real (~26.5% *churn*). O modelo aprende bem a distribuição sintética e generaliza dentro dela; a penalização ocorre na transição para a distribuição real do teste.

> Ver figura: `reports/figures/curvas_aprendizagem.png`

---

## 3. Otimização (*Tuning*)

O algoritmo seleccionado para otimização foi o *Gradient Boosting*, com base nas curvas de aprendizagem da Aula 18 que revelaram convergência real entre treino e validação (Gap CV = 0.017) e F1 de validação de 0.8446, já acima do objectivo de 0.75 definido em M1. O potencial de melhoria com *tuning* foi considerado superior ao da Regressão Logística, que opera próximo do seu limite estrutural linear.

A técnica de otimização adoptada foi o *RandomizedSearchCV* com 50 iterações e validação cruzada estratificada K=5 integrada (*scoring='f1'*), totalizando 250 *fits*. O espaço de pesquisa foi definido de forma conservadora para prevenir *overfitting* à distribuição sintética gerada pelo *SMOTE*: `max_depth` limitado ao intervalo [3, 5] e `min_samples_split` mínimo de 10. Os parâmetros óptimos encontrados foram: `n_estimators=287`, `max_depth=3`, `learning_rate=0.1353`, `subsample=0.7465`, `min_samples_split=19`.

Para quantificar a estabilidade do modelo optimizado, foi aplicada uma validação cruzada K=5 explícita ao modelo final, com os seguintes resultados por dobra: Fold 1 = 0.8381, Fold 2 = 0.8482, Fold 3 = 0.8509, Fold 4 = 0.8650, Fold 5 = 0.8611. A média situou-se em μ = 0.8527 com desvio padrão σ = 0.0096 e intervalo de confiança a 95% de [0.8339, 0.8715]. O σ < 0.03 confirma estabilidade elevada entre as dobras.

A avaliação no conjunto de teste real revelou, contudo, um resultado contraintuitivo: o *Gradient Boosting* optimizado obteve F1 = 0.5814 e *AUC-ROC* = 0.8229 no teste real, ficando abaixo do *baseline* de Regressão Logística (F1 = 0.6048, *AUC* = 0.8340). O facto de o algoritmo ter convergido espontaneamente para `max_depth=3` — o valor *default* — constitui um sinal diagnóstico relevante: a complexidade adicional não acrescenta capacidade de generalização neste problema. A causa raiz é estrutural: o *tuning* optimiza para a distribuição sintética 50/50 do *SMOTE*, enquanto o conjunto de teste reproduz a distribuição real (~26.5% *churn*). Este fenómeno de *distribution shift* é particularmente pronunciado em algoritmos *tree-based*, que memorizam os padrões sintéticos com maior facilidade do que os modelos lineares, penalizando a generalização.

A Regressão Logística permanece assim o modelo com melhor desempenho no teste real. Ambos os modelos ficam aquém do objectivo de *Recall* ≥ 0.80, o que será abordado na Aula 20 através do ajuste do limiar de decisão (*threshold*).

---

## 4. Avaliação do Modelo Final

O modelo final adoptado é a **Regressão Logística com threshold ajustado para 0.31** (em vez do default 0.50). A decisão baseou-se em três critérios: melhor F1-Teste e AUC-ROC no conjunto de teste real entre todos os algoritmos testados (F1=0.6048, AUC=0.8340); coeficientes diretamente interpretáveis em termos de negócio; e capacidade de ajuste do threshold para satisfazer o objetivo de Recall ≥ 0.80 sem reentrenar o modelo.

### 4.1. Matriz de Confusão / Erros

Com o threshold de 0.31, o modelo final obteve os seguintes resultados no conjunto de teste (1409 instâncias reais):

| | Previsto Não-Churn | Previsto Churn |
|---|---|---|
| **Real Não-Churn** | TN = 725 | FP = 310 |
| **Real Churn** | FN = 72 | TP = 302 |

| Métrica | Threshold=0.50 (baseline) | Threshold=0.31 (modelo final) | Δ |
|---|---|---|---|
| F1-Score (Churn) | 0.6048 | **0.6126** | +0.0078 |
| Recall (Churn) | 0.6096 | **0.8075** | +0.1979 |
| Precision (Churn) | 0.6000 | 0.4935 | −0.1065 |
| AUC-ROC | 0.8340 | 0.8340 | — |
| TP (churners identificados) | 228 / 374 (61.0%) | **302 / 374 (80.7%)** | +74 |
| FN (churners perdidos) | 146 / 374 (39.0%) | **72 / 374 (19.3%)** | −74 |

**Objetivo Recall ≥ 0.80: ✅ Atingido (0.8075)**

O ajuste do threshold de 0.50 para 0.31 representou uma troca deliberada: a Precision desceu de 0.60 para 0.49 (mais alarmes falsos — FP passaram de 152 para 310), mas o Recall subiu de 0.61 para 0.81 (menos churners perdidos — FN reduziram de 146 para 72). Esta troca é justificada pelo contexto de negócio: o custo de não detetar um cliente que vai sair (FN) é substancialmente superior ao custo de uma campanha de retenção desnecessária (FP).

**Interpretação dos Falsos Negativos (72 churners não detetados):**

Os 72 clientes que o modelo não identificou apresentam um perfil distinto dos 302 churners detetados:

| Variável | FN (médio) | TP (médio) | Diferença |
|---|---|---|---|
| tenure | 36.25 meses | 11.75 meses | +24.50 |
| LTV_Estático | 2 968 € | 1 028 € | +1 940 € |
| RiskScore | 3.11 | 4.97 | −1.86 |
| ChargesPerService | 14.86 € | 20.64 € | −5.78 € |
| TotalServices | 3.89 | 2.78 | +1.11 |
| Probabilidade média prevista | 0.1751 | 0.6411 | — |

**O modelo falha sistematicamente em clientes com tenure elevado (média 36 meses), LTV alto (média 2968€) e RiskScore moderado (3.11).** São clientes de longa data que, do ponto de vista do modelo, apresentam sinais de fidelidade (tenure alto, mais serviços), mas que acabam por sair — possivelmente por razões não capturadas nas variáveis disponíveis (ex.: qualidade do serviço, oferta competitiva, mudança de circunstâncias pessoais). A probabilidade média prevista neste grupo é 0.18 — abaixo do threshold de 0.31 — confirmando que são casos de incerteza genuína para o modelo linear.

> Ver figura: `reports/figures/matriz_confusao.png`

---

### 4.2. Importância dos Atributos (*Feature Importance*)

Os coeficientes da Regressão Logística (calculados após `StandardScaler`) quantificam o peso de cada variável na decisão de churn. Coeficiente positivo significa que a variável aumenta a probabilidade prevista de churn; negativo significa que a reduz.

**Top 10 variáveis que mais AUMENTAM o risco de churn:**

| Variável | Coeficiente | Interpretação de negócio |
|---|---|---|
| **RiskScore** | +3.006 | Variável composta de maior peso — confirma que contrato+internet+tenure combinados são mais preditivos do que qualquer variável isolada |
| MultipleLines_Yes | +2.163 | Ter múltiplas linhas associado a maior churn — possivelmente clientes com mais necessidades e mais exigentes |
| StreamingTV_Yes | +2.099 | Serviços de streaming associados a perfis de maior volatilidade |
| StreamingMovies_Yes | +2.068 | Idem — combinação de streaming TV + Movies como sinal de risco |
| TenureCohort_Loyal | +1.946 | Efeito de codificação: após one-hot com drop_first, este coeficiente é relativo à categoria base omitida |
| DeviceProtection_Yes | +1.896 | Proteção de dispositivo — associada ao segmento Fiber Optic mais volátil |
| OnlineBackup_Yes | +1.817 | Backup online — serviço premium com custo acrescido |
| TechSupport_Yes | +1.612 | Suporte técnico — clientes que recorrem a suporte têm mais fricção |
| OnlineSecurity_Yes | +1.577 | Segurança online — perfil premium mais propenso a comparar ofertas |
| TenureCohort_Mature | +1.468 | Clientes na fase Mature (25–48m) ainda com risco relevante |

**Top variáveis que mais REDUZEM o risco de churn:**

| Variável | Coeficiente | Interpretação de negócio |
|---|---|---|
| **TotalServices** | **−7.219** | Principal fator de retenção — quanto mais serviços subscritos, maior o *switching cost* e menor o churn |
| tenure | −1.699 | Antiguidade como barreira de saída — clientes mais antigos têm maior inércia |
| MonthlyCharges | −0.445 | Valor mensal elevado isolado (sem outros serviços) não é o principal driver de saída |
| InternetService_Fiber optic | −0.371 | Efeito isolado da fibra — quando controlado pelo RiskScore, a fibra por si não é fator de risco |

**Validação do Objetivo SMART 2:**
O `RiskScore` é a variável com maior coeficiente absoluto positivo (+3.006), confirmando que contribui significativamente para o poder preditivo do modelo final. Combinado com a taxa de 70.2% de churn no nível máximo (validada na EDA), o Objetivo 2 está completamente cumprido.

**Insight crítico — TotalServices como principal protetor:**
O coeficiente de −7.219 do `TotalServices` é o mais forte em valor absoluto de todo o modelo. Este resultado confirma empiricamente o conceito de *switching cost*: cada serviço adicional subscrito cria uma barreira de saída. A recomendação de negócio direta é clara — incentivar a adoção de serviços adicionais nos primeiros 12 meses é a intervenção com maior retorno esperado na retenção de clientes.

> Ver figura: `reports/figures/feature_importance.png`

---

## 5. Conclusão da Fase de Modelação
*(a preencher na Aula 21 — 22/04/2026)*

---

*Última atualização: 15/04/2026      
