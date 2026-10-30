# Classificação de Cobertura do Solo (Land Cover Classification)

[![Python](https://img.shields.io/badge/Python-3.8%2B-blue.svg)](https://www.python.org/)
[![Scikit-Learn](https://img.shields.io/badge/Scikit--Learn-1.0%2B-orange.svg)](https://scikit-learn.org/)
[![Imbalanced-Learn](https://img.shields.io/badge/Imbalanced--Learn-0.9%2B-red.svg)](https://imbalanced-learn.org/)
[![Status](https://img.shields.io/badge/Status-Concluído-success.svg)]()

## Visão Geral do Projeto
Este projeto implementa um pipeline robusto de Machine Learning para a classificação multissectorial de cobertura do solo a partir de dados de sensoriamento remoto. O principal desafio deste projeto é o **extremo desbalanceamento de classes** (variando de 97 amostras para Prédios a apenas 14 para Piscinas) num espaço de alta dimensionalidade (147 *features*).

O modelo final foi otimizado para não negligenciar as classes minoritárias, garantindo validade estatística (zero *data leakage*) e alta capacidade de generalização.

## O Conjunto de Dados (Dataset)
O dataset é composto por características extraídas de imagens (como cor, textura e espectro), totalizando **147 variáveis independentes** para classificar 9 tipos de cobertura.

**Distribuição das Classes (Extremo Desbalanceamento):**
*   **Maioritárias:** Prédio (97), Concreto (93), Árvore (89), Grama (83)
*   **Intermédias:** Sombra (45), Asfalto (45)
*   **Minoritárias (Críticas):** Carro (21), Solo (20), Piscina (14)

## Metodologia e Modelagem

### 1. Prevenção de *Data Leakage*
Para garantir a honestidade das métricas e simular o ambiente de produção, todo o pré-processamento e treino foi encapsulado utilizando o `Pipeline` do Scikit-Learn. Isto garantiu que transformações matemáticas ocorressem estritamente sobre os dados de treino (`.fit_transform()`), preservando o conjunto de teste de qualquer vazamento de informação.

### 2. Seleção do Algoritmo
Foram testadas diversas arquiteturas de base (Regressão Logística, XGBoost, LightGBM, SVM, CatBoost e Extra Trees). Embora modelos de *Boosting* tenham apresentado alta precisão global, eles penalizaram severamente o *Recall* das classes raras (ex: Solo obteve apenas 43% de Recall com XGBoost). 
A escolha técnica recaiu sobre o **Balanced Random Forest (BRF)**, que força um sub-amostragem balanceada na construção de cada árvore, protegendo a representatividade das minorias.

### 3. Otimização de Hiperparâmetros (*Tuning*)
Foi executado um `GridSearchCV` com Validação Cruzada Estratificada (*StratifiedKFold* com 5 partições) e foco na métrica **F1-Macro**. Isto obrigou o otimizador a dar o mesmo peso a todas as 9 classes durante a procura.

**Parâmetros Vencedores (Mitigação de Overfitting):**
*   `max_depth`: 10 (Evita árvores infinitas focadas em ruído local)
*   `min_samples_leaf`: 2 (Exige consistência mínima nas folhas finais)
*   `n_estimators`: 100 (Estabilidade do comitê de árvores)

## Resultados Finais
O modelo atingiu uma **Acurácia Global de 85%**, mas o seu verdadeiro triunfo encontra-se no desempenho sobre as classes minoritárias face à *baseline* tradicional:

*   **Classe Solo (20 amostras):** O *Recall* saltou para **79%** (A precisão foi mantida em 85%).
*   **Classe Piscina (14 amostras):** A classe mais rara do dataset obteve um *Recall* impressionante de **88%**.
*   **Geral:** O *F1-Score Macro* atingiu 0.86, comprovando a eficácia da arquitetura para resgatar dados escassos sem alucinar (*Falsos Positivos*).

## 🚀 Como Utilizar o Modelo

O pipeline final encontra-se exportado e pronto para produção através da biblioteca `joblib`.

### Pré-requisitos
```bash
pip install pandas scikit-learn imbalanced-learn joblib
