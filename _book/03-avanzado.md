# Funcionamiento avanzado de R {#avanzado}

En el capitulo anterior se mostro de manera muy rapida las funciones basicas de **R**, esto porque de ahora en adelante se va a enfocar en el uso de funciones del *tidyverse* [@tidyverse2019]. Este meta-paquete es uno que engloba a un monton de paquetes que se rigen bajo el paradigma de datos ordenados (tidy data). Datos ordenados quiere decir una observacion por fila y cada variable en su columna. Lo que se cubre en este capitulo y mas se puede encontrar en "R for Data Science" [@grolemund2016].

En este capitulo se van a utilizar los siguientes paquetes:


```r
library(babynames)
library(nycflights13)
library(gapminder)
library(DT)
library(rio)
library(tidyverse)
```

Los tres primeros corresponden con conjuntos de datos. Asi mismo se vuelven a importar los datos con que se venia trabajando:


```r
data("airquality")
dat1 <- import("data/LungCapData2.csv", setclass = 'tibble')
titanic <- import("data/titanic.csv", setclass = 'tibble')
```

Los paquetes mas usados e importantes del *tidyverse* son:

* *dplyr*: Manipulacion de datos mediante seleccion, filtrado, creacion, agrupamiento, arreglo y resumen de tablas
* *tidyr*: Convierte datos a ordenados y viceversa
* *ggplot2*: Paquete para crear graficos de alta calidad y personalizables
* *purrr*: Brinda funciones para la iteracion sobre vectores, listas y tablas
* *forcats*: Manipulacion de datos categoricos (factores)
* *stringr*: Manipulacion de datos de texto
* *lubridate*: Manipulacion de fechas

Un punto importante a destacar, es que en todas (sino la mayoria) de las funciones del *tidyverse* la tabla de datos es el primer argumento.

## Operadores logicos

Operadores logicos permiten hacer comparaciones o pruebas, donde usualmente el resultado es `TRUE` o `FALSE`. En terminos numericos `TRUE` equivale a 1 y `FALSE` a 0. Los operadores logicos mas usados son:

* `<`, menor que
* `<=`, menor o igual que
* `==`, igual que
* `!=`, no igual que
* `>`, mayor que
* `>=`, mayor o igual que
* `%in%`, pertenencia

Aqui se asigna 4 a `x` y se aplican varios de los operadores logicos para cuestionar el contenido de el objeto `x`.


```r
x <- 4
x == 4
```

```
## [1] TRUE
```

```r
x != 4
```

```
## [1] FALSE
```

```r
x < 4
```

```
## [1] FALSE
```

```r
x <= 5
```

```
## [1] TRUE
```

Estos operdores se pueden usar igualmente en vectores. Recordando que el resultado de una operacion logica es `TRUE` o `FALSE`, el vector resultante es del tipo logico. Si se desea acceder a los elementos que cumplen la condicion hay que aplicar el vector logico sobre el vector, donde va a extrear los elementos que coinciden con `TRUE`. 

Aqui se crea un vector numerico, y se tratan de extraer los valores menores a 70. Se muestra la forma basica de **R** (`vector[condicion]`) y una forma mas directa e intuitiva que ofrece *purrr*.


```r
y <- c(95, 90, 58, 87, 62, 75)
y < 70
```

```
## [1] FALSE FALSE  TRUE FALSE  TRUE FALSE
```

```r
y[y < 70]
```

```
## [1] 58 62
```

```r
keep(y, ~.x < 70) # purrr
```

```
## [1] 58 62
```

## Operador de secuencia (Pipe operator)

Uno de los operadores basicos en el *tidyverse* es el pipe operator (`%>%`). Este permite que el resultado antes del operador sea la (primer) entrada de lo que se va a hacer despues del operador (`x %>% f(y)` es lo mismo que `f(x,y)`). 

El shortcut para escribirlo es:

* Mac: *Cmd + Shift + M*
* Windows: *Ctrl + Shift + M*

La ventaja es que permite encadenar operaciones sin necesidad de salvar objetos intermedios y es mas facil de leer que encerrar operaciones una dentro de la otra. Se ejemplifica con un caso sencillo, donde se tiene un vector de errores y se quiere calcular el error cuadratico medio (RMSE por sus isglas en ingles).


```r
set.seed(26)
e = runif(50,-10,10)
round(sqrt(mean(e^2)),3) # forma clasica
```

```
## [1] 5.595
```

```r
e %>% .^2 %>% mean() %>% sqrt() %>% round(3) # usando el operador
```

```
## [1] 5.595
```

Lo anterior se lee: agarre el vector `e`, eleve sus valores al cuadrado, despues calcule la media, despues saquele la raiz y por ultimo redondeelo a 3 cifras.

## Resumen de variables

Para resumir datos la funcion principal es `summarise`, que colapsa una o varias columnas a un dato resumen. Muchas veces se tiene una variable agrupadora (factor) en los datos y se requiere calcular estadisticas por grupo, para esto se usa `group_by` junto con `summarise`. En `group_by` se pueden incluir mas de una variable agrupadora.

Funciones que ayudan a resumir datos son:

* `first(n)`, el primer elemento del vector `x`
* `last(x)`, el ultimo elemento del vector `x`
* `nth(x, n)`, el elemento `n` del vector `x`
* `n()`, el numero de filas en una tabla u observaciones en un grupo
* `n_distinct(x)`, el numero de valores unicos en el vector `x`


```r
dat1 %>% 
  group_by(Gender) %>% 
  summarise(mean(Age))
```

```
## # A tibble: 2 x 2
##   Gender `mean(Age)`
##   <chr>        <dbl>
## 1 female        9.84
## 2 male         10.0
```

```r
dat1 %>% 
  group_by(Gender,Smoke) %>% 
  summarise(mean(Age))
```

```
## # A tibble: 4 x 3
## # Groups:   Gender [2]
##   Gender Smoke `mean(Age)`
##   <chr>  <chr>       <dbl>
## 1 female no           9.37
## 2 female yes         13.3 
## 3 male   no           9.69
## 4 male   yes         13.9
```

```r
dat1 %>% 
  group_by(Gender) %>% 
  summarise(N=n(),
            mean(Age),
            mean(Height),
            mean(LungCap))
```

```
## # A tibble: 2 x 5
##   Gender     N `mean(Age)` `mean(Height)` `mean(LungCap)`
##   <chr>  <int>       <dbl>          <dbl>           <dbl>
## 1 female   318        9.84           60.2            5.35
## 2 male     336       10.0            62.0            6.44
```

## Seleccion y renombre de variables

Para seleccionar columnas la funcion es `select`, la cual puede usar numeros o nombres y los nombres no tienen que llevar comillas. Esto tambien permite reordenar las columans de una tabla. Para el caso de obtener una columna como vector se usa `pull` con el numero o nombre de la columna a jalar. Durante la seleccion se puede cambiar el nombre de la variable, o usando `rename`.

Funciones que ayudan a seleccionar variables son:

* `starts_with('X')`, todas las columnas que empiezan con 'X',
* `ends_with('X')`, todas las columnas que terminan con 'X',
* `contains('X')`, todas las columnas que contienen 'X',
* `matches('X')`, todas las columnas que coinciden con 'X'

Aqui se mezcla funciones de resumen y seleccion para crear resumen de las variables seleccionadas.


```r
dat1 %>% 
  group_by(Gender) %>% 
  select(Age,Height) %>% 
  summarise_all(mean)
```

```
## Adding missing grouping variables: `Gender`
```

```
## # A tibble: 2 x 3
##   Gender   Age Height
##   <chr>  <dbl>  <dbl>
## 1 female  9.84   60.2
## 2 male   10.0    62.0
```

```r
dat1 %>% 
  group_by(Gender) %>% 
  select(Age,Height) %>% 
  summarise_all(.funs = list(~mean(.),
                             ~sd(.)))
```

```
## Adding missing grouping variables: `Gender`
```

```
## # A tibble: 2 x 5
##   Gender Age_mean Height_mean Age_sd Height_sd
##   <chr>     <dbl>       <dbl>  <dbl>     <dbl>
## 1 female     9.84        60.2   2.93      4.79
## 2 male      10.0         62.0   2.98      6.33
```

Aqui se muestra la seleccion de variables, que resulta en un reordenamiento de las mismas. Asi mismo se puede deseleccionar lo que no se quiere, usando `-` para indicar las columans que no se quieren.


```r
airquality %>% 
  select(Temp,Wind,Ozone)
```

```
##     Temp Wind Ozone
## 1     67  7.4    41
## 2     72  8.0    36
## 3     74 12.6    12
## 4     62 11.5    18
## 5     56 14.3    NA
## 6     66 14.9    28
## 7     65  8.6    23
## 8     59 13.8    19
## 9     61 20.1     8
## 10    69  8.6    NA
## 11    74  6.9     7
## 12    69  9.7    16
## 13    66  9.2    11
## 14    68 10.9    14
## 15    58 13.2    18
## 16    64 11.5    14
## 17    66 12.0    34
## 18    57 18.4     6
## 19    68 11.5    30
## 20    62  9.7    11
## 21    59  9.7     1
## 22    73 16.6    11
## 23    61  9.7     4
## 24    61 12.0    32
## 25    57 16.6    NA
## 26    58 14.9    NA
## 27    57  8.0    NA
## 28    67 12.0    23
## 29    81 14.9    45
## 30    79  5.7   115
## 31    76  7.4    37
## 32    78  8.6    NA
## 33    74  9.7    NA
## 34    67 16.1    NA
## 35    84  9.2    NA
## 36    85  8.6    NA
## 37    79 14.3    NA
## 38    82  9.7    29
## 39    87  6.9    NA
## 40    90 13.8    71
## 41    87 11.5    39
## 42    93 10.9    NA
## 43    92  9.2    NA
## 44    82  8.0    23
## 45    80 13.8    NA
## 46    79 11.5    NA
## 47    77 14.9    21
## 48    72 20.7    37
## 49    65  9.2    20
## 50    73 11.5    12
## 51    76 10.3    13
## 52    77  6.3    NA
## 53    76  1.7    NA
## 54    76  4.6    NA
## 55    76  6.3    NA
## 56    75  8.0    NA
## 57    78  8.0    NA
## 58    73 10.3    NA
## 59    80 11.5    NA
## 60    77 14.9    NA
## 61    83  8.0    NA
## 62    84  4.1   135
## 63    85  9.2    49
## 64    81  9.2    32
## 65    84 10.9    NA
## 66    83  4.6    64
## 67    83 10.9    40
## 68    88  5.1    77
## 69    92  6.3    97
## 70    92  5.7    97
## 71    89  7.4    85
## 72    82  8.6    NA
## 73    73 14.3    10
## 74    81 14.9    27
## 75    91 14.9    NA
## 76    80 14.3     7
## 77    81  6.9    48
## 78    82 10.3    35
## 79    84  6.3    61
## 80    87  5.1    79
## 81    85 11.5    63
## 82    74  6.9    16
## 83    81  9.7    NA
## 84    82 11.5    NA
## 85    86  8.6    80
## 86    85  8.0   108
## 87    82  8.6    20
## 88    86 12.0    52
## 89    88  7.4    82
## 90    86  7.4    50
## 91    83  7.4    64
## 92    81  9.2    59
## 93    81  6.9    39
## 94    81 13.8     9
## 95    82  7.4    16
## 96    86  6.9    78
## 97    85  7.4    35
## 98    87  4.6    66
## 99    89  4.0   122
## 100   90 10.3    89
## 101   90  8.0   110
## 102   92  8.6    NA
## 103   86 11.5    NA
## 104   86 11.5    44
## 105   82 11.5    28
## 106   80  9.7    65
## 107   79 11.5    NA
## 108   77 10.3    22
## 109   79  6.3    59
## 110   76  7.4    23
## 111   78 10.9    31
## 112   78 10.3    44
## 113   77 15.5    21
## 114   72 14.3     9
## 115   75 12.6    NA
## 116   79  9.7    45
## 117   81  3.4   168
## 118   86  8.0    73
## 119   88  5.7    NA
## 120   97  9.7    76
## 121   94  2.3   118
## 122   96  6.3    84
## 123   94  6.3    85
## 124   91  6.9    96
## 125   92  5.1    78
## 126   93  2.8    73
## 127   93  4.6    91
## 128   87  7.4    47
## 129   84 15.5    32
## 130   80 10.9    20
## 131   78 10.3    23
## 132   75 10.9    21
## 133   73  9.7    24
## 134   81 14.9    44
## 135   76 15.5    21
## 136   77  6.3    28
## 137   71 10.9     9
## 138   71 11.5    13
## 139   78  6.9    46
## 140   67 13.8    18
## 141   76 10.3    13
## 142   68 10.3    24
## 143   82  8.0    16
## 144   64 12.6    13
## 145   71  9.2    23
## 146   81 10.3    36
## 147   69 10.3     7
## 148   63 16.6    14
## 149   70  6.9    30
## 150   77 13.2    NA
## 151   75 14.3    14
## 152   76  8.0    18
## 153   68 11.5    20
```

```r
dat1 %>% 
  select(-Smoke)
```

```
## # A tibble: 654 x 4
##      Age LungCap Height Gender
##    <int>   <dbl>  <dbl> <chr> 
##  1     9    3.12   57   female
##  2     8    3.17   67.5 female
##  3     7    3.16   54.5 female
##  4     9    2.67   53   male  
##  5     9    3.68   57   male  
##  6     8    5.01   61   female
##  7     6    3.76   58   female
##  8     6    2.24   56   female
##  9     8    3.96   58.5 female
## 10     9    3.83   60   female
## # … with 644 more rows
```

```r
dat1 %>% 
  pull(Age)
```

