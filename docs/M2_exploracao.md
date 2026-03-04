# Milestone 2: Análise Exploratória e Engenharia de Atributos

## 1. Análise Exploratória de Dados (EDA)

### 1.1. Distribuição da Variável Alvo
A nossa variável alvo **Churn** foi analisada e apresenta o seguinte comportamento:
* **Estado:** Desequilibrada (*Imbalanced*).
* **Valores:** ~73.5% de retenção (0) contra **26.5% de abandono (1)**.
* **Impacto:** Esta distribuição dita que o sucesso do modelo não será medido por *Accuracy*, mas sim por métricas que penalizem a perda de clientes, como o **Recall**.

### 1.2. Correlações Relevantes
Identificámos relações factuais importantes através da matriz de correlação e análise bivariada:
* **Tenure vs. Churn:** Correlação negativa de $r = -0.35$. Quanto maior o tempo de casa, menor a propensão de saída.
* **MonthlyCharges vs. Churn:** Correlação positiva de $r = 0.19$. Clientes com faturas mais altas tendem a abandonar mais a empresa.
* **Contrato vs. Churn:** O contrato "Month-to-month" é o maior preditor visual de risco, com uma taxa de abandono muito superior aos contratos anuais.

## 2. Qualidade dos Dados e Limpeza

### 2.1. Identificação e Auditoria de Dados Omissos (Missing Data)

Para garantir a integridade da análise, realizámos uma auditoria exaustiva a todas as colunas do dataset para detetar valores nulos ou inconsistentes.

* **O Problema dos "Nulos Invisíveis":** Inicialmente, a função padrão `df.isnull().sum()` indicava **0 valores nulos** em todas as colunas. No entanto, detetámos que a coluna `TotalCharges` continha espaços em branco (`" "`) que o Pandas interpretava como texto válido.
* **Função Utilizada:** Para expor estes dados, utilizámos a função `pd.to_numeric(df['TotalCharges'], errors='coerce')`. O parâmetro `errors='coerce'` foi fundamental, pois forçou a conversão de texto para número e transformou automaticamente os espaços vazios em `NaN` (*Not a Number*), permitindo a sua contabilização real através do método `.isnull().sum()`.
* **Resultado da Auditoria:**
    * **TotalCharges:** 11 valores nulos identificados (0.15% do dataset).
    * **Restantes Colunas:** 0% de nulos detetados após varrimento completo.

### 2.2. Estratégia de Tratamento (Imputação Lógica)

* **Estratégia Escolhida:** Imputação de valor constante (**0.0**).
* **Análise de Causa:** Cruzámos os 11 registos nulos com a variável `tenure` e verificámos que todos pertenciam a clientes com **0 meses de permanência**. 
* **Justificação Técnica:**
* 1.  **Lógica de Negócio:** Se o cliente tem 0 meses de contrato, a sua faturação acumulada é matematicamente zero. O valor `0.0` é o único que mantém a coerência financeira do ciclo de vida do cliente.
* 2.  **Rejeição de Alternativas:** Atribuir a média ou mediana de faturação a quem ainda não completou o primeiro mês introduziria um erro estatístico (ruído). A eliminação de linhas foi descartada para preservar a amostra de novos clientes, essencial para estudar o fenómeno do abandono precoce (*Early Churn*).
* **Resultado Final:** O tratamento garantiu a preservação das **7043 linhas** originais, tornando o dataset tecnicamente apto para a modelação.

### 2.3. Outliers e Integridade

* **Análise de Outliers:** Através da visualização de *Boxplots* para as variáveis `tenure`, `MonthlyCharges` e `TotalCharges`, confirmámos que as distribuições se encontram dentro dos limites operacionais expectáveis da indústria de telecomunicações, não apresentando valores aberrantes que exijam remoção.
* **Tipificação:** A coluna `TotalCharges` foi formalmente convertida de texto (*object*) para numérico (*float64*), permitindo cálculos estatísticos e correlações futuras.
* **Verificação de Duplicados:** Executámos uma verificação de duplicidade através da função `df.duplicated().sum()` e confirmámos que **não existem registos duplicados** no dataset, garantindo que cada entrada representa um cliente único.

