# *Milestone* 2: Análise Exploratória e Engenharia de Atributos

## 1. Análise Exploratória de Dados (EDA)

### 1.1. Distribuição da Variável Alvo
A nossa variável alvo ***Churn*** foi analisada e apresenta o seguinte comportamento:
* **Estado:** Desequilibrada (*Imbalanced*).
* **Valores:** ~73.5% de retenção (0) contra **26.5% de abandono (1)**.
* **Impacto:** Esta distribuição dita que o sucesso do modelo não será medido por *Accuracy*, mas sim por métricas que penalizem a perda de clientes, como o ***Recall*** e o *F1-Score*.

> **Nota Técnica:** Um modelo "preguiçoso" que previsse sempre "Não *Churn*" atingiria 73.5% de *Accuracy*, mas falharia 100% no objetivo de retenção — confirmando que *Accuracy* é uma métrica enganadora neste contexto.

### 1.2. Correlações Relevantes
Identificámos relações factuais importantes através da matriz de correlação de Pearson e da análise bivariada:

| Par de Variáveis | Correlação (r) | Interpretação |
| :--- | :---: | :--- |
| `tenure` vs `Churn` | **-0.35** | Correlação negativa mais forte — quanto maior a permanência, menor o risco de abandono. Principal fator de retenção identificado. |
| `MonthlyCharges` vs `Churn` | **+0.19** | Correlação positiva moderada — faturas mensais mais elevadas associadas a maior propensão de saída. |
| `TotalCharges` vs `tenure` | **+0.83** | Multicolinearidade elevada — selecionar apenas uma destas variáveis em modelos lineares (M3) para evitar redundância. |

### 1.3. Conclusões da Análise Bivariada (Aula 27/02)
1. **Risco Temporal:** O abandono concentra-se no **primeiro ano de contrato** ("zona de perigo": meses 0–12), estabilizando após os 24 meses.
2. **Vulnerabilidade Contratual:** O contrato mensal (*Month-to-month*) é o principal ponto de falha, com uma taxa de *Churn* de **~42%** contra menos de **5%** nos contratos anuais.
3. **Dependência Tecnológica:** Clientes com ***Fiber Optic*** apresentam maior rotatividade do que os de *DSL*, possivelmente pela agressividade competitiva neste segmento *premium*.

### 1.4. Comparação por Classe *Churn* (*Boxplots*)
* **`tenure`:** Clientes que abandonam concentram-se nos primeiros meses (mediana ≈ 10 meses vs ≈ 38 meses nos retidos) — reforça o padrão de *Early Churn*.
* **`MonthlyCharges`:** Clientes que saem pagam, em mediana, mais por mês — possivelmente associado ao segmento *Fiber Optic* (mais caro e mais volátil).
* **`TotalCharges`:** Naturalmente inferior no grupo *Churn* = 1 devido ao menor tempo de permanência.

---

## 2. Qualidade dos Dados e Limpeza

### 2.1. Identificação e Auditoria de Dados Omissos (Aula 9 — 04/03/2026)

Para garantir a integridade da análise, realizámos uma auditoria exaustiva **antes de qualquer transformação**, para detetar valores nulos reais e "nulos invisíveis" (espaços em branco lidos como texto válido pelo *Pandas*).

#### Metodologia da Auditoria
* **Problema detetado — "Nulos Invisíveis":** A função padrão `df.isnull().sum()` indicava **0 valores nulos** em todas as colunas. No entanto, a coluna `TotalCharges` continha espaços em branco (`" "`) que o *Pandas* interpretava como texto válido.
* **Técnica de Exposição:** Utilizámos `df['TotalCharges'].str.strip() == ''` para identificar os registos com espaços em branco ainda antes da conversão, seguido de `pd.to_numeric(df['TotalCharges'], errors='coerce')` para os forçar a `NaN` e contabilizá-los com `.isnull().sum()`.

#### Resultados da Auditoria Completa

| Coluna | Nulos Reais | Percentagem (%) |
| :--- | :---: | :---: |
| `TotalCharges` | **11** | **0.15%** |
| Todas as restantes (19 colunas) | 0 | 0.00% |
| **Total** | **11** | **0.15%** |

### 2.2. Estratégia de Tratamento dos Dados em Falta (Aula 9 — 04/03/2026)

#### Decisão: Imputação por Valor Constante (`0.0`)

