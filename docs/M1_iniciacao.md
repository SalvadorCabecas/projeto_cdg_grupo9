# Milestone 1: Iniciação e Definição do Projeto

## 1. Descrição Detalhada do Problema
O setor das telecomunicações é altamente competitivo. Reter um cliente custa significativamente menos do que adquirir um novo. Este projeto foca-se em analisar o comportamento de consumo e contrato de clientes para prever o risco de abandono (Churn), otimizando a saúde financeira da empresa.

## 2. Objetivos SMART
Nesta secção, definimos as metas estratégicas que orientam o desenvolvimento do projeto, alinhando a performance técnica com as necessidades de negócio da Telco.
---
### Objetivo 1: Modelação Preditiva e Performance
**Prever se um cliente irá cancelar o serviço com um benchmark de performance robusto.**
* Desenvolver um modelo de classificação binária que atinja um **F1-Score mínimo de 0.75**. Esta métrica garante o equilíbrio necessário entre a precisão das previsões e a capacidade de identificação real do abandono.

### Objetivo 2: Identificação de Perfis de Risco
**Mapear os padrões de comportamento que levam ao Churn.**
* Identificar os **3 principais perfis de risco** (segmentos de clientes mais voláteis) até ao final da Milestone 3. Este objetivo visa transformar dados brutos em *insights* acionáveis para a equipa de marketing.

### Objetivo 3: Otimização do Recall para Retenção (Negócio)
**Priorizar a métrica de Recall (Sensibilidade) no ajuste e seleção dos modelos.**
* **Fundamentação:** Em termos de gestão, o custo de um **Falso Negativo** (não detetar um cliente que vai efetivamente sair) é drasticamente superior ao custo de um **Falso Positivo** (oferecer um incentivo de retenção a um cliente que acabaria por ficar).
* **Meta de Negócio:** Garantir que o modelo identifique, pelo menos, **80% dos clientes em risco de churn** (Recall >= 0.80), maximizando a eficácia das campanhas de retenção preventiva.
---
### 2.1. Questões de Partida 
Para orientar a exploração de dados e garantir que o modelo final entrega valor de negócio, o grupo definiu as seguintes perguntas de investigação:

* **P1 (Contratual):** Qual a diferença percentual na taxa de Churn entre clientes com contrato mensal (`Month-to-month`) e contratos de fidelização (1 ou 2 anos)?
* **P2 (Ciclo de Vida):** Existe um ponto de inflexão na variável `tenure` (meses de permanência) onde a probabilidade de abandono aumenta drasticamente (ex: primeiros 6 meses)?
* **P3 (Financeiro):** De que forma o método de pagamento (`PaymentMethod`), especificamente o `Electronic check`, influencia a propensão ao Churn comparado com métodos automáticos?
* **P4 (Demográfico):** O segmento de clientes séniores (`SeniorCitizen`) apresenta padrões de consumo ou taxas de abandono significativamente distintos do restante público?

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
Nesta etapa de Data Understanding, realizámos uma revisão exaustiva dos tipos de dados para garantir que a estrutura técnica do dataset está alinhada com a realidade do negócio e as exigências da modelação.

Conversão Crítica (TotalCharges): Identificámos que esta variável estava incorretamente tipificada como texto (object). A causa raiz eram espaços em branco em clientes com tenure = 0 (novos contratos). Forçámos a conversão para float64, isolando os valores nulos para tratamento na fase de Data Preparation.

Codificação Binária (Churn, SeniorCitizen): Estas variáveis foram definidas como inteiros (int64). Esta transformação é vital para permitir o cálculo de correlações estatísticas e para que os algoritmos de Machine Learning possam processar a variável alvo (0 e 1).

Dimensão Temporal (tenure): Mantida como inteiro, representando a unidade discreta de meses de fidelização, essencial para análises de sobrevivência e retenção.

Otimização Categórica: Todas as variáveis qualitativas (ex: tipo de contrato, serviços e método de pagamento) foram convertidas para o tipo category. Isto reduz o consumo de memória e sinaliza ao modelo a natureza discreta destas características.

Nota de Integridade: Esta organização assegura que não existem "variáveis fantasma" (números lidos como texto) que possam enviesar a análise exploratória ou invalidar o treino dos modelos.

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