## 3. Engenharia de Atributos (Feature Engineering)

### 3.1. Transformações Realizadas
* **Encoding:** A variável `Churn` foi convertida de qualitativa para binária (0 e 1).
* **Tipificação:** 16 variáveis foram convertidas para o tipo `category` para otimização de memória.
* **SeniorCitizen:** Convertida para categórica (0/1) para refletir o seu perfil qualitativo.

### 3.2. Criação de Novos Atributos
* **Estado:** *Em curso*. 

## 4. Dicionário de Dados Final (Pós-Limpeza)


### 4.1. Variáveis Demográficas e de Identificação
| Variável | Descrição | Tipo | Observação |
| :--- | :--- | :--- | :--- |
| **customerID** | Identificador único do cliente | String (ID) | Removido na modelação (não preditivo). |
| **gender** | Género do cliente | Categórica | Male, Female. |
| **SeniorCitizen** | Indica se o cliente tem 65 anos ou mais | Binária | 0 (Não), 1 (Sim). |
| **Partner** | Indica se o cliente tem parceiro(a) | Categórica | Yes, No. |
| **Dependents** | Indica se o cliente tem dependentes | Categórica | Yes, No. |

### 4.2. Variáveis de Serviços Subscritos
| Variável | Descrição | Tipo | Observação |
| :--- | :--- | :--- | :--- |
| **PhoneService** | Se o cliente tem serviço telefónico | Categórica | Yes, No. |
| **MultipleLines** | Se o cliente tem múltiplas linhas | Categórica | Yes, No, No phone service. |
| **InternetService** | Tipo de fornecedor de internet | Categórica | DSL, Fiber optic, No. |
| **OnlineSecurity** | Se tem serviço de segurança online | Categórica | Yes, No, No internet service. |
| **OnlineBackup** | Se tem serviço de backup online | Categórica | Yes, No, No internet service. |
| **DeviceProtection** | Se tem proteção de dispositivo | Categórica | Yes, No, No internet service. |
| **TechSupport** | Se tem suporte técnico prioritário | Categórica | Yes, No, No internet service. |
| **StreamingTV** | Se utiliza streaming de TV | Categórica | Yes, No, No internet service. |
| **StreamingMovies** | Se utiliza streaming de filmes | Categórica | Yes, No, No internet service. |

### 4.3. Variáveis Contratuais e Financeiras
| Variável | Descrição | Tipo | Observação |
| :--- | :--- | :--- | :--- |
| **tenure** | Meses de permanência na empresa | Numérica (int) | Intervalo: 0 a 72 meses. |
| **Contract** | Prazo do contrato do cliente | Categórica | Month-to-month, One year, Two year. |
| **PaperlessBilling** | Se utiliza faturação eletrónica | Categórica | Yes, No. |
| **PaymentMethod** | Método de pagamento escolhido | Categórica | 4 categorias (e.g., Electronic check). |
| **MonthlyCharges** | Valor debitado mensalmente | Numérica (float) | Valor contínuo. |
| **TotalCharges** | Valor total faturado ao cliente | Numérica (float) | **Convertido de object para float.** |
| **Churn** | Indica se o cliente abandonou a empresa | Binária | **Variável Alvo (Target):** 0 (não), 1 (sim). |


## 5. Conclusões Visuais Importantes (Aula 27/02)
1. **Risco Temporal:** O abandono concentra-se no primeiro ano de contrato, estabilizando após os 24 meses.
2. **Vulnerabilidade Contratual:** O contrato mensal é o principal ponto de falha na retenção de clientes.
3. **Dependência Tecnológica:** Clientes com Fibra Ótica apresentam maior rotatividade do que os de DSL.

---
*Atualizado em: 02/03/2026*
