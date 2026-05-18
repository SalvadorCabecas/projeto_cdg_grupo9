# Relatório de Conclusão e Entrega de Valor

> **Milestone 4: Finalização | *Churn Prediction* | ISCAC 2025/2026**
> Este documento encerra o ciclo CRISP-DM do projeto, convertendo os resultados técnicos em valor estratégico e conhecimento acionável. Consultar `docs/M3_modelacao.md` para os detalhes técnicos completos.

---

## 1. Síntese de Resultados e Impacto

### O Problema Resolvido

O projeto partiu de uma pergunta de negócio concreta: **é possível identificar, com antecedência, quais os clientes de uma operadora de telecomunicações que vão cancelar o contrato?**

A resposta é sim. O modelo construído (uma Regressão Logística com limiar de decisão ajustado) consegue identificar **8 em cada 10 clientes** que efetivamente vão abandonar o serviço, antes de esse abandono acontecer. Isto dá à empresa uma janela de intervenção: contactar o cliente, oferecer um incentivo ou resolver um problema antes que a decisão de sair seja irreversível.

### Interpretação dos Resultados em Palavras Simples

Antes deste modelo, a empresa ou tratava todos os clientes da mesma forma (sem priorização), ou tentava adivinhar quem estava em risco com base na experiência empírica. Agora tem um sistema que, ao analisar o perfil de cada cliente (tempo de contrato, serviços subscritos, tipo de contrato e serviço de internet), atribui automaticamente uma probabilidade de abandono.

Os clientes sinalizados pelo modelo como "em risco" não são uma adivinhação: são os que combinam os fatores mais associados ao abandono (contrato mensal, serviço de fibra ótica, poucos serviços adicionais e menos de 12 meses de contrato). Neste grupo de risco máximo, 70 em cada 100 clientes efetivamente saem.

### Valor para o Negócio

| O que a empresa consegue fazer agora | Impacto |
| :--- | :--- |
| Priorizar campanhas de retenção nos 302 clientes identificados como em risco | Evitar a perda de receita recorrente associada a ~80% dos abandonos |
| Intervir nos clientes nos primeiros 12 meses (taxa de abandono 47.4%) | Reduzir o abandono precoce com programas de acolhimento inicial dirigidos |
| Incentivar a subscrição de serviços adicionais (coeficiente `TotalServices` = −7.281) | Cada serviço adicional reduz significativamente o risco de abandono |
| Vigilância paralela dos 72 clientes de alto valor não detetados (*LTV* médio ~2 968€) | Criar canal de monitorização dedicado ao segmento de maior valor |

### Avaliação Face aos Objetivos Iniciais

| Objetivo | Meta | Resultado | Estado |
| :--- | :---: | :---: | :---: |
| F1 em validação cruzada (k=5) | ≥ 0.75 | **0.8433** | ✅ Atingido |
| *Recall* no conjunto de teste | ≥ 0.80 | **0.8075** | ✅ Atingido |
| *RiskScore* nível máximo: taxa de abandono | > 60% | **70.2%** | ✅ Atingido |
| F1 no conjunto de teste real | referência | 0.6126 | ℹ️ Ver nota |

> **Nota sobre o F1 no teste real:** O F1 de 0.61 no conjunto de teste é inferior ao objetivo de 0.75 definido para validação cruzada interna. Esta diferença resulta do *distribution shift*: o treino foi balanceado pelo *SMOTE* (50/50), mas o teste tem a distribuição real (26.5% abandono). O fenómeno afetou todos os 8 algoritmos testados. O critério de negócio prioritário (Recall ≥ 0.80) foi cumprido através do ajuste do limiar de decisão.

---

## 2. Análise Crítica e Limitações

### Limitações dos Dados

**Período único e empresa única:** O conjunto de dados representa um único momento temporal de uma única operadora norte-americana. O modelo aprendeu os padrões de abandono *desta* empresa, *neste* momento. Aplicado a uma operadora diferente ou ao mesmo mercado noutra época, o desempenho pode ser substancialmente diferente.

**Ausência de variáveis comportamentais:** O modelo não tem acesso a informação sobre interações do cliente com o suporte (número de reclamações, tempo de resolução, NPS), nem sobre a qualidade percebida do serviço ou ofertas de concorrentes. Estas variáveis são frequentemente decisivas para o abandono e a sua ausência limita o poder preditivo máximo alcançável.

**Dados transversais:** O conjunto de dados é uma fotografia de um único instante. Um modelo longitudinal (com registos mensais por cliente) teria potencialmente mais poder preditivo, especialmente para detetar mudanças súbitas de comportamento.

**Sobre o `LTV_Estatico`:** O `LTV_Estatico` criado como `MonthlyCharges × tenure` é matematicamente equivalente à variável original `TotalCharges`, com correlação perfeita r=1.000. A criação desta variável melhorou a semântica e resolveu o problema de tipagem da variável original, mas não acrescentou informação preditiva nova ao modelo. Adicionalmente, dado que `MonthlyCharges` e `tenure` permanecem no conjunto de variáveis do modelo, existe redundância parcial que poderia ser quantificada através do VIF (*Variance Inflation Factor*) em trabalho futuro.

### Limitações do Modelo

**Relações lineares apenas:** A Regressão Logística assume que a relação entre cada variável e a probabilidade de abandono é linear após transformação logística. Padrões não-lineares complexos não são capturados, o que contribui para o resultado de F1 = 0.61 no teste real.

