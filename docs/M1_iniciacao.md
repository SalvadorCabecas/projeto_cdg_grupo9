# Milestone 1: Iniciação e Definição do Problema

## 1. Introdução e Contexto
O setor das telecomunicações é altamente competitivo. Reter um cliente custa muito menos do que adquirir um novo. Este projeto é relevante porque permite à empresa agir preventivamente sobre clientes em risco de abandono, otimizando as receitas e a satisfação do consumidor através de uma estratégia baseada em dados.

## 2. Objetivos SMART
* *S (Específico):* Prever se um cliente irá cancelar o serviço (Churn).
* *M (Mensurável):* Alcançar um *F1-Score mínimo de 0.75*.
* *A (Atingível):* O dataset possui 21 variáveis e +7000 registos, permitindo uma modelação sólida.
* *R (Relevante):* Impacto direto na estratégia de retenção e saúde financeira da empresa.
* *T (Temporal):* Concluir a modelação e avaliação até ao final da Milestone 3.

## 3. Perguntas de Investigação
1. Clientes com contratos mensais (Month-to-month) abandonam mais do que os que têm contratos anuais?
2. A falta de suporte técnico (TechSupport) está correlacionada com um churn mais elevado?
3. O valor da fatura mensal (MonthlyCharges) é o fator decisivo para a saída do cliente?

## 4. Dicionário de Dados Completo (21 Variáveis)

Após a inspeção inicial realizada no dia 18/02, identificámos as 21 variáveis que compõem o dataset "Telco Customer Churn":

### 4.1. Variáveis de Identificação e Demográficas (5)
| Variável | Descrição | Tipo |
| :--- | :--- | :--- |
| *customerID* | Identificador único do cliente. | Categórica (ID) |
| *gender* | Género do cliente (Male/Female). | Categórica |
| *SeniorCitizen* | Indica se o cliente tem 65 anos ou mais (1: Sim, 0: Não). | Categórica (Binária) |
| *Partner* | Se o cliente tem parceiro/a (Yes/No). | Categórica |
| *Dependents* | Se o cliente tem dependentes (Yes/No). | Categórica |

### 4.2. Variáveis de Serviços Subscritos (9)
| Variável | Descrição | Tipo |
| :--- | :--- | :--- |
| *PhoneService* | Se o cliente tem serviço telefónico (Yes/No). | Categórica |
| *MultipleLines* | Se tem múltiplas linhas (Yes, No, No phone service). | Categórica |
| *InternetService* | Tipo de internet (DSL, Fiber optic, No). | Categórica |
| *OnlineSecurity* | Se tem serviço de segurança online (Yes, No, No internet). | Categórica |
| *OnlineBackup* | Se tem backup online (Yes, No, No internet). | Categórica |
| *DeviceProtection*| Se tem proteção de dispositivo (Yes, No, No internet). | Categórica |
| *TechSupport* | Se tem suporte técnico (Yes, No, No internet). | Categórica |
| *StreamingTV* | Se utiliza streaming de TV (Yes, No, No internet). | Categórica |
| *StreamingMovies* | Se utiliza streaming de filmes (Yes, No, No internet). | Categórica |

### 4.3. Variáveis Contratuais e Financeiras (7)
| Variável | Descrição | Tipo / Observação |
| :--- | :--- | :--- |
| *tenure* | Meses de permanência na empresa (0 a 72). | Numérica (int64) |
| *Contract* | Tipo de contrato (Month-to-month, One year, Two year). | Categórica |
| *PaperlessBilling* | Se utiliza faturação eletrónica (Yes/No). | Categórica |
| *PaymentMethod* | Método de pagamento (4 categorias). | Categórica |
| *MonthlyCharges* | Valor da fatura mensal. | Numérica (float64) |
| *TotalCharges* | Valor total faturado. | *Erro detetado: object (requer limpeza)* |
| *Churn* | *Variável Alvo*: Indica se o cliente saiu (Yes/No). | Categórica (Target) |

## 5. Divisão de Papéis (Inicial)
* *Salvador Cabeças:* Setup de infraestrutura (GitHub/Kaggle) e Engenharia de Dados.
* *Vasco Coelho:* Escrita de Documentação Técnica e Modelação Estatística.

## 6. Ferramentas e Bibliotecas
* *Ambiente:* Kaggle Notebooks e GitHub.
* *Bibliotecas:* Pandas (limpeza), NumPy (matemática), Matplotlib/Seaborn (gráficos), Scikit-Learn (Machine Learning).

---
Última atualização: 18/02/2026
