# Construção de um Modelo Preditivo de Abandono de Clientes no Setor das Telecomunicações

> Projeto académico de Ciência de Dados — construção e avaliação de um modelo de classificação supervisionada para prever o abandono de clientes de uma empresa de telecomunicações, com base nas suas características contratuais, de utilização e de faturação.

---

## Equipa

| Nome | Número de Aluno |
| :--- | :---: |
| Vasco Firmino Coelho | 2023145934 |
| José Salvador Cabeças | 2023132088 |

---

## Organização do Repositório

A estrutura deste projeto segue as boas práticas de Ciência de Dados:

- **`data/`**: Armazenamento de dados (dados brutos em `raw/` e processados em `processed/`).
- **`docs/`**: Documentação técnica dividida por fases (`M1_iniciacao.md`, `M2_exploracao.md`, `M3_modelacao.md`).
- **`notebooks/`**: Versões exportadas do Kaggle Code, seguindo a ordem numérica das fases.
- **`src/`**: Reservada para código-fonte modular reutilizável (funções de pré-processamento, avaliação, etc.). Não foi utilizada neste projeto porque toda a lógica foi desenvolvida diretamente nos notebooks do Kaggle, não havendo necessidade de partilha de código entre cadernos.
- **`reports/`**: Figuras e relatórios (`figures/`).
- **`requirements.txt`**: Bibliotecas necessárias para reprodução do projeto.

---

## Estado do Projeto

| Fase | Foco CRISP-DM | Prazo | Estado |
| :--- | :--- | :---: | :---: |
| **M1 — Iniciação** | Compreensão do negócio e dos dados | 24/02/2026 | Concluído |
| **M2 — Exploração** | Análise exploratória e preparação dos dados | 24/03/2026 | Concluído |
| **M3 — Modelação** | Modelação e avaliação | 23/04/2026 | Concluído |
| **M4 — Finalização** | Comunicação e entrega | A definir | Por iniciar |

---

## Kaggle

