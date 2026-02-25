# 📊 Projeto: Telco Customer Churn - Grupo 9

## Identificação da Equipa
* **Instituição:** Coimbra Business School | ISCAC
* **Curso:** Licenciatura em Ciência de Dados para a Gestão
* **Unidade Curricular:** Projeto em Ciência de Dados
* **Grupo nº:** 9
* **Membros:**
  * **Vasco Firmino Coelho** (2023145934)
  * **José Salvador Cabeças** (2023132088)

## Organização do Repositório
* **`data/`**: Dados brutos (`raw/`) e processados (`processed/`).
* **`docs/`**: Documentação técnica detalhada das Milestones.
* **`notebooks/`**: Jupyter Notebooks para experimentação (Kaggle).
* **`src/`**: Scripts Python modulares para funções reutilizáveis.
* **`reports/`**: Relatórios e figuras exportadas.

## 1. Iniciação (Milestone 1)
### Contexto e Problema de Negócio
O setor das telecomunicações enfrenta taxas de rotatividade elevadas. Este projeto visa prever o **Churn** (abandono) para permitir ações de retenção proativas, reduzindo custos de aquisição de novos clientes.

### Objetivos do Projeto
* **Objetivo 1:** Desenvolver um modelo preditivo com um **F1-Score mínimo de 0.75**.
* **Objetivo 2:** Identificar os principais fatores que influenciam a saída dos clientes.

## 2. Exploração (Milestone 2) - [EM CURSO]
### Limpeza e Preparação
* **Tratamento de Missing Values:** Identificação de valores nulos (ex: `TotalCharges` em novos clientes).
* **Feature Engineering:** Criação de variáveis de valor (Tenure Cohorts, Total Services, LTV Estático).
* **Encoding:** Aplicação de One-Hot Encoding para variáveis nominais e Label Encoding para ordinais.

### Estratégia de EDA Avançada
> *Foco: Identificar o "Momento de Atrito" (Pain Points).*
* **Análise de Coorte:** O risco de churn diminui com o tempo de contrato?
* **Análise Multivariada:** Como a combinação de "Contrato Mensal" + "Sem Suporte Técnico" potencia o abandono?
* **Estatística:** Teste Qui-Quadrado para validar a dependência entre variáveis categóricas e o Churn.

### Fonte de Dados
* **Dataset:** [Telco Customer Churn (Kaggle)](https://www.kaggle.com/datasets/blastchar/telco-customer-churn)
* **Dimensão:** 7043 linhas e 21 colunas.

---
**Docente Responsável:** Dora Melo (dmelo@iscac.pt)