| Critério | Avaliação |
| :--- | :--- |
| **Causa raiz** | Os 11 registos nulos pertencem **exclusivamente** a clientes com `tenure = 0` (novos contratos que ainda não geraram faturação acumulada). |
| **Lógica de negócio** | Se o cliente tem 0 meses de contrato, a faturação acumulada é matematicamente **zero**. O valor `0.0` é o único que mantém coerência financeira. |
| **Rejeição da média/mediana** | Imputar a média ou mediana de faturação a clientes que ainda não completaram o primeiro mês introduziria ruído estatístico — estes clientes teriam um `TotalCharges` artificialmente inflacionado. |
| **Rejeição da eliminação de linhas** | Eliminar os 11 registos desperdiçaria dados de *Early Churn* — a fase mais crítica para o modelo. |

#### Resultado Final da Imputação

```
✅ Validação concluída (via assert):
   Nulos em TotalCharges : 0  (esperado: 0)
   Nulos totais          : 0  (esperado: 0)
   Linhas preservadas    : 7043 / 7043
```

### 2.3. Tratamento de *Outliers* e Integridade dos Dados (Aula 10 — 06/03/2026)

#### Análise de *Outliers* pelo Método IQR

Aplicámos o método do **Intervalo Interquartil (IQR)** às 3 variáveis numéricas para detetar valores aberrantes com potencial de enviesar o modelo:

| Variável | Limite Inferior (Q1 − 1.5×IQR) | Limite Superior (Q3 + 1.5×IQR) | *Outliers* Detetados |
| :--- | :---: | :---: | :---: |
| `tenure` | -60.00 meses | 124.00 meses | **0 (0.00%)** |
| `MonthlyCharges` | -46.02 € | 171.38 € | **0 (0.00%)** |
| `TotalCharges` | -4683.52 € | 8868.67 € | **0 (0.00%)** |

**Conclusão:** As distribuições encontram-se dentro dos limites operacionais expectáveis da indústria de telecomunicações. **Nenhuma remoção de *outliers* foi necessária.**

#### Verificação e Correção de Tipos de Dados

Confirmámos que não existem "variáveis fantasma" (números lidos como texto):

| Variável | Tipo Inicial | Tipo Final | Ação |
| :--- | :--- | :--- | :--- |
| `TotalCharges` | `object` (texto) | numérica (`float64`) | Convertida com `pd.to_numeric(..., errors='coerce')` |
| `Churn` | `object` (Yes/No) | binária (1, 0) | Mapeada com `.map({'Yes': 1, 'No': 0})` |
| `SeniorCitizen` | `int64` | binária (1, 0) | Já codificada como 0/1 no *dataset* original |
| 15 variáveis categóricas | `object` | `category` | Otimização de memória e sinalização de natureza discreta |
| `tenure`, `MonthlyCharges` | Numérica (`int64`, `float64`) | Sem alteração | Tipos já corretos |

#### Verificação de Duplicados

```
df.duplicated().sum() → 0
✅ Nenhum registo duplicado — cada linha representa um cliente único.
```

---

## 3. Estado do *Dataset* (Pós-Limpeza)

```
📌 DATASET PRONTO PARA FEATURE ENGINEERING:
   Linhas      : 7043  (100% preservadas)
   Colunas     : 21
   Nulos       : 0
   Duplicados  : 0
   Tipos       : float64 (2), int64 (2), category (16), object (1 — customerID)
```

---

## 4. Engenharia de Atributos (*Feature Engineering*)

### 4.1. Transformações Realizadas
* ***Encoding* da variável alvo:** A variável `Churn` foi convertida de qualitativa (Yes/No) para binária (`int64`: 0 e 1), essencial para correlações e algoritmos de *machine learning*.
* **Tipificação categórica:** 16 variáveis qualitativas foram convertidas para o tipo `category`, reduzindo o consumo de memória e sinalizando a natureza discreta das variáveis aos algoritmos.
* **Conversão crítica:** `TotalCharges` convertida de `object` para `float64` com imputação de `0.0` nos 11 casos de `tenure = 0`.
* **Escalonamento de atributos — decisão fundamentada:** Avaliámos a necessidade de normalizar/padronizar as variáveis numéricas (`tenure`, `MonthlyCharges`, `LTV_Estatico`, `ChargesPerService`). **Decisão: não aplicado nesta fase.** Algoritmos baseados em árvores de decisão — como Random Forest, Gradient Boosting e XGBoost, todos candidatos a avaliar em M3 — são invariantes à escala. Para modelos sensíveis à escala (Regressão Logística, KNN, SVM), o `StandardScaler` será integrado no *pipeline* de treino em M3, com *fit* exclusivo no conjunto de treino, evitando *data leakage*. Manter os valores originais na fase de EDA preserva também a legibilidade durante a exploração (`tenure` em meses, `MonthlyCharges` em €).

