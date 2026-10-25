# Classificação do IDEB em Municípios de Santa Catarina com Modelos Baseados em Árvores

Análise de como os indicadores do Índice de Desenvolvimento Sustentável das Cidades (IDSC-BR 2025) se relacionam com o desempenho educacional (IDEB) dos municípios catarinenses. O projeto classifica os municípios de Santa Catarina em faixas do IDEB com Random Forest, XGBoost, LightGBM e CatBoost, compara experimentos com todos os atributos e com atributos selecionados, investiga agrupamentos de municípios (clustering) e identifica quais indicadores mais influenciam os modelos.

Projeto desenvolvido como Trabalho de Conclusão de Curso de Engenharia de Software (UDESC Ibirama).

## Sumário

- [Visão geral](#visão-geral)
- [Dados](#dados)
- [Estrutura do repositório](#estrutura-do-repositório)
- [Pipeline e notebooks](#pipeline-e-notebooks)
- [Metodologia](#metodologia)
- [Como executar](#como-executar)
- [Reprodutibilidade](#reprodutibilidade)
- [Limitações](#limitações)

## Visão geral

**Pergunta de pesquisa:** quais indicadores do IDSC-BR estão mais associados ao desempenho no IDEB dos municípios catarinenses, e com que qualidade é possível classificar esse desempenho a partir deles?

O IDEB é analisado em duas etapas de ensino, tratadas como alvos independentes:

- **AF**: anos finais do ensino fundamental (`SDG4_15_IDEB_AF`);
- **AI**: anos iniciais do ensino fundamental (`SDG4_16_IDEB_AI`).

O estudo se restringe aos **295 municípios de Santa Catarina**, para reduzir a heterogeneidade em relação à base nacional.

## Dados

| Arquivo | Descrição |
|---|---|
| `data/raw/Base_de_Dados_IDSC-BR_2025.xlsx` | Base original do IDSC-BR 2025 (5.570 municípios, aba "Todos os Dados"). Não é alterada pelo projeto. |
| `data/processed/Base_de_Dados_Processada.csv` | Base limpa e usada nos modelos (metadados e colunas brutas removidos). |
| `data/processed/Base_de_Alvos_IDEB.csv` | Nota bruta do IDEB e classificação do Painel, **em arquivo separado** e ligados ao município por `id_municipio`. |
| `data/processed/features_selecionadas_sc.json` | Lista de atributos do experimento com atributos selecionados. |
| `data/processed/importancia_*.csv` | Resultados da análise de importância dos atributos. |

> A nota bruta do IDEB e o Painel **nunca** entram como atributos: ficam em arquivo separado para evitar vazamento de informação.

### Alvos de classificação

Por padrão, a nota bruta do IDEB é discretizada em cinco faixas:

| Faixa | Nota do IDEB |
|---|---|
| Insuficiente / Crítico | abaixo de 4,0 |
| Básico / Baixo | 4,0 a 4,9 |
| Intermediário | 5,0 a 5,9 |
| Adequado (Meta PNE) | 6,0 a 6,9 |
| Avançado / Alto Desempenho | 7,0 ou mais |

Apenas municípios com IDEB divulgado para a etapa correspondente são usados, e classes com menos de dois municípios são removidas (inviabilizam a divisão estratificada). Em Santa Catarina isso resulta em:

| Alvo | Municípios | Classes |
|---|---|---|
| AF | 151 | Insuficiente (7), Básico (55), Intermediário (75), Adequado (14) |
| AI | 272 | Intermediário (65), Adequado (176), Avançado (31) |

A variável `MODO_ALVO` nos notebooks de predição alterna entre `'faixas_bruta'` (padrão) e `'painel'` (as quatro cores do Painel do IDSC-BR).

## Estrutura do repositório

```
idsc-analysis/
├── data/
│   ├── raw/                  # base original do IDSC-BR (não alterada)
│   └── processed/            # bases processadas, alvos, atributos selecionados e resultados
├── notebooks/                # etapas do pipeline, na ordem de execução
└── README.md
```

## Pipeline e notebooks

Os notebooks seguem o fluxo de dados e cada um lê os arquivos gerados pela etapa anterior.

| Notebook | Etapa |
|---|---|
| `01_data_analisys.ipynb` | Análise exploratória: Brasil vs. Santa Catarina, completude dos dados e correlações com o IDEB. |
| `02_data_cleaning.ipynb` | Limpeza: remoção de metadados, colunas brutas e colunas com risco de vazamento (`Goal 4 Score`, `ODS4_reg`). |
| `03_targets_definition.ipynb` | Geração de `Base_de_Alvos_IDEB.csv` (nota bruta e Painel) e conferência das distribuições. |
| `05_feature_selection.ipynb` | Seleção de atributos: filtro de correlação, remoção dos indicadores do ODS 4 e PCA exploratório. |
| `04_global_prediction.ipynb` | Classificação com **todos os atributos** (132 atributos). |
| `06_partial_prediction.ipynb` | Classificação com **atributos selecionados** (105 atributos). |
| `07_feature_importance.ipynb` | Importância dos atributos pelas métricas intrínsecas das árvores. |

## Metodologia

### 1. Análise exploratória e pré-processamento
Resumo da base nacional e catarinense (tamanho, completude, distribuição do IDEB) e limpeza para uso nos modelos. Valores ausentes são mantidos: XGBoost, LightGBM e CatBoost os tratam nativamente, e o Random Forest recebe imputação iterativa ajustada apenas no treino.

### 2. Seleção de atributos
1. **Correlação de Pearson:** em cada par com correlação absoluta acima de 0,80, descarta-se a variável com maior correlação média com as demais (de 135 para 121 colunas).
2. **Remoção do ODS 4:** retiram-se os indicadores de educação e do próprio IDEB, para que a resposta não seja inferida de atributos dependentes do alvo (resultando em 105 atributos).
3. **PCA (exploratório):** aplicado sobre os dados padronizados para analisar a estrutura de variância. **Não é usado para eliminar variáveis.**

### 3. Modelos e ajuste de hiperparâmetros
- Algoritmos: Random Forest, XGBoost, LightGBM e CatBoost.
- Divisão 80% treino / 20% teste, estratificada pela classe, com `random_state = 42`.
- Hiperparâmetros ajustados por `RandomizedSearchCV` (10 combinações, validação cruzada estratificada de três folds sobre o treino, critério de acurácia).
- Dois experimentos comparáveis: **todos os atributos** e **atributos selecionados**.

### 4. Avaliação
Acurácia, precisão, *recall* e F1-*score* no conjunto de teste. Precisão, *recall* e F1 usam a **média macro** (todas as classes pesam igual), o que penaliza o desempenho nas classes minoritárias. Os resultados são comparados com o *baseline* da classe majoritária (49,7% em AF e 64,7% em AI).

### 5. Agrupamento de municípios
Identificação de perfis de municípios catarinenses com K-Means, agrupamento hierárquico e DBSCAN, e avaliação da predição por cluster em comparação com a base integral.

### 6. Importância dos atributos
Importância intrínseca de cada algoritmo, calculada a partir da redução de impureza ou do ganho de informação nos nós de decisão:

- Random Forest: redução média de impureza (Gini);
- XGBoost e LightGBM: ganho (`importance_type='gain'`);
- CatBoost: `PredictionValuesChange`.

Os modelos são reajustados com cinco sementes, as importâncias são normalizadas (soma igual a 1 por modelo), promediadas e consolidadas entre os quatro algoritmos (média, desvio, posição média no ranking e número de modelos em que o atributo aparece entre os 20 mais importantes). Há também uma agregação por ODS. A importância indica associação, não causalidade.

## Como executar

**Requisitos:** Python 3.10 ou superior.

```bash
pip install pandas numpy scikit-learn xgboost lightgbm catboost matplotlib seaborn openpyxl jupyter
```

1. Clone o repositório e coloque a base original em `data/raw/Base_de_Dados_IDSC-BR_2025.xlsx`.
2. Execute os notebooks **a partir da pasta `notebooks/`** (os caminhos são relativos a ela), na ordem:
   `02_data_cleaning` → `02b_alvos_ideb` → `feature_selection` → `03_global_prediction` → `partial_prediction` → `feature_importance`.
   O `01_data_analisys` é independente e pode ser executado a qualquer momento.
3. Os notebooks de agrupamento (`04_*` a `07_*`) usam os mesmos arquivos de `data/processed/`.

A busca de hiperparâmetros é demorada; reduzir `N_ITER` no `feature_importance.ipynb` ou o número de iterações nos notebooks de predição acelera testes.

## Reprodutibilidade

- Semente fixa (`random_state = 42`) na divisão treino/teste, na busca de hiperparâmetros e nos modelos.
- A análise de importância repete o ajuste final com cinco sementes para medir a estabilidade.
- Os alvos ficam isolados em `Base_de_Alvos_IDEB.csv`, gerado por `02b_alvos_ideb.ipynb`.

## Limitações

- **Base pequena:** 151 municípios (AF) e 272 (AI) após os filtros; o conjunto de teste tem 31 e 55 municípios, e um único erro altera a acurácia em cerca de 3,2 e 1,8 pontos percentuais.
- **Uma única divisão treino/teste**, sem intervalos de confiança: diferenças pequenas entre modelos não indicam superioridade.
- **Classes raras:** poucos municípios nas faixas extremas, o que torna as métricas macro instáveis.
- **Atributos agregados** (`Goal N Score`, `Pontuação Indice ODS 2025`) resumem vários indicadores e a pontuação geral inclui o ODS 4; podem carregar informação do próprio IDEB.
- **Atributos correlacionados** dividem a importância entre si, e as métricas de impureza e ganho favorecem variáveis contínuas.
- O Painel do IDSC-BR não foi avaliado como alvo alternativo nos resultados reportados.

## Autoria

Clara dos Santos Becker — Engenharia de Software, UDESC Ibirama.