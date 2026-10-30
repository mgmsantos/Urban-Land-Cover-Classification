# Classificação de Cobertura do Solo

[![Python](https://img.shields.io/badge/Python-3.8%2B-blue.svg)](https://www.python.org/)
[![Scikit-Learn](https://img.shields.io/badge/Scikit--Learn-1.0%2B-orange.svg)](https://scikit-learn.org/)
[![Imbalanced-Learn](https://img.shields.io/badge/Imbalanced--Learn-0.9%2B-red.svg)](https://imbalanced-learn.org/)
[![Status](https://img.shields.io/badge/Status-Concluído-success.svg)]()

## Visão Geral do Projeto
Este projeto implementa um pipeline de Machine Learning para a classificação multiclasse de cobertura do solo a partir de dados de sensoriamento remoto. O principal desafio deste projeto é o desbalanceamento de classes (variando de 97 amostras para Prédios a apenas 14 para Piscinas) num espaço de alta dimensionalidade.

O modelo final foi otimizado para não negligenciar as classes minoritárias, garantindo zero *data leakage*) e capacidade de generalização.

## O Conjunto de Dados
O dataset é composto por características extraídas de imagens por sensoriamento remoto, totalizando 147 variáveis independentes para classificar 9 tipos de classes. O conjunto de dados pode ser obtido em:
- Johnson B. Urban Land Cover [dataset]. 2013. UCI Machine Learning Repository. Available from: https://doi.org/10.24432/C53S48.

- A documentação das features pode ser acessada em: [Documentação Features](https://github.com/mgmsantos/Urban-Land-Cover-Classification/blob/main/documentacao_features.md)

**Distribuição das Classes:**

<p align="center">
  <img src="https://github.com/user-attachments/assets/866cd537-3aca-4e03-8af3-9241c7125c7c" alt="Distribuição das classes no treino" width="589">
</p>

**Sobreposição espectral e variáveis não-lineares**

A fim de obter uma noção da complexidade acerca das 147 variáveis, aplicou-se o algoritmo t-SNE para reduzir a dimensionalidade das features, as quais apresentam comportamento não-linear. A avaliação demonstra que os dados obtidos por sensoriamento remoto promovem uma separação clara entre algumas classes, como: asfalto, sombra, carro e piscina, em virtude das suas assinaturas espectrais serem únicas. Em contrapartida, classes como árvore e grama que compartilham reflectâncias semelhantes; concreto e prédio, visto que muitas construções possuem lajes de concreto; e o solo que está espalhado por todo o gráfico, se misturando com grama, concreto e asfalto, o que é explicado pela variação da reflectância em virtude das condições do solo (seco, molhado, compactado, etc).

<p align="center">
  <img width="790" height="590" alt="image" src="https://github.com/user-attachments/assets/85222ad7-ea05-48ce-a0d7-84c02877a1b5">
</p>

**Índicador biofísico NDVI**

O NDVI (índice de Vegetação por Diferença Normalizada) é um indicador biofísico capaz de separar elementos vegetais vivos de outros materias com base na reflexão da luz infravermelha. Observando o comportamento dessa variável nos dados de treino, percebe-se a sua relevância dentro da modelagem. Classes como árvore e grama se destacam de forma isolada, com maiores valores médios de NDVI (entre 0.17 e 0.30). Dessa forma, isso explica a elevada capacidade da separação dessas classes durante a modelagem, com destaque para as árvores que possuem variação menor em torno da média.

A classe "piscina" apresentou os valores mais baixos de NDVI, e isso é explicado em virtude da absorção de radiação da água, resultando em valores fortemente negativos para esse índice. Aliado a isso, essa classe possui pouquissimas amostras, fazendo com que seja mais difícil de detectar.

As classes concreto, asfalto, prédio e carro se encontram em zonas de transição, com mediana nos limites de -0.15 a 0.00. Para essas classes, o NDVI falha na distinção dos materiais, o que justifica a necessidade de manter as outras diversas features para treinar o modelo.

A classe sombra é caracterizada por apresentar a maior dispersão de NDVI, variando de -0.18 a 0.25. Isso pode estar relacionado à realidade física do terreno, visto que uma sombra projetada sobre uma árvore tera uma assinatura espectral diferente de uma sombra projetada por um prédio.

<p align="center">
  <img width="790" height="390" alt="image" src="https://github.com/user-attachments/assets/a570daea-4d33-49fd-b061-4efbeb653143">
</p>

## Metodologia e Modelagem

### 1. Prevenção de *Data Leakage*
Para garantir a honestidade das métricas e simular o ambiente de produção, todo o pré-processamento e treino foi encapsulado utilizando o `Pipeline` do Scikit-Learn. Isto garantiu que transformações matemáticas ocorressem estritamente sobre os dados de treino, preservando o conjunto de teste de qualquer vazamento de informação.

### 2. Seleção do Algoritmo
Foram testadas diversas arquiteturas de base (Regressão Logística, XGBoost, LightGBM, SVM, CatBoost e Extra Trees). Embora modelos de *Boosting* tenham apresentado alta precisão global, eles penalizaram severamente o *Recall* das classes raras (ex: Solo obteve apenas 43% de Recall com XGBoost). 
A escolha técnica recaiu sobre o **Balanced Random Forest (BRF)** da biblioteca *Imbalanced-Learn*, que força um sub-amostragem balanceada na construção de cada árvore, protegendo a representatividade das minorias.

Durante o processo de modelagem, avaliou-se a importancia das features a fim de fazer uma seleção de variáveis, todavia os testes demonstraram que, apesar da baixa contribuição, todas as features foram importantes para a classificação, especialmente em classes com poucas amostras. Nesse sentido, a remoção de algumas features prejudicava as métricas do modelo para essas classes consideradas raras em virtude do desbalanceamento dos dados.

### 3. Tuning de Hiperparâmetros
Foi executado um `GridSearchCV` com Validação Cruzada Estratificada (*StratifiedKFold*) e foco na métrica **F1-Macro**. Isto obrigou o otimizador a dar o mesmo peso a todas às 9 classes durante a procura.

**Hiperparâmetros otimizados:**
*   `max_depth`: 10
*   `min_samples_leaf`: 2
*   `n_estimators`: 100

## Resultados Finais
O modelo atingiu uma **Acurácia Global de 85%**, mas o seu verdadeiro triunfo encontra-se no desempenho sobre as classes minoritárias, em especial solo e piscina:

*   **Classe Solo (20 amostras):** *Recall* equivalente a 71%.
*   **Classe Piscina (14 amostras):** A classe mais rara do dataset obteve um *Recall* 93%.
*   **Geral:** O *F1-Score Macro* atingiu 0.85, demonstrando que o modelo consegue resgatar dados escassos sem alucinar.

<div align="center">

**Acurácia Global:** 0.85

| Classe | Precision | Recall | F1-Score | Support |
| :---: | :---: | :---: | :---: | :---: |
| **Asphalt** | 0.70 | 1.00 | 0.82 | 14 |
| **Building** | 0.75 | 0.84 | 0.79 | 25 |
| **Car** | 0.93 | 0.87 | 0.90 | 15 |
| **Concrete** | 0.81 | 0.74 | 0.77 | 23 |
| **Grass** | 0.89 | 0.86 | 0.88 | 29 |
| **Pool** | 1.00 | 0.93 | 0.97 | 15 |
| **Shadow** | 1.00 | 0.81 | 0.90 | 16 |
| **Soil** | 0.83 | 0.71 | 0.77 | 14 |
| **Tree** | 0.83 | 0.88 | 0.86 | 17 |
| **Macro Avg** | 0.86 | 0.85 | 0.85 | 168 |
| **Weighted Avg** | 0.86 | 0.85 | 0.85 | 168 |

</div>

As classes Piscina e Sombra obtiveram precisão de 100%, e esse resultado pode ser corroborado com auxílio do t-SNE, o qual agrega e isola essas classes em comparação às outras. Devido a isso, o modelo se torna altamente confiante e preciso nessas categorias. Todavia, o recall ainda se encontrou em 93% e 81%, evidenciando a postura conservadora do modelo em sua previsão ao deixar passar falsos negativos.

A classe concreto obteve um recall de apenas 74% e prédio obteve precisão de 75%, mesmo sendo classes com maior valor de instâncias no treino. Dessa forma, o modelo gera falsos negativos e falsos positivos para ambas as categorias em virtude da similaridade de seus materiais.

A classe solo obteve o menor recall (71%) do modelo, falhando na detecção de 1/3 das amostras de solo exposto. Isso ocorre por conta das diferentes ocorrências de solo, como solo úmido, solo seco, solo compactado, solo com diferentes classes texturais, impactando na avaliação devido à nuances na refletância dessas amostras.

## Aplicações Práticas
A modelagem da classificação de cobertura do solo é uma ferramenta valiosa para setores de Geointeligência e Administração pública. Desde que o modelo esteja apto para detecção de diversas classes, incluindo aquelas com menor representabilidade, é possível realizar auditorias e monitorar excessões, como:

- Auditoria fiscal e atualização cadastral: detecção de obras irregulares e atualização de IPTU;
- Gestão de risco hídrico e prevenção de enchentes: cálculo de taxa de permeabilidade do solo urbano;
- Monitoramento ambiental e alertas de desmatamento: fiscalização de áreas de proteção ambiental e controle de erosão;
- Planejamento de infraestrutura e mobilidade urbana: mapeamento de malha viária e ilhas de calor.

## Como Utilizar o Modelo

O pipeline final encontra-se exportado e pronto para produção através da biblioteca `joblib`.

### Pré-requisitos
```bash
pip install pandas scikit-learn imbalanced-learn joblib

import pandas as pd
import joblib

# 1. Carregar o modelo treinado
modelo_producao = joblib.load('modelo_brf_cobertura_solo.pkl')

# 2. Carregar novos dados (Certifique-se que possui as 147 features originais)
novos_dados = pd.read_csv('novas_amostras_satelite.csv')

# 3. Executar a previsão
previsoes = modelo_producao.predict(novos_dados)

print(previsoes)
# Saída esperada: ['building', 'soil', 'pool', ...]
```

## Possíveis melhorias

### Visão Computacional (Deep Learning)
Caso seja possível acessar as imagens originais de satélite ou as fotografias obtidas por drone com câmera de sensor multiespectral, pode-se utilizar Redes Neurais Convulacionais para entender formatos e resolver problemas entre classes como concreto e prédio, por exemplo.

### Data Fusion
Pode-se enriquecer e melhorar as métricas do modelo adicionando uma nova fonte de dados, como por exemplo: Modelo Digital de Elevação (LiDAR), fazendo com que a máquina saiba quais classes estão em diferentes altitudes.

### Feature Engineering
Utilizando as variáveis já existentes, pode-se calcular outros indicadores biofísicos que contribuam na separação de diferentes materiais, como por exemplo o NDWI para detecção de água e melhora das métricas na classificação de piscinas.