### 4.2. Criação de Novos Atributos — Fase 1 (Aula 11 — 11/03/2026)

Criámos 3 novos atributos para enriquecer o poder preditivo do modelo, passando o *dataset* de **21 para 24 colunas**.

---

#### Atributo 1: `TenureCohort` — Segmentação por Fase do Ciclo de Vida

**Justificação:** A análise *KDE* (Secção 1.3) demonstrou que o comportamento de *Churn* não é linear ao longo do tempo — existem fases distintas com riscos muito diferentes. Segmentar o `tenure` em grupos captura esta não-linearidade de forma que um valor numérico contínuo não consegue.

| Grupo | Intervalo | Nº de Clientes | Taxa de *Churn* |
| :--- | :---: | :---: | :---: |
| *Early* | 0 – 12 meses | 2186 | **47.4%** |
| *Growing* | 13 – 24 meses | 1024 | 28.7% |
| *Mature* | 25 – 48 meses | 1594 | 20.4% |
| *Loyal* | 49 – 72 meses | 2239 | **9.5%** |

**Conclusão (resposta à P2):** Um cliente na fase *Early* tem **5× mais probabilidade de abandonar** do que um cliente *Loyal*. As campanhas de retenção devem ser concentradas nos primeiros 12 meses de contrato.

---

#### Atributo 2: `TotalServices` — Índice de *Lock-in* Comportamental

**Justificação:** Mede quantos serviços adicionais o cliente subscreve (intervalo: 0 a 8). A hipótese é que quanto mais serviços um cliente utiliza, maior o custo percebido de saída (*switching cost*), independentemente do preço de cada serviço.

> **Nota metodológica:** Este atributo **não pretende medir valor financeiro** — para isso existe o `LTV_Estatico`. O `TotalServices` mede exclusivamente o grau de integração e dependência do cliente com a empresa.

Serviços considerados: `PhoneService`, `MultipleLines`, `OnlineSecurity`, `OnlineBackup`, `DeviceProtection`, `TechSupport`, `StreamingTV`, `StreamingMovies`.

| Nº de Serviços | Nº de Clientes | Taxa de *Churn* |
| :---: | :---: | :---: |
| 0 | 80 | **43.8%** |
| 1 | 1701 | 21.1% |
| 2 | 1188 | 32.8% |
| 3 | 965 | 36.5% |
| 4 | 922 | 31.3% |
| 5 | 908 | 25.6% |
| 6 | 676 | 22.5% |
| 7 | 395 | 12.4% |
| 8 | 208 | **5.3%** |

**Conclusão:** A relação não é perfeitamente linear — clientes com 2 a 4 serviços apresentam taxas de *Churn* superiores ao esperado, possivelmente porque este grupo inclui clientes com *Fiber Optic* (segmento mais volátil identificado na Secção 1.3). A tendência geral confirma a hipótese: mais serviços → menor *Churn*.

---

#### Atributo 3: `LTV_Estatico` — Valor de Vida Estimado do Cliente

**Justificação:** Complementa o `TotalServices` ao capturar a dimensão financeira. A fórmula `MonthlyCharges × tenure` usa o encargo mensal real do cliente — que já reflete o preço de todos os serviços subscritos — multiplicado pelo tempo de permanência. Responde à questão: *"Quanto já gerou este cliente para a empresa?"*

| Estatística | Valor |
| :--- | :---: |
| Média | 2 279.58 € |
| Mediana | 1 393.60 € |
| Mínimo | 0.00 € |
| Máximo | 8 550.00 € |
| Desvio Padrão | 2 264.73 € |

**Conclusão:** A assimetria positiva (mediana muito abaixo da média) confirma que a maioria dos clientes tem um LTV baixo — clientes novos com pouca faturação acumulada. O gráfico de densidade (*KDE*) demonstra que o *Churn* se concentra precisamente neste segmento de baixo LTV, validando o atributo como preditor relevante para o M3.

