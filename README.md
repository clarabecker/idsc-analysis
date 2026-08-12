# Predição do IDEB e Análise de Determinantes Sociais via Machine Learning

## Visão Geral do Projeto

Este repositório contém a implementação de um projeto focado em investigar como os determinantes sociais e a infraestrutura municipal afetam a qualidade da educação básica. Utilizando técnicas de Aprendizado de Máquina Não Supervisionado e Supervisionado, o objetivo é entender o cenário e fundamentar a escolha de algoritmos para obter os melhores resultados na predição do Índice de Desenvolvimento da Educação Básica (IDEB), levando em consideração uma separação por agrupamento consistente dos dados.

### Metodologia
O pipeline analítico e preditivo do projeto está estruturado nas seguintes etapas:

#### 1. Análise Exploratória de Dados (EDA): 
Realizada com o objetivo de compreender a distribuição dos indicadores do IDSC e identificar padrões preliminares na base de dados.

#### 2. Pré-processamento:
Executado visando à adequação dos dados para a execução dos modelos de aprendizado de máquina, garantindo a confiabilidade e o desempenho dos algoritmos.

#### 3. Estratégia de Seleção de variáveis 

A seleção de atributos utilizou a Matriz de Correlação de Pearson para eliminar variáveis redundantes e mitigar a maldição da dimensionalidade nos algoritmos. Em seguida, a Análise de Componentes Principais (PCA) identificou os eixos de maior variância, extraindo os indicadores com maior carga fatorial (loadings) para garantir aos modelos dados independentes e de alto valor preditivo.

#### 4. Agrupamento Não Supervisionado

Identificação de perfis de municípios semelhantes através de técnicas de aprendizado não supervisionado.Algoritmos Analisados:

* K-Means
* Agrupamento Hierárquico
* DBSCAN (Agrupamento Espacial Baseado em Densidade para Aplicações com Ruído).

#### 5. Etapa Preditiva Inicial (Baseline Global)

Aplicação de modelos de regressão na base de dados integral para a obtenção de um resultado preditivo de referência.

Modelo de Referência (Baseline): 
* Regressão Linear, escolhida por se caracterizar como um modelo mais simples em comparação com os demais.
Modelos Baseados em Árvores de Decisão:
* Random Forest
* XGBoost (Extreme Gradient Boosting)
* LightGBM (Light Gradient Boosting Machine)
* CatBoost (Categorical Boosting)

Métricas de Avaliação: Erro Médio Absoluto ($MAE$), Raiz do Erro Quadrático Médio ($RMSE$) e Coeficiente de Determinação ($R^2$).

#### 5. Predição Localizada por Subconjunto

Realização da predição em cada subconjunto da base de dados, definidos pelo modelo de agrupamento que apresentar o melhor desempenho na etapa anterior.

- Estratégia: Serão replicados os mesmos modelos de regressão aplicados na base integral. - - Objetivo: Permitir avaliar e comparar o impacto do agrupamento dos municípios em bases de dados menores na precisão dos resultados (avaliados por $MAE$, $RMSE$ e $R^2$).

#### 6. Interpretabilidade dos Modelos

Extração da relevância dos atributos (indicadores do IDSC) na estimativa do IDEB por meio de duas abordagens complementares:
- Métricas Intrínsecas (Redução de Variância): Análise da importância estrutural dos atributos calculada a partir da redução da variância dos dados nos nós de decisão dos algoritmos de árvore.
- Método TreeSHAP: Adotado para mitigar vieses estruturais das métricas nativas e garantir maior consistência teórica. Os valores SHAP quantificam a magnitude e a direção do impacto de cada variável, permitindo identificar de forma clara se um indicador atua de forma positiva ou negativa no desempenho educacional.


