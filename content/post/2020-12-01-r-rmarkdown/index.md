---
title: "Hello R Markdown"
author: "Frida Gomam"
date: 2020-12-01T21:13:14-05:00
categories: ["R"]
tags: ["R Markdown", "plot", "regression"]
---



# R Markdown

This is an R Markdown document. Markdown is a simple formatting syntax for authoring HTML, PDF, and MS Word documents. For more details on using R Markdown see <http://rmarkdown.rstudio.com>.

Teste editar post. 


```r

list.files("..../TS_gdalcubes/Sentinel2_15bands/raw/")
## character(0)
```

# Including Plots

You can also embed plots. See Figure <a href="#fig:pie">1</a> for example:


```r
par(mar = c(0, 1, 0, 1))
pie(
  c(280, 60, 20),
  c('Sky', 'Sunny side of pyramid', 'Shady side of pyramid'),
  col = c('#0292D8', '#F7EA39', '#C4B632'),
  init.angle = -50, border = NA
)
```

<div class="figure">
<img src="{{< blogdown/postref >}}index_files/figure-html/pie-1.png" alt="A fancy pie chart." width="672" />
<p class="caption"><span id="fig:pie"></span>Figure 1: A fancy pie chart.</p>
</div>
