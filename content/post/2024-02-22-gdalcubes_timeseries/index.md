---
title: "Cubos de dados para análise de séries de imagens de satélite"
author: "Vinicio C. Lima "
date: 2024-02-18
categories: ["R"]
tags: ["time-series", "image-processing", "data-cube", "changedetection", "gdalcubes", "r-markdown"]
---


No contexto de sensoriamento remoto e observação da Terra, _spatiotemporal array_ refere-se a matrizes multidimensionais onde duas dimensões representam as dimensões  espaciais do raster e o terceiro o tempo (ou a banda). Essa abordagem permite otimizar o acesso e a recuperação de informações específicas em determinados momentos ou locais para séries espaço-temporais.



<img src="./images/fig-cube-1.png" width="100%" style="display: block; margin: auto;" />


Na estrutura de um cubo de dados raster também são considerados cubos de dimensões superiores (hipercubos), como um cubo de cinco dimensões onde, além do tempo, a banda espectral e o sensor formam dimensões.



<img src="./images/cube.png" width="100%" style="display: block; margin: auto;" />


É uma estrutura eficiente para a manipulação de séries temporais de dados raster, permitindo declarar operações algébricas e aplicar funções a um conjunto limitado de dimensões para realizar cálculos e transformações pixel a pixel e criar novas representações dos dados. 

A seguir iremos explorar a biblioteca gdalcubes para analisar um conjunto de imagens do sensor MSI a bordo dos satélites Sentinel-2A e Sentinel-2B. 


## Conjunto de dados


O conjunto de dados contém 19 cenas de uma área de Cerrado do Estado de Mato Grosso do Sul adquiridas entre fevereiro e julho de 2020.






```r
path_file <- "/home/vinicio/Documentos/Codigo/TS_gdalcubes/Sentinel2_15bands/"

s2_files <- list.files( paste(path_file,  "raw", sep="/"),
    pattern = ".zip$", recursive = TRUE )

s2_files
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



```r
files.size <- sum(file.size(s2_files)) / 1000^3 # gigabytes

files.size # gigabytes
```



```
## [1] 21.2
```



## gdalcubes

[gdalcubes](https://gdalcubes.github.io/) é um pacote R e uma biblioteca C++ para o processamento de grandes coleções de imagens de satélite. 



```r
library(magrittr)
library(gdalcubes)
packageVersion("gdalcubes")
```

```
## [1] '0.7.0'
```



```r
# Cria uma coleção de imagens gdalcubes

if(!file.exists(file.path(path_file, "S2_collection.db" ))){
 S2_download_zip = list.files( paste(path_file, "raw", sep = "/"),
                     pattern = ".zip$",  full.names = TRUE, recursive = TRUE )

 create_image_collection(
   files = S2_download_zip, format='Sentinel2_L2A',
   unroll_archives=TRUE, out_file=file.path(
     path_file,  "S2_collection.db"))
}
```



```r
S2_collection <- image_collection(file.path(
  path_file, "S2_collection.db"))  