---

### 4.3. Criação de Novos Atributos — Fase 2 (Aula 12 — 13/03/2026)

Criámos 2 atributos adicionais que combinam informação de múltiplas colunas, gerando conhecimento que não existe diretamente no *dataset* original. O *dataset* passou de **24 para 26 colunas**.

---

#### Atributo 4: `ChargesPerService` — Rácio de Custo por Serviço

**Justificação:** Mede a relação entre o valor pago mensalmente e o número de serviços subscritos. Um cliente que paga muito por poucos serviços encontra-se numa situação de *poor value* — a hipótese é que este desequilíbrio aumenta a propensão para o abandono.

**Fórmula:** `MonthlyCharges / (TotalServices + 1)` — o `+1` evita divisão por zero em clientes sem serviços adicionais.

| Estatística | Valor |
| :--- | :---: |
| Média | 15.41 € |
| Mediana | 13.98 € |
| Mínimo | 7.65 € |
| Máximo | 36.12 € |

**Correlação com *Churn*:** r = +0.393 (p < 0.001) — **significativa**. Clientes com custo por serviço mais elevado apresentam maior propensão para o abandono.

---

#### Atributo 5: `RiskScore` — Índice de Risco Composto

**Justificação:** Combina os 3 maiores preditores de *Churn* identificados na EDA (Secções 1.2 e 1.3) num único índice numérico de 0 (baixo risco) a 6 (risco máximo).

**Composição do índice:**

| Componente | Peso 0 | Peso 1 | Peso 2 |
| :--- | :---: | :---: | :---: |
| **Tipo de contrato** | Bienal (*Two year*) | Anual (*One year*) | Mensal (*Month-to-month*) |
| **Serviço de internet** | Sem internet | *DSL* | *Fiber Optic* |
| **Antiguidade** | > 12 meses | — | ≤ 12 meses |

**Validação empírica — Taxa de *Churn* por nível de *RiskScore*:**

| *RiskScore* | Nº de Clientes | Taxa de *Churn* |
| :---: | :---: | :---: |
| 0 | 588 | **0.9%** |
| 1 | 899 | 1.9% |
| 2 | 1143 | 7.7% |
| 3 | 1158 | 17.9% |
| 4 | 1642 | 37.4% |
| 5 | 697 | 42.3% |
| 6 | 916 | **70.2%** |

**Correlação com *Churn*:** r = +0.487 (p < 0.001) — **a correlação mais forte de todas as variáveis do *dataset***, confirmando que o índice composto captura informação preditiva superior à de qualquer variável isolada.

**Conclusão (resposta à P5):** O *RiskScore* demonstra uma progressão monotónica clara — de 0.9% no nível 0 para 70.2% no nível 6 — validando a utilidade desta variável para a modelação no M3.

---

### 4.4. Resumo dos Atributos Criados

| Atributo | Tipo | Intervalo | O que mede |
| :--- | :--- | :---: | :--- |
| `TenureCohort` | `category` (4 grupos) | *Early* → *Loyal* | Fase do ciclo de vida do cliente |
| `TotalServices` | `int64` | 0 – 8 | Grau de *lock-in* comportamental |
| `LTV_Estatico` | `float64` | 0 – 8550 € | Valor financeiro acumulado estimado |
| `ChargesPerService` | `float64` | 7.65 – 36.12 € | Rácio custo/serviço (*poor value*) |
| `RiskScore` | `int64` | 0 – 6 | Índice de risco composto de abandono |

### 4.5. Correlação das Novas Variáveis com *Churn*

| Variável | Correlação (r) | p-valor | Significância |
| :--- | :---: | :---: | :---: |
| `RiskScore` | +0.487 | < 0.001 | Significativa |
| `ChargesPerService` | +0.393 | < 0.001 | Significativa |
| `LTV_Estatico` | -0.199 | < 0.001 | Significativa |
| `TotalServices` | -0.067 | < 0.001 | Significativa |

---

## 5. Seleção de Atributos — *Feature Selection* (Aula 13 — 18/03/2026)

### 5.1. Verificação de Multicolinearidade

Identificámos pares de variáveis com correlação elevada (|r| > 0.80) que representam informação redundante:

