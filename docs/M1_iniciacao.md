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

## 4. Dicionário de Dados e Descrição de Variáveis
Após a inspeção inicial realizada via Kaggle (18/02), identificámos a seguinte estrutura:

### 4.1. Variáveis Categóricas
| Variável | Descrição | Observações |
| :--- | :--- | :--- |
| *customerID* | ID único do cliente | Identificador alfanumérico. |
| *gender* | Género | Male / Female. |
| *SeniorCitizen* | Idoso | Binário (1: Sim, 0: Não). |
| *Partner / Dependents* | Relações familiares | Se tem parceiro ou dependentes (Yes/No). |
| *Serviços* | Phone, Internet, Security, etc. | Vários campos Yes/No/No service. |
| *Contract* | Tipo de contrato | Month-to-month, One year, Two year. |
| *PaymentMethod* | Método de pagamento | Cartão, Transferência, Electronic check, etc. |
| *Churn* | *Variável Alvo* | Indica se o cliente saiu (Yes/No). |

### 4.2. Variáveis Numéricas
| Variável | Descrição | Estatísticas Iniciais |
| :--- | :--- | :--- |
| *tenure* | Meses na empresa | Média: ~32 meses; Máximo: 72 meses. |
| *MonthlyCharges* | Fatura mensal | Média: 64.76; Máximo: 118.75. |
| *TotalCharges* | Fatura total | *Anomalia:* Tipo object. Requer limpeza de espaços vazios. |

## 5. Divisão de Papéis (Inicial)
* *Salvador Cabeças:* Setup de infraestrutura (GitHub/Kaggle) e Engenharia de Dados.
* *Vasco Coelho:* Escrita de Documentação Técnica e Modelação Estatística.

## 6. Ferramentas e Bibliotecas
* *Ambiente:* Kaggle Notebooks e GitHub.
* *Bibliotecas:* Pandas (limpeza), NumPy (matemática), Matplotlib/Seaborn (gráficos), Scikit-Learn (Machine Learning).

---
Última atualização: 18/02/2026
