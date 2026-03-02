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

### 2.1. Tratamento de Dados em Falta (Missing Data)
* **Colunas afetadas:** `TotalCharges` (11 valores nulos encontrados).
* **Estratégia adotada:** Identificámos que os nulos ocorriam apenas em clientes com `tenure = 0`. 
* **Resolução:** Imputámos o valor **0.0** nestes registos, garantindo que o dataset final tem 7043 linhas completas.

### 2.2. Outliers e Inconsistências
* Através de boxplots, confirmámos que as variáveis numéricas não apresentam outliers que exijam remoção imediata.
* A coluna `TotalCharges` foi convertida de texto para numérico (*float64*).

## 3. Engenharia de Atributos (Feature Engineering)

### 3.1. Transformações Realizadas
* **Encoding:** A variável `Churn` foi convertida de qualitativa para binária (0 e 1).
* **Tipificação:** 16 variáveis foram convertidas para o tipo `category` para otimização de memória.
* **SeniorCitizen:** Convertida para categórica (0/1) para refletir o seu perfil qualitativo.

### 3.2. Criação de Novos Atributos
* **Estado:** *Em curso*. 

## 4. Dicionário de Dados Final (Pós-Limpeza)

| Atributo | Tipo | Descrição |
| :--- | :--- | :--- |
| `tenure` | Int64 | Meses na empresa |
| `MonthlyCharges` | Float64 | Faturação mensal |
| `TotalCharges` | Float64 | Faturação acumulada (sem nulos) |
| `Churn` | Int64 | Variável Alvo (0=Fica, 1=Sai) |
| `SeniorCitizen` | Category | Indicador de cidadão sénior |

## 5. Conclusões Visuais Importantes (Aula 27/02)
1. **Risco Temporal:** O abandono concentra-se no primeiro ano de contrato, estabilizando após os 24 meses.
2. **Vulnerabilidade Contratual:** O contrato mensal é o principal ponto de falha na retenção de clientes.
3. **Dependência Tecnológica:** Clientes com Fibra Ótica apresentam maior rotatividade do que os de DSL.

---
*Atualizado em: 02/03/2026*
