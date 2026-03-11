# 📊 Projeto: Telco Customer Churn — Grupo 9

## Identificação da Equipa

* **Instituição:** Coimbra Business School | ISCAC
* **Curso:** Licenciatura em Ciência de Dados para a Gestão
* **Unidade Curricular:** Projeto em Ciência de Dados
* **Grupo nº:** 9
* **Membros:**
  * **Vasco Firmino Coelho** (2023145934)
  * **José Salvador Cabeças** (2023132088)

**Docente Responsável:** Dora Melo (dmelo@iscac.pt)

---

## Organização do Repositório

* **`data/`**: Dados brutos (`raw/`) e processados (`processed/`).
* **`docs/`**: Documentação técnica detalhada das *Milestones*.
* **`notebooks/`**: *Jupyter Notebooks* para experimentação (Kaggle).
* **`src/`**: *Scripts* Python modulares para funções reutilizáveis.
* **`reports/`**: Relatórios e figuras exportadas.

---

## Estado do Projeto

| *Milestone* | Fase CRISP-DM | Prazo | Estado |
| :--- | :--- | :---: | :---: |
| **M1 — Iniciação** | *Business & Data Understanding* | 24/02/2026 | ✅ Completo |
| **M2 — Exploração** | *Data Preparation* | 24/03/2026 | 🔄 Em curso |
| **M3 — Modelação** | *Modeling & Evaluation* | TBD | ⏳ Por iniciar |
| **M4 — Finalização** | *Deployment & Communication* | TBD | ⏳ Por iniciar |

---

## 1. Iniciação (*Milestone* 1) ✅

### Contexto e Problema de Negócio
O setor das telecomunicações enfrenta taxas de rotatividade elevadas. Reter um cliente custa significativamente menos do que adquirir um novo. Este projeto visa prever o ***Churn*** (abandono) para permitir ações de retenção proativas, reduzindo custos de aquisição de novos clientes.

### Objetivos do Projeto
* **Objetivo 1:** Desenvolver um modelo preditivo com um *F1-Score* mínimo de **0.75**.
* **Objetivo 2:** Identificar os **3 principais perfis de risco** de *Churn* até ao final do M3.
* **Objetivo 3:** Garantir que o modelo identifica pelo menos **80% dos clientes em risco** (*Recall* ≥ 0.80).

### Fonte de Dados
* **Dataset:** [Telco Customer Churn (Kaggle)](https://www.kaggle.com/datasets/blastchar/telco-customer-churn)
* **Dimensão original:** 7043 linhas × 21 colunas
* **Variável alvo:** `Churn` — binária (0 = não abandona, 1 = abandona)
* **Desequilíbrio de classes:** ~73.5% não *Churn* / ~26.5% *Churn*

---

## 2. Exploração (*Milestone* 2) 🔄

### Limpeza e Preparação dos Dados

* **Tratamento de *Missing Values*:** 11 valores nulos em `TotalCharges` (0.15%) — imputados com `0.0` (clientes com `tenure = 0` não têm faturação acumulada).
* **Tratamento de *Outliers*:** 0 *outliers* nas 3 variáveis numéricas pelo método IQR — distribuições dentro dos limites operacionais da indústria.
* **Correção de Tipos:** `TotalCharges` convertida de `object` para `float64`; `Churn` mapeada para binária (`int64`); 16 variáveis convertidas para `category`.
* **Verificação de Duplicados:** 0 registos duplicados — cada linha representa um cliente único.
* ***Dataset* final:** 7043 linhas × 24 colunas → `data/processed/telco_churn_clean.csv`

### *Feature Engineering*

Foram criados 3 novos atributos (de 21 para **24 colunas**):

| Atributo | Natureza | O que mede |
| :--- | :--- | :--- |
| `TenureCohort` | Categórica (ordinal) | Fase do ciclo de vida: *Early* (0–12m) / *Growing* (13–24m) / *Mature* (25–48m) / *Loyal* (49–72m) |
| `TotalServices` | Numérica (discreta) | Nº de serviços subscritos — indicador de *lock-in* comportamental (0–8) |
| `LTV_Estatico` | Numérica (contínua) | Valor financeiro acumulado estimado (`MonthlyCharges × tenure`) |

### Estratégia de EDA Avançada

> *Foco: Identificar o "Momento de Atrito" (*Pain Points*).*

* **Análise de Coorte:** O risco de *Churn* diminui com o tempo de contrato? ✅ Confirmado — *Early* tem 47.4% vs. *Loyal* com 9.5%.
* **Análise Multivariada:** Como a combinação de "Contrato Mensal" + "Sem Suporte Técnico" potencia o abandono? ⏳ A desenvolver.
* **Estatística:** Teste Qui-Quadrado para validar a dependência entre variáveis categóricas e o *Churn*. ⏳ A desenvolver.
* ***Encoding*:** Aplicação de *One-Hot Encoding* para variáveis nominais e *Label Encoding* para ordinais. ⏳ A desenvolver.

### Principais *Insights* da EDA

1. **Risco temporal:** Clientes *Early* (0–12 meses) têm **47.4% de taxa de *Churn*** — 5× superior aos clientes *Loyal*.
2. **Vulnerabilidade contratual:** Contratos mensais (*Month-to-month*) atingem **~42% de *Churn*** contra menos de 5% nos contratos anuais.
3. **Dependência tecnológica:** Clientes com *Fiber Optic* apresentam maior rotatividade do que os de *DSL*.
4. ***Lock-in*:** Clientes com 8 serviços têm apenas **5.3% de *Churn*** contra 43.8% sem serviços adicionais.

---

## Decisões Técnicas

| Decisão | Justificação |
| :--- | :--- |
| Métrica principal: *Recall* e *F1-Score* | *Accuracy* é enganadora com classes desequilibradas (73/27) |
| Imputação `TotalCharges = 0.0` | Clientes com `tenure = 0` não têm faturação acumulada — média/mediana introduziria ruído estatístico |
| `TenureCohort` em vez de `tenure` contínuo | Captura a não-linearidade do risco ao longo do tempo |
| `TotalServices` separado do `LTV_Estatico` | Medem dimensões distintas: *lock-in* comportamental vs. valor financeiro |

---

*Atualizado em: 11/03/2026*
