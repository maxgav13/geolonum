--- 
title: "Geologia Numerica: Ciencia de Datos para Geociencias"
author: "Maximiliano Garnier Villarreal"
affiliation: 'Escuela Centroamericana de Geologia, Universidad de Costa Rica'
date: "2020-03-31"
output: 
  bookdown::gitbook:
    df_print: tibble
site: bookdown::bookdown_site
documentclass: book
bibliography: [book.bib, packages.bib, references.bib]
csl: apa.csl
link-citations: yes
lang: es
description: "Este documento compila el funcionamiento basico del software estadistico y de programacion R, asi como rutinas y/o comandos especificos para resolver problemas de analisis de datos en geociencias."
---

# Prefacio {-}

Este libro es producido en **Markdown** [@rmarkdown2018], usando **bookdown** [@bookdown2016].

El paquete **bookdown** puede ser instalado desde  CRAN o Github:


```r
install.packages("bookdown")
# version en desarrollo
# devtools::install_github("rstudio/bookdown")
```

Este documento se basa en el material creado para el curso **G-4101 Geologia Numerica**, de la [Escuela Centroamericana de Geologia](geologia.ucr.ac.cr), [Universidad de Costa Rica](ucr.ac.cr). El curso hace uso del software estadistico y de programacion **R** [@R-base] para desarrollar la parte practica de los temas cubiertos en la teoria.

Existe un repositorio en GitHub para este proyecto ([geolonum](https://github.com/maxgav13/geolonum)), donde se pueden acceder los documentos y datos usados en este libro. Adicionalmente, se ha creado un paquete que ofrece funciones adicionales que seran usadas en algunos de los temas que se van a tratar. Para instalar el paquete se puede ir a [GMisc](https://github.com/maxgav13/GMisc) y seguir las instrucciones de la instalacion (el paquete no esta en CRAN, esta en Github, por lo que no se puede instalar de la forma convencional).

El curso cubre un poco de algebra lineal, para luego caer en la parte gruesa del curso que es estadistica. En esta parte se ve lo que es estadistica descriptiva (univariable y bivariable), principios de probabilidad, estadistica inferencial (pruebas de hipotesis, estimacion). La parte final del curso cubre temas relacionados con geociencias (datos direccionales, secuencias, y geoestadistica), donde se aplican conceptos anteriormente adquiridos en la seccion de estadistica.

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="Licencia de Creative Commons" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />Este obra está bajo una <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">licencia de Creative Commons Reconocimiento-NoComercial-CompartirIgual 4.0 Internacional</a>.

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License</a>.