| Par | Correlação (r) | Decisão |
| :--- | :---: | :--- |
| `TotalCharges` vs `LTV_Estatico` | **+1.000** | **Remover `TotalCharges`** — `LTV_Estatico` captura a mesma informação com nomenclatura explícita. |
| `tenure` vs `LTV_Estatico` | +0.827 | **Manter ambas** — `LTV_Estatico` combina `tenure` e `MonthlyCharges`, contendo informação adicional. |
| `MonthlyCharges` vs `TotalServices` | +0.802 | **Manter ambas** — conceitos distintos (preço pago vs número de serviços subscritos). |

### 5.2. Variáveis Removidas

| Variável | Motivo da Remoção |
| :--- | :--- |
| `customerID` | Identificador único sem valor preditivo. |
| `TotalCharges` | Redundância total (r = 1.000) com `LTV_Estatico`. A fórmula `MonthlyCharges × tenure` é equivalente a `TotalCharges` após imputação. |

---

## 6. Estado Final do *Dataset* (Pronto para Modelação)

```
📌 DATASET FINAL — PRONTO PARA MODELAÇÃO (M3):
   Linhas      : 7043
   Colunas     : 24  (21 originais + 5 novos atributos − 2 removidos)
   Nulos       : 0
   Duplicados  : 42 linhas com perfil coincidente (clientes distintos — `customerID` único em todos os casos, não são erros)
   Ficheiro    : data/processed/telco_churn_processed.csv
```

---

## 7. Dicionário de Dados Final (24 Variáveis)

### 7.1. Variáveis Demográficas
| Variável | Descrição | Natureza | Tipo Técnico | Observação |
| :--- | :--- | :--- | :--- | :--- |
| **gender** | Género do cliente | Categórica | `category` | *Male*, *Female*. |
| **SeniorCitizen** | Indica se o cliente tem 65 anos ou mais | Binária | `category` | 0 (Não), 1 (Sim). |
| **Partner** | Indica se o cliente tem parceiro(a) | Binária | `category` | *Yes*, *No*. |
| **Dependents** | Indica se o cliente tem dependentes | Binária | `category` | *Yes*, *No*. |

### 7.2. Variáveis de Serviços Subscritos
| Variável | Descrição | Natureza | Tipo Técnico | Observação |
| :--- | :--- | :--- | :--- | :--- |
| **PhoneService** | Se o cliente tem serviço telefónico | Binária | `category` | *Yes*, *No*. |
| **MultipleLines** | Se o cliente tem múltiplas linhas | Categórica | `category` | *Yes*, *No*, *No phone service*. |
| **InternetService** | Tipo de fornecedor de internet | Categórica | `category` | *DSL*, *Fiber optic*, *No*. |
| **OnlineSecurity** | Se tem serviço de segurança *online* | Categórica | `category` | *Yes*, *No*, *No internet service*. |
| **OnlineBackup** | Se tem serviço de *backup online* | Categórica | `category` | *Yes*, *No*, *No internet service*. |
| **DeviceProtection** | Se tem proteção de dispositivo | Categórica | `category` | *Yes*, *No*, *No internet service*. |
| **TechSupport** | Se tem suporte técnico prioritário | Categórica | `category` | *Yes*, *No*, *No internet service*. |
| **StreamingTV** | Se utiliza *streaming* de TV | Categórica | `category` | *Yes*, *No*, *No internet service*. |
| **StreamingMovies** | Se utiliza *streaming* de filmes | Categórica | `category` | *Yes*, *No*, *No internet service*. |

### 7.3. Variáveis Contratuais e Financeiras
| Variável | Descrição | Natureza | Tipo Técnico | Observação |
| :--- | :--- | :--- | :--- | :--- |
| **tenure** | Meses de permanência na empresa | Numérica (discreta) | `int64` | Intervalo: 0 a 72 meses. |
| **Contract** | Prazo do contrato do cliente | Categórica (ordinal) | `category` | *Month-to-month*, *One year*, *Two year*. |
| **PaperlessBilling** | Se utiliza faturação eletrónica | Binária | `category` | *Yes*, *No*. |
| **PaymentMethod** | Método de pagamento escolhido | Categórica | `category` | 4 categorias (e.g., *Electronic check*). |
| **MonthlyCharges** | Valor debitado mensalmente | Numérica (contínua) | `float64` | Valor contínuo. |

