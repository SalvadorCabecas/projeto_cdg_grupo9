# Construção de um Modelo Preditivo de Abandono de Clientes no Setor das Telecomunicações

> Projeto académico de Ciência de Dados: construção e avaliação de um modelo de classificação supervisionada para prever o abandono de clientes de uma empresa de telecomunicações, com base nas suas características contratuais, de utilização e de faturação.

---

## Resumo Executivo

Construímos um modelo de *Machine Learning* que **identifica 8 em cada 10 clientes** de uma operadora de telecomunicações que estão prestes a cancelar o contrato, antes que estes saiam. Com base no perfil contratual e de serviços de 7043 clientes reais, o modelo permite à empresa focar as campanhas de retenção nos clientes de maior risco, evitando perdas de receita de forma eficiente e dirigida. O principal fator de proteção descoberto é a subscrição de múltiplos serviços: cada serviço adicional cria uma barreira de saída que reduz significativamente a probabilidade de abandono.

---


## Vídeo Pitch

[![Vídeo Pitch — Churn Prediction | Grupo 9](https://img.shields.io/badge/▶_Ver_Vídeo-Pitch_5_min-red?style=for-the-badge)](https://drive.google.com/file/d/1UGqhpdktsMnBJAcZUpJF4lnCIfRS5fek/view?usp=drive_link)

> Apresentação final do projeto em formato audiovisual (5 minutos).
> Demonstração do modelo a classificar clientes em tempo real.


---

## Equipa

| Nome | Número de Aluno |
| :--- | :---: |
| Vasco Firmino Coelho | 2023145934 |
| José Salvador Cabeças | 2023132088 |

---

## Resultado Final em Linguagem de Negócio

| O que o modelo faz | Resultado |
| :--- | :--- |
| Identifica clientes em risco de abandono | **302 em cada 374** clientes que vão sair são detetados (80.7%) |
| Taxa de deteção (*Recall*) | **8 em cada 10** clientes em risco de abandono identificados antes de saírem |
| Poder discriminativo (*AUC-ROC*) | De dois clientes escolhidos ao acaso (um que vai sair e outro que fica), o modelo distingue-os corretamente **83% das vezes** |
| Principal fator de risco | Contrato mensal + Fibra ótica + menos de 12 meses → **70% de probabilidade de abandono** |
| Principal fator de proteção | Cada serviço adicional subscrito **reduz significativamente** o risco de saída |
| Clientes de alto risco não detetados | 72 clientes com valor acumulado (*LTV*) médio de **2 968€** (monitorização paralela recomendada) |

---

## Kaggle

| Item | Ligação |
| :--- | :--- |
| Conjunto de dados | [Telco Customer Churn (IBM/Kaggle)](https://www.kaggle.com/datasets/blastchar/telco-customer-churn) |
| Caderno de exploração e preparação (M2) | [`notebooks/1.0_eda_limpeza.ipynb`](notebooks/1.0_eda_limpeza.ipynb) |
| Caderno de modelação e treino (M3) | [`notebooks/2.0_modelacao_treino.ipynb`](notebooks/2.0_modelacao_treino.ipynb) |

---

## Navegação Rápida

| Documento | Conteúdo |
| :--- | :--- |
| [`docs/M1_iniciacao.md`](docs/M1_iniciacao.md) | Problema de negócio, objetivos SMART, dicionário de dados |
| [`docs/M2_exploracao.md`](docs/M2_exploracao.md) | Análise exploratória, limpeza, engenharia de atributos |
| [`docs/M3_modelacao.md`](docs/M3_modelacao.md) | Comparação de 8 algoritmos, modelo final, métricas técnicas |
| [`docs/M4_conclusoes.md`](docs/M4_conclusoes.md) | Síntese de valor, limitações, ética e roteiro futuro |
| [`reports/figures/`](reports/figures/) | Todas as figuras exportadas (curvas de aprendizagem, matriz de confusão, importância de atributos) |

---

## Estado do Projeto

| Fase | Foco CRISP-DM | Prazo | Estado |
| :--- | :--- | :---: | :---: |
| **M1: Iniciação** | Compreensão do negócio e dos dados | 24/02/2026 | Concluído |
| **M2: Exploração** | Análise exploratória e preparação dos dados | 24/03/2026 | Concluído |
| **M3: Modelação** | Modelação e avaliação | 23/04/2026 | Concluído |
| **M4: Finalização** | Comunicação e entrega | 18/05/2026 | Concluído |

---

## Organização do Repositório

```
projeto_cdg_grupo9/
├── data/
│   ├── raw/                  # Conjunto de dados original IBM/Kaggle
│   └── processed/            # Conjunto de dados processado (7043 × 24 colunas)
├── docs/
│   ├── M1_iniciacao.md       # Compreensão do negócio
│   ├── M2_exploracao.md      # Preparação dos dados
│   ├── M3_modelacao.md       # Modelação e avaliação
│   └── M4_conclusoes.md      # Conclusão e entrega de valor
├── notebooks/
│   ├── 1.0_eda_limpeza.ipynb       # Exploração e limpeza (M2)
│   └── 2.0_modelacao_treino.ipynb  # Modelação e treino (M3)
├── reports/figures/          # Figuras exportadas (PNG)
├── src/                      # Reservado, não utilizado neste projeto
└── requirements.txt          # Dependências Python
```

---

## 1. Iniciação (Milestone 1)

### Contexto e Problema de Negócio

O setor das telecomunicações caracteriza-se por mercados saturados e elevada concorrência. O abandono de clientes representa uma perda direta de receita recorrente e implica custos operacionais significativos. Identificar antecipadamente os clientes com maior propensão para abandonar permite adotar medidas de retenção eficientes e dirigidas.

Este projeto utiliza o conjunto de dados público *Telco Customer Churn* (IBM/Kaggle) com 7043 clientes e 21 variáveis contratuais, de serviços e de faturação.

### Objetivo SMART

Construir um modelo de classificação supervisionada para prever o abandono de clientes, atingindo um F1 igual ou superior a 0.75 na classe positiva (abandono = 1) em validação cruzada estratificada com k=5, utilizando o conjunto de dados *Telco Customer Churn* da IBM, até ao dia 23/04/2026.

### Questões de Investigação

1. Quais as variáveis com maior associação estatística com o abandono?
2. Existe um limiar de antiguidade abaixo do qual o risco é substancialmente superior?
3. A combinação de contrato mensal com ausência de suporte técnico aumenta o risco?
4. Um índice de risco composto consegue estratificar clientes com taxa de abandono superior a 60% no nível máximo?
5. As novas variáveis criadas aumentam o poder preditivo do modelo?
6. É possível atingir *Recall* ≥ 0.80 com balanceamento de classes e ajuste do limiar de decisão?

### Observações Iniciais ao Conjunto de Dados

- **Dimensão:** 7043 registos e 21 variáveis; variável alvo `Churn` com desequilíbrio de classes (~73.5% sem abandono / ~26.5% com abandono).
- **Problema de tipagem:** a variável `TotalCharges` estava armazenada como texto devido à presença de espaços em branco em 11 registos com `tenure = 0`, exigindo conversão e imputação.
- **Distribuição temporal:** a variável `tenure` apresenta uma distribuição bimodal, com concentração de clientes novos (primeiros meses) e clientes com 6 anos de contrato, sugerindo um padrão de abandono precoce.
- **Ausência de valores nulos reais e duplicados** nas restantes variáveis após auditoria inicial.
- **Desequilíbrio de classes:** a proporção de 26.5% de abandono determinou a escolha do F1 como métrica principal em detrimento da exatidão global.

Detalhes completos em [`docs/M1_iniciacao.md`](docs/M1_iniciacao.md).

---

## 2. Exploração (Milestone 2)

### Principais Descobertas

- **Risco temporal:** clientes nos primeiros 12 meses têm taxa de abandono de **47.4%**, 5× superior à dos clientes fidelizados.
- **Tipo de contrato:** contratos mensais atingem **~42% de abandono**, contra menos de 5% nos anuais.
- **Índice de risco:** `RiskScore` nível máximo → **70.2% de abandono** (questão de investigação 4 validada ✅).
- **Fidelização por serviços:** 8 serviços subscritos → apenas **5.3% de abandono**.

### Atributos Criados

| Atributo | O que mede |
| :--- | :--- |
| `TenureCohort` | Fase do ciclo de vida do cliente (*Early* / *Growing* / *Mature* / *Loyal*) |
| `TotalServices` | Número de serviços subscritos (indicador de fidelização comportamental) |
| `LTV_Estatico` | Valor de vida acumulado estimado (`MonthlyCharges × tenure`) |
| `ChargesPerService` | Rácio custo/serviço (indicador de relação custo-benefício desfavorável) |
| `RiskScore` | Índice de risco composto (0 a 6): contrato + internet + antiguidade |

Detalhes em [`docs/M2_exploracao.md`](docs/M2_exploracao.md).

---

## 3. Modelação (Milestone 3)

### Algoritmos Avaliados (8 no total)

| Algoritmo | F1 Treino | F1 Teste | AUC-ROC |
| :--- | :---: | :---: | :---: |
| **Regressão Logística** ⭐ | 0.8433 | **0.6032** | **0.8340** |
| Gradiente Progressivo | 0.8602 | 0.5949 | 0.8325 |
| SVM (RBF) | 0.8535 | 0.5858 | 0.8138 |
| Floresta Aleatória | 0.9982 | 0.5815 | 0.8205 |
| Naïve Bayes | 0.7682 | 0.5804 | 0.8076 |
| XGBoost | 0.9610 | 0.5646 | 0.8066 |
| KNN | 0.8759 | 0.5387 | 0.7687 |
| Árvore de Decisão | 0.9983 | 0.4756 | 0.6413 |

### Modelo Final: Resultados em Linguagem de Negócio

**Regressão Logística com limiar de decisão ajustado para 0.31.**

| Métrica técnica | Significado prático |
| :--- | :--- |
| *Recall* = **0.8075** ✅ | O modelo deteta **8 em cada 10** clientes que vão abandonar |
| *AUC-ROC* = **0.8340** | Distingue corretamente clientes em risco de abandono em **83%** dos casos |
| Clientes identificados: **302 / 374** | De 374 clientes que iam sair, o modelo sinalizou **302 com antecedência** |
| Falsos negativos: **72** | 72 clientes saíram sem serem detetados, com valor de vida (*LTV*) médio de **2 968€** |
| Falsos positivos: **310** | Cada descida de 0.01 no limiar troca precisão por sensibilidade; a decisão sobre o equilíbrio adequado depende do custo real de uma campanha de retenção desnecessária, que varia por empresa |

Detalhes técnicos completos em [`docs/M3_modelacao.md`](docs/M3_modelacao.md).

---

## 4. Finalização (Milestone 4)

### Resposta ao Problema

O modelo identifica **302 dos 374 clientes** que efetivamente vão abandonar (80.7%), dando à empresa uma janela de intervenção antes que a decisão de sair seja irreversível. O objetivo de *Recall* ≥ 0.80 foi atingido, o critério de negócio mais relevante para minimizar os clientes em risco não detetados.

### Recomendações

1. **Intervir nos primeiros 12 meses:** taxa de abandono de 47.4% nesta fase, 5× superior à dos clientes fidelizados. Programas de acolhimento inicial e incentivos à subscrição de serviços adicionais têm o maior retorno.
2. **Estratégia de pacotes combinados:** cada serviço adicional cria uma barreira de saída (coeficiente `TotalServices` = −7.219). Ofertas agrupadas com desconto são a intervenção com maior impacto comprovado.
3. **Vigilância paralela de alto valor:** os 72 clientes em risco de abandono não detetados têm valor de vida (*LTV*) médio de 2 968€. Recomenda-se monitorização por critérios diretos: antiguidade elevada + contrato mensal + sem renovação recente.
4. **Ajuste dinâmico do limiar de decisão:** o limiar de 0.31 maximiza o *Recall*. Em campanhas de custo elevado, deve ser reajustado para equilibrar precisão e sensibilidade.

Análise completa em [`docs/M4_conclusoes.md`](docs/M4_conclusoes.md).

---

## Como Reproduzir este Projeto

```bash
# 1. Clonar o repositório
git clone https://github.com/SalvadorCabecas/projeto_cdg_grupo9

# 2. Instalar as dependências
pip install -r requirements.txt

# 3. Executar os cadernos na ordem numérica
#    notebooks/1.0_eda_limpeza.ipynb        : exploração e preparação dos dados
#    notebooks/2.0_modelacao_treino.ipynb   : modelação, avaliação e modelo final
```

Os cadernos detetam automaticamente o ambiente (Kaggle ou local) e ajustam os caminhos de ficheiros sem necessidade de configuração manual.

---

## Referências

1. IBM. (s.d.). *Telco Customer Churn* [Conjunto de dados]. Kaggle. Disponível em: <https://www.kaggle.com/datasets/blastchar/telco-customer-churn>

2. Verbeke, W., Dejaeger, K., Martens, D., Hur, J., & Baesens, B. (2012). New insights into churn prediction in the telecommunication sector. *European Journal of Operational Research*, 218(1), 211–229.

3. Chawla, N. V., Bowyer, K. W., Hall, L. O., & Kegelmeyer, W. P. (2002). SMOTE: Synthetic minority over-sampling technique. *Journal of Artificial Intelligence Research*, 16, 321–357.

4. Wirth, R., & Hipp, J. (2000). CRISP-DM: Towards a standard process model for data mining. *Proceedings of the 4th International Conference on Knowledge Discovery and Data Mining* (pp. 29–39).

5. Melo, D. (2026). *Metodologia CRISP-DM* [Materiais de apoio]. ISCAC, Coimbra Business School.

6. Gallo, A. (2014). *The value of keeping the right customers*. *Harvard Business Review*. Disponível em: <https://hbr.org/2014/10/the-value-of-keeping-the-right-customers>

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

Última atualização: 18/05/2026
