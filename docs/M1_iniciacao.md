# Milestone 1: Iniciação e Definição do Projeto

---

## 1. Descrição Detalhada do Problema

O setor das telecomunicações caracteriza-se por mercados saturados e elevada concorrência entre operadores, o que torna a retenção de clientes um fator determinante para a sustentabilidade das empresas. O abandono de clientes representa uma perda direta de receita recorrente e impõe custos operacionais associados à necessidade de substituição dessa base de clientes.

Este projeto tem como objetivo construir um modelo de classificação supervisionada capaz de prever o abandono de clientes com base nos seus dados contratuais, de serviços e de faturação. O modelo deverá identificar, com antecedência, os clientes com maior propensão para abandonar o serviço, fornecendo à empresa uma ferramenta de apoio à decisão que permita priorizar intervenções de retenção de forma eficiente e direcionada.

O conjunto de dados utilizado é o *Telco Customer Churn*, disponibilizado publicamente pela IBM no Kaggle, e contém registos de 7043 clientes de uma empresa de telecomunicações norte-americana, com 21 variáveis que descrevem as características contratuais, os serviços subscritos e a faturação de cada cliente.

---

## 2. Objetivo SMART

Construir um modelo de classificação supervisionada para prever o abandono de clientes, atingindo um F1-Score igual ou superior a 0.75 na classe positiva (abandono = 1) em validação cruzada estratificada com k=5, utilizando o conjunto de dados *Telco Customer Churn* da IBM, até ao dia 23/04/2026.

---

## 3. Questões de Investigação

1. Quais as variáveis contratuais e de utilização que apresentam maior associação estatística com o abandono de clientes?
2. Existe um limiar de antiguidade abaixo do qual a probabilidade de abandono é substancialmente superior?
3. A combinação de contrato mensal com ausência de serviços de suporte técnico aumenta significativamente a probabilidade de abandono, comparativamente a clientes com contrato anual?
4. Um índice de risco composto, construído a partir das variáveis contratuais e de serviço, consegue estratificar os clientes de forma a que o grupo de risco mais elevado apresente uma taxa de abandono superior a 60%?
5. As novas variáveis criadas aumentam o poder preditivo do modelo face às variáveis originais do conjunto de dados?
6. É possível atingir uma taxa de deteção igual ou superior a 0.80 para a classe de abandono, recorrendo a técnicas de balanceamento de classes e ajuste do limiar de decisão?

---

## 4. Metodologia de Gestão

**Divisão de tarefas:**

- Salvador Cabeças — infraestrutura, engenharia de dados e gestão do repositório.
- Vasco Coelho — documentação técnica e modelação estatística.

**Ferramentas de colaboração:** GitHub para controlo de versões e portefólio; Kaggle para desenvolvimento partilhado dos cadernos.

---

## 5. Análise de Viabilidade dos Dados

### 5.1. Disponibilidade e Origem

O conjunto de dados *Telco Customer Churn* foi disponibilizado publicamente pela IBM no Kaggle e pode ser acedido em: <https://www.kaggle.com/datasets/blastchar/telco-customer-churn>. Os dados foram importados diretamente para o ambiente Kaggle e a versão processada é mantida no repositório em `data/processed/`. O conjunto de dados original encontra-se em `data/raw/`.

| Característica | Valor |
| :--- | :--- |
| Fonte | IBM — disponibilizado no Kaggle |
| Dimensão original | 7043 registos × 21 variáveis |
| Variável alvo | `Churn` — binária (0 = permanece, 1 = abandona) |
| Taxa de abandono | ~26.5% (1869 clientes) |
| Período de referência | Não especificado (dados transversais) |

### 5.2. Critérios de Qualidade dos Dados

Foram utilizados os seguintes critérios para verificar a qualidade dos dados na fase de iniciação:

| Critério | Método de verificação | Conclusão |
| :--- | :--- | :--- |
| **Completude** | Contagem de valores nulos por coluna (`isnull().sum()`) | 11 valores em falta em `TotalCharges` (0.15%) — todos em clientes com `tenure = 0` |
| **Consistência de tipos** | Verificação dos tipos de dados (`dtypes`) | `TotalCharges` estava armazenada como texto (`object`), exigindo conversão para numérico |
| **Unicidade** | Contagem de registos duplicados (`duplicated().sum()`) | 0 registos duplicados — cada linha representa um cliente único |
| **Validade dos domínios** | Análise estatística descritiva e verificação de valores únicos | Todos os valores dentro dos intervalos operacionais esperados — 0 valores atípicos pelo método IQR |
| **Ética e privacidade** | Verificação da natureza dos dados | Dados públicos e totalmente anonimizados (identificador do cliente mascarado) — conformidade com o RGPD |

