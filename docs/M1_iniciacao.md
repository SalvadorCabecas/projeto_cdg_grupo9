# Milestone 1: Iniciação e Definição do Projeto - Grupo 9

## 1. Descrição Detalhada do Problema
O setor das telecomunicações enfrenta taxas de rotatividade elevadas. [cite_start]O custo de adquirir um novo cliente é significativamente superior ao de reter um atual [cite: 850-851]. [cite_start]Este projeto foca-se no dataset "Telco Customer Churn" para prever quais clientes têm maior probabilidade de abandonar a operadora, permitindo ações de marketing preventivas [cite: 849-850].

## 2. Objetivos SMART
1. [cite_start]*Objetivo 1 (Técnico):* Desenvolver um modelo de classificação binária (Churn: Yes/No) com um *F1-Score mínimo de 0.75*, garantindo um bom equilíbrio entre Precision e Recall [cite: 258-261, 742-747].
2. [cite_start]*Objetivo 2 (Gestão):* Identificar os 3 principais fatores que influenciam o abandono (ex: tipo de contrato ou faturação mensal) até à entrega da Milestone 3 [cite: 211-215, 764].

## 3. Metodologia de Gestão (PBL)
* *Divisão de Tarefas:*
    * [cite_start]*Salvador Cabeças:* Responsável pela Infraestrutura GitHub, Link com Kaggle e Engenharia de Dados [cite: 263-267, 635-639].
    * [cite_start]*Vasco Coelho:* Responsável pela Documentação Técnica (Milestones), Análise Estatística e Modelação [cite: 263-267, 635-639].
* [cite_start]*Ferramentas:* GitHub Projects para gestão de tarefas (Kanban) e Kaggle para ambiente de desenvolvimento Python[cite: 1354, 1378].

## 4. Análise de Viabilidade dos Dados
* [cite_start]*Disponibilidade:* O dataset já foi importado com sucesso para o Kaggle Notebook[cite: 45, 421].
* [cite_start]*Qualidade Inicial:* Através do comando df.info(), detetámos que a coluna TotalCharges está configurada como 'object' (texto) e precisa de conversão para numérica[cite: 47, 423].
* [cite_start]*Ética:* Os dados são públicos e estão anonimizados, respeitando os princípios do RGPD[cite: 273, 645].

## 5. Perguntas de Investigação
1. [cite_start]Clientes com contratos "Month-to-month" têm uma taxa de abandono superior aos de contratos anuais? [cite: 414, 772]
2. [cite_start]Existe uma relação direta entre o valor da fatura mensal (MonthlyCharges) e a probabilidade de Churn? [cite: 414, 770]
3. [cite_start]A ausência de serviços de suporte técnico aumenta a probabilidade de o cliente abandonar a operadora? [cite: 414, 771]

---
[cite_start]Data de última atualização: 18/02/2026 [cite: 281, 653]
