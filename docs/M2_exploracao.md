# Milestone 2: Análise Exploratória e Engenharia de Atributos

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

---

## 2. Qualidade dos Dados e Limpeza

### 2.1. Identificação e Auditoria de Dados Omissos (Aula 9 — 04/03/2026)

Para garantir a integridade da análise, realizámos uma auditoria exaustiva **antes de qualquer transformação**, para detetar valores nulos reais e "nulos invisíveis" (espaços em branco lidos como texto válido pelo Pandas).

#### Metodologia da Auditoria
* **Problema detetado — "Nulos Invisíveis":** A função padrão `df.isnull().sum()` indicava **0 valores nulos** em todas as colunas. No entanto, a coluna `TotalCharges` continha espaços em branco (`" "`) que o Pandas interpretava como texto válido.
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

### 2.3. Tratamento de Outliers e Integridade dos Dados (Aula 10 — 06/03/2026)

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
| `TotalCharges` | `object` (texto) | `numérica (float64) ` | Convertida com `pd.to_numeric(..., errors='coerce')` |
| `Churn` | `object` (Yes/No) | `binária (1,0) ` | Mapeada com `.map({'Yes': 1, 'No': 0})` |
| `SeniorCitizen` | `int64` | `binária (1,0) ` | Convertida para 'Yes': 1, 'No': 0 |
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

### 4.2. Criação de Novos Atributos (Aula 11 — 11/03/2026)

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

### 4.3. Resumo dos Atributos Criados

| Atributo | Tipo | Intervalo | O que mede |
| :--- | :--- | :---: | :--- |
| `TenureCohort` | `category` (4 grupos) | *Early* → *Loyal* | Fase do ciclo de vida do cliente |
| `TotalServices` | `int64` | 0 – 8 | Grau de *lock-in* comportamental |
| `LTV_Estatico` | `float64` | 0 – 8550 € | Valor financeiro acumulado estimado |

---

## 5. Estado Final do *Dataset* (Pós-*Feature Engineering*)

```
📌 DATASET FINAL — PRONTO PARA MODELAÇÃO (M3):
   Linhas      : 7043
   Colunas     : 24  (21 originais + 3 novos atributos)
   Nulos       : 0
   Duplicados  : 0
   Ficheiro    : data/processed/telco_churn_clean.csv
```

---

## 6. Dicionário de Dados Final (24 Variáveis)

### 6.1. Variáveis Demográficas e de Identificação
| Variável | Descrição | Natureza | Tipo Técnico | Observação |
| :--- | :--- | :--- | :--- | :--- |
| **customerID** | Identificador único do cliente | Identificador | `String (ID) ` | Removido na modelação (não preditivo). |
| **gender** | Género do cliente | Categórica | `category` | Male, Female. |
| **SeniorCitizen** | Indica se o cliente tem 65 anos ou mais | Binária | `category` | 0 (Não), 1 (Sim). |
| **Partner** | Indica se o cliente tem parceiro(a) | Binária | `category` | Yes, No. |
| **Dependents** | Indica se o cliente tem dependentes | Binária | `category` | Yes, No. |

### 6.2. Variáveis de Serviços Subscritos
| Variável | Descrição | Natureza | Tipo Técnico | Observação |
| :--- | :--- | :--- | :--- | :--- |
| **PhoneService** | Se o cliente tem serviço telefónico | Binária | `category` | Yes, No. |
| **MultipleLines** | Se o cliente tem múltiplas linhas | Categórica | `category` | Yes, No, No phone service. |
| **InternetService** | Tipo de fornecedor de internet | Categórica | `category` | DSL, Fiber optic, No. |
| **OnlineSecurity** | Se tem serviço de segurança online | Categórica | `category` | Yes, No, No internet service. |
| **OnlineBackup** | Se tem serviço de backup online | Categórica | `category` | Yes, No, No internet service. |
| **DeviceProtection** | Se tem proteção de dispositivo | Categórica | `category` | Yes, No, No internet service. |
| **TechSupport** | Se tem suporte técnico prioritário | Categórica | `category` | Yes, No, No internet service. |
| **StreamingTV** | Se utiliza *streaming* de TV | Categórica | `category` | Yes, No, No internet service. |
| **StreamingMovies** | Se utiliza *streaming* de filmes | Categórica | `category` | Yes, No, No internet service. |

### 6.3. Variáveis Contratuais e Financeiras
| Variável | Descrição | Natureza | Tipo Técnico | Observação |
| :--- | :--- | :--- | :--- | :--- |
| **tenure** | Meses de permanência na empresa | Numérica (discreta) | `int64` | Intervalo: 0 a 72 meses. |
| **Contract** | Prazo do contrato do cliente | Categórica (ordinal) | `category` | Month-to-month, One year, Two year. |
| **PaperlessBilling** | Se utiliza faturação eletrónica | Binária | `category` | Yes, No. |
| **PaymentMethod** | Método de pagamento escolhido | Categórica | `category` | 4 categorias (e.g., *Electronic check*). |
| **MonthlyCharges** | Valor debitado mensalmente | Numérica (contínua) | `float64` | Valor contínuo. |
| **TotalCharges** | Valor total faturado ao cliente | Numérica (contínua) | `float64` | **Convertido de `object` para `float64`.** Imputado com `0.0` em 11 registos com `tenure=0`. |
| **Churn** | Indica se o cliente abandonou a empresa | Binária | `int64` | **Variável alvo (*Target*):** 0 (não abandona), 1 (abandona). |

### 6.4. Atributos Derivados (*Feature Engineering*)
| Variável | Descrição | Natureza | Tipo Técnico | Observação |
| :--- | :--- | :--- | :--- | :--- |
| **TenureCohort** | Fase do ciclo de vida do cliente | Categórica (ordinal) | `category` | *Early* (0–12m), *Growing* (13–24m), *Mature* (25–48m), *Loyal* (49–72m). |
| **TotalServices** | Nº de serviços adicionais subscritos | Numérica (discreta) | `int64` | Intervalo: 0 a 8. Indicador de *lock-in* comportamental. |
| **LTV_Estatico** | Valor de vida estimado do cliente | Numérica (contínua) | `float64` | `MonthlyCharges × tenure`. Intervalo: 0 a 8550 €. |

---

*Atualizado em: 11/03/2026*
