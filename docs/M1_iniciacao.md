# Milestone 1: Iniciação e Definição do Projeto

## 1. Descrição Detalhada do Problema

O setor das telecomunicações caracteriza-se por mercados saturados e elevada concorrência, o que torna a retenção de clientes um fator crítico para a sustentabilidade das empresas.

Este projeto foca-se em analisar o comportamento de consumo, contrato e faturação de clientes de uma empresa de telecomunicações para construir um modelo preditivo capaz de identificar, com antecedência, os clientes com maior probabilidade de abandonar o serviço. O objetivo final é fornecer à empresa uma ferramenta de apoio à decisão que permita priorizar intervenções de retenção de forma eficiente e direcionada.

---

## 2. Objetivos SMART

Nesta secção, definimos as metas estratégicas que orientam o desenvolvimento do projeto, alinhando a performance técnica com as necessidades de negócio.

---

### Objetivo 1: Modelação Preditiva e Comparação de Modelos

Desenvolver e comparar três modelos de classificação supervisionada — Regressão Logística, Random Forest e XGBoost — para prever o abandono de clientes (Churn), selecionando o modelo com melhor F1-Score na classe positiva (Churn = 1), com um limiar mínimo de 0,75 em validação cruzada estratificada (k=5), até ao dia 23/04/2026 (Milestone 3).

### Objetivo 2: Validação do Índice de Risco Composto

Validar que o índice de risco composto (RiskScore), construído a partir de variáveis contratuais e de serviço durante a fase de feature engineering, estratifica eficazmente os clientes em níveis de risco de abandono, confirmando que o grupo de risco mais elevado apresenta uma taxa de abandono superior a 60% e que a variável contribui significativamente para o poder preditivo do modelo final (feature importance), até ao dia 23/04/2026 (Milestone 3).

### Objetivo 3: Otimização do Recall para Retenção

Assegurar uma taxa de deteção (Recall) igual ou superior a 0,80 para a classe positiva (Churn = 1), minimizando o número de clientes em risco não identificados pelo modelo, recorrendo a técnicas de balanceamento de classes (SMOTE e/ou ajuste de class_weight) e otimização do limiar de decisão, até ao dia 23/04/2026 (Milestone 3).

*Fundamentação de negócio:* O custo de um *Falso Negativo* (não detetar um cliente que vai efetivamente sair) é drasticamente superior ao custo de um *Falso Positivo* (oferecer um incentivo de retenção a um cliente que acabaria por ficar). Por esta razão, priorizamos o Recall sobre a Precision.

---

### 2.1. Questões de Investigação

Para orientar a exploração de dados e garantir que o modelo final entrega valor de negócio, o grupo definiu as seguintes perguntas de investigação:

1. *P1 (Contratual):* Quais as variáveis contratuais e de utilização que apresentam maior associação estatística com o abandono de clientes?
2. *P2 (Ciclo de Vida):* Existe um limiar de antiguidade (tenure) abaixo do qual a probabilidade de abandono é substancialmente superior?
3. *P3 (Financeiro):* A combinação de contrato mensal com ausência de serviços de suporte técnico aumenta significativamente a probabilidade de abandono, comparativamente a clientes com contrato anual?
4. *P4 (Modelo):* Um modelo de aprendizagem supervisionada consegue identificar corretamente, pelo menos, 80% dos clientes que efetivamente abandonam o serviço?
5. *P5 (*Feature Engineering):* As novas variáveis criadas (*RiskScore, ChargesPerService) aumentam o poder preditivo do modelo face às variáveis originais do conjunto de dados?

---

## 3. Metodologia de Gestão (PBL)

* *Divisão de Tarefas:*
  * *Salvador Cabeças:* Responsável pelo setup de infraestrutura e Engenharia de Dados.
  * *Vasco Coelho:* Responsável pela Documentação Técnica e Modelação Estatística.
* *Ferramentas de Colaboração:* GitHub para controlo de versões e portefólio, Kaggle para desenvolvimento partilhado dos notebooks.

---

## 4. Análise de Viabilidade dos Dados

* *Disponibilidade:* Os dados foram importados via Kaggle e estão acessíveis no repositório. O dataset original encontra-se em data/raw/.
* *Qualidade Inicial:* Identificámos que a coluna TotalCharges estava tipificada como object devido a espaços em branco em registos com tenure = 0, o que exigiu tratamento na M2.
* *Ética:* O dataset é público, disponibilizado pela IBM no Kaggle, e está totalmente anonimizado (customerID mascarado), cumprindo as normas de proteção de dados e RGPD.

---

## 5. Cronograma Interno

