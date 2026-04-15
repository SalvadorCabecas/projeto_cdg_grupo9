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
| **M3 — Modelação** | Modelação e avaliação | 23/04/2026 | Em Curso |
| **M4 — Finalização** | Comunicação e entrega | A definir | Por iniciar |

---

## *Kaggle*

| Item | Ligação |
| :--- | :--- |
| *Notebook* EDA e preparação | `notebooks/1.0_eda_limpeza.ipynb` (*Kaggle* — acesso restrito) |
| *Notebook* modelação e treino | `notebooks/2.0_modelacao_treino.ipynb` (*Kaggle* — acesso restrito) |
| Conjunto de dados | [*Telco Customer Churn*](https://www.kaggle.com/datasets/blastchar/telco-customer-churn) |

---

## 1. Iniciação (*Milestone* 1)

### Contexto e Problema de Negócio

O setor das telecomunicações caracteriza-se por mercados saturados e elevada concorrência, o que torna a retenção de clientes um fator crítico para a sustentabilidade das empresas. O abandono de clientes (*Churn*) representa uma perda direta de receita recorrente e impõe custos operacionais associados à necessidade de substituição dessa base de clientes.

Este projeto visa construir um modelo preditivo capaz de identificar, com antecedência, os clientes com maior probabilidade de abandono, com base nas suas características contratuais, de utilização e de faturação. O objetivo final é fornecer à empresa uma ferramenta de apoio à decisão que permita priorizar intervenções de retenção de forma eficiente.

### Objetivos do Projeto (SMART)

1. **Objetivo 1:** Desenvolver e comparar modelos de classificação supervisionada para prever o abandono de clientes (*Churn*), selecionando o modelo com melhor *F1-Score* na classe positiva (*Churn* = 1), com um limiar mínimo de 0,75 em validação cruzada estratificada (k=5), até ao dia 23/04/2026 (*Milestone* 3).
2. **Objetivo 2:** Validar que o índice de risco composto (*RiskScore*), construído a partir de variáveis contratuais e de serviço durante a fase de *feature engineering*, estratifica eficazmente os clientes em níveis de risco de abandono, confirmando que o grupo de risco mais elevado apresenta uma taxa de abandono superior a 60% e que a variável contribui significativamente para o poder preditivo do modelo final (*feature importance*), até ao dia 23/04/2026 (*Milestone* 3).
3. **Objetivo 3:** Assegurar uma taxa de deteção (*Recall*) igual ou superior a 0,80 para a classe positiva (*Churn* = 1), minimizando o número de clientes em risco não identificados pelo modelo, recorrendo a técnicas de balanceamento de classes (*SMOTE* e/ou ajuste de *class_weight*) e otimização do limiar de decisão, até ao dia 23/04/2026 (*Milestone* 3).

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
- Resultado final: 7043 linhas × 24 colunas → `data/processed/telco_churn_processed.csv`

Detalhes completos em `docs/M2_exploracao.md`.

### Principais Conclusões da Análise Exploratória

> Figura mais relevante do projeto: taxa de abandono por *RiskScore* (`reports/figures/09_novas_features_correlacao.png`)

- **Risco temporal:** Clientes nos primeiros 12 meses apresentam uma taxa de abandono de **47.4%** — cerca de 5 vezes superior à dos clientes com mais de 4 anos de contrato.
- **Tipo de contrato:** Contratos mensais atingem **~42% de abandono**, contra menos de 5% nos contratos anuais.
- **Índice de risco:** O *RiskScore* máximo (6) corresponde a uma taxa de abandono de **70.2%**, confirmando o poder preditivo combinado das variáveis contratuais e de serviço.
- **Fidelização:** Clientes com 8 serviços subscritos têm apenas **5.3% de abandono**, contra 43.8% sem serviços adicionais.

---

## 3. Modelação (*Milestone* 3)

### Estratégia de Modelação

O *dataset* processado foi dividido em 80% para treino e 20% para teste com divisão estratificada (`stratify=y, random_state=42`), preservando a proporção real de *churn* (~26.5%) em ambas as partições. Para corrigir o desequilíbrio de classes, foi aplicado *SMOTE* exclusivamente no conjunto de treino (~4508 Não-*Churn* / ~4508 *Churn* após balanceamento). O `StandardScaler` foi ajustado no treino balanceado e aplicado a ambos os conjuntos. A métrica principal é o *F1-Score* na classe *Churn* (objetivo ≥ 0.75 em CV), complementada pelo *AUC-ROC* e *Recall* (objetivo ≥ 0.80).

### Algoritmos Testados

| Algoritmo | F1 Treino | F1 Teste | *AUC-ROC* | *Gap* F1 |
| :--- | :---: | :---: | :---: | :---: |
| *Logistic Regression* (*baseline*) | 0.8436 | **0.6048** | **0.8340** | 0.2388 |
| *Gradient Boosting* | 0.8588 | 0.5898 | 0.8319 | 0.2690 |
| *Random Forest* | 0.9982 | 0.5812 | 0.8194 | 0.4169 |
| *Naive Bayes* | 0.7684 | 0.5799 | 0.8077 | 0.1885 |
| *XGBoost* | 0.9606 | 0.5677 | 0.8009 | 0.3930 |
| *KNN* | 0.8758 | 0.5398 | 0.7690 | 0.3360 |
| *Decision Tree* | 0.9983 | 0.4881 | 0.6502 | 0.5102 |

A Regressão Logística obteve o melhor *F1-Score* no teste real (0.6048) e o melhor *AUC-ROC* (0.8340). Os modelos *tree-based* revelaram *overfitting* severo à distribuição sintética do *SMOTE*, não generalizando para a distribuição real do teste.

### Otimização (*Tuning*)

O *Gradient Boosting* foi seleccionado para otimização com `RandomizedSearchCV` (50 iterações × 5 *folds* = 250 *fits*, `scoring='f1'`). Os hiperparâmetros óptimos encontrados foram: `n_estimators=287`, `max_depth=3`, `learning_rate=0.1353`, `subsample=0.7465`, `min_samples_split=19`. A validação cruzada K=5 sobre o modelo optimizado produziu μ=0.8527 ± σ=0.0096 (IC 95%: [0.8339, 0.8715]), confirmando estabilidade elevada. No conjunto de teste real, o modelo optimizado obteve F1=0.5814, abaixo do *baseline* (F1=0.6048) — fenómeno de *distribution shift* decorrente da diferença entre a distribuição *SMOTE*-balanceada e a distribuição real do teste.

### Estado Atual e Próximos Passos

| Etapa | Estado |
| :--- | :---: |
| Divisão treino/teste + *SMOTE* + `StandardScaler` | ✅ Concluído |
| *Baseline* (Regressão Logística) — F1=0.6048, *AUC*=0.8340 | ✅ Concluído |
| Comparação de 7 algoritmos candidatos | ✅ Concluído |
| Diagnóstico de *overfitting* + curvas de aprendizagem | ✅ Concluído |
| Otimização *Gradient Boosting* (*RandomizedSearchCV*) | ✅ Concluído |
| Ajuste de *threshold* + matriz de confusão final | ⏳ Aula 20 — 17/04/2026 |
| *Feature importance* + análise de erros | ⏳ Aula 20 — 17/04/2026 |
| Consolidação e limpeza do *notebook* | ⏳ Aula 21 — 22/04/2026 |

Detalhes completos em `docs/M3_modelacao.md`.

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

*Última atualização: 15/04/2026 | Aula 19 — Semana 10*