### 5.3. Estatísticas Iniciais das Variáveis

**Variáveis numéricas (antes de qualquer tratamento):**

| Variável | Média | Desvio Padrão | Mínimo | Máximo | Mediana |
| :--- | :---: | :---: | :---: | :---: | :---: |
| `tenure` | 32.37 | 24.56 | 0 | 72 | 29 |
| `MonthlyCharges` | 64.76 | 30.09 | 18.25 | 118.75 | 70.35 |
| `TotalCharges` | 2279.73 | 2266.79 | 0.00 | 8684.80 | 1394.55 |

**Variáveis qualitativas (frequências mais relevantes):**

| Variável | Categoria mais frequente | Frequência | Observação |
| :--- | :--- | :---: | :--- |
| `Churn` | Não abandona (0) | 73.5% | Desequilíbrio de classes |
| `Contract` | Mensal | 55.0% | Maioria sem compromisso de longo prazo |
| `InternetService` | Fibra ótica | 44.0% | Segmento predominante |
| `PaymentMethod` | Cheque eletrónico | 33.6% | Método mais comum |
| `gender` | Masculino | 50.5% | Distribuição equilibrada |
| `SeniorCitizen` | Não (0) | 83.8% | Minoria de clientes sénior |
| `Partner` | Não | 51.7% | Ligeira maioria sem parceiro |
| `Dependents` | Não | 70.0% | Maioria sem dependentes |

---

## 6. Cronograma Interno

| Fase | Data Limite | Entregável Esperado |
| :--- | :---: | :--- |
| M1 — Iniciação | 24/02/2026 | Repositório estruturado e plano de projeto |
| M2 — Exploração | 24/03/2026 | Caderno de análise exploratória e dados processados |
| M3 — Modelação | 23/04/2026 | Comparação de algoritmos, modelo final e métricas |
| M4 — Finalização | Maio 2026 | Apresentação final e relatório |

---

## 7. Dicionário de Dados (21 Variáveis Originais)

### 7.1. Variáveis Demográficas e de Identificação

| Variável | Descrição | Tipo Estatístico | Domínio |
| :--- | :--- | :--- | :--- |
| `customerID` | Identificador único do cliente | Qualitativa nominal | Cadeia alfanumérica única por cliente |
| `gender` | Género do cliente | Qualitativa nominal | Masculino, Feminino |
| `SeniorCitizen` | Indica se o cliente tem 65 anos ou mais | Qualitativa nominal (binária) | 0 (Não), 1 (Sim) |
| `Partner` | Indica se o cliente tem parceiro(a) | Qualitativa nominal (binária) | Sim, Não |
| `Dependents` | Indica se o cliente tem dependentes | Qualitativa nominal (binária) | Sim, Não |

### 7.2. Variáveis de Serviços Subscritos

| Variável | Descrição | Tipo Estatístico | Domínio |
| :--- | :--- | :--- | :--- |
| `PhoneService` | Se o cliente tem serviço telefónico | Qualitativa nominal (binária) | Sim, Não |
| `MultipleLines` | Se o cliente tem múltiplas linhas | Qualitativa nominal | Sim, Não, Sem serviço telefónico |
| `InternetService` | Tipo de fornecedor de internet | Qualitativa nominal | DSL, Fibra ótica, Não |
| `OnlineSecurity` | Se tem serviço de segurança em linha | Qualitativa nominal | Sim, Não, Sem serviço de internet |
| `OnlineBackup` | Se tem serviço de cópia de segurança em linha | Qualitativa nominal | Sim, Não, Sem serviço de internet |
| `DeviceProtection` | Se tem proteção de dispositivo | Qualitativa nominal | Sim, Não, Sem serviço de internet |
| `TechSupport` | Se tem suporte técnico prioritário | Qualitativa nominal | Sim, Não, Sem serviço de internet |
| `StreamingTV` | Se utiliza transmissão de televisão em linha | Qualitativa nominal | Sim, Não, Sem serviço de internet |
| `StreamingMovies` | Se utiliza transmissão de filmes em linha | Qualitativa nominal | Sim, Não, Sem serviço de internet |