S2_collection
```

```
## Image collection object, referencing 19 images with 15 bands
## Images:
##                                                           name      left
## 1 S2A_MSIL2A_20200215T134201_N0214_R124_T22KCD_20200215T161217 -52.91137
## 2 S2A_MSIL2A_20200306T134211_N0214_R124_T22KCD_20200306T160502 -52.91137
## 3 S2A_MSIL2A_20200316T134211_N0214_R124_T22KCD_20200316T192808 -52.91137
## 4 S2A_MSIL2A_20200326T134211_N0214_R124_T22KCD_20200326T160948 -52.91137
## 5 S2A_MSIL2A_20200405T134211_N0214_R124_T22KCD_20200405T161002 -52.91137
## 6 S2A_MSIL2A_20200425T134211_N0214_R124_T22KCD_20200425T161132 -52.91137
##         top    bottom     right            datetime        srs
## 1 -18.98277 -19.98271 -51.85693 2020-02-15T13:42:01 EPSG:32722
## 2 -18.98277 -19.98271 -51.85693 2020-03-06T13:42:11 EPSG:32722
## 3 -18.98277 -19.98271 -51.85693 2020-03-16T13:42:11 EPSG:32722
## 4 -18.98277 -19.98271 -51.85693 2020-03-26T13:42:11 EPSG:32722
## 5 -18.98277 -19.98271 -51.85693 2020-04-05T13:42:11 EPSG:32722
## 6 -18.98277 -19.98271 -51.85693 2020-04-25T13:42:11 EPSG:32722
## [ omitted 13 images ] 
## 
## Bands:
##    name offset scale unit   nodata image_count
## 1   AOT      0 1e+00                        19
## 2   B01      0 1e-04      0.000000          19
## 3   B02      0 1e-04      0.000000          19
## 4   B03      0 1e-04      0.000000          19
## 5   B04      0 1e-04      0.000000          19
## 6   B05      0 1e-04      0.000000          19
## 7   B06      0 1e-04      0.000000          19
## 8   B07      0 1e-04      0.000000          19
## 9   B08      0 1e-04      0.000000          19
## 10  B09      0 1e-04      0.000000          19
## 11  B11      0 1e-04      0.000000          19
## 12  B12      0 1e-04      0.000000          19
## 13  B8A      0 1e-04      0.000000          19
## 14  SCL      0 1e+00                        19
## 15  WVP      0 1e+00                        19
```



Uma view define a geometria espaço-temporal sem conectá-la ao conjuntos de dados específicos. Isso é especialmente útil quando se trabalha com grandes volumes de dados raster. É uma representação específica da série temporal que pode ser definida pela geometria do cubo.

Uma view do cubo de dados contém o sistema de referência espacial (SRS), a extensão espaçotemporal (esquerda, direita, inferior, superior, data/hora de início, data/hora de término) e o tamanho espaçotemporal dos pixels (tamanho espacial, duração temporal).

Por exemplo, podemos criar uma view que aplica uma reamostragem dos pixels para 250 m e uma agragação temporal pela mediana de um intervalo mensal para obter uma visão geral da série. 




```r
 # Visão geral da cena
v.overview = cube_view(
  extent=S2_collection, 
  dx=250, dy=250, # 250 m x 250 m
  resampling = "bilinear", # interpolador da reamostragem
  srs="EPSG:31982", # projeção de destino
  dt="P1M", aggregation = "median" # mediana para o mês.
  )

v.overview 
```

```
## A data cube view object
## 
## Dimensions:
##                low             high count pixel_size
## t       2020-02-01       2020-07-31     6        P1M
## y 7789200.49146679 7900950.49146679   447        250
## x 298688.080453591 410438.080453591   447        250
## 
## SRS: "EPSG:31982"
## Temporal aggregation method: "median"
## Spatial resampling method: "bilinear"
```



A combinação de uma visualização `cube_view()` da geometria do cubo  com a coleção de imagens produz um cubo de dados raster regular contendo os metadados da coleção de imagens e a geometria da view `raster_cube()` .




```r
 cube.overview <-  raster_cube(S2_collection, v.overview) 

 cube.overview
```

```
## A data cube proxy object
## 
## Dimensions:
##                low             high count pixel_size chunk_size
## t       2020-02-01       2020-07-31     6        P1M          1
## y 7789200.49146679 7900950.49146679   447        250        320
## x 298688.080453591 410438.080453591   447        250        320
## 
## Bands:
##    name offset scale nodata unit
## 1   AOT      0 1e+00    NaN     
## 2   B01      0 1e-04    NaN     
## 3   B02      0 1e-04    NaN     
## 4   B03      0 1e-04    NaN     
## 5   B04      0 1e-04    NaN     
## 6   B05      0 1e-04    NaN     
## 7   B06      0 1e-04    NaN     
## 8   B07      0 1e-04    NaN     
## 9   B08      0 1e-04    NaN     
## 10  B09      0 1e-04    NaN     
## 11  B11      0 1e-04    NaN     
## 12  B12      0 1e-04    NaN     
## 13  B8A      0 1e-04    NaN     
## 14  SCL      0 1e+00    NaN     
## 15  WVP      0 1e+00    NaN
```


<img src="{{< blogdown/postref >}}index_files/figure-html/cubeview_plot-1.png" width="150%" />


A região da cena compreende uma área ocupada predominantemente por pastagens extensivas e reflorestamento de eucalipto. Podemos obter o NDVI da série para e definir uma região específica para observar um povoamento de Eucalyptus Urograndis.



```
## Warning in plot.cube(., zlim = c(0, 1200), rgb = 3:1, key.pos = 1, ncol = 1, :
## Ignoring key.pos for RGB plots
```

<img src="{{< blogdown/postref >}}index_files/figure-html/euc_rgb-1.png" width="100%" style="display: block; margin: auto;" />


O produto [L2A](https://docs.sentinel-hub.com/api/latest/data/sentinel-2-l2a/) inclui bandas de máscara e  sinalizadores de qualidade de pixels, entre outras camadas que podem ser usadas para filtrar pixels espúrios ou atender determinada análise.

As máscaras são aplicadas em imagens e não em cubos. Os valores mascarados não contribuirão para a agregação de pixels.



```r
s2.clear.mask <- image_mask("SCL", values= c(0,1,2,3,5,6,7,8,9,10,11 )) # Vegetação
```




```r
v.euc = cube_view(
  extent=list(S2_collection, left=321434.9, right=326500,
              bottom=7813432, top=7819363,
             t0="2020-02-15", t1="2020-07-31"),
            dt="P1M", dx=100, dy=100, srs="EPSG:31982",
            aggregation = "median", resampling = "bilinear")