```
##   [1]  9  8  7  9  9  8  6  6  8  9  6  8  8  8  8  7  5  6  9  9  5  5  4  7  9
##  [26]  3  9  5  8  9  5  9  8  7  5  8  9  8  8  8  9  8  5  8  5  9  7  8  6  8
##  [51]  5  9  9  8  6  9  9  7  4  8  8  8  6  4  8  6  9  7  5  9  8  8  9  9  9
##  [76]  7  5  5  9  6  7  6  8  8  7  8  7  9  5  9  9  9  7  8  8  9  9  9  7  8
## [101]  8  7  9  4  9  6  8  6  7  7  8  7  7  7  7  8  7  5  8  7  9  7  7  6  8
## [126]  8  8  9  7  8  9  8  8  9  8  6  6  8  9  5  7  9  6  9  9  9  6  8  9  8
## [151]  8  9  9  9  7  8  6  9  9  9  7  8  5  8  9  6  9  6  8  5  7  7  4  9  8
## [176]  9  9  9  5  9  7  6  9  9  9  7  5  8  9  7  9  8  9  6  6  8  9  5  6  6
## [201]  9  7  9  8  5  7  6  9  7  9  9  8  9  7  9  4  9  5  8  9  8  3  9  8  6
## [226]  9  8  8  7  6  8  9  4  7  8  8  9  6  8  6  8  9  8  7  9  8  7  9  8  9
## [251]  6  8  9  8  9  9  8  7  5  7  8  9  9  6  8  7  9  7  7  5  9  9  8  8  9
## [276]  6  7  5  9  5  7  6  8  7  8  4  8  5  8  7  7  9  9  8  9  6  8  9  4  6
## [301]  7  9  8  6  8  7  5  8  7 11 10 14 11 11 12 10 11 10 14 13 14 12 12 10 13
## [326] 10 11 10 11 10 13 14 11 10 11 13 10 10 12 10 10 10 11 11 11 10 11 11 13 13
## [351] 11 11 14 11 10 10 10 14 13 10 14 10 11 13 12 13 10 13 11 14 11 13 11 11 10
## [376] 11 11 10 11 13 12 10 10 14 11 10 11 10 11 13 13 10 11 11 12 10 10 11 10 11
## [401] 14 13 12 11 11 11 14 12 10 12 11 10 11 13 10 10 11 13 10 11 10 13 11 10 11
## [426] 11 14 11 13 11 11 10 13 10 13 10 12 10 14 12 10 11 14 12 10 10 10 10 12 13
## [451] 11 12 11 12 11 11 12 12 13 11 12 10 12 13 10 12 10 12 10 11 10 12 14 10 10
## [476] 12 10 10 13 12 12 11 13 12 10 11 11 13 12 13 13 10 12 12 14 11 10 13 11 11
## [501] 13 12 10 10 12 13 11 10 11 11 11 11 11 14 12 13 13 10 12 10 10 12 11 12 11
## [526] 11 12 12 14 11 10 11 12 13 12 11 11 11 14 11 13 12 10 12 13 10 10 10 10 14
## [551] 12 11 11 12 14 14 10 11 11 10 10 12 12 11 12 10 12 13 10 12 10 13 12 10 12
## [576] 10 11 12 11 12 10 13 12 11 11 11 11 12 14 11 11 12 14 11 13 11 10 13 12 11
## [601] 13 14 10 11 11 15 15 18 19 19 16 17 15 15 15 15 15 19 18 16 17 16 15 15 15
## [626] 18 17 15 17 17 16 17 15 15 16 16 15 18 15 16 17 16 16 15 18 15 16 17 16 16
## [651] 15 18 16 15
```

### `select` helpers

Se muestran diferentes usos y resultados de usar `select` helpers para facilidad de seleccion de columnas que cumplan con ciertos criterios. A su vez, se ejemplifica el renombrar las columnas durante la seleccion o usando `rename` (nuevo_nombre = nombre_actual). Un operador especial es `everything()` que selecciona todo; esto es util cuando se quiere reordenar y poner una o varias columnas de primero y despues el resto sin tener que escribir todos los nombres.


```r
select(storms, name:pressure) # columnas desde name hasta pressure
```

```
## # A tibble: 10,010 x 11
##    name   year month   day  hour   lat  long status      category  wind pressure
##    <chr> <dbl> <dbl> <int> <dbl> <dbl> <dbl> <chr>       <ord>    <int>    <int>
##  1 Amy    1975     6    27     0  27.5 -79   tropical d… -1          25     1013
##  2 Amy    1975     6    27     6  28.5 -79   tropical d… -1          25     1013
##  3 Amy    1975     6    27    12  29.5 -79   tropical d… -1          25     1013
##  4 Amy    1975     6    27    18  30.5 -79   tropical d… -1          25     1013
##  5 Amy    1975     6    28     0  31.5 -78.8 tropical d… -1          25     1012
##  6 Amy    1975     6    28     6  32.4 -78.7 tropical d… -1          25     1012
##  7 Amy    1975     6    28    12  33.3 -78   tropical d… -1          25     1011
##  8 Amy    1975     6    28    18  34   -77   tropical d… -1          30     1006
##  9 Amy    1975     6    29     0  34.4 -75.8 tropical s… 0           35     1004
## 10 Amy    1975     6    29     6  34   -74.8 tropical s… 0           40     1002
## # … with 10,000 more rows
```

```r
storms %>% 
  select(-c(name, pressure)) # columnas menos name y pressure
```

```
## # A tibble: 10,010 x 11
##     year month   day  hour   lat  long status category  wind ts_diameter
##    <dbl> <dbl> <int> <dbl> <dbl> <dbl> <chr>  <ord>    <int>       <dbl>
##  1  1975     6    27     0  27.5 -79   tropi… -1          25          NA
##  2  1975     6    27     6  28.5 -79   tropi… -1          25          NA
##  3  1975     6    27    12  29.5 -79   tropi… -1          25          NA
##  4  1975     6    27    18  30.5 -79   tropi… -1          25          NA
##  5  1975     6    28     0  31.5 -78.8 tropi… -1          25          NA
##  6  1975     6    28     6  32.4 -78.7 tropi… -1          25          NA
##  7  1975     6    28    12  33.3 -78   tropi… -1          25          NA
##  8  1975     6    28    18  34   -77   tropi… -1          30          NA
##  9  1975     6    29     0  34.4 -75.8 tropi… 0           35          NA
## 10  1975     6    29     6  34   -74.8 tropi… 0           40          NA
## # … with 10,000 more rows, and 1 more variable: hu_diameter <dbl>
```

```r
iris %>% 
  select(starts_with("Sepal")) # columnas que empiezan con 'Sepal'
```

```
##     Sepal.Length Sepal.Width
## 1            5.1         3.5
## 2            4.9         3.0
## 3            4.7         3.2
## 4            4.6         3.1
## 5            5.0         3.6
## 6            5.4         3.9
## 7            4.6         3.4
## 8            5.0         3.4
## 9            4.4         2.9
## 10           4.9         3.1
## 11           5.4         3.7
## 12           4.8         3.4
## 13           4.8         3.0
## 14           4.3         3.0
## 15           5.8         4.0
## 16           5.7         4.4
## 17           5.4         3.9
## 18           5.1         3.5
## 19           5.7         3.8
## 20           5.1         3.8
## 21           5.4         3.4
## 22           5.1         3.7
## 23           4.6         3.6
## 24           5.1         3.3
## 25           4.8         3.4
## 26           5.0         3.0
## 27           5.0         3.4
## 28           5.2         3.5
## 29           5.2         3.4
## 30           4.7         3.2
## 31           4.8         3.1
## 32           5.4         3.4
## 33           5.2         4.1
## 34           5.5         4.2
## 35           4.9         3.1
## 36           5.0         3.2
## 37           5.5         3.5
## 38           4.9         3.6
## 39           4.4         3.0
## 40           5.1         3.4
## 41           5.0         3.5
## 42           4.5         2.3
## 43           4.4         3.2
## 44           5.0         3.5
## 45           5.1         3.8
## 46           4.8         3.0
## 47           5.1         3.8
## 48           4.6         3.2
## 49           5.3         3.7
## 50           5.0         3.3
## 51           7.0         3.2
## 52           6.4         3.2
## 53           6.9         3.1
## 54           5.5         2.3
## 55           6.5         2.8
## 56           5.7         2.8
## 57           6.3         3.3
## 58           4.9         2.4
## 59           6.6         2.9
## 60           5.2         2.7
## 61           5.0         2.0
## 62           5.9         3.0
## 63           6.0         2.2
## 64           6.1         2.9
## 65           5.6         2.9
## 66           6.7         3.1
## 67           5.6         3.0
## 68           5.8         2.7
## 69           6.2         2.2
## 70           5.6         2.5
## 71           5.9         3.2
## 72           6.1         2.8
## 73           6.3         2.5
## 74           6.1         2.8
## 75           6.4         2.9
## 76           6.6         3.0
## 77           6.8         2.8
## 78           6.7         3.0
## 79           6.0         2.9
## 80           5.7         2.6
## 81           5.5         2.4
## 82           5.5         2.4
## 83           5.8         2.7
## 84           6.0         2.7
## 85           5.4         3.0
## 86           6.0         3.4
## 87           6.7         3.1
## 88           6.3         2.3
## 89           5.6         3.0
## 90           5.5         2.5
## 91           5.5         2.6
## 92           6.1         3.0
## 93           5.8         2.6
## 94           5.0         2.3
## 95           5.6         2.7
## 96           5.7         3.0
## 97           5.7         2.9
## 98           6.2         2.9
## 99           5.1         2.5
## 100          5.7         2.8
## 101          6.3         3.3
## 102          5.8         2.7
## 103          7.1         3.0
## 104          6.3         2.9
## 105          6.5         3.0
## 106          7.6         3.0
## 107          4.9         2.5
## 108          7.3         2.9
## 109          6.7         2.5
## 110          7.2         3.6
## 111          6.5         3.2
## 112          6.4         2.7
## 113          6.8         3.0
## 114          5.7         2.5
## 115          5.8         2.8
## 116          6.4         3.2
## 117          6.5         3.0
## 118          7.7         3.8
## 119          7.7         2.6
## 120          6.0         2.2
## 121          6.9         3.2
## 122          5.6         2.8
## 123          7.7         2.8
## 124          6.3         2.7
## 125          6.7         3.3
## 126          7.2         3.2
## 127          6.2         2.8
## 128          6.1         3.0
## 129          6.4         2.8
## 130          7.2         3.0
## 131          7.4         2.8
## 132          7.9         3.8
## 133          6.4         2.8
## 134          6.3         2.8
## 135          6.1         2.6
## 136          7.7         3.0
## 137          6.3         3.4
## 138          6.4         3.1
## 139          6.0         3.0
## 140          6.9         3.1
## 141          6.7         3.1
## 142          6.9         3.1
## 143          5.8         2.7
## 144          6.8         3.2
## 145          6.7         3.3
## 146          6.7         3.0
## 147          6.3         2.5
## 148          6.5         3.0
## 149          6.2         3.4
## 150          5.9         3.0
```

```r
iris %>% 
  select(ends_with("Width")) # columnas que terminan con 'Width'
```

```
##     Sepal.Width Petal.Width
## 1           3.5         0.2
## 2           3.0         0.2
## 3           3.2         0.2
## 4           3.1         0.2
## 5           3.6         0.2
## 6           3.9         0.4
## 7           3.4         0.3
## 8           3.4         0.2
## 9           2.9         0.2
## 10          3.1         0.1
## 11          3.7         0.2
## 12          3.4         0.2
## 13          3.0         0.1
## 14          3.0         0.1
## 15          4.0         0.2
## 16          4.4         0.4
## 17          3.9         0.4
## 18          3.5         0.3
## 19          3.8         0.3
## 20          3.8         0.3
## 21          3.4         0.2
## 22          3.7         0.4
## 23          3.6         0.2
## 24          3.3         0.5
## 25          3.4         0.2
## 26          3.0         0.2
## 27          3.4         0.4
## 28          3.5         0.2
## 29          3.4         0.2
## 30          3.2         0.2
## 31          3.1         0.2
## 32          3.4         0.4
## 33          4.1         0.1
## 34          4.2         0.2
## 35          3.1         0.2
## 36          3.2         0.2
## 37          3.5         0.2
## 38          3.6         0.1
## 39          3.0         0.2
## 40          3.4         0.2
## 41          3.5         0.3
## 42          2.3         0.3
## 43          3.2         0.2
## 44          3.5         0.6
## 45          3.8         0.4
## 46          3.0         0.3
## 47          3.8         0.2
## 48          3.2         0.2
## 49          3.7         0.2
## 50          3.3         0.2
## 51          3.2         1.4
## 52          3.2         1.5
## 53          3.1         1.5
## 54          2.3         1.3
## 55          2.8         1.5
## 56          2.8         1.3
## 57          3.3         1.6
## 58          2.4         1.0
## 59          2.9         1.3
## 60          2.7         1.4
## 61          2.0         1.0
## 62          3.0         1.5
## 63          2.2         1.0
## 64          2.9         1.4
## 65          2.9         1.3
## 66          3.1         1.4
## 67          3.0         1.5
## 68          2.7         1.0
## 69          2.2         1.5
## 70          2.5         1.1
## 71          3.2         1.8
## 72          2.8         1.3
## 73          2.5         1.5
## 74          2.8         1.2
## 75          2.9         1.3
## 76          3.0         1.4
## 77          2.8         1.4
## 78          3.0         1.7
## 79          2.9         1.5
## 80          2.6         1.0
## 81          2.4         1.1
## 82          2.4         1.0
## 83          2.7         1.2
## 84          2.7         1.6
## 85          3.0         1.5
## 86          3.4         1.6
## 87          3.1         1.5
## 88          2.3         1.3
## 89          3.0         1.3
## 90          2.5         1.3
## 91          2.6         1.2
## 92          3.0         1.4
## 93          2.6         1.2
## 94          2.3         1.0
## 95          2.7         1.3
## 96          3.0         1.2
## 97          2.9         1.3
## 98          2.9         1.3
## 99          2.5         1.1
## 100         2.8         1.3
## 101         3.3         2.5
## 102         2.7         1.9
## 103         3.0         2.1
## 104         2.9         1.8
## 105         3.0         2.2
## 106         3.0         2.1
## 107         2.5         1.7
## 108         2.9         1.8
## 109         2.5         1.8
## 110         3.6         2.5
## 111         3.2         2.0
## 112         2.7         1.9
## 113         3.0         2.1
## 114         2.5         2.0
## 115         2.8         2.4
## 116         3.2         2.3
## 117         3.0         1.8
## 118         3.8         2.2
## 119         2.6         2.3
## 120         2.2         1.5
## 121         3.2         2.3
## 122         2.8         2.0
## 123         2.8         2.0
## 124         2.7         1.8
## 125         3.3         2.1
## 126         3.2         1.8
## 127         2.8         1.8
## 128         3.0         1.8
## 129         2.8         2.1
## 130         3.0         1.6
## 131         2.8         1.9
## 132         3.8         2.0
## 133         2.8         2.2
## 134         2.8         1.5
## 135         2.6         1.4
## 136         3.0         2.3
## 137         3.4         2.4
## 138         3.1         1.8
## 139         3.0         1.8
## 140         3.1         2.1
## 141         3.1         2.4
## 142         3.1         2.3
## 143         2.7         1.9
## 144         3.2         2.3
## 145         3.3         2.5
## 146         3.0         2.3
## 147         2.5         1.9
## 148         3.0         2.0
## 149         3.4         2.3
## 150         3.0         1.8
```

