# Milestone 1: Iniciação e Definição do Projeto

## 1. Descrição Detalhada do Problema
O setor das telecomunicações é altamente competitivo. Reter um cliente custa significativamente menos do que adquirir um novo. Este projeto foca-se em analisar o comportamento de consumo e contrato de clientes para prever o risco de abandono (Churn), otimizando a saúde financeira da empresa.

## 2. Objetivos SMART
1. *Objetivo 1:* Prever se um cliente irá cancelar o serviço com um *F1-Score mínimo de 0.75*.
2. *Objetivo 2:* Identificar os 3 principais perfis de risco até ao final da Milestone 3.

## 3. Metodologia de Gestão (PBL)
* *Divisão de Tarefas:*
  * *Salvador Cabeças:* Responsável pelo Setup de infraestrutura e Engenharia de Dados.
  * *Vasco Coelho:* Responsável pela Documentação Técnica e Modelação Estatística.
* *Ferramentas de Colaboração:* GitHub Projects (Kanban) e Kaggle para desenvolvimento partilhado.

## 4. Análise de Viabilidade dos Dados
* *Disponibilidade:* Os dados foram importados via Kaggle API e estão acessíveis no repositório.
* *Qualidade Inicial:* Identificámos que a coluna TotalCharges está como object devido a espaços vazios, o que exigirá tratamento na M2.
* *Ética:* O dataset é público e está totalmente anonimizado (customerID mascarado), cumprindo as normas de proteção de dados e RGPD.

## 5. Cronograma Interno
| Fase | Data Limite | Entregável Esperado |
| :--- | :--- | :--- |
| M1: Iniciação | 20/02/2026 | Repositório estruturado e Plano de Projeto. |
| M2: Exploração | Março 2026 | Notebook de EDA e Dados Processados. |
| M3: Modelação | Abril 2026 | Comparação de algoritmos e métricas. |
| M4: Finalização| Maio 2026 | Pitch e Relatório Final. |

## 6. Dicionário de Dados Completo (21 Variáveis)

### 6.1. Identificação e Demográficas
| Variável | Descrição | Tipo |
| :--- | :--- | :--- |
| *customerID* | Identificador único do cliente. | ID |
| *gender* | Género (Male/Female). | Categórica |
| *SeniorCitizen* | Se o cliente é idoso (1/0). | Binária |
| *Partner* | Se tem parceiro (Yes/No). | Categórica |
| *Dependents* | Se tem dependentes (Yes/No). | Categórica |

### 6.2. Serviços Subscritos
| Variável | Descrição |
| :--- | :--- |
| *tenure* | Meses na empresa (Numérica). |
| *PhoneService, MultipleLines* | Serviços de voz. |
| *InternetService* | DSL, Fiber optic, No. |
| *OnlineSecurity, OnlineBackup, DeviceProtection, TechSupport* | Serviços de suporte/segurança. |
| *StreamingTV, StreamingMovies* | Serviços de entretenimento. |

### 6.3. Contratuais e Financeiras
| Variável | Descrição | Observação |
| :--- | :--- | :--- |
| *Contract* | Tipo de contrato. | Month-to-month, 1 yr, 2 yr |
| *PaperlessBilling*| Fatura digital (Yes/No). | Categórica |
| *PaymentMethod* | Método de pagamento. | 4 categorias |
| *MonthlyCharges* | Fatura mensal. | Numérica (float) |
| *TotalCharges* | Fatura total. | *Requer Limpeza (object)* |
| *Churn* | *Variável Alvo*. | Yes / No |

---
Data de última atualização: 20/02/2026