# NDVI

month_euc_ndvi <- raster_cube( S2_collection, v.euc, mask = s2.clear.mask) %>%
   select_bands(c("B04","B08", "SCL")) %>%
  apply_pixel(c("(B08-B04)/(B08+B04)"), names="NDVI") 

# kNDVI

month_euc_kndvi <- raster_cube( S2_collection, v.euc, mask = s2.clear.mask) %>%
   select_bands(c("B04","B04","B08", "SCL")) %>% 
 apply_pixel("tanh(((B08-B04)/(B08+B04))^2)", "kNDVI") 

library(viridis)
month_euc_ndvi %>%  filter_pixel("NDVI > 0.7") %>% # filtra outros usos
  plot(zlim=c(0.55, .9), key.pos=1,  ncol =3, nrow=2, col=viridis)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/euc_ndvi-1.png" width="150%" style="display: block; margin: auto;" />



```r
month_euc_ndvi %>% filter_pixel("NDVI > 0.7") %>% 
  reduce_space(#"sum(NDVI)",
               "mean(NDVI)",
               "min(NDVI)",
               "max(NDVI)",
               "median(NDVI)",
               "sd(NDVI))", 
                "count(NDVI)", 
               "var(NDVI)") %>%
  plot(ncol =4, nrow=2)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/reduce_space_ndvi-1.png" width="95%" style="display: block; margin: auto;" />


Os dados do NDVI seguem a tendência esperada para o período seco caracteristico da região para o intervalo analisado. É de se esperar que quanto mais próximo do auge do período seco os fatores climáticos afetem o metabolismo das plantas, resultando em valores menores para o NDVI.


Os [dados do INMET](https://tempo.inmet.gov.br/GraficosAnuais) apontam para uma condição típica do clima da região para o período, coincindindo com os mínimos de umidade do ar e longos períodos de estiagem.




<div class="figure" style="text-align: center">
<img src="./images/estao-agua-clara-a756 (6).png" alt="Precipitação (mm) para estação Água Clara (A758)." width="60%" />
<p class="caption">(\#fig:fig_precp_AC)Precipitação (mm) para estação Água Clara (A758).</p>
</div>



<div class="figure" style="text-align: center">
<img src="./images/estao-agua-clara-a756 (2).png" alt="Umidade do ar (%) para estação Água Clara (A758)." width="60%" />
<p class="caption">(\#fig:fig_umid_AC)Umidade do ar (%) para estação Água Clara (A758).</p>
</div>




<div class="figure" style="text-align: center">
<img src="./images/estao-agua-clara-a756 (1).png" alt="Temperatura do ar (ºC) para estação Água Clara (A758)." width="60%" />
<p class="caption">(\#fig:fig_temp_AC)Temperatura do ar (ºC) para estação Água Clara (A758).</p>
</div>



Dado esse contexto, uma pergunta interessante de ser respondida é se:

Mesmo havendo uma tendência de global de diminuição do valor do NDVI nos dados analisados, seria possível detectar anomalias da atividade da fotossíntese correspondendo a fatores externos ao clima, como intervenções de manejo ou estresse causado por vento, pragas ou doenças ou fatores fisiológicos da planta?


Ao analisar o comportamento da média e da  mediana no período é possível observar uma atenuação da tendência de diminuição do NDVI no intervalo abril-junho, com uma inflexão no mês de maio. 


<div class="figure" style="text-align: center">
<img src="{{< blogdown/postref >}}index_files/figure-html/reduce_space_ndvi2-1.png" alt="Média e a mediana da série mensal." width="75%" />
<p class="caption">(\#fig:reduce_space_ndvi2)Média e a mediana da série mensal.</p>
</div>






## Detecção de mudanças

Podemos derivar as diferenças mensais para analisar o comportamento da diminuição do valor do NDVI no decorrer dos meses Tt - T (t-1).   A função  `window_time()`  aplica o filtro de diferença de kernel para a série mensal. 


```r
# Diferença mensal