| Fase | Data Limite | Entregável Esperado |
| :--- | :---: | :--- |
| M1: Iniciação | 24/02/2026 | Repositório estruturado e Plano de Projeto. |
| M2: Exploração | 24/03/2026 | Notebook de EDA e dados processados. |
| M3: Modelação | 23/04/2026 | Comparação de algoritmos e métricas. |
| M4: Finalização | Maio 2026 | Apresentação final e relatório. |

---

## 6. Dicionário de Dados (21 Variáveis Originais)

Nesta etapa de Data Understanding, realizámos uma revisão exaustiva dos tipos de dados para garantir que a estrutura técnica do dataset está alinhada com a realidade do negócio e as exigências da modelação.

*Conversão Crítica* (TotalCharges): Identificámos que esta variável estava incorretamente tipificada como texto (object). A causa raiz eram espaços em branco em clientes com tenure = 0 (novos contratos). Forçámos a conversão para float64, isolando os valores nulos para tratamento na fase de Data Preparation.

*Codificação Binária* (Churn, SeniorCitizen): Estas variáveis foram definidas como inteiros (int64). Esta transformação é vital para permitir o cálculo de correlações estatísticas e para que os algoritmos de machine learning possam processar a variável alvo (0 e 1).

*Dimensão Temporal* (tenure): Mantida como inteiro, representando a unidade discreta de meses de fidelização, essencial para análises de sobrevivência e retenção.

*Otimização Categórica:* Todas as variáveis qualitativas (ex.: tipo de contrato, serviços e método de pagamento) foram convertidas para o tipo category. Isto reduz o consumo de memória e sinaliza ao modelo a natureza discreta destas características.

> *Nota de Integridade:* Esta organização assegura que não existem "variáveis fantasma" (números lidos como texto) que possam enviesar a análise exploratória ou invalidar o treino dos modelos.

### 6.1. Variáveis Demográficas e de Identificação

| Variável | Descrição | Tipo | Observação |
| :--- | :--- | :--- | :--- |
| *customerID* | Identificador único do cliente | String (ID) | Removido na modelação (não preditivo). |
| *gender* | Género do cliente | Categórica | Male, Female. |
| *SeniorCitizen* | Indica se o cliente tem 65 anos ou mais | Binária | 0 (Não), 1 (Sim). |
| *Partner* | Indica se o cliente tem parceiro(a) | Binária | Yes, No. |
| *Dependents* | Indica se o cliente tem dependentes | Binária | Yes, No. |

### 6.2. Variáveis de Serviços Subscritos

| Variável | Descrição | Tipo | Observação |
| :--- | :--- | :--- | :--- |
| *PhoneService* | Se o cliente tem serviço telefónico | Binária | Yes, No. |
| *MultipleLines* | Se o cliente tem múltiplas linhas | Categórica | Yes, No, No phone service. |
| *InternetService* | Tipo de fornecedor de internet | Categórica | DSL, Fiber optic, No. |
| *OnlineSecurity* | Se tem serviço de segurança online | Categórica | Yes, No, No internet service. |
| *OnlineBackup* | Se tem serviço de backup online | Categórica | Yes, No, No internet service. |
| *DeviceProtection* | Se tem proteção de dispositivo | Categórica | Yes, No, No internet service. |
| *TechSupport* | Se tem suporte técnico prioritário | Categórica | Yes, No, No internet service. |
| *StreamingTV* | Se utiliza streaming de TV | Categórica | Yes, No, No internet service. |
| *StreamingMovies* | Se utiliza streaming de filmes | Categórica | Yes, No, No internet service. |

### 6.3. Variáveis Contratuais e Financeiras

| Variável | Descrição | Tipo | Observação |
| :--- | :--- | :--- | :--- |
| *tenure* | Meses de permanência na empresa | Numérica (int64) | Intervalo: 0 a 72 meses. |
| *Contract* | Prazo do contrato do cliente | Categórica | Month-to-month, One year, Two year. |
| *PaperlessBilling* | Se utiliza faturação eletrónica | Binária | Yes, No. |
| *PaymentMethod* | Método de pagamento escolhido | Categórica | 4 categorias (e.g., Electronic check). |
| *MonthlyCharges* | Valor debitado mensalmente | Numérica (float64) | Valor contínuo. |
| *TotalCharges* | Valor total faturado ao cliente | Numérica (float64) | *Convertido de object para float64.* Espaços em branco em registos com tenure = 0 tratados como nulos. |
| *Churn* | Indica se o cliente abandonou a empresa | Binária (int64) | *Variável alvo (*Target):** 0 (não abandona), 1 (abandona). |

---

### Fonte de Dados

- *Conjunto de dados:* [Telco Customer Churn — IBM/Kaggle](https://www.kaggle.com/datasets/blastchar/telco-customer-churn)
- *Dimensão:* 7043 linhas × 21 colunas

---

Data de última atualização: 25/03/2026
