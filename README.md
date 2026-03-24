# Previsão de Abandono de Clientes em Telecomunicações

> Projeto académico de Ciência de Dados — previsão do abandono (*Churn*) de clientes de uma empresa de telecomunicações, com recurso a técnicas de aprendizagem supervisionada.

---

## Equipa

| Nome | Número de Aluno |
| :--- | :---: |
| Vasco Firmino Coelho | 2023145934 |
| José Salvador Cabeças | 2023132088 |

---

## Organização do Repositório

A estrutura deste projeto segue as boas práticas de Ciência de Dados e Engenharia de Software:

- **`data/`**: Armazenamento de dados (dados brutos em `raw/` e processados em `processed/`).
- **`docs/`**: Documentação técnica detalhada dividida por fases (`M1_iniciacao.md`, `M2_exploracao.md`, `M3_modelacao.md`, `M4_conclusoes.md`).
- **`notebooks/`**: Versões exportadas do *Kaggle Code*, seguindo a ordem numérica das fases.
- **`src/`**: Código-fonte modular (*scripts* `.py`) para funções reutilizáveis.
- **`reports/`**: Relatórios finais, apresentação e exportação de figuras (`figures/`).
- **`requirements.txt`**: Ficheiro de configuração com as bibliotecas necessárias.

---

## Estado do Projeto

| Fase | Foco CRISP-DM | Prazo | Estado |
| :--- | :--- | :---: | :---: |
| **M1 — Iniciação** | Compreensão do negócio e dos dados | 24/02/2026 | Concluído |
| **M2 — Exploração** | Análise exploratória e preparação dos dados | 24/03/2026 | Concluído |
| **M3 — Modelação** | Modelação e avaliação | A definir | Por iniciar |
| **M4 — Finalização** | Comunicação e entrega | A definir | Por iniciar |

---

## *Kaggle*