```r
storms %>% 
  select(contains("d")) # columnas que contienen 'd'
```

```
## # A tibble: 10,010 x 4
##      day  wind ts_diameter hu_diameter
##    <int> <int>       <dbl>       <dbl>
##  1    27    25          NA          NA
##  2    27    25          NA          NA
##  3    27    25          NA          NA
##  4    27    25          NA          NA
##  5    28    25          NA          NA
##  6    28    25          NA          NA
##  7    28    25          NA          NA
##  8    28    30          NA          NA
##  9    29    35          NA          NA
## 10    29    40          NA          NA
## # … with 10,000 more rows
```

```r
iris %>% 
  select(Especie = Species, everything()) # renombrar seleccion y seleccionar el resto
```

```
##        Especie Sepal.Length Sepal.Width Petal.Length Petal.Width
## 1       setosa          5.1         3.5          1.4         0.2
## 2       setosa          4.9         3.0          1.4         0.2
## 3       setosa          4.7         3.2          1.3         0.2
## 4       setosa          4.6         3.1          1.5         0.2
## 5       setosa          5.0         3.6          1.4         0.2
## 6       setosa          5.4         3.9          1.7         0.4
## 7       setosa          4.6         3.4          1.4         0.3
## 8       setosa          5.0         3.4          1.5         0.2
## 9       setosa          4.4         2.9          1.4         0.2
## 10      setosa          4.9         3.1          1.5         0.1
## 11      setosa          5.4         3.7          1.5         0.2
## 12      setosa          4.8         3.4          1.6         0.2
## 13      setosa          4.8         3.0          1.4         0.1
## 14      setosa          4.3         3.0          1.1         0.1
## 15      setosa          5.8         4.0          1.2         0.2
## 16      setosa          5.7         4.4          1.5         0.4
## 17      setosa          5.4         3.9          1.3         0.4
## 18      setosa          5.1         3.5          1.4         0.3
## 19      setosa          5.7         3.8          1.7         0.3
## 20      setosa          5.1         3.8          1.5         0.3
## 21      setosa          5.4         3.4          1.7         0.2
## 22      setosa          5.1         3.7          1.5         0.4
## 23      setosa          4.6         3.6          1.0         0.2
## 24      setosa          5.1         3.3          1.7         0.5
## 25      setosa          4.8         3.4          1.9         0.2
## 26      setosa          5.0         3.0          1.6         0.2
## 27      setosa          5.0         3.4          1.6         0.4
## 28      setosa          5.2         3.5          1.5         0.2
## 29      setosa          5.2         3.4          1.4         0.2
## 30      setosa          4.7         3.2          1.6         0.2
## 31      setosa          4.8         3.1          1.6         0.2
## 32      setosa          5.4         3.4          1.5         0.4
## 33      setosa          5.2         4.1          1.5         0.1
## 34      setosa          5.5         4.2          1.4         0.2
## 35      setosa          4.9         3.1          1.5         0.2
## 36      setosa          5.0         3.2          1.2         0.2
## 37      setosa          5.5         3.5          1.3         0.2
## 38      setosa          4.9         3.6          1.4         0.1
## 39      setosa          4.4         3.0          1.3         0.2
## 40      setosa          5.1         3.4          1.5         0.2
## 41      setosa          5.0         3.5          1.3         0.3
## 42      setosa          4.5         2.3          1.3         0.3
## 43      setosa          4.4         3.2          1.3         0.2
## 44      setosa          5.0         3.5          1.6         0.6
## 45      setosa          5.1         3.8          1.9         0.4
## 46      setosa          4.8         3.0          1.4         0.3
## 47      setosa          5.1         3.8          1.6         0.2
## 48      setosa          4.6         3.2          1.4         0.2
## 49      setosa          5.3         3.7          1.5         0.2
## 50      setosa          5.0         3.3          1.4         0.2
## 51  versicolor          7.0         3.2          4.7         1.4
## 52  versicolor          6.4         3.2          4.5         1.5
## 53  versicolor          6.9         3.1          4.9         1.5
## 54  versicolor          5.5         2.3          4.0         1.3
## 55  versicolor          6.5         2.8          4.6         1.5
## 56  versicolor          5.7         2.8          4.5         1.3
## 57  versicolor          6.3         3.3          4.7         1.6
## 58  versicolor          4.9         2.4          3.3         1.0
## 59  versicolor          6.6         2.9          4.6         1.3
## 60  versicolor          5.2         2.7          3.9         1.4
## 61  versicolor          5.0         2.0          3.5         1.0
## 62  versicolor          5.9         3.0          4.2         1.5
## 63  versicolor          6.0         2.2          4.0         1.0
## 64  versicolor          6.1         2.9          4.7         1.4
## 65  versicolor          5.6         2.9          3.6         1.3
## 66  versicolor          6.7         3.1          4.4         1.4
## 67  versicolor          5.6         3.0          4.5         1.5
## 68  versicolor          5.8         2.7          4.1         1.0
## 69  versicolor          6.2         2.2          4.5         1.5
## 70  versicolor          5.6         2.5          3.9         1.1
## 71  versicolor          5.9         3.2          4.8         1.8
## 72  versicolor          6.1         2.8          4.0         1.3
## 73  versicolor          6.3         2.5          4.9         1.5
## 74  versicolor          6.1         2.8          4.7         1.2
## 75  versicolor          6.4         2.9          4.3         1.3
## 76  versicolor          6.6         3.0          4.4         1.4
## 77  versicolor          6.8         2.8          4.8         1.4
## 78  versicolor          6.7         3.0          5.0         1.7
## 79  versicolor          6.0         2.9          4.5         1.5
## 80  versicolor          5.7         2.6          3.5         1.0
## 81  versicolor          5.5         2.4          3.8         1.1
## 82  versicolor          5.5         2.4          3.7         1.0
## 83  versicolor          5.8         2.7          3.9         1.2
## 84  versicolor          6.0         2.7          5.1         1.6
## 85  versicolor          5.4         3.0          4.5         1.5
## 86  versicolor          6.0         3.4          4.5         1.6
## 87  versicolor          6.7         3.1          4.7         1.5
## 88  versicolor          6.3         2.3          4.4         1.3
## 89  versicolor          5.6         3.0          4.1         1.3
## 90  versicolor          5.5         2.5          4.0         1.3
## 91  versicolor          5.5         2.6          4.4         1.2
## 92  versicolor          6.1         3.0          4.6         1.4
## 93  versicolor          5.8         2.6          4.0         1.2
## 94  versicolor          5.0         2.3          3.3         1.0
## 95  versicolor          5.6         2.7          4.2         1.3
## 96  versicolor          5.7         3.0          4.2         1.2
## 97  versicolor          5.7         2.9          4.2         1.3
## 98  versicolor          6.2         2.9          4.3         1.3
## 99  versicolor          5.1         2.5          3.0         1.1
## 100 versicolor          5.7         2.8          4.1         1.3
## 101  virginica          6.3         3.3          6.0         2.5
## 102  virginica          5.8         2.7          5.1         1.9
## 103  virginica          7.1         3.0          5.9         2.1
## 104  virginica          6.3         2.9          5.6         1.8
## 105  virginica          6.5         3.0          5.8         2.2
## 106  virginica          7.6         3.0          6.6         2.1
## 107  virginica          4.9         2.5          4.5         1.7
## 108  virginica          7.3         2.9          6.3         1.8
## 109  virginica          6.7         2.5          5.8         1.8
## 110  virginica          7.2         3.6          6.1         2.5
## 111  virginica          6.5         3.2          5.1         2.0
## 112  virginica          6.4         2.7          5.3         1.9
## 113  virginica          6.8         3.0          5.5         2.1
## 114  virginica          5.7         2.5          5.0         2.0
## 115  virginica          5.8         2.8          5.1         2.4
## 116  virginica          6.4         3.2          5.3         2.3
## 117  virginica          6.5         3.0          5.5         1.8
## 118  virginica          7.7         3.8          6.7         2.2
## 119  virginica          7.7         2.6          6.9         2.3
## 120  virginica          6.0         2.2          5.0         1.5
## 121  virginica          6.9         3.2          5.7         2.3
## 122  virginica          5.6         2.8          4.9         2.0
## 123  virginica          7.7         2.8          6.7         2.0
## 124  virginica          6.3         2.7          4.9         1.8
## 125  virginica          6.7         3.3          5.7         2.1
## 126  virginica          7.2         3.2          6.0         1.8
## 127  virginica          6.2         2.8          4.8         1.8
## 128  virginica          6.1         3.0          4.9         1.8
## 129  virginica          6.4         2.8          5.6         2.1
## 130  virginica          7.2         3.0          5.8         1.6
## 131  virginica          7.4         2.8          6.1         1.9
## 132  virginica          7.9         3.8          6.4         2.0
## 133  virginica          6.4         2.8          5.6         2.2
## 134  virginica          6.3         2.8          5.1         1.5
## 135  virginica          6.1         2.6          5.6         1.4
## 136  virginica          7.7         3.0          6.1         2.3
## 137  virginica          6.3         3.4          5.6         2.4
## 138  virginica          6.4         3.1          5.5         1.8
## 139  virginica          6.0         3.0          4.8         1.8
## 140  virginica          6.9         3.1          5.4         2.1
## 141  virginica          6.7         3.1          5.6         2.4
## 142  virginica          6.9         3.1          5.1         2.3
## 143  virginica          5.8         2.7          5.1         1.9
## 144  virginica          6.8         3.2          5.9         2.3
## 145  virginica          6.7         3.3          5.7         2.5
## 146  virginica          6.7         3.0          5.2         2.3
## 147  virginica          6.3         2.5          5.0         1.9
## 148  virginica          6.5         3.0          5.2         2.0
## 149  virginica          6.2         3.4          5.4         2.3
## 150  virginica          5.9         3.0          5.1         1.8
```

```r
iris %>% 
  rename(Especie = Species) # renombrar columna
```

```
##     Sepal.Length Sepal.Width Petal.Length Petal.Width    Especie
## 1            5.1         3.5          1.4         0.2     setosa
## 2            4.9         3.0          1.4         0.2     setosa
## 3            4.7         3.2          1.3         0.2     setosa
## 4            4.6         3.1          1.5         0.2     setosa
## 5            5.0         3.6          1.4         0.2     setosa
## 6            5.4         3.9          1.7         0.4     setosa
## 7            4.6         3.4          1.4         0.3     setosa
## 8            5.0         3.4          1.5         0.2     setosa
## 9            4.4         2.9          1.4         0.2     setosa
## 10           4.9         3.1          1.5         0.1     setosa
## 11           5.4         3.7          1.5         0.2     setosa
## 12           4.8         3.4          1.6         0.2     setosa
## 13           4.8         3.0          1.4         0.1     setosa
## 14           4.3         3.0          1.1         0.1     setosa
## 15           5.8         4.0          1.2         0.2     setosa
## 16           5.7         4.4          1.5         0.4     setosa
## 17           5.4         3.9          1.3         0.4     setosa
## 18           5.1         3.5          1.4         0.3     setosa
## 19           5.7         3.8          1.7         0.3     setosa
## 20           5.1         3.8          1.5         0.3     setosa
## 21           5.4         3.4          1.7         0.2     setosa
## 22           5.1         3.7          1.5         0.4     setosa
## 23           4.6         3.6          1.0         0.2     setosa
## 24           5.1         3.3          1.7         0.5     setosa
## 25           4.8         3.4          1.9         0.2     setosa
## 26           5.0         3.0          1.6         0.2     setosa
## 27           5.0         3.4          1.6         0.4     setosa
## 28           5.2         3.5          1.5         0.2     setosa
## 29           5.2         3.4          1.4         0.2     setosa
## 30           4.7         3.2          1.6         0.2     setosa
## 31           4.8         3.1          1.6         0.2     setosa
## 32           5.4         3.4          1.5         0.4     setosa
## 33           5.2         4.1          1.5         0.1     setosa
## 34           5.5         4.2          1.4         0.2     setosa
## 35           4.9         3.1          1.5         0.2     setosa
## 36           5.0         3.2          1.2         0.2     setosa
## 37           5.5         3.5          1.3         0.2     setosa
## 38           4.9         3.6          1.4         0.1     setosa
## 39           4.4         3.0          1.3         0.2     setosa
## 40           5.1         3.4          1.5         0.2     setosa
## 41           5.0         3.5          1.3         0.3     setosa
## 42           4.5         2.3          1.3         0.3     setosa
## 43           4.4         3.2          1.3         0.2     setosa
## 44           5.0         3.5          1.6         0.6     setosa
## 45           5.1         3.8          1.9         0.4     setosa
## 46           4.8         3.0          1.4         0.3     setosa
## 47           5.1         3.8          1.6         0.2     setosa
## 48           4.6         3.2          1.4         0.2     setosa
## 49           5.3         3.7          1.5         0.2     setosa
## 50           5.0         3.3          1.4         0.2     setosa
## 51           7.0         3.2          4.7         1.4 versicolor
## 52           6.4         3.2          4.5         1.5 versicolor
## 53           6.9         3.1          4.9         1.5 versicolor
## 54           5.5         2.3          4.0         1.3 versicolor
## 55           6.5         2.8          4.6         1.5 versicolor
## 56           5.7         2.8          4.5         1.3 versicolor
## 57           6.3         3.3          4.7         1.6 versicolor
## 58           4.9         2.4          3.3         1.0 versicolor
## 59           6.6         2.9          4.6         1.3 versicolor
## 60           5.2         2.7          3.9         1.4 versicolor
## 61           5.0         2.0          3.5         1.0 versicolor
## 62           5.9         3.0          4.2         1.5 versicolor
## 63           6.0         2.2          4.0         1.0 versicolor
## 64           6.1         2.9          4.7         1.4 versicolor
## 65           5.6         2.9          3.6         1.3 versicolor
## 66           6.7         3.1          4.4         1.4 versicolor
## 67           5.6         3.0          4.5         1.5 versicolor
## 68           5.8         2.7          4.1         1.0 versicolor
## 69           6.2         2.2          4.5         1.5 versicolor
## 70           5.6         2.5          3.9         1.1 versicolor
## 71           5.9         3.2          4.8         1.8 versicolor
## 72           6.1         2.8          4.0         1.3 versicolor
## 73           6.3         2.5          4.9         1.5 versicolor
## 74           6.1         2.8          4.7         1.2 versicolor
## 75           6.4         2.9          4.3         1.3 versicolor
## 76           6.6         3.0          4.4         1.4 versicolor
## 77           6.8         2.8          4.8         1.4 versicolor
## 78           6.7         3.0          5.0         1.7 versicolor
## 79           6.0         2.9          4.5         1.5 versicolor
## 80           5.7         2.6          3.5         1.0 versicolor
## 81           5.5         2.4          3.8         1.1 versicolor
## 82           5.5         2.4          3.7         1.0 versicolor
## 83           5.8         2.7          3.9         1.2 versicolor
## 84           6.0         2.7          5.1         1.6 versicolor
## 85           5.4         3.0          4.5         1.5 versicolor
## 86           6.0         3.4          4.5         1.6 versicolor
## 87           6.7         3.1          4.7         1.5 versicolor
## 88           6.3         2.3          4.4         1.3 versicolor
## 89           5.6         3.0          4.1         1.3 versicolor
## 90           5.5         2.5          4.0         1.3 versicolor
## 91           5.5         2.6          4.4         1.2 versicolor
## 92           6.1         3.0          4.6         1.4 versicolor
## 93           5.8         2.6          4.0         1.2 versicolor
## 94           5.0         2.3          3.3         1.0 versicolor
## 95           5.6         2.7          4.2         1.3 versicolor
## 96           5.7         3.0          4.2         1.2 versicolor
## 97           5.7         2.9          4.2         1.3 versicolor
## 98           6.2         2.9          4.3         1.3 versicolor
## 99           5.1         2.5          3.0         1.1 versicolor
## 100          5.7         2.8          4.1         1.3 versicolor
## 101          6.3         3.3          6.0         2.5  virginica
## 102          5.8         2.7          5.1         1.9  virginica
## 103          7.1         3.0          5.9         2.1  virginica
## 104          6.3         2.9          5.6         1.8  virginica
## 105          6.5         3.0          5.8         2.2  virginica
## 106          7.6         3.0          6.6         2.1  virginica
## 107          4.9         2.5          4.5         1.7  virginica
## 108          7.3         2.9          6.3         1.8  virginica
## 109          6.7         2.5          5.8         1.8  virginica
## 110          7.2         3.6          6.1         2.5  virginica
## 111          6.5         3.2          5.1         2.0  virginica
## 112          6.4         2.7          5.3         1.9  virginica
## 113          6.8         3.0          5.5         2.1  virginica
## 114          5.7         2.5          5.0         2.0  virginica
## 115          5.8         2.8          5.1         2.4  virginica
## 116          6.4         3.2          5.3         2.3  virginica
## 117          6.5         3.0          5.5         1.8  virginica
## 118          7.7         3.8          6.7         2.2  virginica
## 119          7.7         2.6          6.9         2.3  virginica
## 120          6.0         2.2          5.0         1.5  virginica
## 121          6.9         3.2          5.7         2.3  virginica
## 122          5.6         2.8          4.9         2.0  virginica
## 123          7.7         2.8          6.7         2.0  virginica
## 124          6.3         2.7          4.9         1.8  virginica
## 125          6.7         3.3          5.7         2.1  virginica
## 126          7.2         3.2          6.0         1.8  virginica
## 127          6.2         2.8          4.8         1.8  virginica
## 128          6.1         3.0          4.9         1.8  virginica
## 129          6.4         2.8          5.6         2.1  virginica
## 130          7.2         3.0          5.8         1.6  virginica
## 131          7.4         2.8          6.1         1.9  virginica
## 132          7.9         3.8          6.4         2.0  virginica
## 133          6.4         2.8          5.6         2.2  virginica
## 134          6.3         2.8          5.1         1.5  virginica
## 135          6.1         2.6          5.6         1.4  virginica
## 136          7.7         3.0          6.1         2.3  virginica
## 137          6.3         3.4          5.6         2.4  virginica
## 138          6.4         3.1          5.5         1.8  virginica
## 139          6.0         3.0          4.8         1.8  virginica
## 140          6.9         3.1          5.4         2.1  virginica
## 141          6.7         3.1          5.6         2.4  virginica
## 142          6.9         3.1          5.1         2.3  virginica
## 143          5.8         2.7          5.1         1.9  virginica
## 144          6.8         3.2          5.9         2.3  virginica
## 145          6.7         3.3          5.7         2.5  virginica
## 146          6.7         3.0          5.2         2.3  virginica
## 147          6.3         2.5          5.0         1.9  virginica
## 148          6.5         3.0          5.2         2.0  virginica
## 149          6.2         3.4          5.4         2.3  virginica
## 150          5.9         3.0          5.1         1.8  virginica
```

