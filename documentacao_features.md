# Documentação de Dados: Urban Land Cover Dataset (OBIA)

Este dicionário de dados detalha as 148 variáveis extraídas por Análise de Imagens Baseada em Objetos (OBIA). O dataset não trabalha com pixels individuais, mas com **polígonos (segmentos de imagem)**. 

A estrutura é composta por **1 variável alvo** e **21 atributos base**, que são repetidos ao longo de **7 escalas espaciais** (o próprio objeto + 6 buffers de vizinhança), totalizando 148 colunas (1 + 21 * 7 = 148).

---

## 1. Variável Alvo (Target)

| Variável | Tipo | Descrição e Classes |
| :--- | :--- | :--- |
| **`class`** | Categórica | A classe de cobertura do solo real do polígono. Valores comuns: *Asphalt* (Asfalto), *Grass* (Grama), *Tree* (Árvore), *Roof* (Telhado), *Bare Soil* (Solo Exposto), *Shadow* (Sombra). |

## 2. Atributos Espectrais (Assinatura de Luz e Cor)
Capturam como o material do objeto interage com a luz solar.

| Variável | Descrição Matemática | Aplicação Prática |
| :--- | :--- | :--- |
| **`Mean_G`, `Mean_R`, `Mean_NIR`** | Valor médio de reflectância nos canais Verde (Green), Vermelho (Red) e Infravermelho Próximo (NIR) dentro do polígono. | Identifica a "cor" predominante. Vegetação tem pico alto em `Mean_NIR` e baixo em `Mean_R`. |
| **`SD_G`, `SD_R`, `SD_NIR`** | Desvio padrão (Standard Deviation) dos pixels dentro do polígono para cada banda. | Mede a homogeneidade. Um telhado metálico liso tem `SD` baixo; uma copa de árvore tem `SD` alto. |
| **`Bright`** | Luminosidade geral do objeto (média ponderada das bandas). | Separa objetos muito claros (concreto, telhados brancos) de escuros (asfalto úmido, sombras). |
| **`NDVI`** | Normalized Difference Vegetation Index. Fórmula: `(NIR - R) / (NIR + R)`. | A métrica definitiva para separar vegetação (Grama/Árvore) de áreas impermeáveis (Telhado/Asfalto). |

## 3. Atributos de Forma e Geometria
Diferenciam objetos que têm cores idênticas, mas estruturas físicas diferentes no ambiente urbano.

| Variável | Descrição Matemática | Aplicação Prática |
| :--- | :--- | :--- |
| **`Area`** | Área total do polígono em pixels. | Filtra ruídos pequenos ou diferencia pequenas casas de grandes galpões industriais. |
| **`LW`** | Length-to-Width (Comprimento / Largura). | Separa perfeitamente vias/ruas (LW altíssimo) de casas (LW próximo a 1). |
| **`Rect`** | Grau de Retangularidade (quão bem o polígono se encaixa num retângulo perfeito). | Excelente para isolar telhados, quadras esportivas e construções artificiais. |
| **`Round`** | Grau de Arredondamento. | Útil para separar copas de árvores ou canteiros redondos. |
| **`Compact`** | Compacidade (Área em relação ao perímetro). | Separa formas simples e fechadas (casas) de formas complexas e esparramadas. |
| **`ShpIndx`** | Índice de Forma (Shape Index). Mede a complexidade da borda do objeto. | Áreas naturais tendem a ter bordas mais irregulares (alto ShpIndx) que áreas construídas pelo homem. |
| **`Assym`** | Grau de Assimetria do polígono. | Ajuda a refinar a detecção de estruturas que não são simétricas. |
| **`BordLngth`** | Comprimento total da borda (perímetro) do objeto. | Usado em conjunto com a área para entender o formato. |
| **`BrdIndx`** | Índice de Borda. Compara a borda do objeto com a borda de um quadrado de mesma área. | Diferencia padrões ortogonais urbanos de padrões orgânicos naturais. |
| **`Dens`** | Densidade do formato (Área / Raio do menor círculo circunscrito). | Mede quão "sólido" o objeto é. |

## 4. Atributos de Textura (GLCM)
Baseados na Matriz de Co-ocorrência de Níveis de Cinza (Gray-Level Co-occurrence Matrix), avaliam o relevo visual.

| Variável | Descrição Matemática | Aplicação Prática |
| :--- | :--- | :--- |
| **`GLCM1`** | Contraste (Diferença de intensidade entre pixels vizinhos). | Diferencia texturas rugosas (árvores) de lisas (asfalto, telhado metálico). |
| **`GLCM2`** | Correlação (Relação linear dos níveis de cinza). | Detecta padrões lineares repetitivos (ex: telhas onduladas, faixas de pedestre). |
| **`GLCM3`** | Entropia / Homogeneidade. | Mede a desordem visual. Uma sombra tem textura muito homogênea; um solo exposto e com pedregulhos tem alta entropia. |

---

## 5. O Multiplicador Espacial (Sufixos de Escala)

As 21 variáveis descritas acima correspondem às características do **próprio objeto isolado**. Para dar "contexto espacial" ao modelo de Inteligência Artificial, o algoritmo gera círculos (buffers) de diferentes tamanhos ao redor do centro geométrico do objeto e recalcula exatamente as mesmas 21 variáveis considerando o que existe dentro desse raio.

Isso gera as 126 colunas adicionais (21 variáveis * 6 raios):

*   **`_40`** (Ex: `NDVI_40`, `Bright_40`): Características no raio de 40 metros/pixels do centro do objeto.
*   **`_60`**
*   **`_80`**
*   **`_100`**
*   **`_120`**
*   **`_140`** (Ex: `NDVI_140`, `Bright_140`): O contexto espacial mais amplo (macro-vizinhança).

**Por que isso importa?**
O modelo usa o contexto (a vizinhança) para ter mais certeza da classificação. Por exemplo, se o modelo identifica um retângulo cinza (podendo ser um telhado ou asfalto), mas o `NDVI_140` ao redor é alto (indicando grama e árvores), a probabilidade de ser uma casa aumenta significativamente.