**Impacto do *distribution shift*:** O modelo foi treinado com dados artificialmente balanceados pelo *SMOTE* (50% abandono / 50% retenção), mas o mundo real tem 26.5% de abandono. Esta discrepância estrutural explica porque o desempenho em validação cruzada (~0.85) é substancialmente superior ao desempenho no teste real (0.61). Nenhum dos 8 algoritmos testados conseguiu superar este teto, o que sugere que a estratégia de balanceamento é o principal fator limitativo.

### Contextos de Falha

O modelo falha sistematicamente num segmento específico: **clientes de longa data com contrato mensal e *LTV* elevado.** Os 72 clientes não detetados têm em média 36 meses de contrato e ~2 968€ de valor gerado, o triplo dos clientes detetados. O modelo interpreta a sua antiguidade como sinal de fidelidade e atribui-lhes uma probabilidade de abandono de apenas 18% (abaixo do limiar de 31%). Saem por razões que as variáveis disponíveis não conseguem capturar: uma oferta mais atrativa de um concorrente, uma mudança de circunstâncias pessoais ou uma degradação gradual da qualidade percebida do serviço. São paradoxalmente os clientes de maior valor e os mais difíceis de identificar.

---

## 3. Considerações Éticas e de Viés

### Privacidade e Proteção de Dados

O conjunto de dados utilizado é público e totalmente anonimizado: o `customerID` é uma cadeia alfanumérica sem qualquer ligação a dados pessoais identificáveis. O modelo analisa exclusivamente padrões de comportamento contratual e de utilização de serviços, em conformidade com o RGPD. Numa implementação real em produção, seria necessário garantir que os dados de clientes são tratados com consentimento explícito e base legal adequada.

### Transparência e Explicabilidade

A Regressão Logística é um dos modelos mais transparentes disponíveis: cada decisão é explicável através dos seus coeficientes. Não existe caixa negra (é possível mostrar a qualquer cliente ou auditor exatamente quais as variáveis que contribuíram para a classificação e com que peso). O `RiskScore` composto é igualmente explicável: o cliente pode verificar os três critérios que o compõem (tipo de contrato, tipo de internet, antiguidade) e compreender porque foi sinalizado.

Esta transparência é particularmente relevante em contextos regulatórios, onde a explicabilidade das decisões automatizadas pode ser legalmente exigida.

### Viés Potencial

**Viés geográfico e cultural:** Os dados são de uma operadora norte-americana. Os padrões de abandono podem ser diferentes em mercados com regulação distinta ou culturas de contratação diferentes, como Portugal.

**Viés de seleção temporal:** O conjunto de dados não indica o período a que se refere. Se os dados foram recolhidos num período atípico (crise, campanha de concorrente, mudança regulatória), os padrões capturados podem não ser representativos do comportamento normal.

**Risco de uso discriminatório:** Um modelo de *churn* não deve ser utilizado para negar serviços, aplicar preços diferenciados não autorizados ou penalizar clientes identificados como "em risco". O seu propósito legítimo é a retenção proativa (oferecer mais valor aos clientes em risco, não penalizá-los). A variável `SeniorCitizen` está presente no modelo; numa implementação real, seria obrigatória uma auditoria de equidade (*fairness audit*) para garantir que a taxa de falsos negativos não é sistematicamente superior em grupos demográficos específicos.

---

## 4. Roadmap e Trabalhos Futuros

### 1. Melhoria Técnica: Eliminação do *Distribution Shift*

O principal obstáculo de desempenho é o *distribution shift* introduzido pelo *SMOTE*. A melhoria técnica prioritária seria substituir o *SMOTE* por `class_weight='balanced'` na Regressão Logística, que penaliza os erros na classe minoritária sem alterar a distribuição dos dados de treino. Esta abordagem elimina o *distribution shift* e poderia aproximar o F1 no teste real dos valores obtidos em validação cruzada. A calibração do modelo com `CalibratedClassifierCV` tornaria também as probabilidades previstas mais fiáveis como estimativas reais de risco.

### 2. Novas Variáveis: Dados Comportamentais e Temporais

As variáveis com maior potencial de melhoria são as que atualmente não existem no conjunto de dados:

- **Histórico de reclamações:** número de contactos com suporte, tempo médio de resolução, NPS. Clientes insatisfeitos com o suporte têm risco muito superior (o coeficiente positivo de `TechSupport_Yes` no modelo atual é um indicador indireto desta fricção).
- **Dados longitudinais:** transformar o conjunto de dados num painel com registos mensais por cliente permitiria detetar mudanças de padrão, como um cliente que reduziu subitamente o consumo de serviços ou aumentou as chamadas ao suporte nos últimos três meses.
- **Dados de mercado:** informação sobre campanhas promocionais ativas de operadores concorrentes, que são frequentemente o gatilho imediato do abandono.

### 3. Disponibilização: Da Previsão à Ação em Produção

O modelo atual é um protótipo académico. Para ser utilizável em contexto real, precisaria de:

- **API REST de classificação:** encapsular o modelo num serviço que receba o perfil de um cliente e devolva em tempo real a probabilidade de abandono e o RiskScore. Tecnologias candidatas: FastAPI (Python) com disponibilização em AWS Lambda ou Azure Functions.
- **Painel de monitorização (Streamlit):** interface para a equipa de retenção visualizar os clientes de maior risco, filtrar por segmento e registar as ações tomadas (lista dos clientes prioritários, RiskScore, *LTV* e probabilidade prevista).
- **Sistema de reentrenamento periódico:** mecanismo para registar os resultados das campanhas de retenção e usar esses dados para reentrenar o modelo, evitando a degradação de desempenho ao longo do tempo (*concept drift*).

---

**Data de Conclusão:** 18/05/2026