## Filtrado de observaciones

Para filtrar observaciones de acuerdo a uno a varios criterios se usa la funcion `filter`, asi como operadores logicos y funciones auxiliares. 

Funciones que ayudan a filtrar observaciones son las mismas de los [Operadores logicos].

Dos de las funciones auxiliares mas utiles son:

* `between(x,left,right)` que filtra observaciones para la variable `x` que se encuentren entre `left` (limite inferior) y `right` (limite superior); esta es mas util para variables numericas,
* `x %in% c(a,b,c)` que filtra observaciones para la variable `x` que se encuentren en el vector `c(a,b,c)`; esta es mas util para variables de texto o factor

Se muestran diferentes ejemplos de como filtrar observaciones. Cuando se requiere que una observacion cumpla varios criterios, estas condiciones se pueden separar por medio de comas (`,`), que es lo mismo que usar el operador logico `&`. Si se requiere una u otra condicion se puede usar el operador logico `|`, pero en ese caso y dependiendo de lo deseado es mejor usar `between()` o `%in%`.


```r
filter(airquality,Temp > 85)
```

```
##    Ozone Solar.R Wind Temp Month Day
## 1     NA     273  6.9   87     6   8
## 2     71     291 13.8   90     6   9
## 3     39     323 11.5   87     6  10
## 4     NA     259 10.9   93     6  11
## 5     NA     250  9.2   92     6  12
## 6     77     276  5.1   88     7   7
## 7     97     267  6.3   92     7   8
## 8     97     272  5.7   92     7   9
## 9     85     175  7.4   89     7  10
## 10    NA     291 14.9   91     7  14
## 11    79     187  5.1   87     7  19
## 12    80     294  8.6   86     7  24
## 13    52      82 12.0   86     7  27
## 14    82     213  7.4   88     7  28
## 15    50     275  7.4   86     7  29
## 16    78      NA  6.9   86     8   4
## 17    66      NA  4.6   87     8   6
## 18   122     255  4.0   89     8   7
## 19    89     229 10.3   90     8   8
## 20   110     207  8.0   90     8   9
## 21    NA     222  8.6   92     8  10
## 22    NA     137 11.5   86     8  11
## 23    44     192 11.5   86     8  12
## 24    73     215  8.0   86     8  26
## 25    NA     153  5.7   88     8  27
## 26    76     203  9.7   97     8  28
## 27   118     225  2.3   94     8  29
## 28    84     237  6.3   96     8  30
## 29    85     188  6.3   94     8  31
## 30    96     167  6.9   91     9   1
## 31    78     197  5.1   92     9   2
## 32    73     183  2.8   93     9   3
## 33    91     189  4.6   93     9   4
## 34    47      95  7.4   87     9   5
```

```r
airquality %>% 
  filter(Temp > 85)
```

```
##    Ozone Solar.R Wind Temp Month Day
## 1     NA     273  6.9   87     6   8
## 2     71     291 13.8   90     6   9
## 3     39     323 11.5   87     6  10
## 4     NA     259 10.9   93     6  11
## 5     NA     250  9.2   92     6  12
## 6     77     276  5.1   88     7   7
## 7     97     267  6.3   92     7   8
## 8     97     272  5.7   92     7   9
## 9     85     175  7.4   89     7  10
## 10    NA     291 14.9   91     7  14
## 11    79     187  5.1   87     7  19
## 12    80     294  8.6   86     7  24
## 13    52      82 12.0   86     7  27
## 14    82     213  7.4   88     7  28
## 15    50     275  7.4   86     7  29
## 16    78      NA  6.9   86     8   4
## 17    66      NA  4.6   87     8   6
## 18   122     255  4.0   89     8   7
## 19    89     229 10.3   90     8   8
## 20   110     207  8.0   90     8   9
## 21    NA     222  8.6   92     8  10
## 22    NA     137 11.5   86     8  11
## 23    44     192 11.5   86     8  12
## 24    73     215  8.0   86     8  26
## 25    NA     153  5.7   88     8  27
## 26    76     203  9.7   97     8  28
## 27   118     225  2.3   94     8  29
## 28    84     237  6.3   96     8  30
## 29    85     188  6.3   94     8  31
## 30    96     167  6.9   91     9   1
## 31    78     197  5.1   92     9   2
## 32    73     183  2.8   93     9   3
## 33    91     189  4.6   93     9   4
## 34    47      95  7.4   87     9   5
```

```r
airquality %>% 
  filter(Temp > 75, Wind > 10)
```

```
##    Ozone Solar.R Wind Temp Month Day
## 1     45     252 14.9   81     5  29
## 2     NA     264 14.3   79     6   6
## 3     71     291 13.8   90     6   9
## 4     39     323 11.5   87     6  10
## 5     NA     259 10.9   93     6  11
## 6     NA     332 13.8   80     6  14
## 7     NA     322 11.5   79     6  15
## 8     21     191 14.9   77     6  16
## 9     13     137 10.3   76     6  20
## 10    NA      98 11.5   80     6  28
## 11    NA      31 14.9   77     6  29
## 12    NA     101 10.9   84     7   4
## 13    40     314 10.9   83     7   6
## 14    27     175 14.9   81     7  13
## 15    NA     291 14.9   91     7  14
## 16     7      48 14.3   80     7  15
## 17    35     274 10.3   82     7  17
## 18    63     220 11.5   85     7  20
## 19    NA     295 11.5   82     7  23
## 20    52      82 12.0   86     7  27
## 21     9      24 13.8   81     8   2
## 22    89     229 10.3   90     8   8
## 23    NA     137 11.5   86     8  11
## 24    44     192 11.5   86     8  12
## 25    28     273 11.5   82     8  13
## 26    NA      64 11.5   79     8  15
## 27    22      71 10.3   77     8  16
## 28    31     244 10.9   78     8  19
## 29    44     190 10.3   78     8  20
## 30    21     259 15.5   77     8  21
## 31    32      92 15.5   84     9   6
## 32    20     252 10.9   80     9   7
## 33    23     220 10.3   78     9   8
## 34    44     236 14.9   81     9  11
## 35    21     259 15.5   76     9  12
## 36    13      27 10.3   76     9  18
## 37    36     139 10.3   81     9  23
## 38    NA     145 13.2   77     9  27
```

```r
airquality %>% 
  filter(between(Temp, 70, 80))
```

```
##    Ozone Solar.R Wind Temp Month Day
## 1     36     118  8.0   72     5   2
## 2     12     149 12.6   74     5   3
## 3      7      NA  6.9   74     5  11
## 4     11     320 16.6   73     5  22
## 5    115     223  5.7   79     5  30
## 6     37     279  7.4   76     5  31
## 7     NA     286  8.6   78     6   1
## 8     NA     287  9.7   74     6   2
## 9     NA     264 14.3   79     6   6
## 10    NA     332 13.8   80     6  14
## 11    NA     322 11.5   79     6  15
## 12    21     191 14.9   77     6  16
## 13    37     284 20.7   72     6  17
## 14    12     120 11.5   73     6  19
## 15    13     137 10.3   76     6  20
## 16    NA     150  6.3   77     6  21
## 17    NA      59  1.7   76     6  22
## 18    NA      91  4.6   76     6  23
## 19    NA     250  6.3   76     6  24
## 20    NA     135  8.0   75     6  25
## 21    NA     127  8.0   78     6  26
## 22    NA      47 10.3   73     6  27
## 23    NA      98 11.5   80     6  28
## 24    NA      31 14.9   77     6  29
## 25    10     264 14.3   73     7  12
## 26     7      48 14.3   80     7  15
## 27    16       7  6.9   74     7  21
## 28    65     157  9.7   80     8  14
## 29    NA      64 11.5   79     8  15
## 30    22      71 10.3   77     8  16
## 31    59      51  6.3   79     8  17
## 32    23     115  7.4   76     8  18
## 33    31     244 10.9   78     8  19
## 34    44     190 10.3   78     8  20
## 35    21     259 15.5   77     8  21
## 36     9      36 14.3   72     8  22
## 37    NA     255 12.6   75     8  23
## 38    45     212  9.7   79     8  24
## 39    20     252 10.9   80     9   7
## 40    23     220 10.3   78     9   8
## 41    21     230 10.9   75     9   9
## 42    24     259  9.7   73     9  10
## 43    21     259 15.5   76     9  12
## 44    28     238  6.3   77     9  13
## 45     9      24 10.9   71     9  14
## 46    13     112 11.5   71     9  15
## 47    46     237  6.9   78     9  16
## 48    13      27 10.3   76     9  18
## 49    23      14  9.2   71     9  22
## 50    30     193  6.9   70     9  26
## 51    NA     145 13.2   77     9  27
## 52    14     191 14.3   75     9  28
## 53    18     131  8.0   76     9  29
```

```r
airquality %>% 
  filter(Temp > 75, Wind > 10) %>% 
  select(Ozone,Solar.R)
```

```
##    Ozone Solar.R
## 1     45     252
## 2     NA     264
## 3     71     291
## 4     39     323
## 5     NA     259
## 6     NA     332
## 7     NA     322
## 8     21     191
## 9     13     137
## 10    NA      98
## 11    NA      31
## 12    NA     101
## 13    40     314
## 14    27     175
## 15    NA     291
## 16     7      48
## 17    35     274
## 18    63     220
## 19    NA     295
## 20    52      82
## 21     9      24
## 22    89     229
## 23    NA     137
## 24    44     192
## 25    28     273
## 26    NA      64
## 27    22      71
## 28    31     244
## 29    44     190
## 30    21     259
## 31    32      92
## 32    20     252
## 33    23     220
## 34    44     236
## 35    21     259
## 36    13      27
## 37    36     139
## 38    NA     145
```

```r
babynames %>% 
  filter(name %in% c("Acura", "Lexus", "Yugo"))
```

```
## # A tibble: 57 x 5
##     year sex   name      n       prop
##    <dbl> <chr> <chr> <int>      <dbl>
##  1  1990 F     Lexus    36 0.0000175 
##  2  1990 M     Lexus    12 0.00000558
##  3  1991 F     Lexus   102 0.0000502 
##  4  1991 M     Lexus    16 0.00000755
##  5  1992 F     Lexus   193 0.0000963 
##  6  1992 M     Lexus    25 0.0000119 
##  7  1993 F     Lexus   285 0.000145  
##  8  1993 M     Lexus    30 0.0000145 
##  9  1994 F     Lexus   381 0.000195  
## 10  1994 F     Acura     6 0.00000308
## # … with 47 more rows
```

