# Perguntas e Respostas
## Projeto: *Churn Prediction* | ISCAC 2025/2026 | Grupo 9

---

## Pergunta 1

**"Porque é que o modelo mais simples ganhou aos modelos complexos?"**

### Resposta

O modelo mais simples ganhou por uma combinação de três fatores que se reforçam mutuamente.

O *SMOTE* equilibrou as classes no treino para 50/50, uma distribuição artificial. Os modelos baseados em árvores — Floresta Aleatória, XGBoost, Árvore de Decisão — são muito eficazes a memorizar padrões e memorizaram exatamente essa distribuição sintética. Quando chegaram ao teste com a distribuição real (26.5% abandono), falharam na generalização. A Regressão Logística aprende fronteiras lineares e é menos susceptível a este efeito de *distribution shift*.

As relações mais preditivas neste conjunto de dados — antiguidade baixa, contrato mensal, poucos serviços — têm uma estrutura aproximadamente linear com o abandono. Complexidade adicional não acrescenta valor quando o problema é linearmente separável.

A normalização (*StandardScaler*) beneficia diretamente os modelos lineares. Para os modelos baseados em árvores, é neutra.

O processo de otimização do Gradiente Progressivo confirmou este diagnóstico: os parâmetros óptimos encontrados foram conservadores (profundidade máxima = 3, valor por omissão), e o desempenho no teste real foi inferior ao da linha de base. Mais complexidade não resolve um problema de *distribution shift*.

---

## Pergunta 2

**"O LTV\_Estático não introduz multicolinearidade se as variáveis que o compõem já estão no modelo?"**

### Resposta

É uma observação correcta, que reconhecemos como limitação do projeto.

O `LTV_Estatico` é calculado como `MonthlyCharges × tenure`. Ambas as variáveis permanecem no modelo após a seleção de atributos, pelo que existe correlação parcial entre as três — não é multicolinearidade perfeita, mas é uma redundância parcial.

A justificação para manter as três variáveis é que o produto captura uma interação que as variáveis isoladas não capturam: um cliente com 60 meses de contrato e 80€/mês tem um perfil distinto de um cliente com 60 meses e 20€/mês, mesmo que a antiguidade seja idêntica. O `LTV_Estatico` codifica essa combinação num único valor com semântica de negócio direta.

A limitação reconhecida é que esta variável olha apenas para o passado — o valor que o cliente já gerou — e não para o valor futuro esperado. Um valor de vida dinâmico exigiria modelação financeira adicional com dados que não estão disponíveis neste conjunto. Esta limitação está documentada em [`docs/M4_conclusoes.md`](M4_conclusoes.md) e identificada como melhoria futura, nomeadamente através do cálculo explícito do VIF (*Variance Inflation Factor*).

---

## Pergunta 3

**"O F1-Score de 0.60 está muito abaixo do objetivo SMART de 0.75 — o objetivo não foi cumprido?"**

### Resposta

O objetivo SMART de F1 ≥ 0.75 foi definido para a validação cruzada interna, não para o teste final. Em validação cruzada estratificada com k=5 no conjunto de treino balanceado, a Regressão Logística atingiu F1 = 0.8433 — acima do limiar definido.

O F1 de 0.60 no teste final resulta do *distribution shift*: o treino tem distribuição artificial 50/50 (após *SMOTE*) e o teste tem a distribuição real de 26.5% de abandono. Esta diferença de distribuições faz descer o F1 de forma sistemática em todos os algoritmos testados — é um fenómeno esperado e documentado na literatura, não uma falha específica do modelo.

O critério operacional cumprido é o *Recall* ≥ 0.80, que corresponde ao objetivo de negócio mais relevante: minimizar os clientes em risco não detetados. Com o limiar ajustado para 0.31, o modelo atingiu *Recall* = 0.8075, satisfazendo a questão de investigação 6. A escolha de maximizar o *Recall* em detrimento do F1 é uma decisão de negócio consciente: num contexto de retenção, o custo de não identificar um cliente em risco é sistematicamente superior ao custo de uma campanha de retenção desnecessária.

---

## Pergunta 4

**"Porque usaram *SMOTE* em vez de simplesmente penalizar a classe minoritária no algoritmo?"**

### Resposta

O SMOTE foi a abordagem escolhida, mas reconhecemos que `class_weight='balanced'` seria metodologicamente mais limpo para este problema.

O `class_weight='balanced'` penaliza os erros na classe minoritária sem alterar a distribuição dos dados de treino, eliminando o *distribution shift* que foi o principal obstáculo de desempenho neste projeto. Com esta abordagem, o modelo seria treinado e avaliado sobre a mesma distribuição real de 26.5% de abandono, e o F1 no conjunto de teste seria comparável ao obtido em validação cruzada.

A opção pelo SMOTE foi tomada na fase de modelação como técnica padrão para desequilíbrio de classes, seguindo a literatura da área. O custo desta decisão — o *distribution shift* — só se tornou evidente ao comparar sistematicamente os resultados de validação cruzada com os do teste real. A substituição por `class_weight='balanced'` está identificada como a primeira melhoria técnica prioritária em [`docs/M4_conclusoes.md`](M4_conclusoes.md).

---

## Pergunta 5

**"O conjunto de dados é uma fotografia de um único momento — como justificam usar a variável `tenure` para inferir comportamento ao longo do tempo?"**

### Resposta

A limitação é real e está documentada no projeto como dado transversal (*cross-sectional*).

A variável `tenure` regista há quantos meses o cliente está com a operadora no momento da recolha dos dados. Não é uma série temporal — não temos acesso à evolução do comportamento do mesmo cliente ao longo do tempo. O que fazemos é inferir padrões de coorte: clientes com `tenure` baixo (0–12 meses) têm uma taxa de abandono de 47.4%, enquanto clientes com `tenure` elevado (mais de 48 meses) têm apenas 9.5%. Este padrão permite segmentar o risco por fase do ciclo de vida, mas não permite distinguir se um cliente específico com 3 anos de contrato está a tornar-se mais ou menos propenso ao abandono.

Um modelo longitudinal — com registos mensais por cliente — teria potencialmente mais poder preditivo, especialmente para detetar mudanças súbitas de comportamento. A recolha deste tipo de dados e a sua incorporação num modelo de painel estão identificadas como melhoria futura em [`docs/M4_conclusoes.md`](M4_conclusoes.md).

---

## Pergunta 6

**"A variável `SeniorCitizen` está no modelo — como garantem que o modelo não discrimina clientes com base na idade?"**

### Resposta

A questão é legítima e foi considerada na análise ética do projeto.

O modelo usa `SeniorCitizen` como variável preditiva porque tem associação estatística com o abandono nos dados — não porque exista uma relação causal com a idade em si. Na maioria dos casos, a correlação é mediada por outras variáveis: clientes idosos tendem a ter contratos de longa data e a subscrever menos serviços digitais, fatores que o modelo já capta através de `tenure` e `TotalServices`.

O uso legítimo deste modelo é a retenção proativa — identificar clientes em risco para lhes oferecer mais valor, não para penalizá-los. Aplicar o modelo para negar serviços, aplicar preços diferenciados não autorizados ou discriminar clientes com base em características demográficas seria um uso indevido, independentemente da variável em causa.

Numa implementação real em produção, a auditoria de equidade (*fairness audit*) seria obrigatória: verificar se a taxa de falsos negativos — clientes em risco não detetados — é sistematicamente superior em algum grupo demográfico, o que constituiria viés discriminatório. Este passo não foi realizado neste projeto académico e está identificado como requisito para qualquer implementação real.

---

Última atualização: 15/05/2026