### 7.4. Variável Alvo
| Variável | Descrição | Natureza | Tipo Técnico | Observação |
| :--- | :--- | :--- | :--- | :--- |
| **Churn** | Indica se o cliente abandonou a empresa | Binária | `int64` | **Variável alvo (*Target*):** 0 (não abandona), 1 (abandona). |

### 7.5. Atributos Derivados (*Feature Engineering*)
| Variável | Descrição | Natureza | Tipo Técnico | Observação |
| :--- | :--- | :--- | :--- | :--- |
| **TenureCohort** | Fase do ciclo de vida do cliente | Categórica (ordinal) | `category` | *Early* (0–12m), *Growing* (13–24m), *Mature* (25–48m), *Loyal* (49–72m). |
| **TotalServices** | Nº de serviços adicionais subscritos | Numérica (discreta) | `int64` | Intervalo: 0 a 8. Indicador de *lock-in* comportamental. |
| **LTV_Estatico** | Valor de vida estimado do cliente | Numérica (contínua) | `float64` | `MonthlyCharges × tenure`. Intervalo: 0 a 8550 €. |
| **ChargesPerService** | Custo mensal por serviço subscrito | Numérica (contínua) | `float64` | `MonthlyCharges / (TotalServices + 1)`. Intervalo: 7.65 a 36.12 €. |
| **RiskScore** | Índice de risco composto de abandono | Numérica (discreta) | `int64` | Soma de 3 componentes (contrato + internet + antiguidade). Intervalo: 0 a 6. |

### 7.6. Variáveis Removidas (não presentes no *dataset* final)
| Variável | Motivo da Remoção |
| :--- | :--- |
| **customerID** | Identificador único sem valor preditivo. |
| **TotalCharges** | Redundância total (r = 1.000) com `LTV_Estatico`. |

---

## 8. Conclusões da Fase de Exploração

### O que aprendemos que não sabíamos no final do M1?

**1. A antiguidade (*tenure*) é o preditor mais robusto de retenção.**
No M1 identificámos as variáveis existentes, mas não sabíamos a magnitude do efeito. A análise *KDE* e bivariada revelou que os primeiros 12 meses são uma "zona de perigo" crítica — 47.4% de *churn* na fase *Early* contra 9.5% na fase *Loyal*. Esta descoberta justifica a criação do `TenureCohort` e orienta diretamente a estratégia de retenção.

**2. O tipo de contrato amplifica exponencialmente o risco.**
Contratos mensais apresentam ~42% de *churn* face a <5% nos contratos anuais — uma diferença de 8× que não era quantificada no M1. Este insight responde à P1 e é um dos componentes centrais do `RiskScore`.

**3. O `RiskScore` composto supera qualquer variável isolada.**
A correlação de +0.487 com *Churn* é a mais elevada de todo o *dataset* — superior ao `tenure` (-0.35) e ao `MonthlyCharges` (+0.19). Isto confirma que a combinação de contrato + internet + antiguidade captura informação preditiva que nenhuma variável original capturava sozinha (resposta à P5).

**4. A `TotalCharges` é redundante após o *feature engineering*.**
A correlação perfeita (r = 1.000) com o `LTV_Estatico` — que foi criado por nós — confirmou que a variável original não acrescenta informação nova. A sua remoção simplifica o modelo sem perda de poder preditivo.

**5. O desequilíbrio de classes exige estratégia de balanceamento.**
A distribuição 73.5%/26.5% confirma que *Accuracy* não pode ser a métrica de avaliação em M3. A decisão de usar SMOTE (no treino) e F1-Score como métrica principal decorre diretamente desta análise.

### Os dados são suficientes para avançar para a modelação?

**Sim.** O *dataset* final reúne as condições necessárias:
- **Dimensão:** 7043 registos — suficiente para divisão treino/teste estratificada e validação cruzada k=5.
- **Qualidade:** 0 nulos, 0 duplicados problemáticos, tipos de dados corretos.
- **Poder preditivo:** 5 novos atributos com correlações estatisticamente significativas (p < 0.001), especialmente o `RiskScore` (+0.487) e o `ChargesPerService` (+0.393).
- **Limitação conhecida:** O desequilíbrio de classes (~26.5% *churn*) será mitigado com SMOTE em M3, aplicado exclusivamente ao conjunto de treino.

---

*Última atualização: 22/04/2026*