## Orden de acuerdo a variables

`arrange` se usa para ordenar los datos de acuerdo a una o mas variables, donde por defecto lo hace de manera ascendente, para ordenarlos de manera descendente se encierra la variable dentro de `desc(var)`. Si se ordena por una variable numerica se hara de menor a mayor o viceversa, si se ordena por una variable factor se hara de acuerdo al orden de los niveles del factor, y si se ordena por una variable de texto se hara por orden alfabetico.


```r
airquality %>% 
  arrange(Temp)
```

```
##     Ozone Solar.R Wind Temp Month Day
## 1      NA      NA 14.3   56     5   5
## 2       6      78 18.4   57     5  18
## 3      NA      66 16.6   57     5  25
## 4      NA      NA  8.0   57     5  27
## 5      18      65 13.2   58     5  15
## 6      NA     266 14.9   58     5  26
## 7      19      99 13.8   59     5   8
## 8       1       8  9.7   59     5  21
## 9       8      19 20.1   61     5   9
## 10      4      25  9.7   61     5  23
## 11     32      92 12.0   61     5  24
## 12     18     313 11.5   62     5   4
## 13     11      44  9.7   62     5  20
## 14     14      20 16.6   63     9  25
## 15     14     334 11.5   64     5  16
## 16     13     238 12.6   64     9  21
## 17     23     299  8.6   65     5   7
## 18     20      37  9.2   65     6  18
## 19     28      NA 14.9   66     5   6
## 20     11     290  9.2   66     5  13
## 21     34     307 12.0   66     5  17
## 22     41     190  7.4   67     5   1
## 23     23      13 12.0   67     5  28
## 24     NA     242 16.1   67     6   3
## 25     18     224 13.8   67     9  17
## 26     14     274 10.9   68     5  14
## 27     30     322 11.5   68     5  19
## 28     24     238 10.3   68     9  19
## 29     20     223 11.5   68     9  30
## 30     NA     194  8.6   69     5  10
## 31     16     256  9.7   69     5  12
## 32      7      49 10.3   69     9  24
## 33     30     193  6.9   70     9  26
## 34      9      24 10.9   71     9  14
## 35     13     112 11.5   71     9  15
## 36     23      14  9.2   71     9  22
## 37     36     118  8.0   72     5   2
## 38     37     284 20.7   72     6  17
## 39      9      36 14.3   72     8  22
## 40     11     320 16.6   73     5  22
## 41     12     120 11.5   73     6  19
## 42     NA      47 10.3   73     6  27
## 43     10     264 14.3   73     7  12
## 44     24     259  9.7   73     9  10
## 45     12     149 12.6   74     5   3
## 46      7      NA  6.9   74     5  11
## 47     NA     287  9.7   74     6   2
## 48     16       7  6.9   74     7  21
## 49     NA     135  8.0   75     6  25
## 50     NA     255 12.6   75     8  23
## 51     21     230 10.9   75     9   9
## 52     14     191 14.3   75     9  28
## 53     37     279  7.4   76     5  31
## 54     13     137 10.3   76     6  20
## 55     NA      59  1.7   76     6  22
## 56     NA      91  4.6   76     6  23
## 57     NA     250  6.3   76     6  24
## 58     23     115  7.4   76     8  18
## 59     21     259 15.5   76     9  12
## 60     13      27 10.3   76     9  18
## 61     18     131  8.0   76     9  29
## 62     21     191 14.9   77     6  16
## 63     NA     150  6.3   77     6  21
## 64     NA      31 14.9   77     6  29
## 65     22      71 10.3   77     8  16
## 66     21     259 15.5   77     8  21
## 67     28     238  6.3   77     9  13
## 68     NA     145 13.2   77     9  27
## 69     NA     286  8.6   78     6   1
## 70     NA     127  8.0   78     6  26
## 71     31     244 10.9   78     8  19
## 72     44     190 10.3   78     8  20
## 73     23     220 10.3   78     9   8
## 74     46     237  6.9   78     9  16
## 75    115     223  5.7   79     5  30
## 76     NA     264 14.3   79     6   6
## 77     NA     322 11.5   79     6  15
## 78     NA      64 11.5   79     8  15
## 79     59      51  6.3   79     8  17
## 80     45     212  9.7   79     8  24
## 81     NA     332 13.8   80     6  14
## 82     NA      98 11.5   80     6  28
## 83      7      48 14.3   80     7  15
## 84     65     157  9.7   80     8  14
## 85     20     252 10.9   80     9   7
## 86     45     252 14.9   81     5  29
## 87     32     236  9.2   81     7   3
## 88     27     175 14.9   81     7  13
## 89     48     260  6.9   81     7  16
## 90     NA     258  9.7   81     7  22
## 91     59     254  9.2   81     7  31
## 92     39      83  6.9   81     8   1
## 93      9      24 13.8   81     8   2
## 94    168     238  3.4   81     8  25
## 95     44     236 14.9   81     9  11
## 96     36     139 10.3   81     9  23
## 97     29     127  9.7   82     6   7
## 98     23     148  8.0   82     6  13
## 99     NA     139  8.6   82     7  11
## 100    35     274 10.3   82     7  17
## 101    NA     295 11.5   82     7  23
## 102    20      81  8.6   82     7  26
## 103    16      77  7.4   82     8   3
## 104    28     273 11.5   82     8  13
## 105    16     201  8.0   82     9  20
## 106    NA     138  8.0   83     6  30
## 107    64     175  4.6   83     7   5
## 108    40     314 10.9   83     7   6
## 109    64     253  7.4   83     7  30
## 110    NA     186  9.2   84     6   4
## 111   135     269  4.1   84     7   1
## 112    NA     101 10.9   84     7   4
## 113    61     285  6.3   84     7  18
## 114    32      92 15.5   84     9   6
## 115    NA     220  8.6   85     6   5
## 116    49     248  9.2   85     7   2
## 117    63     220 11.5   85     7  20
## 118   108     223  8.0   85     7  25
## 119    35      NA  7.4   85     8   5
## 120    80     294  8.6   86     7  24
## 121    52      82 12.0   86     7  27
## 122    50     275  7.4   86     7  29
## 123    78      NA  6.9   86     8   4
## 124    NA     137 11.5   86     8  11
## 125    44     192 11.5   86     8  12
## 126    73     215  8.0   86     8  26
## 127    NA     273  6.9   87     6   8
## 128    39     323 11.5   87     6  10
## 129    79     187  5.1   87     7  19
## 130    66      NA  4.6   87     8   6
## 131    47      95  7.4   87     9   5
## 132    77     276  5.1   88     7   7
## 133    82     213  7.4   88     7  28
## 134    NA     153  5.7   88     8  27
## 135    85     175  7.4   89     7  10
## 136   122     255  4.0   89     8   7
## 137    71     291 13.8   90     6   9
## 138    89     229 10.3   90     8   8
## 139   110     207  8.0   90     8   9
## 140    NA     291 14.9   91     7  14
## 141    96     167  6.9   91     9   1
## 142    NA     250  9.2   92     6  12
## 143    97     267  6.3   92     7   8
## 144    97     272  5.7   92     7   9
## 145    NA     222  8.6   92     8  10
## 146    78     197  5.1   92     9   2
## 147    NA     259 10.9   93     6  11
## 148    73     183  2.8   93     9   3
## 149    91     189  4.6   93     9   4
## 150   118     225  2.3   94     8  29
## 151    85     188  6.3   94     8  31
## 152    84     237  6.3   96     8  30
## 153    76     203  9.7   97     8  28
```

```r
airquality %>% 
  arrange(desc(Temp))
```

```
##     Ozone Solar.R Wind Temp Month Day
## 1      76     203  9.7   97     8  28
## 2      84     237  6.3   96     8  30
## 3     118     225  2.3   94     8  29
## 4      85     188  6.3   94     8  31
## 5      NA     259 10.9   93     6  11
## 6      73     183  2.8   93     9   3
## 7      91     189  4.6   93     9   4
## 8      NA     250  9.2   92     6  12
## 9      97     267  6.3   92     7   8
## 10     97     272  5.7   92     7   9
## 11     NA     222  8.6   92     8  10
## 12     78     197  5.1   92     9   2
## 13     NA     291 14.9   91     7  14
## 14     96     167  6.9   91     9   1
## 15     71     291 13.8   90     6   9
## 16     89     229 10.3   90     8   8
## 17    110     207  8.0   90     8   9
## 18     85     175  7.4   89     7  10
## 19    122     255  4.0   89     8   7
## 20     77     276  5.1   88     7   7
## 21     82     213  7.4   88     7  28
## 22     NA     153  5.7   88     8  27
## 23     NA     273  6.9   87     6   8
## 24     39     323 11.5   87     6  10
## 25     79     187  5.1   87     7  19
## 26     66      NA  4.6   87     8   6
## 27     47      95  7.4   87     9   5
## 28     80     294  8.6   86     7  24
## 29     52      82 12.0   86     7  27
## 30     50     275  7.4   86     7  29
## 31     78      NA  6.9   86     8   4
## 32     NA     137 11.5   86     8  11
## 33     44     192 11.5   86     8  12
## 34     73     215  8.0   86     8  26
## 35     NA     220  8.6   85     6   5
## 36     49     248  9.2   85     7   2
## 37     63     220 11.5   85     7  20
## 38    108     223  8.0   85     7  25
## 39     35      NA  7.4   85     8   5
## 40     NA     186  9.2   84     6   4
## 41    135     269  4.1   84     7   1
## 42     NA     101 10.9   84     7   4
## 43     61     285  6.3   84     7  18
## 44     32      92 15.5   84     9   6
## 45     NA     138  8.0   83     6  30
## 46     64     175  4.6   83     7   5
## 47     40     314 10.9   83     7   6
## 48     64     253  7.4   83     7  30
## 49     29     127  9.7   82     6   7
## 50     23     148  8.0   82     6  13
## 51     NA     139  8.6   82     7  11
## 52     35     274 10.3   82     7  17
## 53     NA     295 11.5   82     7  23
## 54     20      81  8.6   82     7  26
## 55     16      77  7.4   82     8   3
## 56     28     273 11.5   82     8  13
## 57     16     201  8.0   82     9  20
## 58     45     252 14.9   81     5  29
## 59     32     236  9.2   81     7   3
## 60     27     175 14.9   81     7  13
## 61     48     260  6.9   81     7  16
## 62     NA     258  9.7   81     7  22
## 63     59     254  9.2   81     7  31
## 64     39      83  6.9   81     8   1
## 65      9      24 13.8   81     8   2
## 66    168     238  3.4   81     8  25
## 67     44     236 14.9   81     9  11
## 68     36     139 10.3   81     9  23
## 69     NA     332 13.8   80     6  14
## 70     NA      98 11.5   80     6  28
## 71      7      48 14.3   80     7  15
## 72     65     157  9.7   80     8  14
## 73     20     252 10.9   80     9   7
## 74    115     223  5.7   79     5  30
## 75     NA     264 14.3   79     6   6
## 76     NA     322 11.5   79     6  15
## 77     NA      64 11.5   79     8  15
## 78     59      51  6.3   79     8  17
## 79     45     212  9.7   79     8  24
## 80     NA     286  8.6   78     6   1
## 81     NA     127  8.0   78     6  26
## 82     31     244 10.9   78     8  19
## 83     44     190 10.3   78     8  20
## 84     23     220 10.3   78     9   8
## 85     46     237  6.9   78     9  16
## 86     21     191 14.9   77     6  16
## 87     NA     150  6.3   77     6  21
## 88     NA      31 14.9   77     6  29
## 89     22      71 10.3   77     8  16
## 90     21     259 15.5   77     8  21
## 91     28     238  6.3   77     9  13
## 92     NA     145 13.2   77     9  27
## 93     37     279  7.4   76     5  31
## 94     13     137 10.3   76     6  20
## 95     NA      59  1.7   76     6  22
## 96     NA      91  4.6   76     6  23
## 97     NA     250  6.3   76     6  24
## 98     23     115  7.4   76     8  18
## 99     21     259 15.5   76     9  12
## 100    13      27 10.3   76     9  18
## 101    18     131  8.0   76     9  29
## 102    NA     135  8.0   75     6  25
## 103    NA     255 12.6   75     8  23
## 104    21     230 10.9   75     9   9
## 105    14     191 14.3   75     9  28
## 106    12     149 12.6   74     5   3
## 107     7      NA  6.9   74     5  11
## 108    NA     287  9.7   74     6   2
## 109    16       7  6.9   74     7  21
## 110    11     320 16.6   73     5  22
## 111    12     120 11.5   73     6  19
## 112    NA      47 10.3   73     6  27
## 113    10     264 14.3   73     7  12
## 114    24     259  9.7   73     9  10
## 115    36     118  8.0   72     5   2
## 116    37     284 20.7   72     6  17
## 117     9      36 14.3   72     8  22
## 118     9      24 10.9   71     9  14
## 119    13     112 11.5   71     9  15
## 120    23      14  9.2   71     9  22
## 121    30     193  6.9   70     9  26
## 122    NA     194  8.6   69     5  10
## 123    16     256  9.7   69     5  12
## 124     7      49 10.3   69     9  24
## 125    14     274 10.9   68     5  14
## 126    30     322 11.5   68     5  19
## 127    24     238 10.3   68     9  19
## 128    20     223 11.5   68     9  30
## 129    41     190  7.4   67     5   1
## 130    23      13 12.0   67     5  28
## 131    NA     242 16.1   67     6   3
## 132    18     224 13.8   67     9  17
## 133    28      NA 14.9   66     5   6
## 134    11     290  9.2   66     5  13
## 135    34     307 12.0   66     5  17
## 136    23     299  8.6   65     5   7
## 137    20      37  9.2   65     6  18
## 138    14     334 11.5   64     5  16
## 139    13     238 12.6   64     9  21
## 140    14      20 16.6   63     9  25
## 141    18     313 11.5   62     5   4
## 142    11      44  9.7   62     5  20
## 143     8      19 20.1   61     5   9
## 144     4      25  9.7   61     5  23
## 145    32      92 12.0   61     5  24
## 146    19      99 13.8   59     5   8
## 147     1       8  9.7   59     5  21
## 148    18      65 13.2   58     5  15
## 149    NA     266 14.9   58     5  26
## 150     6      78 18.4   57     5  18
## 151    NA      66 16.6   57     5  25
## 152    NA      NA  8.0   57     5  27
## 153    NA      NA 14.3   56     5   5
```

```r
gss_cat %>% 
  arrange(marital)
```