month_euc_ndvi %>%  filter_pixel("NDVI > 0.7") %>% 
  window_time(kernel=c(-1,1), window=c(1,0)) %>%
  plot(zlim=c(-.13, .15), key.pos=1,   col= viridis,  t = 2:6, ncol = 3 )
```

<img src="{{< blogdown/postref >}}index_files/figure-html/euc_change-1.png" width="150%" style="display: block; margin: auto;" />



<div class="figure" style="text-align: center">
<img src="{{< blogdown/postref >}}index_files/figure-html/reduce_space_change-1.png" alt="Média e a Mediana da diferença simples para a série mensal Tt - T(t-1)" width="75%" />
<p class="caption">(\#fig:reduce_space_change)Média e a Mediana da diferença simples para a série mensal Tt - T(t-1)</p>
</div>


A média e a mediana da diferença simples mostram uma estagnação da tendência de diminuição do NDVI, com o valor da média da diferença superior a mediana da diferença no mês de junho. Esse comportamento aponta para a diminuição da magnitude da taxa de mudança que pode indicar uma redução na intensidade dessa tendência, o que ser observado no gráfico entre os mêses de maio e junho. 


A mudança positiva no intervalo fevereiro-março pode ser decorrente das chuvas que ocorreram durante o período, o que não pode ser observado para o intervalo seguinte mesmo com a continuação de eventos de chuva no mês de abril. Isso poderia estar relacionado a algum 'fator externo' aos dados analisados até aqui?


Observando o gráfico da média e da mediana da diferença podemos considerar três padrões de tendência, representados no gráfico pelos intervalos (1) março-abril, (2) abril-junho, (3) junho-julho. 


A inclinação do intervalo (3) junho-julho  é menor que a do intervalo (1) março-abril, o que é contrário a tendência das variáveis climáticas, enquanto a série avança para o auge do período seco do ano, com os mínimos da umidade do ar entre agosto e outubro. 


Diferenças abslutas menores do NDVI ocorreram de maneira significativa na área analisada, entre os meses de abril e junho, consecutivamente (maio - abril ; junho - maio). Existe algum padrão ou arranjo espacial dos valores da diferença simples. Orientação preferencial SO-NE?




```r
month_euc_ndvi %>%  filter_pixel("NDVI > 0.7") %>% 
  window_time(kernel=c(-1,1), window=c(1,0)) %>%
  filter_pixel("NDVI > 0.0") %>% 
  plot(zlim=c(0.0, .12), key.pos=1,   col= viridis,  t = 2:6, ncol = 3 )
```

<img src="{{< blogdown/postref >}}index_files/figure-html/euc_change_filt-1.png" width="150%" style="display: block; margin: auto;" />


## Considerações finais

Essa análise foi motivada por uma intervenção de manejo em um povoamento adulto de Eucalipytus. Onde no mês de Maio foi empregado um agente como medida de controle sobre a população da _Glena bipennaria bipennaria_ (lagarta desfolhadora).


° Em uma coleção de imagens aplicou-se uma redução temporal da série para um intervalo mensal.


° Foi criado um cubo de dados raster contendo 6 imagens correspondendo ao intervalo mês a mês da série e calculado o índice de vegetação de diferença normalizada NDVI.


° No cubo de dados foi aplicado uma função redutora em uma janela móvel sobre a dimensão do tempo. Filtro de kernel da diferença simples Tt - T (t-1). 


A diferença consecutiva mês a mês para o intervalo mostrou uma atenuação na intensidade de tendência de diminuição do NDVI, com inflexões em maio e junho. Uma análise de tendências e anomalias pode fornecer uma abordagem complementar de maneira a validar os valores encontrados para a diferença simples.  