| Item | Ligação |
| :--- | :--- |
| Conjunto de dados | [Telco Customer Churn — IBM/Kaggle](https://www.kaggle.com/datasets/blastchar/telco-customer-churn) |
| Caderno de exploração e preparação (M2) | `notebooks/1.0_eda_limpeza.ipynb` |
| Caderno de modelação e treino (M3) | `notebooks/2.0_modelacao_treino.ipynb` |

---

## 1. Iniciação (Milestone 1)

### Contexto e Problema de Negócio

O setor das telecomunicações caracteriza-se por mercados saturados e elevada concorrência entre operadores. Neste contexto, o abandono de clientes representa uma perda direta de receita recorrente e implica custos operacionais associados à necessidade de substituição dessa base de clientes. A capacidade de identificar, com antecedência, os clientes com maior propensão para o abandono permite à empresa adotar medidas de retenção dirigidas e atempadas, preservando a sua base de clientes de forma eficiente.

Este projeto tem como objetivo construir um modelo de classificação supervisionada capaz de prever o abandono de clientes com base nos seus dados contratuais, de serviços e de faturação, utilizando o conjunto de dados público *Telco Customer Churn* disponibilizado pela IBM no Kaggle (7043 clientes, 21 variáveis).

### Objetivo do Projeto (SMART)

Construir um modelo de classificação supervisionada para prever o abandono de clientes, atingindo um F1-Score igual ou superior a 0.75 na classe positiva (abandono = 1) em validação cruzada estratificada com k=5, utilizando o conjunto de dados *Telco Customer Churn* da IBM, até ao dia 23/04/2026.

### Questões de Investigação

1. Quais as variáveis contratuais e de utilização que apresentam maior associação estatística com o abandono de clientes?
2. Existe um limiar de antiguidade abaixo do qual a probabilidade de abandono é substancialmente superior?
3. A combinação de contrato mensal com ausência de serviços de suporte técnico aumenta significativamente a probabilidade de abandono, comparativamente a clientes com contrato anual?
4. Um índice de risco composto, construído a partir das variáveis contratuais e de serviço, consegue estratificar os clientes de forma a que o grupo de risco mais elevado apresente uma taxa de abandono superior a 60%?
5. As novas variáveis criadas aumentam o poder preditivo do modelo face às variáveis originais do conjunto de dados?
6. É possível atingir uma taxa de deteção igual ou superior a 0.80 para a classe de abandono, recorrendo a técnicas de balanceamento de classes e ajuste do limiar de decisão?

### Observações Iniciais ao Conjunto de Dados

Na análise inicial ao conjunto de dados, foram identificados os seguintes aspetos:

- **Dimensão:** 7043 registos e 21 variáveis; variável alvo `Churn` com desequilíbrio de classes (~73.5% sem abandono / ~26.5% com abandono).
- **Problema de tipagem:** a variável `TotalCharges` estava armazenada como texto devido à presença de espaços em branco em 11 registos com `tenure = 0`, exigindo conversão e imputação.
- **Distribuição temporal:** a variável `tenure` apresenta uma distribuição bimodal — concentração de clientes novos (primeiros meses) e clientes com 6 anos de contrato, sugerindo um padrão de abandono precoce.
- **Ausência de valores nulos reais e duplicados** nas restantes variáveis após auditoria inicial.
- **Desequilíbrio de classes:** a proporção de 26.5% de abandono determinou a escolha do F1-Score como métrica principal em detrimento da exatidão global.

Detalhes completos em `docs/M1_iniciacao.md`.

---

## 2. Exploração (Milestone 2)

### Limpeza e Preparação

- 11 valores nulos em `TotalCharges` (0.15%) — imputados com `0.0`, por corresponderem a clientes com `tenure = 0` sem faturação acumulada.
- 0 valores atípicos nas 3 variáveis numéricas (método IQR).
- `TotalCharges` convertida de texto para número; `Churn` codificada para binária; 16 variáveis convertidas para categórica.
- 0 registos duplicados.
- 5 novos atributos criados: `TenureCohort`, `TotalServices`, `LTV_Estatico`, `ChargesPerService` e `RiskScore`.
- 2 variáveis removidas: `customerID` (sem valor preditivo) e `TotalCharges` (redundante face ao `LTV_Estatico`).
- Resultado final: 7043 linhas × 24 colunas.

### Principais Conclusões

- **Risco temporal:** clientes nos primeiros 12 meses apresentam uma taxa de abandono de **47.4%** — cerca de 5 vezes superior à dos clientes com mais de 4 anos de contrato.
- **Tipo de contrato:** contratos mensais atingem **~42% de abandono**, contra menos de 5% nos contratos anuais.
- **Índice de risco:** o `RiskScore` máximo (nível 6) corresponde a uma taxa de abandono de **70.2%** — resposta positiva à questão de investigação 4.
- **Fidelização por serviços:** clientes com 8 serviços subscritos têm apenas **5.3% de abandono**, contra 43.8% sem serviços adicionais.

Detalhes completos em `docs/M2_exploracao.md`.

---

## 3. Modelação (Milestone 3)

### Estratégia

Divisão estratificada 80/20 (`stratify=y, random_state=42`). Balanceamento de classes com SMOTE aplicado exclusivamente ao conjunto de treino (~4508 por classe). Normalização com `StandardScaler` ajustado no treino. Métrica principal: F1-Score na classe de abandono; métricas complementares: AUC-ROC e Recall.

### Algoritmos Avaliados

| Algoritmo | F1 Treino | F1 Teste | AUC-ROC | Desvio F1 |
| :--- | :---: | :---: | :---: | :---: |
| Regressão Logística (referência) | 0.8433 | **0.6032** | **0.8340** | 0.2401 |
| Gradiente Progressivo | 0.8602 | 0.5949 | 0.8325 | 0.2653 |
| SVM (RBF) | 0.8535 | 0.5858 | 0.8138 | 0.2677 |
| Floresta Aleatória | 0.9982 | 0.5815 | 0.8205 | 0.4167 |
| Naïve Bayes | 0.7682 | 0.5804 | 0.8076 | 0.1878 |
| XGBoost | 0.9610 | 0.5646 | 0.8066 | 0.3964 |
| KNN | 0.8759 | 0.5387 | 0.7687 | 0.3372 |
| Árvore de Decisão | 0.9983 | 0.4756 | 0.6413 | 0.5227 |

### Modelo Final

**Regressão Logística com limiar de decisão ajustado para 0.31.**

| Métrica | Valor |
| :--- | :---: |
| F1-Score (abandono) | **0.6126** |
| Recall (abandono) | **0.8075** ✅ |
| Precisão (abandono) | 0.4935 |
| AUC-ROC | 0.8340 |
| Clientes em risco identificados | 302 / 374 (80.7%) |

O ajuste do limiar de 0.50 para 0.31 permitiu atingir o objetivo de Recall ≥ 0.80, identificando mais 74 clientes em risco à custa de um aumento nos falsos positivos — troca justificada pelo custo de negócio de não detetar um cliente que vai abandonar.

Detalhes completos em `docs/M3_modelacao.md`.

---

## 4. Finalização (Milestone 4)

### Resposta ao Problema

O modelo construído — **Regressão Logística com limiar de decisão ajustado para 0.31** — demonstra que é possível prever o abandono de clientes com base em variáveis contratuais, de serviço e de faturação, identificando **302 dos 374 churners reais (80.7%)** no conjunto de teste. O objetivo SMART de F1-Score ≥ 0.75 não foi atingido (resultado: 0.61), mas o objetivo operacional de Recall ≥ 0.80 foi cumprido, que é o critério mais relevante para o negócio: minimizar os clientes em risco não detetados.

A análise revelou que os principais fatores de risco são o **tipo de contrato mensal** (taxa de abandono ~42%), a **antiguidade reduzida** (clientes com ≤ 12 meses têm taxa de 47.4%) e o **RiskScore elevado** (nível máximo: 70.2%). O principal fator protetor é a **subscrição de múltiplos serviços** (`TotalServices`, coeficiente −7.281): cada serviço adicional cria uma barreira de saída.

### Recomendações

1. **Intervir nos primeiros 12 meses:** Programas de *onboarding* e incentivos à subscrição de serviços adicionais para os clientes novos, que apresentam taxa de abandono 5× superior à dos clientes fidelizados.
2. **Estratégia de *bundling*:** Ofertas combinadas com desconto têm o maior retorno esperado na retenção — empiricamente confirmado pelo coeficiente de `TotalServices`.
3. **Vigilância de clientes de alto LTV:** Os 72 churners não detetados têm valor médio de 2 968€. Recomenda-se monitorização paralela por critérios de negócio diretos (tenure alto + contrato mensal + sem renovação recente).
4. **Ajuste dinâmico do limiar:** O threshold de 0.31 maximiza o Recall; em campanhas de custo elevado, deve ser reajustado conforme o orçamento disponível para equilibrar Precisão e Recall.

---

## Como Reproduzir este Projeto

1. Clonar o repositório: `git clone https://github.com/SalvadorCabecas/projeto_cdg_grupo9`
2. Instalar as dependências: `pip install -r requirements.txt`
3. Executar os cadernos na pasta `notebooks/` seguindo a ordem numérica.

---

## Referências

1. IBM. (s.d.). *Telco Customer Churn* [Conjunto de dados]. Kaggle. Disponível em: https://www.kaggle.com/datasets/blastchar/telco-customer-churn

2. Verbeke, W., Dejaeger, K., Martens, D., Hur, J., & Baesens, B. (2012). New insights into churn prediction in the telecommunication sector: A profit driven data mining approach. *European Journal of Operational Research*, 218(1), 211–229.

3. Chawla, N. V., Bowyer, K. W., Hall, L. O., & Kegelmeyer, W. P. (2002). SMOTE: Synthetic minority over-sampling technique. *Journal of Artificial Intelligence Research*, 16, 321–357.

4. Wirth, R., & Hipp, J. (2000). CRISP-DM: Towards a standard process model for data mining. In *Proceedings of the 4th International Conference on the Practical Applications of Knowledge Discovery and Data Mining* (pp. 29–39).

5. Melo, D. (2026). *Metodologia CRISP-DM* [Materiais de apoio]. ISCAC — Coimbra Business School.

---

## Identificação Académica

| | |
| :--- | :--- |
| **Instituição** | Coimbra Business School \| ISCAC |
| **Curso** | Licenciatura em Ciência de Dados para a Gestão |
| **Unidade Curricular** | Projeto em Ciência de Dados |
| **Docente Responsável** | Dora Melo (dmelo@iscac.pt) |
| **Ano Letivo** | 2025/2026 |

---

*Última atualização: 23/04/2026 