```
## # A tibble: 21,483 x 9
##     year marital    age race  rincome     partyid      relig   denom     tvhours
##    <int> <fct>    <int> <fct> <fct>       <fct>        <fct>   <fct>       <int>
##  1  2000 No answ…    28 Other $10000 - 1… Ind,near dem Buddhi… Not appl…       2
##  2  2006 No answ…    NA White Not applic… Strong repu… Protes… Other          NA
##  3  2006 No answ…    NA White No answer   Strong repu… None    Not appl…       2
##  4  2006 No answ…    63 White No answer   Strong demo… None    Not appl…      NA
##  5  2006 No answ…    40 Other $20000 - 2… Not str dem… Protes… No denom…      NA
##  6  2006 No answ…    45 White No answer   No answer    No ans… No answer      NA
##  7  2006 No answ…    NA White No answer   No answer    No ans… No answer      NA
##  8  2008 No answ…    62 White Not applic… Strong demo… Protes… Episcopal      NA
##  9  2008 No answ…    43 White No answer   Independent  Christ… No denom…       1
## 10  2008 No answ…    50 White No answer   Ind,near dem Protes… Other           4
## # … with 21,473 more rows
```

## Creacion de variables

Para crear o modificar variables se usa `mutate`. Algunas veces se requiere o desea categorizar una variable continua de acuerdo a ciertos criterios o puntos de quiebre; lo anterior puede realizarse por medio de lo que se conoce como *if statements*, donde una funcion que realiza la misma tarea pero de forma mas eficiente es `case_when`.

En el primer ejemplo se trabaja con la tabla del `titanic`, donde se tienen varias variables como texto ('Pclass', 'Survived', 'Sex') y se quieren convertir a factor, por lo que simplemente se re-definen estas variables. Este cambio se puede ver con `glimpse` para el antes y despues, donde el tipo de variable cambia.


```r
glimpse(titanic)
```

```
## Observations: 891
## Variables: 12
## $ PassengerId <int> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17…
## $ Survived    <int> 0, 1, 1, 1, 0, 0, 0, 0, 1, 1, 1, 1, 0, 0, 0, 1, 0, 1, 0, …
## $ Pclass      <int> 3, 1, 3, 1, 3, 3, 1, 3, 3, 2, 3, 1, 3, 3, 3, 2, 3, 2, 3, …
## $ Name        <chr> "Braund, Mr. Owen Harris", "Cumings, Mrs. John Bradley (F…
## $ Sex         <chr> "male", "female", "female", "female", "male", "male", "ma…
## $ Age         <dbl> 22, 38, 26, 35, 35, NA, 54, 2, 27, 14, 4, 58, 20, 39, 14,…
## $ SibSp       <int> 1, 1, 0, 1, 0, 0, 0, 3, 0, 1, 1, 0, 0, 1, 0, 0, 4, 0, 1, …
## $ Parch       <int> 0, 0, 0, 0, 0, 0, 0, 1, 2, 0, 1, 0, 0, 5, 0, 0, 1, 0, 0, …
## $ Ticket      <chr> "A/5 21171", "PC 17599", "STON/O2. 3101282", "113803", "3…
## $ Fare        <dbl> 7.2500, 71.2833, 7.9250, 53.1000, 8.0500, 8.4583, 51.8625…
## $ Cabin       <chr> "", "C85", "", "C123", "", "", "E46", "", "", "", "G6", "…
## $ Embarked    <chr> "S", "C", "S", "S", "S", "Q", "S", "S", "S", "C", "S", "S…
```

```r
titanic = titanic %>% 
  mutate(Pclass = as_factor(Pclass),
         Survived = as_factor(Survived),
         Sex = as_factor(Sex))
glimpse(titanic)
```

```
## Observations: 891
## Variables: 12
## $ PassengerId <int> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17…
## $ Survived    <fct> 0, 1, 1, 1, 0, 0, 0, 0, 1, 1, 1, 1, 0, 0, 0, 1, 0, 1, 0, …
## $ Pclass      <fct> 3, 1, 3, 1, 3, 3, 1, 3, 3, 2, 3, 1, 3, 3, 3, 2, 3, 2, 3, …
## $ Name        <chr> "Braund, Mr. Owen Harris", "Cumings, Mrs. John Bradley (F…
## $ Sex         <fct> male, female, female, female, male, male, male, male, fem…
## $ Age         <dbl> 22, 38, 26, 35, 35, NA, 54, 2, 27, 14, 4, 58, 20, 39, 14,…
## $ SibSp       <int> 1, 1, 0, 1, 0, 0, 0, 3, 0, 1, 1, 0, 0, 1, 0, 0, 4, 0, 1, …
## $ Parch       <int> 0, 0, 0, 0, 0, 0, 0, 1, 2, 0, 1, 0, 0, 5, 0, 0, 1, 0, 0, …
## $ Ticket      <chr> "A/5 21171", "PC 17599", "STON/O2. 3101282", "113803", "3…
## $ Fare        <dbl> 7.2500, 71.2833, 7.9250, 53.1000, 8.0500, 8.4583, 51.8625…
## $ Cabin       <chr> "", "C85", "", "C123", "", "", "E46", "", "", "", "G6", "…
## $ Embarked    <chr> "S", "C", "S", "S", "S", "Q", "S", "S", "S", "C", "S", "S…
```

Se pueden crear variables nuevas que dependen de otra en la tabla. En el ejemplo se calcula la altura en centimetros a partir de la altura en pulgadas (1 pulgada = 2.54 cm)


```r
dat1 %>% 
  mutate(Altura = Height*2.54)
```

```
## # A tibble: 654 x 6
##      Age LungCap Height Gender Smoke Altura
##    <int>   <dbl>  <dbl> <chr>  <chr>  <dbl>
##  1     9    3.12   57   female no      145.
##  2     8    3.17   67.5 female no      171.
##  3     7    3.16   54.5 female no      138.
##  4     9    2.67   53   male   no      135.
##  5     9    3.68   57   male   no      145.
##  6     8    5.01   61   female no      155.
##  7     6    3.76   58   female no      147.
##  8     6    2.24   56   female no      142.
##  9     8    3.96   58.5 female no      149.
## 10     9    3.83   60   female no      152.
## # … with 644 more rows
```

En el tercer ejemplo se re define la variable 'Month' pasandola a factor donde se le cambian las etiquetas a algo mas explicito. A su vez, se define una nueva variable condicionada en los valores de otra (sensacion dependiendo del valor de la temperatura). Aqui se ejemplifica `case_when`, donde la estructura es:


```r
case_when(condicion1 ~ resultado1,
          condicion2 ~ resultado2,
          T ~ resultado3)
```



```r
airq = airquality %>% 
  mutate(Month = factor(Month,
                        levels = 5:9,
                        labels = c("Mayo", "Junio", "Julio", 
                                   "Agosto", "Setiembre")),
         Sensation = case_when(Temp < 60 ~ 'Cold',
                               Temp < 70 ~ 'Cool',
                               Temp < 85 ~ 'Warm',
                               T ~ 'Hot') %>% 
           as.factor())
airq
```

```
##     Ozone Solar.R Wind Temp     Month Day Sensation
## 1      41     190  7.4   67      Mayo   1      Cool
## 2      36     118  8.0   72      Mayo   2      Warm
## 3      12     149 12.6   74      Mayo   3      Warm
## 4      18     313 11.5   62      Mayo   4      Cool
## 5      NA      NA 14.3   56      Mayo   5      Cold
## 6      28      NA 14.9   66      Mayo   6      Cool
## 7      23     299  8.6   65      Mayo   7      Cool
## 8      19      99 13.8   59      Mayo   8      Cold
## 9       8      19 20.1   61      Mayo   9      Cool
## 10     NA     194  8.6   69      Mayo  10      Cool
## 11      7      NA  6.9   74      Mayo  11      Warm
## 12     16     256  9.7   69      Mayo  12      Cool
## 13     11     290  9.2   66      Mayo  13      Cool
## 14     14     274 10.9   68      Mayo  14      Cool
## 15     18      65 13.2   58      Mayo  15      Cold
## 16     14     334 11.5   64      Mayo  16      Cool
## 17     34     307 12.0   66      Mayo  17      Cool
## 18      6      78 18.4   57      Mayo  18      Cold
## 19     30     322 11.5   68      Mayo  19      Cool
## 20     11      44  9.7   62      Mayo  20      Cool
## 21      1       8  9.7   59      Mayo  21      Cold
## 22     11     320 16.6   73      Mayo  22      Warm
## 23      4      25  9.7   61      Mayo  23      Cool
## 24     32      92 12.0   61      Mayo  24      Cool
## 25     NA      66 16.6   57      Mayo  25      Cold
## 26     NA     266 14.9   58      Mayo  26      Cold
## 27     NA      NA  8.0   57      Mayo  27      Cold
## 28     23      13 12.0   67      Mayo  28      Cool
## 29     45     252 14.9   81      Mayo  29      Warm
## 30    115     223  5.7   79      Mayo  30      Warm
## 31     37     279  7.4   76      Mayo  31      Warm
## 32     NA     286  8.6   78     Junio   1      Warm
## 33     NA     287  9.7   74     Junio   2      Warm
## 34     NA     242 16.1   67     Junio   3      Cool
## 35     NA     186  9.2   84     Junio   4      Warm
## 36     NA     220  8.6   85     Junio   5       Hot
## 37     NA     264 14.3   79     Junio   6      Warm
## 38     29     127  9.7   82     Junio   7      Warm
## 39     NA     273  6.9   87     Junio   8       Hot
## 40     71     291 13.8   90     Junio   9       Hot
## 41     39     323 11.5   87     Junio  10       Hot
## 42     NA     259 10.9   93     Junio  11       Hot
## 43     NA     250  9.2   92     Junio  12       Hot
## 44     23     148  8.0   82     Junio  13      Warm
## 45     NA     332 13.8   80     Junio  14      Warm
## 46     NA     322 11.5   79     Junio  15      Warm
## 47     21     191 14.9   77     Junio  16      Warm
## 48     37     284 20.7   72     Junio  17      Warm
## 49     20      37  9.2   65     Junio  18      Cool
## 50     12     120 11.5   73     Junio  19      Warm
## 51     13     137 10.3   76     Junio  20      Warm
## 52     NA     150  6.3   77     Junio  21      Warm
## 53     NA      59  1.7   76     Junio  22      Warm
## 54     NA      91  4.6   76     Junio  23      Warm
## 55     NA     250  6.3   76     Junio  24      Warm
## 56     NA     135  8.0   75     Junio  25      Warm
## 57     NA     127  8.0   78     Junio  26      Warm
## 58     NA      47 10.3   73     Junio  27      Warm
## 59     NA      98 11.5   80     Junio  28      Warm
## 60     NA      31 14.9   77     Junio  29      Warm
## 61     NA     138  8.0   83     Junio  30      Warm
## 62    135     269  4.1   84     Julio   1      Warm
## 63     49     248  9.2   85     Julio   2       Hot
## 64     32     236  9.2   81     Julio   3      Warm
## 65     NA     101 10.9   84     Julio   4      Warm
## 66     64     175  4.6   83     Julio   5      Warm
## 67     40     314 10.9   83     Julio   6      Warm
## 68     77     276  5.1   88     Julio   7       Hot
## 69     97     267  6.3   92     Julio   8       Hot
## 70     97     272  5.7   92     Julio   9       Hot
## 71     85     175  7.4   89     Julio  10       Hot
## 72     NA     139  8.6   82     Julio  11      Warm
## 73     10     264 14.3   73     Julio  12      Warm
## 74     27     175 14.9   81     Julio  13      Warm
## 75     NA     291 14.9   91     Julio  14       Hot
## 76      7      48 14.3   80     Julio  15      Warm
## 77     48     260  6.9   81     Julio  16      Warm
## 78     35     274 10.3   82     Julio  17      Warm
## 79     61     285  6.3   84     Julio  18      Warm
## 80     79     187  5.1   87     Julio  19       Hot
## 81     63     220 11.5   85     Julio  20       Hot
## 82     16       7  6.9   74     Julio  21      Warm
## 83     NA     258  9.7   81     Julio  22      Warm
## 84     NA     295 11.5   82     Julio  23      Warm
## 85     80     294  8.6   86     Julio  24       Hot
## 86    108     223  8.0   85     Julio  25       Hot
## 87     20      81  8.6   82     Julio  26      Warm
## 88     52      82 12.0   86     Julio  27       Hot
## 89     82     213  7.4   88     Julio  28       Hot
## 90     50     275  7.4   86     Julio  29       Hot
## 91     64     253  7.4   83     Julio  30      Warm
## 92     59     254  9.2   81     Julio  31      Warm
## 93     39      83  6.9   81    Agosto   1      Warm
## 94      9      24 13.8   81    Agosto   2      Warm
## 95     16      77  7.4   82    Agosto   3      Warm
## 96     78      NA  6.9   86    Agosto   4       Hot
## 97     35      NA  7.4   85    Agosto   5       Hot
## 98     66      NA  4.6   87    Agosto   6       Hot
## 99    122     255  4.0   89    Agosto   7       Hot
## 100    89     229 10.3   90    Agosto   8       Hot
## 101   110     207  8.0   90    Agosto   9       Hot
## 102    NA     222  8.6   92    Agosto  10       Hot
## 103    NA     137 11.5   86    Agosto  11       Hot
## 104    44     192 11.5   86    Agosto  12       Hot
## 105    28     273 11.5   82    Agosto  13      Warm
## 106    65     157  9.7   80    Agosto  14      Warm
## 107    NA      64 11.5   79    Agosto  15      Warm
## 108    22      71 10.3   77    Agosto  16      Warm
## 109    59      51  6.3   79    Agosto  17      Warm
## 110    23     115  7.4   76    Agosto  18      Warm
## 111    31     244 10.9   78    Agosto  19      Warm
## 112    44     190 10.3   78    Agosto  20      Warm
## 113    21     259 15.5   77    Agosto  21      Warm
## 114     9      36 14.3   72    Agosto  22      Warm
## 115    NA     255 12.6   75    Agosto  23      Warm
## 116    45     212  9.7   79    Agosto  24      Warm
## 117   168     238  3.4   81    Agosto  25      Warm
## 118    73     215  8.0   86    Agosto  26       Hot
## 119    NA     153  5.7   88    Agosto  27       Hot
## 120    76     203  9.7   97    Agosto  28       Hot
## 121   118     225  2.3   94    Agosto  29       Hot
## 122    84     237  6.3   96    Agosto  30       Hot
## 123    85     188  6.3   94    Agosto  31       Hot
## 124    96     167  6.9   91 Setiembre   1       Hot
## 125    78     197  5.1   92 Setiembre   2       Hot
## 126    73     183  2.8   93 Setiembre   3       Hot
## 127    91     189  4.6   93 Setiembre   4       Hot
## 128    47      95  7.4   87 Setiembre   5       Hot
## 129    32      92 15.5   84 Setiembre   6      Warm
## 130    20     252 10.9   80 Setiembre   7      Warm
## 131    23     220 10.3   78 Setiembre   8      Warm
## 132    21     230 10.9   75 Setiembre   9      Warm
## 133    24     259  9.7   73 Setiembre  10      Warm
## 134    44     236 14.9   81 Setiembre  11      Warm
## 135    21     259 15.5   76 Setiembre  12      Warm
## 136    28     238  6.3   77 Setiembre  13      Warm
## 137     9      24 10.9   71 Setiembre  14      Warm
## 138    13     112 11.5   71 Setiembre  15      Warm
## 139    46     237  6.9   78 Setiembre  16      Warm
## 140    18     224 13.8   67 Setiembre  17      Cool
## 141    13      27 10.3   76 Setiembre  18      Warm
## 142    24     238 10.3   68 Setiembre  19      Cool
## 143    16     201  8.0   82 Setiembre  20      Warm
## 144    13     238 12.6   64 Setiembre  21      Cool
## 145    23      14  9.2   71 Setiembre  22      Warm
## 146    36     139 10.3   81 Setiembre  23      Warm
## 147     7      49 10.3   69 Setiembre  24      Cool
## 148    14      20 16.6   63 Setiembre  25      Cool
## 149    30     193  6.9   70 Setiembre  26      Warm
## 150    NA     145 13.2   77 Setiembre  27      Warm
## 151    14     191 14.3   75 Setiembre  28      Warm
## 152    18     131  8.0   76 Setiembre  29      Warm
## 153    20     223 11.5   68 Setiembre  30      Cool
```