| Item | Ligação |
| :--- | :--- |
| *Notebook* principal | `notebooks/1-0-eda-limpeza.ipynb` (*Kaggle* — acesso restrito) |
| Conjunto de dados | [*Telco Customer Churn*](https://www.kaggle.com/datasets/blastchar/telco-customer-churn) |

---

## 1. Iniciação (*Milestone* 1)

### Contexto e Problema de Negócio

O setor das telecomunicações caracteriza-se por mercados saturados e elevada concorrência, o que torna a retenção de clientes um fator crítico para a sustentabilidade das empresas. O abandono de clientes (*Churn*) representa uma perda direta de receita recorrente e impõe custos operacionais associados à necessidade de substituição dessa base de clientes.

Este projeto visa construir um modelo preditivo capaz de identificar, com antecedência, os clientes com maior probabilidade de abandono, com base nas suas características contratuais, de utilização e de faturação. O objetivo final é fornecer à empresa uma ferramenta de apoio à decisão que permita priorizar intervenções de retenção de forma eficiente.

### Objetivos do Projeto (SMART)

1. **Objetivo 1:** Desenvolver e comparar três modelos de classificação supervisionada — Regressão Logística, *Random Forest* e *XGBoost* — para prever o abandono de clientes (*Churn*), selecionando o modelo com melhor *F1-Score* na classe positiva (*Churn* = 1), com um limiar mínimo de 0,75 em validação cruzada estratificada (k=5), até ao dia 21/04/2026 (*Milestone* 3).
2. **Objetivo 2:** Validar que o índice de risco composto (*RiskScore*), construído a partir de variáveis contratuais e de serviço durante a fase de *feature engineering*, estratifica eficazmente os clientes em níveis de risco de abandono, confirmando que o grupo de risco mais elevado apresenta uma taxa de abandono superior a 60% e que a variável contribui significativamente para o poder preditivo do modelo final (*feature importance*), até ao dia 21/04/2026 (*Milestone* 3).
3. **Objetivo 3:** Assegurar uma taxa de deteção (*Recall*) igual ou superior a 0,80 para a classe positiva (*Churn* = 1), minimizando o número de clientes em risco não identificados pelo modelo, recorrendo a técnicas de balanceamento de classes (SMOTE e/ou ajuste de *class_weight*) e otimização do limiar de decisão, até ao dia 21/04/2026 (*Milestone* 3).

### Questões de Investigação

1. Quais as variáveis contratuais e de utilização que apresentam maior associação estatística com o abandono de clientes?
2. A combinação de contrato mensal com ausência de serviços de suporte técnico aumenta significativamente a probabilidade de abandono, comparativamente a clientes com contrato anual?
3. Existe um limiar de antiguidade (*tenure*) abaixo do qual a probabilidade de abandono é substancialmente superior?
4. Um modelo de aprendizagem supervisionada consegue identificar corretamente, pelo menos, 80% dos clientes que efetivamente abandonam o serviço?
5. As novas variáveis criadas (*RiskScore*, *ChargesPerService*) aumentam o poder preditivo do modelo face às variáveis originais do conjunto de dados?

### Fonte de Dados

- **Conjunto de dados:** [*Telco Customer Churn* — IBM/*Kaggle*](https://www.kaggle.com/datasets/blastchar/telco-customer-churn)
- **Dimensão original:** 7043 linhas × 21 colunas
- **Variável alvo:** `Churn` — binária (0 = permanece, 1 = abandona)
- **Desequilíbrio de classes:** ~73.5% sem abandono / ~26.5% com abandono

---

## 2. Exploração (*Milestone* 2)

### Limpeza e Preparação

- 11 valores nulos em `TotalCharges` (0.15%) — imputados com `0.0`, por corresponderem a clientes com `tenure = 0` sem faturação acumulada.
- 0 valores atípicos nas 3 variáveis numéricas (método IQR).
- `TotalCharges` convertida de texto para `float64`; `Churn` mapeada para binária (`int64`); 16 variáveis convertidas para `category`.
- 0 registos duplicados.
- 5 novos atributos criados: `TenureCohort`, `TotalServices`, `LTV_Estatico`, `ChargesPerService` e `RiskScore`.
- 2 variáveis removidas por redundância: `customerID` (não preditiva) e `TotalCharges` (multicolinearidade com `LTV_Estatico`).
- Resultado final: 7043 linhas × 24 colunas → `data/processed/telco_churn_clean.csv`

Detalhes completos em `docs/M2_exploracao.md`.

### Principais Conclusões da Análise Exploratória

> Figura mais relevante do projeto: taxa de abandono por *RiskScore* (`reports/figures/09_novas_features_correlacao.png`)

- **Risco temporal:** Clientes nos primeiros 12 meses apresentam uma taxa de abandono de **47.4%** — cerca de 5 vezes superior à dos clientes com mais de 4 anos de contrato.
- **Tipo de contrato:** Contratos mensais atingem **~42% de abandono**, contra menos de 5% nos contratos anuais.
- **Índice de risco:** O *RiskScore* máximo (6) corresponde a uma taxa de abandono de **70.2%**, confirmando o poder preditivo combinado das variáveis contratuais e de serviço.
- **Fidelização:** Clientes com 8 serviços subscritos têm apenas **5.3% de abandono**, contra 43.8% sem serviços adicionais.

---

## 3. Modelação (*Milestone* 3)

### Abordagem Técnica

- **Modelos:** Regressão Logística, *Random Forest*, *XGBoost*
- **Métrica principal:** *F1-Score* e *Recall*

*A iniciar após conclusão do M2.*

---

## 4. Finalização (*Milestone* 4)

### Resposta ao Problema

*A preencher após conclusão do M3.*

### Recomendações

*A preencher após conclusão do M3.*

---

## Como Reproduzir este Projeto

1. Clonar o repositório: `git clone https://github.com/SalvadorCabecas/projeto_cdg_grupo9`
2. Instalar as dependências: `pip install -r requirements.txt`
3. Executar os *notebooks* na pasta `notebooks/` seguindo a ordem numérica.

---

## Referências

1. IBM. (s.d.). *Telco Customer Churn* [Conjunto de dados]. *Kaggle*. Disponível em: https://www.kaggle.com/datasets/blastchar/telco-customer-churn
2. Melo, D. (2026). *Metodologia CRISP-DM* [Materiais de apoio]. ISCAC — *Coimbra Business School*.

---

## Identificação Académica

| | |
| :--- | :--- |
| **Instituição** | *Coimbra Business School* \| ISCAC |
| **Curso** | Licenciatura em Ciência de Dados para a Gestão |
| **Unidade Curricular** | Projeto em Ciência de Dados |
| **Docente Responsável** | Dora Melo (dmelo@iscac.pt) |
| **Ano Letivo** | 2025/2026 |

---

*Última atualização: 24/03/2026*
