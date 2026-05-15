# Perguntas e Respostas
## Projeto: Churn Prediction | ISCAC 2025/2026 | Grupo 9

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

Última atualização: 15/05/2026