```r
airquality %>% 
  as_tibble()
```

```
## # A tibble: 153 x 6
##    Ozone Solar.R  Wind  Temp Month   Day
##    <int>   <int> <dbl> <int> <int> <int>
##  1    41     190   7.4    67     5     1
##  2    36     118   8      72     5     2
##  3    12     149  12.6    74     5     3
##  4    18     313  11.5    62     5     4
##  5    NA      NA  14.3    56     5     5
##  6    28      NA  14.9    66     5     6
##  7    23     299   8.6    65     5     7
##  8    19      99  13.8    59     5     8
##  9     8      19  20.1    61     5     9
## 10    NA     194   8.6    69     5    10
## # … with 143 more rows
```

## Conteo de variables cualitativas

Para contar casos de variables discretas de una manera mas expedita se puede usar `count`. Esta funcion realiza un agrupamiento (`group_by`) y resumen (`summarise`) a la vez.


```r
mpg %>% 
  count(manufacturer, year)
```

```
## # A tibble: 30 x 3
##    manufacturer  year     n
##    <chr>        <int> <int>
##  1 audi          1999     9
##  2 audi          2008     9
##  3 chevrolet     1999     7
##  4 chevrolet     2008    12
##  5 dodge         1999    16
##  6 dodge         2008    21
##  7 ford          1999    15
##  8 ford          2008    10
##  9 honda         1999     5
## 10 honda         2008     4
## # … with 20 more rows
```

## Tabla interactiva

Este es un ejemplo de como convertir una tabla estatica a interactiva. Se usa el paquete *DT* [@R-DT] y la funcion `datatable`, donde se pueden definir otra serie de argumentos. Tiene la ventaja de que para columnas numericas puedo filtrar por medio de sliders, y para columnas de facto puedo seleccionar los niveles.


```r
airq %>% 
  DT::datatable(filter = 'top', options = list(dom = 't'))
```

