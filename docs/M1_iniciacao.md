# Milestone 1: Iniciação e Definição do Projeto - Grupo 9

## 1. Descrição Detalhada do Problema
O setor das telecomunicações enfrenta taxas de rotatividade (churn) elevadas, o que impacta diretamente a rentabilidade das empresas. Como o custo de adquirir um novo cliente é muito superior ao de reter um atual, este projeto utiliza o dataset "Telco Customer Churn" para prever comportamentos de abandono e permitir que a gestão tome decisões proativas de fidelização.

## 2. Objetivos SMART
1. *Objetivo 1 (Técnico):* Desenvolver um modelo de classificação binária para prever o Churn com um *F1-Score mínimo de 0.75*, garantindo equilíbrio entre Precision e Recall.
2. *Objetivo 2 (Gestão):* Identificar os 3 principais fatores que influenciam a saída dos clientes (ex: tipo de contrato, tempo de permanência ou serviços adicionais) até à Milestone 3.

## 3. Metodologia de Gestão (PBL)
* *Divisão de Tarefas:*
    * *Salvador Cabeças:* Responsável pela Infraestrutura GitHub, Sincronização com Kaggle e Engenharia de Dados.
    * *Vasco Coelho:* Responsável pela Documentação Técnica (Milestones), Análise Estatística e Modelação.
* *Ferramentas:* GitHub Projects (Kanban) para gestão de tarefas e Kaggle Notebooks para desenvolvimento em Python.

## 4. Análise de Viabilidade dos Dados
* *Disponibilidade:* O dataset foi carregado com sucesso no Kaggle e vinculado ao repositório GitHub.
* *Qualidade Inicial:* Através da inspeção inicial (df.info()), observou-se que a coluna TotalCharges precisa de tratamento, pois está como texto (object) e contém valores em branco.
* *Ética:* Os dados são públicos e anonimizados, cumprindo os requisitos de privacidade.

## 5. Perguntas de Investigação
1. Clientes com contratos mensais ("Month-to-month") têm maior probabilidade de abandono do que clientes com contratos anuais?
2. O valor da fatura mensal (MonthlyCharges) é um indicador determinante para o churn?
3. A presença de serviços de suporte técnico e segurança online influencia positivamente a retenção do cliente?