### 7.3. Variáveis Contratuais e Financeiras

| Variável | Descrição | Tipo Estatístico | Domínio |
| :--- | :--- | :--- | :--- |
| `tenure` | Meses de permanência na empresa | Quantitativa discreta | 0 a 72 meses |
| `Contract` | Prazo do contrato do cliente | Qualitativa ordinal | Mensal, Anual, Bienal |
| `PaperlessBilling` | Se utiliza faturação eletrónica | Qualitativa nominal (binária) | Sim, Não |
| `PaymentMethod` | Método de pagamento escolhido | Qualitativa nominal | Cheque eletrónico, Cheque por correio, Transferência bancária automática, Cartão de crédito automático |
| `MonthlyCharges` | Valor debitado mensalmente | Quantitativa contínua | 18.25 € a 118.75 € |
| `TotalCharges` | Valor total faturado ao cliente | Quantitativa contínua | 0.00 € a 8684.80 € |
| `Churn` | Indica se o cliente abandonou a empresa | Qualitativa nominal (binária) | 0 (permanece), 1 (abandona) — **variável alvo** |

---

## 8. Problemas Detetados na Análise Inicial

Após carregamento do conjunto de dados, foram identificados os seguintes problemas que requereram tratamento na fase de preparação de dados (M2):

| Variável | Problema | Causa | Solução Aplicada |
| :--- | :--- | :--- | :--- |
| `TotalCharges` | Tipo incorreto: armazenada como texto (`object`) em vez de numérico | Presença de espaços em branco em 11 registos com `tenure = 0` | Conversão com `pd.to_numeric(..., errors='coerce')` e imputação com `0.0` |
| `Churn` | Valores textuais ("Yes"/"No") em vez de numéricos | Formato original do conjunto de dados | Mapeamento para binário: "Yes" → 1, "No" → 0 |
| 16 variáveis qualitativas | Tipo `object` (texto) em vez de `category` | Carregamento por omissão do Pandas | Conversão para tipo `category` para otimização de memória |
| `customerID` | Variável de identificação sem valor preditivo | Identificador único por cliente | Removida na fase de preparação para modelação |

---

## 9. Resumo dos Pontos Importantes da Fase de Iniciação

- O conjunto de dados está disponível publicamente, é de qualidade suficiente para modelação e não levanta questões éticas — adequado para os objetivos do projeto.
- O desequilíbrio de classes (73.5%/26.5%) é o principal condicionante metodológico: a exatidão global não pode ser usada como métrica de avaliação, sendo o F1-Score a métrica mais adequada.
- A variável `TotalCharges` apresentava um problema de tipagem que, sem tratamento, invalidaria qualquer análise estatística ou modelo de aprendizagem automática.
- O conjunto de dados possui 21 variáveis cobrindo as dimensões demográfica, contratual, de serviços e financeira — diversidade suficiente para construir um modelo preditivo robusto.
- Não foram identificados registos duplicados nem valores atípicos nas variáveis numéricas, o que simplifica a fase de preparação.
- A natureza do problema (identificar clientes em risco antes do abandono) exige um modelo com elevada taxa de deteção (Recall), sendo prioritário minimizar os falsos negativos face aos falsos positivos.

---

## 10. Referências

1. IBM. (s.d.). *Telco Customer Churn* [Conjunto de dados]. Kaggle. Disponível em: <https://www.kaggle.com/datasets/blastchar/telco-customer-churn>

2. Verbeke, W., Dejaeger, K., Martens, D., Hur, J., & Baesens, B. (2012). New insights into churn prediction in the telecommunication sector: A profit driven data mining approach. *European Journal of Operational Research*, 218(1), 211–229.

3. Chawla, N. V., Bowyer, K. W., Hall, L. O., & Kegelmeyer, W. P. (2002). SMOTE: Synthetic minority over-sampling technique. *Journal of Artificial Intelligence Research*, 16, 321–357.

4. Wirth, R., & Hipp, J. (2000). CRISP-DM: Towards a standard process model for data mining. In *Proceedings of the 4th International Conference on the Practical Applications of Knowledge Discovery and Data Mining* (pp. 29–39).

5. Melo, D. (2026). *Metodologia CRISP-DM* [Materiais de apoio]. ISCAC — Coimbra Business School.

---

Última atualização: 29/04/2026