<!--html_preserve--><div id="htmlwidget-6a8bdb0ff9faaa84961c" style="width:100%;height:auto;" class="datatables html-widget"></div>
<script type="application/json" data-for="htmlwidget-6a8bdb0ff9faaa84961c">{"x":{"filter":"top","filterHTML":"<tr>\n  <td><\/td>\n  <td data-type=\"integer\" style=\"vertical-align: top;\">\n    <div class=\"form-group has-feedback\" style=\"margin-bottom: auto;\">\n      <input type=\"search\" placeholder=\"All\" class=\"form-control\" style=\"width: 100%;\"/>\n      <span class=\"glyphicon glyphicon-remove-circle form-control-feedback\"><\/span>\n    <\/div>\n    <div style=\"display: none; position: absolute; width: 200px;\">\n      <div data-min=\"1\" data-max=\"168\"><\/div>\n      <span style=\"float: left;\"><\/span>\n      <span style=\"float: right;\"><\/span>\n    <\/div>\n  <\/td>\n  <td data-type=\"integer\" style=\"vertical-align: top;\">\n    <div class=\"form-group has-feedback\" style=\"margin-bottom: auto;\">\n      <input type=\"search\" placeholder=\"All\" class=\"form-control\" style=\"width: 100%;\"/>\n      <span class=\"glyphicon glyphicon-remove-circle form-control-feedback\"><\/span>\n    <\/div>\n    <div style=\"display: none; position: absolute; width: 200px;\">\n      <div data-min=\"7\" data-max=\"334\"><\/div>\n      <span style=\"float: left;\"><\/span>\n      <span style=\"float: right;\"><\/span>\n    <\/div>\n  <\/td>\n  <td data-type=\"number\" style=\"vertical-align: top;\">\n    <div class=\"form-group has-feedback\" style=\"margin-bottom: auto;\">\n      <input type=\"search\" placeholder=\"All\" class=\"form-control\" style=\"width: 100%;\"/>\n      <span class=\"glyphicon glyphicon-remove-circle form-control-feedback\"><\/span>\n    <\/div>\n    <div style=\"display: none; position: absolute; width: 200px;\">\n      <div data-min=\"1.7\" data-max=\"20.7\" data-scale=\"1\"><\/div>\n      <span style=\"float: left;\"><\/span>\n      <span style=\"float: right;\"><\/span>\n    <\/div>\n  <\/td>\n  <td data-type=\"integer\" style=\"vertical-align: top;\">\n    <div class=\"form-group has-feedback\" style=\"margin-bottom: auto;\">\n      <input type=\"search\" placeholder=\"All\" class=\"form-control\" style=\"width: 100%;\"/>\n      <span class=\"glyphicon glyphicon-remove-circle form-control-feedback\"><\/span>\n    <\/div>\n    <div style=\"display: none; position: absolute; width: 200px;\">\n      <div data-min=\"56\" data-max=\"97\"><\/div>\n      <span style=\"float: left;\"><\/span>\n      <span style=\"float: right;\"><\/span>\n    <\/div>\n  <\/td>\n  <td data-type=\"factor\" style=\"vertical-align: top;\">\n    <div class=\"form-group has-feedback\" style=\"margin-bottom: auto;\">\n      <input type=\"search\" placeholder=\"All\" class=\"form-control\" style=\"width: 100%;\"/>\n      <span class=\"glyphicon glyphicon-remove-circle form-control-feedback\"><\/span>\n    <\/div>\n    <div style=\"width: 100%; display: none;\">\n      <select multiple=\"multiple\" style=\"width: 100%;\" data-options=\"[&quot;Mayo&quot;,&quot;Junio&quot;,&quot;Julio&quot;,&quot;Agosto&quot;,&quot;Setiembre&quot;]\"><\/select>\n    <\/div>\n  <\/td>\n  <td data-type=\"integer\" style=\"vertical-align: top;\">\n    <div class=\"form-group has-feedback\" style=\"margin-bottom: auto;\">\n      <input type=\"search\" placeholder=\"All\" class=\"form-control\" style=\"width: 100%;\"/>\n      <span class=\"glyphicon glyphicon-remove-circle form-control-feedback\"><\/span>\n    <\/div>\n    <div style=\"display: none; position: absolute; width: 200px;\">\n      <div data-min=\"1\" data-max=\"31\"><\/div>\n      <span style=\"float: left;\"><\/span>\n      <span style=\"float: right;\"><\/span>\n    <\/div>\n  <\/td>\n  <td data-type=\"factor\" style=\"vertical-align: top;\">\n    <div class=\"form-group has-feedback\" style=\"margin-bottom: auto;\">\n      <input type=\"search\" placeholder=\"All\" class=\"form-control\" style=\"width: 100%;\"/>\n      <span class=\"glyphicon glyphicon-remove-circle form-control-feedback\"><\/span>\n    <\/div>\n    <div style=\"width: 100%; display: none;\">\n      <select multiple=\"multiple\" style=\"width: 100%;\" data-options=\"[&quot;Cold&quot;,&quot;Cool&quot;,&quot;Hot&quot;,&quot;Warm&quot;]\"><\/select>\n    <\/div>\n  <\/td>\n<\/tr>","data":[["1","2","3","4","5","6","7","8","9","10","11","12","13","14","15","16","17","18","19","20","21","22","23","24","25","26","27","28","29","30","31","32","33","34","35","36","37","38","39","40","41","42","43","44","45","46","47","48","49","50","51","52","53","54","55","56","57","58","59","60","61","62","63","64","65","66","67","68","69","70","71","72","73","74","75","76","77","78","79","80","81","82","83","84","85","86","87","88","89","90","91","92","93","94","95","96","97","98","99","100","101","102","103","104","105","106","107","108","109","110","111","112","113","114","115","116","117","118","119","120","121","122","123","124","125","126","127","128","129","130","131","132","133","134","135","136","137","138","139","140","141","142","143","144","145","146","147","148","149","150","151","152","153"],[41,36,12,18,null,28,23,19,8,null,7,16,11,14,18,14,34,6,30,11,1,11,4,32,null,null,null,23,45,115,37,null,null,null,null,null,null,29,null,71,39,null,null,23,null,null,21,37,20,12,13,null,null,null,null,null,null,null,null,null,null,135,49,32,null,64,40,77,97,97,85,null,10,27,null,7,48,35,61,79,63,16,null,null,80,108,20,52,82,50,64,59,39,9,16,78,35,66,122,89,110,null,null,44,28,65,null,22,59,23,31,44,21,9,null,45,168,73,null,76,118,84,85,96,78,73,91,47,32,20,23,21,24,44,21,28,9,13,46,18,13,24,16,13,23,36,7,14,30,null,14,18,20],[190,118,149,313,null,null,299,99,19,194,null,256,290,274,65,334,307,78,322,44,8,320,25,92,66,266,null,13,252,223,279,286,287,242,186,220,264,127,273,291,323,259,250,148,332,322,191,284,37,120,137,150,59,91,250,135,127,47,98,31,138,269,248,236,101,175,314,276,267,272,175,139,264,175,291,48,260,274,285,187,220,7,258,295,294,223,81,82,213,275,253,254,83,24,77,null,null,null,255,229,207,222,137,192,273,157,64,71,51,115,244,190,259,36,255,212,238,215,153,203,225,237,188,167,197,183,189,95,92,252,220,230,259,236,259,238,24,112,237,224,27,238,201,238,14,139,49,20,193,145,191,131,223],[7.4,8,12.6,11.5,14.3,14.9,8.6,13.8,20.1,8.6,6.9,9.7,9.2,10.9,13.2,11.5,12,18.4,11.5,9.7,9.7,16.6,9.7,12,16.6,14.9,8,12,14.9,5.7,7.4,8.6,9.7,16.1,9.2,8.6,14.3,9.7,6.9,13.8,11.5,10.9,9.2,8,13.8,11.5,14.9,20.7,9.2,11.5,10.3,6.3,1.7,4.6,6.3,8,8,10.3,11.5,14.9,8,4.1,9.2,9.2,10.9,4.6,10.9,5.1,6.3,5.7,7.4,8.6,14.3,14.9,14.9,14.3,6.9,10.3,6.3,5.1,11.5,6.9,9.7,11.5,8.6,8,8.6,12,7.4,7.4,7.4,9.2,6.9,13.8,7.4,6.9,7.4,4.6,4,10.3,8,8.6,11.5,11.5,11.5,9.7,11.5,10.3,6.3,7.4,10.9,10.3,15.5,14.3,12.6,9.7,3.4,8,5.7,9.7,2.3,6.3,6.3,6.9,5.1,2.8,4.6,7.4,15.5,10.9,10.3,10.9,9.7,14.9,15.5,6.3,10.9,11.5,6.9,13.8,10.3,10.3,8,12.6,9.2,10.3,10.3,16.6,6.9,13.2,14.3,8,11.5],[67,72,74,62,56,66,65,59,61,69,74,69,66,68,58,64,66,57,68,62,59,73,61,61,57,58,57,67,81,79,76,78,74,67,84,85,79,82,87,90,87,93,92,82,80,79,77,72,65,73,76,77,76,76,76,75,78,73,80,77,83,84,85,81,84,83,83,88,92,92,89,82,73,81,91,80,81,82,84,87,85,74,81,82,86,85,82,86,88,86,83,81,81,81,82,86,85,87,89,90,90,92,86,86,82,80,79,77,79,76,78,78,77,72,75,79,81,86,88,97,94,96,94,91,92,93,93,87,84,80,78,75,73,81,76,77,71,71,78,67,76,68,82,64,71,81,69,63,70,77,75,76,68],["Mayo","Mayo","Mayo","Mayo","Mayo","Mayo","Mayo","Mayo","Mayo","Mayo","Mayo","Mayo","Mayo","Mayo","Mayo","Mayo","Mayo","Mayo","Mayo","Mayo","Mayo","Mayo","Mayo","Mayo","Mayo","Mayo","Mayo","Mayo","Mayo","Mayo","Mayo","Junio","Junio","Junio","Junio","Junio","Junio","Junio","Junio","Junio","Junio","Junio","Junio","Junio","Junio","Junio","Junio","Junio","Junio","Junio","Junio","Junio","Junio","Junio","Junio","Junio","Junio","Junio","Junio","Junio","Junio","Julio","Julio","Julio","Julio","Julio","Julio","Julio","Julio","Julio","Julio","Julio","Julio","Julio","Julio","Julio","Julio","Julio","Julio","Julio","Julio","Julio","Julio","Julio","Julio","Julio","Julio","Julio","Julio","Julio","Julio","Julio","Agosto","Agosto","Agosto","Agosto","Agosto","Agosto","Agosto","Agosto","Agosto","Agosto","Agosto","Agosto","Agosto","Agosto","Agosto","Agosto","Agosto","Agosto","Agosto","Agosto","Agosto","Agosto","Agosto","Agosto","Agosto","Agosto","Agosto","Agosto","Agosto","Agosto","Agosto","Setiembre","Setiembre","Setiembre","Setiembre","Setiembre","Setiembre","Setiembre","Setiembre","Setiembre","Setiembre","Setiembre","Setiembre","Setiembre","Setiembre","Setiembre","Setiembre","Setiembre","Setiembre","Setiembre","Setiembre","Setiembre","Setiembre","Setiembre","Setiembre","Setiembre","Setiembre","Setiembre","Setiembre","Setiembre","Setiembre"],[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30],["Cool","Warm","Warm","Cool","Cold","Cool","Cool","Cold","Cool","Cool","Warm","Cool","Cool","Cool","Cold","Cool","Cool","Cold","Cool","Cool","Cold","Warm","Cool","Cool","Cold","Cold","Cold","Cool","Warm","Warm","Warm","Warm","Warm","Cool","Warm","Hot","Warm","Warm","Hot","Hot","Hot","Hot","Hot","Warm","Warm","Warm","Warm","Warm","Cool","Warm","Warm","Warm","Warm","Warm","Warm","Warm","Warm","Warm","Warm","Warm","Warm","Warm","Hot","Warm","Warm","Warm","Warm","Hot","Hot","Hot","Hot","Warm","Warm","Warm","Hot","Warm","Warm","Warm","Warm","Hot","Hot","Warm","Warm","Warm","Hot","Hot","Warm","Hot","Hot","Hot","Warm","Warm","Warm","Warm","Warm","Hot","Hot","Hot","Hot","Hot","Hot","Hot","Hot","Hot","Warm","Warm","Warm","Warm","Warm","Warm","Warm","Warm","Warm","Warm","Warm","Warm","Warm","Hot","Hot","Hot","Hot","Hot","Hot","Hot","Hot","Hot","Hot","Hot","Warm","Warm","Warm","Warm","Warm","Warm","Warm","Warm","Warm","Warm","Warm","Cool","Warm","Cool","Warm","Cool","Warm","Warm","Cool","Cool","Warm","Warm","Warm","Warm","Cool"]],"container":"<table class=\"display\">\n  <thead>\n    <tr>\n      <th> <\/th>\n      <th>Ozone<\/th>\n      <th>Solar.R<\/th>\n      <th>Wind<\/th>\n      <th>Temp<\/th>\n      <th>Month<\/th>\n      <th>Day<\/th>\n      <th>Sensation<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"dom":"t","columnDefs":[{"className":"dt-right","targets":[1,2,3,4,6]},{"orderable":false,"targets":0}],"order":[],"autoWidth":false,"orderClasses":false,"orderCellsTop":true}},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

## Datos relacionales

En caso de tener datos de observaciones en diferentes tablas, estas se pueden unir para juntar los datos en una unica tabla (uniones de transformacion), o relacionar para filtrar los datos de una tabla con respecto a otra (uniones de filtro).

De manera general las uniones se van a realizar de acuerdo a las columnas que tengan el mismo nombre en ambas tablas. Si se desea especificar una columna en especifico se usa el argumento `by = 'col'`. Si el nombre difiere entre las tablas se define la union de acuerdo a `by = c('a' = 'b')`, donde `'a'` corresponde con el nombre de la columna en la primer tabla, y `'b'` corresponde con el nombre de la columna en la segunda tabla. Esto aplica para todas las funciones de union (`*_join`).

### Uniones de transformacion

Estas uniones agregan columnas de una tabla a otra. 

Un tipo de union es `left_join(x, y)`, donde se unen los datos de la tabla de la derecha (`y`) a la de la izquierda (`x`) de acuerdo a una columna en comun, y manteniendo todas las observaciones de `x`.


```r
flights %>% 
  left_join(airlines)
```

```
## Joining, by = "carrier"
```

```
## # A tibble: 336,776 x 20
##     year month   day dep_time sched_dep_time dep_delay arr_time sched_arr_time
##    <int> <int> <int>    <int>          <int>     <dbl>    <int>          <int>
##  1  2013     1     1      517            515         2      830            819
##  2  2013     1     1      533            529         4      850            830
##  3  2013     1     1      542            540         2      923            850
##  4  2013     1     1      544            545        -1     1004           1022
##  5  2013     1     1      554            600        -6      812            837
##  6  2013     1     1      554            558        -4      740            728
##  7  2013     1     1      555            600        -5      913            854
##  8  2013     1     1      557            600        -3      709            723
##  9  2013     1     1      557            600        -3      838            846
## 10  2013     1     1      558            600        -2      753            745
## # … with 336,766 more rows, and 12 more variables: arr_delay <dbl>,
## #   carrier <chr>, flight <int>, tailnum <chr>, origin <chr>, dest <chr>,
## #   air_time <dbl>, distance <dbl>, hour <dbl>, minute <dbl>, time_hour <dttm>,
## #   name <chr>
```

```r
flights %>% 
  left_join(airports, c("dest" = "faa"))
```

```
## # A tibble: 336,776 x 26
##     year month   day dep_time sched_dep_time dep_delay arr_time sched_arr_time
##    <int> <int> <int>    <int>          <int>     <dbl>    <int>          <int>
##  1  2013     1     1      517            515         2      830            819
##  2  2013     1     1      533            529         4      850            830
##  3  2013     1     1      542            540         2      923            850
##  4  2013     1     1      544            545        -1     1004           1022
##  5  2013     1     1      554            600        -6      812            837
##  6  2013     1     1      554            558        -4      740            728
##  7  2013     1     1      555            600        -5      913            854
##  8  2013     1     1      557            600        -3      709            723
##  9  2013     1     1      557            600        -3      838            846
## 10  2013     1     1      558            600        -2      753            745
## # … with 336,766 more rows, and 18 more variables: arr_delay <dbl>,
## #   carrier <chr>, flight <int>, tailnum <chr>, origin <chr>, dest <chr>,
## #   air_time <dbl>, distance <dbl>, hour <dbl>, minute <dbl>, time_hour <dttm>,
## #   name <chr>, lat <dbl>, lon <dbl>, alt <int>, tz <dbl>, dst <chr>,
## #   tzone <chr>
```

Otro tipo de union es `inner_join(x, y)`, donde se mantienen observaciones que se encuentran en ambas tablas.


```r
df1 <- tibble(x = c(1, 2), y = 2:1)
df2 <- tibble(x = c(1, 3), a = 10, b = "a")

df1 %>% 
  inner_join(df2)
```

```
## Joining, by = "x"
```

```
## # A tibble: 1 x 4
##       x     y     a b    
##   <dbl> <int> <dbl> <chr>
## 1     1     2    10 a
```

Otro tripo de union es `full_join(x, y)`, donde se mantienen todas las observaciones de ambas tablas.


```r
df1 %>% 
  full_join(df2)
```

```
## Joining, by = "x"
```

```
## # A tibble: 3 x 4
##       x     y     a b    
##   <dbl> <int> <dbl> <chr>
## 1     1     2    10 a    
## 2     2     1    NA <NA> 
## 3     3    NA    10 a
```

### Uniones de filtro

Se filtran las observaciones de una tabla de acuerdo a si coniciden o no con las de otra tabla.

Un tipo es `semi_join(x, y)`, donde se mantienen todas las observaciones de `x` que coinciden con observaciones en `y`, pero sin agregar columnas de `y`. El opuesto seria `anti_join(x, y)`, donde se eliminan todas las observaciones de `x` que coinciden con observaciones en `y`, pero sin agregar columnas de `y`.


```r
df1 <- tibble(x = c(1, 1, 3, 4), y = 1:4)
df2 <- tibble(x = c(1, 1, 2), z = c("a", "b", "a"))

df1 %>% 
  semi_join(df2)
```

```
## Joining, by = "x"
```

```
## # A tibble: 2 x 2
##       x     y
##   <dbl> <int>
## 1     1     1
## 2     1     2
```

```r
df1 %>% 
  anti_join(df2)
```

```
## Joining, by = "x"
```

```
## # A tibble: 2 x 2
##       x     y
##   <dbl> <int>
## 1     3     3
## 2     4     4
```


## Datos ordenados (Tidy data)

### Formatos largo y ancho

Los datos ordenados corresponden con cada variable en su columna, cada fila corresponde con una observacion, y en las celdas van los valores correspondientes. Esto corresponde con un formato largo (Figura \@ref(fig:tidy-data)).

(ref:tidy-data) Estructura e ideologia de datos ordenados [@grolemund2016].

<div class="figure">
<img src="imgs/tidy-data.png" alt="(ref:tidy-data)" width="960" />
<p class="caption">(\#fig:tidy-data)(ref:tidy-data)</p>
</div>

El ejemplo que se muestra acontinuacion no esta ordenado. La tabla tiene 3 variables pero no definidas correctamente. Una variable seria el pais, otra seria el anho (las columnas), y la tercera seria el numero de casos (las celdas). Esto se conoce como datos en formato ancho (En algunos casos puede ser necesario este formato, pero en la mayoria de ocasiones se prefiere el formato largo).


```r
casos <- tribble(
  ~pais, ~"2011", ~"2012", ~"2013",
   "FR",    7000,    6900,    7000,
   "DE",    5800,    6000,    6200,
   "US",   15000,   14000,   13000
)
```

Para pasar de un formato ancho a largo, se usa la funcion `pivot_longer(cols, names_to, values_to)`, donde `cols` son las columnas a agrupar en una sola, `names_to` es el nombre que se le va a dar a la columna que va a contener las columnas a agrupar, y `values_to` es el nombre que se le va a dar a la columna que va a contener los valores de las celdas y que corresponden con una variable.

En este caso se van a agrupar todas las columnas menos el pais, se le va a llamar 'anho' y lo que estaba en las celdas pasa a ser la columna 'casos'.


```r
casos_tidy = casos %>% 
  pivot_longer(cols = -pais, names_to = 'anho', values_to = 'casos')
casos_tidy
```

```
## # A tibble: 9 x 3
##   pais  anho  casos
##   <chr> <chr> <dbl>
## 1 FR    2011   7000
## 2 FR    2012   6900
## 3 FR    2013   7000
## 4 DE    2011   5800
## 5 DE    2012   6000
## 6 DE    2013   6200
## 7 US    2011  15000
## 8 US    2012  14000
## 9 US    2013  13000
```

De igual manera se puede volver al formato ancho con `pivot_wider(id_cols, names_from, values_from)`, donde `id_cols` es una columna que identifica a cada observacion, `names_from` es la columna a usar para nuevas columnas, y `values_from` es la columna donde estan los valores a poner en las celdas.


```r
casos_tidy %>% 
  pivot_wider(id_cols = pais, names_from = anho, values_from = casos)
```

```
## # A tibble: 3 x 4
##   pais  `2011` `2012` `2013`
##   <chr>  <dbl>  <dbl>  <dbl>
## 1 FR      7000   6900   7000
## 2 DE      5800   6000   6200
## 3 US     15000  14000  13000
```

### Separar y unir

Otro caso de datos no ordenados es cuando una columna contiene 2 o mas datos, por lo que es necesario separar cada dato en un su propia columna.

En el ejemplo la columna 'tasa' corresponde con 'casos' y 'poblacion', por lo que hay que separarla. La funcion `separate` tiene el argumento `into` que corresponde con un vector de texto donse se deben definir los nombres de las columnas resultantes.


```r
casos2 <- tribble(
          ~pais, ~anho,               ~tasa,
  "Afghanistan",  2001,      '745/19987071',
       "Brasil",  2001,   '37737/172006362',
        "China",  2001, '212258/1272915272'
)
```


```r
casos2 %>% 
  separate(tasa, into = c('casos', 'poblacion'))
```

```
## # A tibble: 3 x 4
##   pais         anho casos  poblacion 
##   <chr>       <dbl> <chr>  <chr>     
## 1 Afghanistan  2001 745    19987071  
## 2 Brasil       2001 37737  172006362 
## 3 China        2001 212258 1272915272
```

Por defecto `separate` va a separar la columna en cualquier caracter especial que encuentre. Si se quiere especificar se puede usar el argumento `sep`.


```r
casos2 %>% 
  separate(tasa, into = c('casos', 'poblacion'), sep = '/')
```

```
## # A tibble: 3 x 4
##   pais         anho casos  poblacion 
##   <chr>       <dbl> <chr>  <chr>     
## 1 Afghanistan  2001 745    19987071  
## 2 Brasil       2001 37737  172006362 
## 3 China        2001 212258 1272915272
```

El tipo de columna resultante de `separate` es de texto, pero en algunos casos ese no es el tipo deseado, por lo que se le puede pedir a la funcion que trate de adivinar y convertir las columnas al tipo correcto por medio del argumento `convert = TRUE`.


```r
casos2_sep = casos2 %>% 
  separate(tasa, into = c('casos', 'poblacion'), convert = T)
casos2_sep
```

```
## # A tibble: 3 x 4
##   pais         anho  casos  poblacion
##   <chr>       <dbl>  <int>      <int>
## 1 Afghanistan  2001    745   19987071
## 2 Brasil       2001  37737  172006362
## 3 China        2001 212258 1272915272
```

El unir columnas se hace por medio de `unite`, donde se le pasan, primero, el nombre de la nueva columna, y segundo los nombres de las columnas a unir, asi como el caracter a usar para separar los datos.


```r
casos2_sep %>% 
  unite(tasa, casos, poblacion, sep = '-')
```

```
## # A tibble: 3 x 3
##   pais         anho tasa             
##   <chr>       <dbl> <chr>            
## 1 Afghanistan  2001 745-19987071     
## 2 Brasil       2001 37737-172006362  
## 3 China        2001 212258-1272915272
```


## Datos anidados (Nesting) {#nest}

Esta es una de las ventajas de los tibbles, donde una columna puede ser una lista, y como una lista puede contener lo que sea, esto permite felxibilidad en el analisis y manipulacion de dtos, como se va a ver en el proximo capitulo.

Esto es muy usado junto con `group_by`, donde primero se agrupa la tabla y luego se crea una columna donde para cada grupo se va a tener su tabla unica (las observaciones que corresponden con ese grupo) y diferente al resto.


```r
iris %>% 
  group_by(Species) %>% 
  nest()
```

```
## # A tibble: 3 x 2
## # Groups:   Species [3]
##   Species    data             
##   <fct>      <list>           
## 1 setosa     <tibble [50 × 4]>
## 2 versicolor <tibble [50 × 4]>
## 3 virginica  <tibble [50 × 4]>
```

```r
airq %>% 
  group_by(Month) %>% 
  nest()
```

```
## # A tibble: 5 x 2
## # Groups:   Month [5]
##   Month     data             
##   <fct>     <list>           
## 1 Mayo      <tibble [31 × 6]>
## 2 Junio     <tibble [30 × 6]>
## 3 Julio     <tibble [31 × 6]>
## 4 Agosto    <tibble [31 × 6]>
## 5 Setiembre <tibble [30 × 6]>
```


