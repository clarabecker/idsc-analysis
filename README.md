## Estrutura da Tabela

 - Código IBGE: O primeiro número (ex: 1303403).
 - Nome do Município: (ex: Parintins).
 - População: O valor logo após o nome (ex: 96372).
 - UF: Estado (ex: AM).
 - Indicadores (SDG1 a SDG17): A sequência de números decimais que segue são os valores para os indicadores que você detalhou no dicionário.

 **1 - Clustering de Municípios:**
 
  - Agrupar cidades não por tamanho, mas por perfil de desenvolvimento usando modelos de clustering (DBSCAN, Isolation Forest).
  - Identificar mesmo grupos de desafios por cidade, correlação das variáveis. Exemplo: cidades com alta densidade populacional, baixo saneamento e alta incidência de doenças.
  - Aplicação de redução de dimensionalidade (PCA) para entender quais indicadores (dos 100) realmente definem o sucesso ou fracasso de uma cidade catarinense.
 
 **2 - Predição de Desempenho Educacional (IDEB)**  
 
  - Através de Determinantes Sociais Verificar como o investimento em infraestrutura (Indicador 60) correlaciona-se com a redução da pobreza (Indicador 1 a 4).
  - IA Explicável
  - Criar um modelo de Regressão (Random Forest, XGBoost) para prever a nota do IDEB de um município com base nos outros 98 indicadores.
  - Mostrar quais fatores que oorrem fora da escola (ex: mortalidade infantil ou coleta de lixo) mais impactam o aprendizado na sua região.
  - Biblioteca SHAP em Python para gerar gráficos de importância de atributos para cada cluster.
  - "O que o gestor público de SC deve priorizar fora da escola para que o resultado dentro da escola melhore?"

  - IDEB: https://www.gov.br/inep/pt-br/areas-de-atuacao/pesquisas-estatisticas-e-indicadores/ideb
    
 **3 - Clustering seguido de Predição Localizada**

Fase 1 (Clustering) : Agrupamento dos municípios brasileiros em 4 ou 5 perfis.

Fase 2 (Predição): Treinamento um modelo de predição de IDEB para cada grupo.

Ex de resultado: em "Polos Agrícolas", o que mais afeta a educação é o Indicador X, enquanto em "Cidades Turísticas" é o Indicador Y.
