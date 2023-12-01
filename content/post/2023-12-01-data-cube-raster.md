---
title: "Data cube raster para análise de séries temporais de imagens de satélite"
author: " "
date: 2023-12-01
categories: ["R"]
tags: ["time-series", "gdalcubes", "data-cube", "raster"]
---



```r
# Sentinel
file_path_s2 <- "/home/vinicio/Documentos/Codigo/TS_gdalcubes/Sentinel2_15bands/raw/"

# data cube 
file_path_prj <- "/home/vinicio/Documentos/Codigo/Data cube raster/"
```


No contexto de sensoriamento remoto e observação da Terra, _spatiotemporal array_ refere-se a matrizes multidimensionais onde duas dimensões representam as dimensões  espaciais do raster e o terceiro o tempo (ou a banda). Essa abordagem permite otimizar o acesso e a recuperação de informações específicas em determinados momentos ou locais para séries espaço-temporais.



```r
 knitr::include_graphics("/img/fig-cube-1.png", error = FALSE)
```

<img src="../../../../../../../img/fig-cube-1.png" width="60%" style="display: block; margin: auto;" />

Na estrutura de um cubo de dados raster, entretanto, também são considerados cubos de dimensões superiores (hipercubos), como um cubo de cinco dimensões onde, além do tempo, a banda espectral e o sensor formam dimensões.



```r
getwd()
```

```
## [1] "/home/vinicio/Documentos/Codigo/viniciovcl/content/post"
```

```r
list.files("./img")
```

```
##  [1] "cube.png"                      "estao-agua-clara-a756 (1).png"
##  [3] "estao-agua-clara-a756 (2).png" "estao-agua-clara-a756 (3).png"
##  [5] "estao-agua-clara-a756 (4).png" "estao-agua-clara-a756 (6).png"
##  [7] "estao-agua-clara-a756.png"     "fig-cube-1.png"               
##  [9] "L2A.png"                       "mult-array.png"               
## [11] "paper_illustration.png"        "readme_fig.png"               
## [13] "SCL band.png"
```

É uma estrutura eficiente para a manipulação de séries temporais de dados raster, permitindo declarar operações algébricas e aplicar funções a um conjunto limitado de dimensões para realizar cálculos e transformações pixel a pixel e criar novas representações dos dados. 

A seguir iremos explorar a biblioteca gdalcubes para analisar um conjunto de imagens do sensor MSI a bordo dos satélites Sentinel-2A e Sentinel-2B. 


# R Markdown

This is an R Markdown document. Markdown is a simple formatting syntax for authoring HTML, PDF, and MS Word documents. For more details on using R Markdown see <http://rmarkdown.rstudio.com>.

Teste editar post. 


```r
list.files("/home/vinicio/Documentos/Codigo/TS_gdalcubes/Sentinel2_15bands/raw/",
           full.names = FALSE)
```

```
##  [1] "S2A_MSIL2A_20200215T134201_N0214_R124_T22KCD_20200215T161217.zip"
##  [2] "S2A_MSIL2A_20200306T134211_N0214_R124_T22KCD_20200306T160502.zip"
##  [3] "S2A_MSIL2A_20200316T134211_N0214_R124_T22KCD_20200316T192808.zip"
##  [4] "S2A_MSIL2A_20200326T134211_N0214_R124_T22KCD_20200326T160948.zip"
##  [5] "S2A_MSIL2A_20200405T134211_N0214_R124_T22KCD_20200405T161002.zip"
##  [6] "S2A_MSIL2A_20200425T134211_N0214_R124_T22KCD_20200425T161132.zip"
##  [7] "S2A_MSIL2A_20200614T134221_N0214_R124_T22KCD_20200614T174720.zip"
##  [8] "S2A_MSIL2A_20200714T134221_N0214_R124_T22KCD_20200714T210626.zip"
##  [9] "S2B_MSIL2A_20200311T134209_N0214_R124_T22KCD_20200311T204936.zip"
## [10] "S2B_MSIL2A_20200331T134209_N0214_R124_T22KCD_20200331T173754.zip"
## [11] "S2B_MSIL2A_20200410T134209_N0214_R124_T22KCD_20200410T174302.zip"
## [12] "S2B_MSIL2A_20200420T134209_N0214_R124_T22KCD_20200420T174701.zip"
## [13] "S2B_MSIL2A_20200430T134209_N0214_R124_T22KCD_20200430T160218.zip"
## [14] "S2B_MSIL2A_20200510T134209_N0214_R124_T22KCD_20200510T174712.zip"
## [15] "S2B_MSIL2A_20200520T134209_N0214_R124_T22KCD_20200520T173910.zip"
## [16] "S2B_MSIL2A_20200530T134209_N0214_R124_T22KCD_20200530T155919.zip"
## [17] "S2B_MSIL2A_20200609T134219_N0214_R124_T22KCD_20200609T174956.zip"
## [18] "S2B_MSIL2A_20200619T134219_N0214_R124_T22KCD_20200619T175114.zip"
## [19] "S2B_MSIL2A_20200719T134209_N0214_R124_T22KCD_20200719T174410.zip"
```
