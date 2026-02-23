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
Nesta fase de entendimento dos dados, procedeu-se à tipificação correta de todas as variáveis. Foi identificada a necessidade de conversão da variável `TotalCharges`, que originalmente é lida como texto (`object`), para um formato numérico (`float64`), garantindo a integridade da análise estatística e a compatibilidade com algoritmos de Machine Learning.

### 6.1. Variáveis Demográficas e de Identificação
| Variável | Descrição | Tipo | Observação |
| :--- | :--- | :--- | :--- |
| **customerID** | Identificador único do cliente | String (ID) | Removido na modelação (não preditivo). |
| **gender** | Género do cliente | Categórica | Male, Female. |
| **SeniorCitizen** | Indica se o cliente tem 65 anos ou mais | Binária | 0 (Não), 1 (Sim). |
| **Partner** | Indica se o cliente tem parceiro(a) | Categórica | Yes, No. |
| **Dependents** | Indica se o cliente tem dependentes | Categórica | Yes, No. |

### 6.2. Variáveis de Serviços Subscritos
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

### 6.3. Variáveis Contratuais e Financeiras
| Variável | Descrição | Tipo | Observação |
| :--- | :--- | :--- | :--- |
| **tenure** | Meses de permanência na empresa | Numérica (int) | Intervalo: 0 a 72 meses. |
| **Contract** | Prazo do contrato do cliente | Categórica | Month-to-month, One year, Two year. |
| **PaperlessBilling** | Se utiliza faturação eletrónica | Categórica | Yes, No. |
| **PaymentMethod** | Método de pagamento escolhido | Categórica | 4 categorias (e.g., Electronic check). |
| **MonthlyCharges** | Valor debitado mensalmente | Numérica (float) | Valor contínuo. |
| **TotalCharges** | Valor total faturado ao cliente | Numérica (float) | **Convertido de object para float.** |
| **Churn** | Indica se o cliente abandonou a empresa | Binária | **Variável Alvo (Target):** Yes, No. |

---

> **Nota Crítica de Qualidade:** A variável `TotalCharges` continha espaços em branco em registos onde `tenure = 0`. Estes foram tratados como valores nulos (`NaN`) durante a conversão e serão devidamente imputados na fase de *Data Preparation* (Milestone 2).
---
Data de última atualização: 23/02/2026
