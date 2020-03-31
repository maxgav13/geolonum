# Iteracion con `purrr`

En este capitulo se muestra como iterar funciones sobre diferentes objetos (vectores, tablas, listas). La idea de las iteraciones es ser mas eficiente a la hora de realizar calculos repetitivos. Se va a introducir al paquete *purrr* [@R-purrr] que brinda funciones para realizar diferentes tareas que requieren iterar sobre 1 o mas objetos.

En este capitulo se van a utilizar los siguientes paquetes:


```r
library(gapminder)
library(fs)
library(rio)
library(tidymodels)
library(tidyverse)
```

Asi mismo se vuelven a importar y manipular los datos con que se venia trabajando:


```r
data("airquality")

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
```

## Iterando sobre un objeto

La funcion basica de `purrr` es `map(.x, .f, ...)`, donde `.x` es el objeto sobre el cual iterar (vector, tabla o lista), `.f` es la funcion o tarea a realizar durante la iteracion, y `...` son argumentos extra dependiendo de la funcion. Esta funcion (`map`) siempre va a resultar en una lista; existen variantes de esta que son especificas para cuando se conoce cual va a ser el tipo de dato de salida. Por ejemplo, `map_dbl` se usa cuando el resultado de la funcion es un numero con decimales.

En el siguiente bloque de codigo se generan dos listas ficticias, ambas de 7 elementos, donde la primera corresponde con notas de estudiantes en pruebas durante un semestre, y la segunda son puntos extra para cada estudiante. 


```r
set.seed(4101)
n = 8
minima = 60
maxima = 100
exams <- list(
  student1 = round(runif(n, minima, maxima)),
  student2 = round(runif(n, minima, maxima)),
  student3 = round(runif(n, minima, maxima)),
  student4 = round(runif(n, minima, maxima)),
  student5 = round(runif(n, minima, maxima)),
  student6 = round(runif(n, minima, maxima)),
  student7 = round(runif(n, minima, maxima))
)

extra_credit <- list(10, 5, 0, 15, 5, 0, 5)
```

Usando los datos generados anteriormente, se muestra la funcionabilidad de varias de las funciones `map_*`. Estas funciones se pueden usar con el pipe operator (`%>%`).

El primer ejemplo muestra como con `map` se obtiene una lista de la nota media de los examnes por estudiante. Se itera sobre la lista 'exams', y a cada elemento de la lista (en este caso vectores) se le calcula la media.


```r
map(exams, mean) # media
```

```
## $student1
## [1] 79.25
## 
## $student2
## [1] 75.875
## 
## $student3
## [1] 80.25
## 
## $student4
## [1] 85.125
## 
## $student5
## [1] 84.875
## 
## $student6
## [1] 79.625
## 
## $student7
## [1] 81.875
```

En el segundo ejemplo se utiliza el pipe (`%>%`) y una de las variantes de `map` (`map_dbl`), ya que lo que se va a calcular (nota maxima) se sabe es un numero con decimales.


```r
exams %>% map_dbl(max) # nota maxima
```

```
## student1 student2 student3 student4 student5 student6 student7 
##      100       96       91       99       95       93      100
```

En el tercer ejemplo se itera sobre una tabla, donde en este caso la iteracion es sobre las columnas. Recordemos que una tabla es una lista donde las columnas son los elementos de la lista. Lo que se quiere hacer es obtener el valor de la media para cada columna de la tabla 'airq'. Al hacer esto encontramos dos situaciones; la primera que dice que hay un argumento no numerico o logico (en este caso se refiere a las columnas 'Month' y 'Sensation' que son factor), por lo que al ser un factor no se le puede aplicar una funcion numerica; la segunda que hay valores 'NA' aun en columnas que son numericas ('Ozone', 'Solar.R'), esto porque en esas columnas hay NAs y por defecto la funcion `mean` no los remueve a la hora de hacer el calculo.


```r
airq %>% map_dbl(mean)
```

```
## Warning in mean.default(.x[[i]], ...): argument is not numeric or logical:
## returning NA

## Warning in mean.default(.x[[i]], ...): argument is not numeric or logical:
## returning NA
```

```
##     Ozone   Solar.R      Wind      Temp     Month       Day Sensation 
##        NA        NA  9.957516 77.882353        NA 15.803922        NA
```

Por lo anterior, hay dos soluciones dependiendo de lo que se quiera resolver. Si solo se quiere lidiar con los 'NA', se puede agregar el argumento `na.rm = T` de la funcion `mean`, pero las columnas de tipo factor van a seguir estando presentes y dar 'NA' como resultado.


```r
airq %>% map_dbl(mean, na.rm = T)
```

```
## Warning in mean.default(.x[[i]], ...): argument is not numeric or logical:
## returning NA

## Warning in mean.default(.x[[i]], ...): argument is not numeric or logical:
## returning NA
```

```
##      Ozone    Solar.R       Wind       Temp      Month        Day  Sensation 
##  42.129310 185.931507   9.957516  77.882353         NA  15.803922         NA
```

La solucion mas adecuada en este caso es primero seleccionar las columnas de tipo numerico (`select_if(is.numeric)`) y a estas aplicarle el calculo de la media removiendo los 'NA'. Esta ultima forma de aplicar la funcion es, a mi parecer, las mas practica y clara. Se tiene que escribir la funcion empezando con el simbolo `~`, esto le dice a *purrr* que lo que va a estar a al derecha va a ser una funcion, seguido de este simbolo se escribe la funcion de manera normal, con la excepcion de que en el lugar donde iria tipicamente el vector se pone `.x`, para decirle a *purrr* donde incrustar los elementos del objeto sobre el cual se esta iterando.


```r
airq %>% 
  select_if(is.numeric) %>% 
  map_dbl(~ mean(.x, na.rm = T))
```

```
##      Ozone    Solar.R       Wind       Temp        Day 
##  42.129310 185.931507   9.957516  77.882353  15.803922
```

## Iterando sobre dos objetos

Los ejemplos anteriores se estaba iterando unicamente sobre un objeto. Para iterar sobre dos objetos (que tienen que tener la misma cantidad de elementos), existen las funciones `map2_*`, que tienen al estructura `map2(.x, .y, .f, ...)`, donde `.x` es el primer objeto, `.y` es el segundo objeto, y `.f` es al funcion a utilizar sobre los dos objetos. Un ejemplo de esto es calcular la nota final de los estudiantes por medio de la media de los examenes y agregarle el credito extra.


```r
exams %>% 
  map2_dbl(extra_credit, ~ mean(.x) + .y)
```

```
## student1 student2 student3 student4 student5 student6 student7 
##   89.250   80.875   80.250  100.125   89.875   79.625   86.875
```

## Leyendo archivos y combinandolos

Un caso tipico donde se desea iterar es leer varios archivos de texto que tienen el mismo formato y como combinarlos en una sola tabla para posterior manipulacion. En este caso se usa la funcion `dir_ls` del paquete *fs* [@R-fs], donde se define la carpeta donde se encuentran los archivos (`path`) y con `glob` se define un patron en el nombre de los archivos (en este caso todos los archivos empiezan con 'datos_').


```r
archivos <- dir_ls(path = 'data', glob = "*datos_*") 

archivos
```

```
## data/datos_cuarto_grado.csv data/datos_quinto_grado.csv 
## data/datos_tercer_grado.csv
```

```r
file_info(archivos)
```

```
## # A tibble: 3 x 18
##   path       type   size permissions modification_time   user  group device_id
##   <fs::path> <fct> <fs:> <fs::perms> <dttm>              <chr> <chr>     <dbl>
## 1 data/dato… file  1.79K rw-r--r--   2019-07-01 16:52:58 maxi… staff  16777220
## 2 data/dato… file  1.84K rw-r--r--   2019-07-01 16:52:58 maxi… staff  16777220
## 3 data/dato… file  1.81K rw-r--r--   2019-07-01 16:52:58 maxi… staff  16777220
## # … with 10 more variables: hard_links <dbl>, special_device_id <dbl>,
## #   inode <dbl>, block_size <dbl>, blocks <dbl>, flags <int>, generation <dbl>,
## #   access_time <dttm>, change_time <dttm>, birth_time <dttm>
```

Una vez se tiene le objeto con los nombres de los archivos ('archivos'), se puede proceder a realizar la iteracion. Como estamos importando archivos de texto (.csv) usamos la funcion `import` del paquete *rio*. 

Primeramente podemos generar una lista donde iteramos sobre el objeto 'archivos' e importamos cada uno, para posteriormente "pegar" uno tras otro con la funcion `bind_rows` de *dplyr*.


```r
map(archivos, import) %>% 
  bind_rows()
```

```
##        fecha             nombre matematica ingles matricula
## 1   1/1/2015 Hernandez, Rodrigo         90     60       100
## 2   1/2/2015 Hernandez, Rodrigo         85     70       100
## 3   1/3/2015 Hernandez, Rodrigo         70     80       100
## 4   1/4/2015 Hernandez, Rodrigo         75     85       100
## 5   1/5/2015 Hernandez, Rodrigo         70     90       100
## 6   1/6/2015 Hernandez, Rodrigo         66     90       100
## 7   1/1/2015      Sanchez, Juan         60     80       102
## 8   1/2/2015      Sanchez, Juan         70     80       102
## 9   1/3/2015      Sanchez, Juan         80     90       102
## 10  1/4/2015      Sanchez, Juan         85     85       102
## 11  1/5/2015      Sanchez, Juan         60     90       102
## 12  1/6/2015      Sanchez, Juan         80     99       102
## 13  1/1/2015     Perez, Roberto         60     60       105
## 14  1/2/2015     Perez, Roberto         76     66       105
## 15  1/3/2015     Perez, Roberto         66     62       105
## 16  1/4/2015     Perez, Roberto         74     70       105
## 17  1/5/2015     Perez, Roberto         66     63       105
## 18  1/6/2015     Perez, Roberto         60     64       105
## 19  1/1/2015   Ramirez, Alberto         50     60        99
## 20  1/2/2015   Ramirez, Alberto         55     65        99
## 21  1/3/2015   Ramirez, Alberto         60     64        99
## 22  1/4/2015   Ramirez, Alberto         55     63        99
## 23  1/5/2015   Ramirez, Alberto         50     66        99
## 24  1/6/2015   Ramirez, Alberto         62     70        99
## 25  1/1/2015      Lopez, Ingrid         90     80       103
## 26  1/2/2015      Lopez, Ingrid         95     85       103
## 27  1/3/2015      Lopez, Ingrid         90     84       103
## 28  1/4/2015      Lopez, Ingrid         95     93       103
## 29  1/5/2015      Lopez, Ingrid         90     86       103
## 30  1/6/2015      Lopez, Ingrid         92     80       103
## 31  1/1/2015   Alvarez, Cecilia         92     71       109
## 32  1/2/2015   Alvarez, Cecilia         81     72       109
## 33  1/3/2015   Alvarez, Cecilia         82     73       109
## 34  1/4/2015   Alvarez, Cecilia         74     84       109
## 35  1/5/2015   Alvarez, Cecilia         86     73       109
## 36  1/6/2015   Alvarez, Cecilia         82     71       109
## 37  1/1/2015     Jimenez, Elena         92     91        98
## 38  1/2/2015     Jimenez, Elena         91     92        98
## 39  1/3/2015     Jimenez, Elena         92     93        98
## 40  1/4/2015     Jimenez, Elena         94     94        98
## 41  1/5/2015     Jimenez, Elena         96     93        98
## 42  1/6/2015     Jimenez, Elena         82     99        98
## 43  1/1/2015       Paz, Beatriz         74     81       110
## 44  1/2/2015       Paz, Beatriz         85     82       110
## 45  1/3/2015       Paz, Beatriz         77     83       110
## 46  1/4/2015       Paz, Beatriz         83     84       110
## 47  1/5/2015       Paz, Beatriz         72     83       110
## 48  1/6/2015       Paz, Beatriz         81     89       110
## 49  1/1/2015        Díaz, Bruno         72     68       120
## 50  1/2/2015        Díaz, Bruno         70     72       120
## 51  1/3/2015        Díaz, Bruno         82     73       120
## 52  1/4/2015        Díaz, Bruno         62     68       120
## 53  1/5/2015        Díaz, Bruno         79     69       120
## 54  1/6/2015        Díaz, Bruno         79     71       120
## 55  1/1/2015  Fernández, Gudiel         93     84       122
## 56  1/2/2015  Fernández, Gudiel         75     70       122
## 57  1/3/2015  Fernández, Gudiel         82     65       122
## 58  1/4/2015  Fernández, Gudiel         66     69       122
## 59  1/5/2015  Fernández, Gudiel         85     84       122
## 60  1/6/2015  Fernández, Gudiel         96     68       122
## 61  1/1/2015    Sosa, Guillermo         71     79       125
## 62  1/2/2015    Sosa, Guillermo         76     86       125
## 63  1/3/2015    Sosa, Guillermo         91     99       125
## 64  1/4/2015    Sosa, Guillermo         87     87       125
## 65  1/5/2015    Sosa, Guillermo         68     78       125
## 66  1/6/2015    Sosa, Guillermo         74     74       125
## 67  1/1/2015  Aguirre, Benjamin         74     78       119
## 68  1/2/2015  Aguirre, Benjamin         88     78       119
## 69  1/3/2015  Aguirre, Benjamin         81     70       119
## 70  1/4/2015  Aguirre, Benjamin         89     88       119
## 71  1/5/2015  Aguirre, Benjamin         82     76       119
## 72  1/6/2015  Aguirre, Benjamin         90     73       119
## 73  1/1/2015    Medina, Paulina         77     83       123
## 74  1/2/2015    Medina, Paulina         67     99       123
## 75  1/3/2015    Medina, Paulina         91     87       123
## 76  1/4/2015    Medina, Paulina         96     85       123
## 77  1/5/2015    Medina, Paulina         82     95       123
## 78  1/6/2015    Medina, Paulina         71     91       123
## 79  1/1/2015   Torres, Gabriela         80     94       129
## 80  1/2/2015   Torres, Gabriela         98     63       129
## 81  1/3/2015   Torres, Gabriela         74     78       129
## 82  1/4/2015   Torres, Gabriela         99     84       129
## 83  1/5/2015   Torres, Gabriela         88     97       129
## 84  1/6/2015   Torres, Gabriela         96    100       129
## 85  1/1/2015   Flores, Patricia         67     61       118
## 86  1/2/2015   Flores, Patricia         85     83       118
## 87  1/3/2015   Flores, Patricia        100     90       118
## 88  1/4/2015   Flores, Patricia         65     70       118
## 89  1/5/2015   Flores, Patricia         73     72       118
## 90  1/6/2015   Flores, Patricia         95     90       118
## 91  1/1/2015      Aragón, Maria         91     97       130
## 92  1/2/2015      Aragón, Maria         93     68       130
## 93  1/3/2015      Aragón, Maria         84     74       130
## 94  1/4/2015      Aragón, Maria         80     78       130
## 95  1/5/2015      Aragón, Maria         91     97       130
## 96  1/6/2015      Aragón, Maria         96     75       130
## 97  1/1/2015   Dominguez, Tomas         75     65       140
## 98  1/2/2015   Dominguez, Tomas         83     62       140
## 99  1/3/2015   Dominguez, Tomas         63     90       140
## 100 1/4/2015   Dominguez, Tomas         94     86       140
## 101 1/5/2015   Dominguez, Tomas         92     65       140
## 102 1/6/2015   Dominguez, Tomas         64     95       140
## 103 1/1/2015         Paz, Edwin         84     67       142
## 104 1/2/2015         Paz, Edwin         63     84       142
## 105 1/3/2015         Paz, Edwin         76     62       142
## 106 1/4/2015         Paz, Edwin         85     90       142
## 107 1/5/2015         Paz, Edwin         71     78       142
## 108 1/6/2015         Paz, Edwin         82     94       142
## 109 1/1/2015    Vasquez, Samuel         61    100       145
## 110 1/2/2015    Vasquez, Samuel        100     91       145
## 111 1/3/2015    Vasquez, Samuel         64     89       145
## 112 1/4/2015    Vasquez, Samuel         92     98       145
## 113 1/5/2015    Vasquez, Samuel         66     83       145
## 114 1/6/2015    Vasquez, Samuel         93     80       145
## 115 1/1/2015  Fuentes, Fernando         65     95       139
## 116 1/2/2015  Fuentes, Fernando         62     95       139
## 117 1/3/2015  Fuentes, Fernando         97     76       139
## 118 1/4/2015  Fuentes, Fernando         85     73       139
## 119 1/5/2015  Fuentes, Fernando         82     60       139
## 120 1/6/2015  Fuentes, Fernando         74     72       139
## 121 1/1/2015     Ayala, Antonio         93     74       143
## 122 1/2/2015     Ayala, Antonio         93     78       143
## 123 1/3/2015     Ayala, Antonio         83     88       143
## 124 1/4/2015     Ayala, Antonio         85     67       143
## 125 1/5/2015     Ayala, Antonio         94     88       143
## 126 1/6/2015     Ayala, Antonio         78     79       143
## 127 1/1/2015    Juarez, Roberto         63     63       149
## 128 1/2/2015    Juarez, Roberto         69     70       149
## 129 1/3/2015    Juarez, Roberto         81     72       149
## 130 1/4/2015    Juarez, Roberto         83     85       149
## 131 1/5/2015    Juarez, Roberto         82     80       149
## 132 1/6/2015    Juarez, Roberto         95     77       149
## 133 1/1/2015  Cifuentes, Melisa         93     64       138
## 134 1/2/2015  Cifuentes, Melisa         91     69       138
## 135 1/3/2015  Cifuentes, Melisa         60     65       138
## 136 1/4/2015  Cifuentes, Melisa         77     99       138
## 137 1/5/2015  Cifuentes, Melisa         64     85       138
## 138 1/6/2015  Cifuentes, Melisa         93     80       138
## 139 1/1/2015      Ventura, Juan         90     84       150
## 140 1/2/2015      Ventura, Juan         79     68       150
## 141 1/3/2015      Ventura, Juan         74     60       150
## 142 1/4/2015      Ventura, Juan         71     88       150
## 143 1/5/2015      Ventura, Juan         85     80       150
## 144 1/6/2015      Ventura, Juan         70     66       150
```

Con lo anterior logramos generar una tabla con todos los datos pero no sabemos cuales datos correponden con cual archivo (y consecuentemente con que nivel). Para remediar lo anterior la funcion `bind_rows` tiene un argumento `.id`, al cual se le pasa el nombre de la columna que se quiere agregar mostrando el nombre del archivo al cual pertence cada observacion.


```r
archivos %>%
  map_dfr(import, .id = "archivo")
```

```
##                         archivo    fecha             nombre matematica ingles
## 1   data/datos_cuarto_grado.csv 1/1/2015 Hernandez, Rodrigo         90     60
## 2   data/datos_cuarto_grado.csv 1/2/2015 Hernandez, Rodrigo         85     70
## 3   data/datos_cuarto_grado.csv 1/3/2015 Hernandez, Rodrigo         70     80
## 4   data/datos_cuarto_grado.csv 1/4/2015 Hernandez, Rodrigo         75     85
## 5   data/datos_cuarto_grado.csv 1/5/2015 Hernandez, Rodrigo         70     90
## 6   data/datos_cuarto_grado.csv 1/6/2015 Hernandez, Rodrigo         66     90
## 7   data/datos_cuarto_grado.csv 1/1/2015      Sanchez, Juan         60     80
## 8   data/datos_cuarto_grado.csv 1/2/2015      Sanchez, Juan         70     80
## 9   data/datos_cuarto_grado.csv 1/3/2015      Sanchez, Juan         80     90
## 10  data/datos_cuarto_grado.csv 1/4/2015      Sanchez, Juan         85     85
## 11  data/datos_cuarto_grado.csv 1/5/2015      Sanchez, Juan         60     90
## 12  data/datos_cuarto_grado.csv 1/6/2015      Sanchez, Juan         80     99
## 13  data/datos_cuarto_grado.csv 1/1/2015     Perez, Roberto         60     60
## 14  data/datos_cuarto_grado.csv 1/2/2015     Perez, Roberto         76     66
## 15  data/datos_cuarto_grado.csv 1/3/2015     Perez, Roberto         66     62
## 16  data/datos_cuarto_grado.csv 1/4/2015     Perez, Roberto         74     70
## 17  data/datos_cuarto_grado.csv 1/5/2015     Perez, Roberto         66     63
## 18  data/datos_cuarto_grado.csv 1/6/2015     Perez, Roberto         60     64
## 19  data/datos_cuarto_grado.csv 1/1/2015   Ramirez, Alberto         50     60
## 20  data/datos_cuarto_grado.csv 1/2/2015   Ramirez, Alberto         55     65
## 21  data/datos_cuarto_grado.csv 1/3/2015   Ramirez, Alberto         60     64
## 22  data/datos_cuarto_grado.csv 1/4/2015   Ramirez, Alberto         55     63
## 23  data/datos_cuarto_grado.csv 1/5/2015   Ramirez, Alberto         50     66
## 24  data/datos_cuarto_grado.csv 1/6/2015   Ramirez, Alberto         62     70
## 25  data/datos_cuarto_grado.csv 1/1/2015      Lopez, Ingrid         90     80
## 26  data/datos_cuarto_grado.csv 1/2/2015      Lopez, Ingrid         95     85
## 27  data/datos_cuarto_grado.csv 1/3/2015      Lopez, Ingrid         90     84
## 28  data/datos_cuarto_grado.csv 1/4/2015      Lopez, Ingrid         95     93
## 29  data/datos_cuarto_grado.csv 1/5/2015      Lopez, Ingrid         90     86
## 30  data/datos_cuarto_grado.csv 1/6/2015      Lopez, Ingrid         92     80
## 31  data/datos_cuarto_grado.csv 1/1/2015   Alvarez, Cecilia         92     71
## 32  data/datos_cuarto_grado.csv 1/2/2015   Alvarez, Cecilia         81     72
## 33  data/datos_cuarto_grado.csv 1/3/2015   Alvarez, Cecilia         82     73
## 34  data/datos_cuarto_grado.csv 1/4/2015   Alvarez, Cecilia         74     84
## 35  data/datos_cuarto_grado.csv 1/5/2015   Alvarez, Cecilia         86     73
## 36  data/datos_cuarto_grado.csv 1/6/2015   Alvarez, Cecilia         82     71
## 37  data/datos_cuarto_grado.csv 1/1/2015     Jimenez, Elena         92     91
## 38  data/datos_cuarto_grado.csv 1/2/2015     Jimenez, Elena         91     92
## 39  data/datos_cuarto_grado.csv 1/3/2015     Jimenez, Elena         92     93
## 40  data/datos_cuarto_grado.csv 1/4/2015     Jimenez, Elena         94     94
## 41  data/datos_cuarto_grado.csv 1/5/2015     Jimenez, Elena         96     93
## 42  data/datos_cuarto_grado.csv 1/6/2015     Jimenez, Elena         82     99
## 43  data/datos_cuarto_grado.csv 1/1/2015       Paz, Beatriz         74     81
## 44  data/datos_cuarto_grado.csv 1/2/2015       Paz, Beatriz         85     82
## 45  data/datos_cuarto_grado.csv 1/3/2015       Paz, Beatriz         77     83
## 46  data/datos_cuarto_grado.csv 1/4/2015       Paz, Beatriz         83     84
## 47  data/datos_cuarto_grado.csv 1/5/2015       Paz, Beatriz         72     83
## 48  data/datos_cuarto_grado.csv 1/6/2015       Paz, Beatriz         81     89
## 49  data/datos_quinto_grado.csv 1/1/2015        Díaz, Bruno         72     68
## 50  data/datos_quinto_grado.csv 1/2/2015        Díaz, Bruno         70     72
## 51  data/datos_quinto_grado.csv 1/3/2015        Díaz, Bruno         82     73
## 52  data/datos_quinto_grado.csv 1/4/2015        Díaz, Bruno         62     68
## 53  data/datos_quinto_grado.csv 1/5/2015        Díaz, Bruno         79     69
## 54  data/datos_quinto_grado.csv 1/6/2015        Díaz, Bruno         79     71
## 55  data/datos_quinto_grado.csv 1/1/2015  Fernández, Gudiel         93     84
## 56  data/datos_quinto_grado.csv 1/2/2015  Fernández, Gudiel         75     70
## 57  data/datos_quinto_grado.csv 1/3/2015  Fernández, Gudiel         82     65
## 58  data/datos_quinto_grado.csv 1/4/2015  Fernández, Gudiel         66     69
## 59  data/datos_quinto_grado.csv 1/5/2015  Fernández, Gudiel         85     84
## 60  data/datos_quinto_grado.csv 1/6/2015  Fernández, Gudiel         96     68
## 61  data/datos_quinto_grado.csv 1/1/2015    Sosa, Guillermo         71     79
## 62  data/datos_quinto_grado.csv 1/2/2015    Sosa, Guillermo         76     86
## 63  data/datos_quinto_grado.csv 1/3/2015    Sosa, Guillermo         91     99
## 64  data/datos_quinto_grado.csv 1/4/2015    Sosa, Guillermo         87     87
## 65  data/datos_quinto_grado.csv 1/5/2015    Sosa, Guillermo         68     78
## 66  data/datos_quinto_grado.csv 1/6/2015    Sosa, Guillermo         74     74
## 67  data/datos_quinto_grado.csv 1/1/2015  Aguirre, Benjamin         74     78
## 68  data/datos_quinto_grado.csv 1/2/2015  Aguirre, Benjamin         88     78
## 69  data/datos_quinto_grado.csv 1/3/2015  Aguirre, Benjamin         81     70
## 70  data/datos_quinto_grado.csv 1/4/2015  Aguirre, Benjamin         89     88
## 71  data/datos_quinto_grado.csv 1/5/2015  Aguirre, Benjamin         82     76
## 72  data/datos_quinto_grado.csv 1/6/2015  Aguirre, Benjamin         90     73
## 73  data/datos_quinto_grado.csv 1/1/2015    Medina, Paulina         77     83
## 74  data/datos_quinto_grado.csv 1/2/2015    Medina, Paulina         67     99
## 75  data/datos_quinto_grado.csv 1/3/2015    Medina, Paulina         91     87
## 76  data/datos_quinto_grado.csv 1/4/2015    Medina, Paulina         96     85
## 77  data/datos_quinto_grado.csv 1/5/2015    Medina, Paulina         82     95
## 78  data/datos_quinto_grado.csv 1/6/2015    Medina, Paulina         71     91
## 79  data/datos_quinto_grado.csv 1/1/2015   Torres, Gabriela         80     94
## 80  data/datos_quinto_grado.csv 1/2/2015   Torres, Gabriela         98     63
## 81  data/datos_quinto_grado.csv 1/3/2015   Torres, Gabriela         74     78
## 82  data/datos_quinto_grado.csv 1/4/2015   Torres, Gabriela         99     84
## 83  data/datos_quinto_grado.csv 1/5/2015   Torres, Gabriela         88     97
## 84  data/datos_quinto_grado.csv 1/6/2015   Torres, Gabriela         96    100
## 85  data/datos_quinto_grado.csv 1/1/2015   Flores, Patricia         67     61
## 86  data/datos_quinto_grado.csv 1/2/2015   Flores, Patricia         85     83
## 87  data/datos_quinto_grado.csv 1/3/2015   Flores, Patricia        100     90
## 88  data/datos_quinto_grado.csv 1/4/2015   Flores, Patricia         65     70
## 89  data/datos_quinto_grado.csv 1/5/2015   Flores, Patricia         73     72
## 90  data/datos_quinto_grado.csv 1/6/2015   Flores, Patricia         95     90
## 91  data/datos_quinto_grado.csv 1/1/2015      Aragón, Maria         91     97
## 92  data/datos_quinto_grado.csv 1/2/2015      Aragón, Maria         93     68
## 93  data/datos_quinto_grado.csv 1/3/2015      Aragón, Maria         84     74
## 94  data/datos_quinto_grado.csv 1/4/2015      Aragón, Maria         80     78
## 95  data/datos_quinto_grado.csv 1/5/2015      Aragón, Maria         91     97
## 96  data/datos_quinto_grado.csv 1/6/2015      Aragón, Maria         96     75
## 97  data/datos_tercer_grado.csv 1/1/2015   Dominguez, Tomas         75     65
## 98  data/datos_tercer_grado.csv 1/2/2015   Dominguez, Tomas         83     62
## 99  data/datos_tercer_grado.csv 1/3/2015   Dominguez, Tomas         63     90
## 100 data/datos_tercer_grado.csv 1/4/2015   Dominguez, Tomas         94     86
## 101 data/datos_tercer_grado.csv 1/5/2015   Dominguez, Tomas         92     65
## 102 data/datos_tercer_grado.csv 1/6/2015   Dominguez, Tomas         64     95
## 103 data/datos_tercer_grado.csv 1/1/2015         Paz, Edwin         84     67
## 104 data/datos_tercer_grado.csv 1/2/2015         Paz, Edwin         63     84
## 105 data/datos_tercer_grado.csv 1/3/2015         Paz, Edwin         76     62
## 106 data/datos_tercer_grado.csv 1/4/2015         Paz, Edwin         85     90
## 107 data/datos_tercer_grado.csv 1/5/2015         Paz, Edwin         71     78
## 108 data/datos_tercer_grado.csv 1/6/2015         Paz, Edwin         82     94
## 109 data/datos_tercer_grado.csv 1/1/2015    Vasquez, Samuel         61    100
## 110 data/datos_tercer_grado.csv 1/2/2015    Vasquez, Samuel        100     91
## 111 data/datos_tercer_grado.csv 1/3/2015    Vasquez, Samuel         64     89
## 112 data/datos_tercer_grado.csv 1/4/2015    Vasquez, Samuel         92     98
## 113 data/datos_tercer_grado.csv 1/5/2015    Vasquez, Samuel         66     83
## 114 data/datos_tercer_grado.csv 1/6/2015    Vasquez, Samuel         93     80
## 115 data/datos_tercer_grado.csv 1/1/2015  Fuentes, Fernando         65     95
## 116 data/datos_tercer_grado.csv 1/2/2015  Fuentes, Fernando         62     95
## 117 data/datos_tercer_grado.csv 1/3/2015  Fuentes, Fernando         97     76
## 118 data/datos_tercer_grado.csv 1/4/2015  Fuentes, Fernando         85     73
## 119 data/datos_tercer_grado.csv 1/5/2015  Fuentes, Fernando         82     60
## 120 data/datos_tercer_grado.csv 1/6/2015  Fuentes, Fernando         74     72
## 121 data/datos_tercer_grado.csv 1/1/2015     Ayala, Antonio         93     74
## 122 data/datos_tercer_grado.csv 1/2/2015     Ayala, Antonio         93     78
## 123 data/datos_tercer_grado.csv 1/3/2015     Ayala, Antonio         83     88
## 124 data/datos_tercer_grado.csv 1/4/2015     Ayala, Antonio         85     67
## 125 data/datos_tercer_grado.csv 1/5/2015     Ayala, Antonio         94     88
## 126 data/datos_tercer_grado.csv 1/6/2015     Ayala, Antonio         78     79
## 127 data/datos_tercer_grado.csv 1/1/2015    Juarez, Roberto         63     63
## 128 data/datos_tercer_grado.csv 1/2/2015    Juarez, Roberto         69     70
## 129 data/datos_tercer_grado.csv 1/3/2015    Juarez, Roberto         81     72
## 130 data/datos_tercer_grado.csv 1/4/2015    Juarez, Roberto         83     85
## 131 data/datos_tercer_grado.csv 1/5/2015    Juarez, Roberto         82     80
## 132 data/datos_tercer_grado.csv 1/6/2015    Juarez, Roberto         95     77
## 133 data/datos_tercer_grado.csv 1/1/2015  Cifuentes, Melisa         93     64
## 134 data/datos_tercer_grado.csv 1/2/2015  Cifuentes, Melisa         91     69
## 135 data/datos_tercer_grado.csv 1/3/2015  Cifuentes, Melisa         60     65
## 136 data/datos_tercer_grado.csv 1/4/2015  Cifuentes, Melisa         77     99
## 137 data/datos_tercer_grado.csv 1/5/2015  Cifuentes, Melisa         64     85
## 138 data/datos_tercer_grado.csv 1/6/2015  Cifuentes, Melisa         93     80
## 139 data/datos_tercer_grado.csv 1/1/2015      Ventura, Juan         90     84
## 140 data/datos_tercer_grado.csv 1/2/2015      Ventura, Juan         79     68
## 141 data/datos_tercer_grado.csv 1/3/2015      Ventura, Juan         74     60
## 142 data/datos_tercer_grado.csv 1/4/2015      Ventura, Juan         71     88
## 143 data/datos_tercer_grado.csv 1/5/2015      Ventura, Juan         85     80
## 144 data/datos_tercer_grado.csv 1/6/2015      Ventura, Juan         70     66
##     matricula
## 1         100
## 2         100
## 3         100
## 4         100
## 5         100
## 6         100
## 7         102
## 8         102
## 9         102
## 10        102
## 11        102
## 12        102
## 13        105
## 14        105
## 15        105
## 16        105
## 17        105
## 18        105
## 19         99
## 20         99
## 21         99
## 22         99
## 23         99
## 24         99
## 25        103
## 26        103
## 27        103
## 28        103
## 29        103
## 30        103
## 31        109
## 32        109
## 33        109
## 34        109
## 35        109
## 36        109
## 37         98
## 38         98
## 39         98
## 40         98
## 41         98
## 42         98
## 43        110
## 44        110
## 45        110
## 46        110
## 47        110
## 48        110
## 49        120
## 50        120
## 51        120
## 52        120
## 53        120
## 54        120
## 55        122
## 56        122
## 57        122
## 58        122
## 59        122
## 60        122
## 61        125
## 62        125
## 63        125
## 64        125
## 65        125
## 66        125
## 67        119
## 68        119
## 69        119
## 70        119
## 71        119
## 72        119
## 73        123
## 74        123
## 75        123
## 76        123
## 77        123
## 78        123
## 79        129
## 80        129
## 81        129
## 82        129
## 83        129
## 84        129
## 85        118
## 86        118
## 87        118
## 88        118
## 89        118
## 90        118
## 91        130
## 92        130
## 93        130
## 94        130
## 95        130
## 96        130
## 97        140
## 98        140
## 99        140
## 100       140
## 101       140
## 102       140
## 103       142
## 104       142
## 105       142
## 106       142
## 107       142
## 108       142
## 109       145
## 110       145
## 111       145
## 112       145
## 113       145
## 114       145
## 115       139
## 116       139
## 117       139
## 118       139
## 119       139
## 120       139
## 121       143
## 122       143
## 123       143
## 124       143
## 125       143
## 126       143
## 127       149
## 128       149
## 129       149
## 130       149
## 131       149
## 132       149
## 133       138
## 134       138
## 135       138
## 136       138
## 137       138
## 138       138
## 139       150
## 140       150
## 141       150
## 142       150
## 143       150
## 144       150
```

La siguiente situacion que podemos encontrar es que el nombre del archivo (o cualquier otra columna de la tabla) tiene mas informacion de la necesaria, por lo que hay que separar los contenidos de la columna. Para esto usamos `separate` de *tidyr* para separar la columna en varias. En el caso de la columna 'archivo' podemos esperar tres columnas si especificamos el separador (`sep = '_'`), pero hay columnas que no ofrecen ninguna informacion (de las 3, la 1 y la 3, la 2 es la que tiene el nombre del nivel); para descartar estas columnas a la hora de separarlas se puede incluir `NA` en la posicion de las columnas que se desea descartar.


```r
archivos %>%
  map_dfr(import, .id = "archivo") %>% 
  separate(archivo, into = letters[1:3], sep = '_')
```

```
##              a      b         c    fecha             nombre matematica ingles
## 1   data/datos cuarto grado.csv 1/1/2015 Hernandez, Rodrigo         90     60
## 2   data/datos cuarto grado.csv 1/2/2015 Hernandez, Rodrigo         85     70
## 3   data/datos cuarto grado.csv 1/3/2015 Hernandez, Rodrigo         70     80
## 4   data/datos cuarto grado.csv 1/4/2015 Hernandez, Rodrigo         75     85
## 5   data/datos cuarto grado.csv 1/5/2015 Hernandez, Rodrigo         70     90
## 6   data/datos cuarto grado.csv 1/6/2015 Hernandez, Rodrigo         66     90
## 7   data/datos cuarto grado.csv 1/1/2015      Sanchez, Juan         60     80
## 8   data/datos cuarto grado.csv 1/2/2015      Sanchez, Juan         70     80
## 9   data/datos cuarto grado.csv 1/3/2015      Sanchez, Juan         80     90
## 10  data/datos cuarto grado.csv 1/4/2015      Sanchez, Juan         85     85
## 11  data/datos cuarto grado.csv 1/5/2015      Sanchez, Juan         60     90
## 12  data/datos cuarto grado.csv 1/6/2015      Sanchez, Juan         80     99
## 13  data/datos cuarto grado.csv 1/1/2015     Perez, Roberto         60     60
## 14  data/datos cuarto grado.csv 1/2/2015     Perez, Roberto         76     66
## 15  data/datos cuarto grado.csv 1/3/2015     Perez, Roberto         66     62
## 16  data/datos cuarto grado.csv 1/4/2015     Perez, Roberto         74     70
## 17  data/datos cuarto grado.csv 1/5/2015     Perez, Roberto         66     63
## 18  data/datos cuarto grado.csv 1/6/2015     Perez, Roberto         60     64
## 19  data/datos cuarto grado.csv 1/1/2015   Ramirez, Alberto         50     60
## 20  data/datos cuarto grado.csv 1/2/2015   Ramirez, Alberto         55     65
## 21  data/datos cuarto grado.csv 1/3/2015   Ramirez, Alberto         60     64
## 22  data/datos cuarto grado.csv 1/4/2015   Ramirez, Alberto         55     63
## 23  data/datos cuarto grado.csv 1/5/2015   Ramirez, Alberto         50     66
## 24  data/datos cuarto grado.csv 1/6/2015   Ramirez, Alberto         62     70
## 25  data/datos cuarto grado.csv 1/1/2015      Lopez, Ingrid         90     80
## 26  data/datos cuarto grado.csv 1/2/2015      Lopez, Ingrid         95     85
## 27  data/datos cuarto grado.csv 1/3/2015      Lopez, Ingrid         90     84
## 28  data/datos cuarto grado.csv 1/4/2015      Lopez, Ingrid         95     93
## 29  data/datos cuarto grado.csv 1/5/2015      Lopez, Ingrid         90     86
## 30  data/datos cuarto grado.csv 1/6/2015      Lopez, Ingrid         92     80
## 31  data/datos cuarto grado.csv 1/1/2015   Alvarez, Cecilia         92     71
## 32  data/datos cuarto grado.csv 1/2/2015   Alvarez, Cecilia         81     72
## 33  data/datos cuarto grado.csv 1/3/2015   Alvarez, Cecilia         82     73
## 34  data/datos cuarto grado.csv 1/4/2015   Alvarez, Cecilia         74     84
## 35  data/datos cuarto grado.csv 1/5/2015   Alvarez, Cecilia         86     73
## 36  data/datos cuarto grado.csv 1/6/2015   Alvarez, Cecilia         82     71
## 37  data/datos cuarto grado.csv 1/1/2015     Jimenez, Elena         92     91
## 38  data/datos cuarto grado.csv 1/2/2015     Jimenez, Elena         91     92
## 39  data/datos cuarto grado.csv 1/3/2015     Jimenez, Elena         92     93
## 40  data/datos cuarto grado.csv 1/4/2015     Jimenez, Elena         94     94
## 41  data/datos cuarto grado.csv 1/5/2015     Jimenez, Elena         96     93
## 42  data/datos cuarto grado.csv 1/6/2015     Jimenez, Elena         82     99
## 43  data/datos cuarto grado.csv 1/1/2015       Paz, Beatriz         74     81
## 44  data/datos cuarto grado.csv 1/2/2015       Paz, Beatriz         85     82
## 45  data/datos cuarto grado.csv 1/3/2015       Paz, Beatriz         77     83
## 46  data/datos cuarto grado.csv 1/4/2015       Paz, Beatriz         83     84
## 47  data/datos cuarto grado.csv 1/5/2015       Paz, Beatriz         72     83
## 48  data/datos cuarto grado.csv 1/6/2015       Paz, Beatriz         81     89
## 49  data/datos quinto grado.csv 1/1/2015        Díaz, Bruno         72     68
## 50  data/datos quinto grado.csv 1/2/2015        Díaz, Bruno         70     72
## 51  data/datos quinto grado.csv 1/3/2015        Díaz, Bruno         82     73
## 52  data/datos quinto grado.csv 1/4/2015        Díaz, Bruno         62     68
## 53  data/datos quinto grado.csv 1/5/2015        Díaz, Bruno         79     69
## 54  data/datos quinto grado.csv 1/6/2015        Díaz, Bruno         79     71
## 55  data/datos quinto grado.csv 1/1/2015  Fernández, Gudiel         93     84
## 56  data/datos quinto grado.csv 1/2/2015  Fernández, Gudiel         75     70
## 57  data/datos quinto grado.csv 1/3/2015  Fernández, Gudiel         82     65
## 58  data/datos quinto grado.csv 1/4/2015  Fernández, Gudiel         66     69
## 59  data/datos quinto grado.csv 1/5/2015  Fernández, Gudiel         85     84
## 60  data/datos quinto grado.csv 1/6/2015  Fernández, Gudiel         96     68
## 61  data/datos quinto grado.csv 1/1/2015    Sosa, Guillermo         71     79
## 62  data/datos quinto grado.csv 1/2/2015    Sosa, Guillermo         76     86
## 63  data/datos quinto grado.csv 1/3/2015    Sosa, Guillermo         91     99
## 64  data/datos quinto grado.csv 1/4/2015    Sosa, Guillermo         87     87
## 65  data/datos quinto grado.csv 1/5/2015    Sosa, Guillermo         68     78
## 66  data/datos quinto grado.csv 1/6/2015    Sosa, Guillermo         74     74
## 67  data/datos quinto grado.csv 1/1/2015  Aguirre, Benjamin         74     78
## 68  data/datos quinto grado.csv 1/2/2015  Aguirre, Benjamin         88     78
## 69  data/datos quinto grado.csv 1/3/2015  Aguirre, Benjamin         81     70
## 70  data/datos quinto grado.csv 1/4/2015  Aguirre, Benjamin         89     88
## 71  data/datos quinto grado.csv 1/5/2015  Aguirre, Benjamin         82     76
## 72  data/datos quinto grado.csv 1/6/2015  Aguirre, Benjamin         90     73
## 73  data/datos quinto grado.csv 1/1/2015    Medina, Paulina         77     83
## 74  data/datos quinto grado.csv 1/2/2015    Medina, Paulina         67     99
## 75  data/datos quinto grado.csv 1/3/2015    Medina, Paulina         91     87
## 76  data/datos quinto grado.csv 1/4/2015    Medina, Paulina         96     85
## 77  data/datos quinto grado.csv 1/5/2015    Medina, Paulina         82     95
## 78  data/datos quinto grado.csv 1/6/2015    Medina, Paulina         71     91
## 79  data/datos quinto grado.csv 1/1/2015   Torres, Gabriela         80     94
## 80  data/datos quinto grado.csv 1/2/2015   Torres, Gabriela         98     63
## 81  data/datos quinto grado.csv 1/3/2015   Torres, Gabriela         74     78
## 82  data/datos quinto grado.csv 1/4/2015   Torres, Gabriela         99     84
## 83  data/datos quinto grado.csv 1/5/2015   Torres, Gabriela         88     97
## 84  data/datos quinto grado.csv 1/6/2015   Torres, Gabriela         96    100
## 85  data/datos quinto grado.csv 1/1/2015   Flores, Patricia         67     61
## 86  data/datos quinto grado.csv 1/2/2015   Flores, Patricia         85     83
## 87  data/datos quinto grado.csv 1/3/2015   Flores, Patricia        100     90
## 88  data/datos quinto grado.csv 1/4/2015   Flores, Patricia         65     70
## 89  data/datos quinto grado.csv 1/5/2015   Flores, Patricia         73     72
## 90  data/datos quinto grado.csv 1/6/2015   Flores, Patricia         95     90
## 91  data/datos quinto grado.csv 1/1/2015      Aragón, Maria         91     97
## 92  data/datos quinto grado.csv 1/2/2015      Aragón, Maria         93     68
## 93  data/datos quinto grado.csv 1/3/2015      Aragón, Maria         84     74
## 94  data/datos quinto grado.csv 1/4/2015      Aragón, Maria         80     78
## 95  data/datos quinto grado.csv 1/5/2015      Aragón, Maria         91     97
## 96  data/datos quinto grado.csv 1/6/2015      Aragón, Maria         96     75
## 97  data/datos tercer grado.csv 1/1/2015   Dominguez, Tomas         75     65
## 98  data/datos tercer grado.csv 1/2/2015   Dominguez, Tomas         83     62
## 99  data/datos tercer grado.csv 1/3/2015   Dominguez, Tomas         63     90
## 100 data/datos tercer grado.csv 1/4/2015   Dominguez, Tomas         94     86
## 101 data/datos tercer grado.csv 1/5/2015   Dominguez, Tomas         92     65
## 102 data/datos tercer grado.csv 1/6/2015   Dominguez, Tomas         64     95
## 103 data/datos tercer grado.csv 1/1/2015         Paz, Edwin         84     67
## 104 data/datos tercer grado.csv 1/2/2015         Paz, Edwin         63     84
## 105 data/datos tercer grado.csv 1/3/2015         Paz, Edwin         76     62
## 106 data/datos tercer grado.csv 1/4/2015         Paz, Edwin         85     90
## 107 data/datos tercer grado.csv 1/5/2015         Paz, Edwin         71     78
## 108 data/datos tercer grado.csv 1/6/2015         Paz, Edwin         82     94
## 109 data/datos tercer grado.csv 1/1/2015    Vasquez, Samuel         61    100
## 110 data/datos tercer grado.csv 1/2/2015    Vasquez, Samuel        100     91
## 111 data/datos tercer grado.csv 1/3/2015    Vasquez, Samuel         64     89
## 112 data/datos tercer grado.csv 1/4/2015    Vasquez, Samuel         92     98
## 113 data/datos tercer grado.csv 1/5/2015    Vasquez, Samuel         66     83
## 114 data/datos tercer grado.csv 1/6/2015    Vasquez, Samuel         93     80
## 115 data/datos tercer grado.csv 1/1/2015  Fuentes, Fernando         65     95
## 116 data/datos tercer grado.csv 1/2/2015  Fuentes, Fernando         62     95
## 117 data/datos tercer grado.csv 1/3/2015  Fuentes, Fernando         97     76
## 118 data/datos tercer grado.csv 1/4/2015  Fuentes, Fernando         85     73
## 119 data/datos tercer grado.csv 1/5/2015  Fuentes, Fernando         82     60
## 120 data/datos tercer grado.csv 1/6/2015  Fuentes, Fernando         74     72
## 121 data/datos tercer grado.csv 1/1/2015     Ayala, Antonio         93     74
## 122 data/datos tercer grado.csv 1/2/2015     Ayala, Antonio         93     78
## 123 data/datos tercer grado.csv 1/3/2015     Ayala, Antonio         83     88
## 124 data/datos tercer grado.csv 1/4/2015     Ayala, Antonio         85     67
## 125 data/datos tercer grado.csv 1/5/2015     Ayala, Antonio         94     88
## 126 data/datos tercer grado.csv 1/6/2015     Ayala, Antonio         78     79
## 127 data/datos tercer grado.csv 1/1/2015    Juarez, Roberto         63     63
## 128 data/datos tercer grado.csv 1/2/2015    Juarez, Roberto         69     70
## 129 data/datos tercer grado.csv 1/3/2015    Juarez, Roberto         81     72
## 130 data/datos tercer grado.csv 1/4/2015    Juarez, Roberto         83     85
## 131 data/datos tercer grado.csv 1/5/2015    Juarez, Roberto         82     80
## 132 data/datos tercer grado.csv 1/6/2015    Juarez, Roberto         95     77
## 133 data/datos tercer grado.csv 1/1/2015  Cifuentes, Melisa         93     64
## 134 data/datos tercer grado.csv 1/2/2015  Cifuentes, Melisa         91     69
## 135 data/datos tercer grado.csv 1/3/2015  Cifuentes, Melisa         60     65
## 136 data/datos tercer grado.csv 1/4/2015  Cifuentes, Melisa         77     99
## 137 data/datos tercer grado.csv 1/5/2015  Cifuentes, Melisa         64     85
## 138 data/datos tercer grado.csv 1/6/2015  Cifuentes, Melisa         93     80
## 139 data/datos tercer grado.csv 1/1/2015      Ventura, Juan         90     84
## 140 data/datos tercer grado.csv 1/2/2015      Ventura, Juan         79     68
## 141 data/datos tercer grado.csv 1/3/2015      Ventura, Juan         74     60
## 142 data/datos tercer grado.csv 1/4/2015      Ventura, Juan         71     88
## 143 data/datos tercer grado.csv 1/5/2015      Ventura, Juan         85     80
## 144 data/datos tercer grado.csv 1/6/2015      Ventura, Juan         70     66
##     matricula
## 1         100
## 2         100
## 3         100
## 4         100
## 5         100
## 6         100
## 7         102
## 8         102
## 9         102
## 10        102
## 11        102
## 12        102
## 13        105
## 14        105
## 15        105
## 16        105
## 17        105
## 18        105
## 19         99
## 20         99
## 21         99
## 22         99
## 23         99
## 24         99
## 25        103
## 26        103
## 27        103
## 28        103
## 29        103
## 30        103
## 31        109
## 32        109
## 33        109
## 34        109
## 35        109
## 36        109
## 37         98
## 38         98
## 39         98
## 40         98
## 41         98
## 42         98
## 43        110
## 44        110
## 45        110
## 46        110
## 47        110
## 48        110
## 49        120
## 50        120
## 51        120
## 52        120
## 53        120
## 54        120
## 55        122
## 56        122
## 57        122
## 58        122
## 59        122
## 60        122
## 61        125
## 62        125
## 63        125
## 64        125
## 65        125
## 66        125
## 67        119
## 68        119
## 69        119
## 70        119
## 71        119
## 72        119
## 73        123
## 74        123
## 75        123
## 76        123
## 77        123
## 78        123
## 79        129
## 80        129
## 81        129
## 82        129
## 83        129
## 84        129
## 85        118
## 86        118
## 87        118
## 88        118
## 89        118
## 90        118
## 91        130
## 92        130
## 93        130
## 94        130
## 95        130
## 96        130
## 97        140
## 98        140
## 99        140
## 100       140
## 101       140
## 102       140
## 103       142
## 104       142
## 105       142
## 106       142
## 107       142
## 108       142
## 109       145
## 110       145
## 111       145
## 112       145
## 113       145
## 114       145
## 115       139
## 116       139
## 117       139
## 118       139
## 119       139
## 120       139
## 121       143
## 122       143
## 123       143
## 124       143
## 125       143
## 126       143
## 127       149
## 128       149
## 129       149
## 130       149
## 131       149
## 132       149
## 133       138
## 134       138
## 135       138
## 136       138
## 137       138
## 138       138
## 139       150
## 140       150
## 141       150
## 142       150
## 143       150
## 144       150
```

```r
archivos %>%
  map_dfr(import, .id = "archivo") %>% 
  separate(archivo, into = c(NA, 'grado', NA), sep = '_')
```

```
##      grado    fecha             nombre matematica ingles matricula
## 1   cuarto 1/1/2015 Hernandez, Rodrigo         90     60       100
## 2   cuarto 1/2/2015 Hernandez, Rodrigo         85     70       100
## 3   cuarto 1/3/2015 Hernandez, Rodrigo         70     80       100
## 4   cuarto 1/4/2015 Hernandez, Rodrigo         75     85       100
## 5   cuarto 1/5/2015 Hernandez, Rodrigo         70     90       100
## 6   cuarto 1/6/2015 Hernandez, Rodrigo         66     90       100
## 7   cuarto 1/1/2015      Sanchez, Juan         60     80       102
## 8   cuarto 1/2/2015      Sanchez, Juan         70     80       102
## 9   cuarto 1/3/2015      Sanchez, Juan         80     90       102
## 10  cuarto 1/4/2015      Sanchez, Juan         85     85       102
## 11  cuarto 1/5/2015      Sanchez, Juan         60     90       102
## 12  cuarto 1/6/2015      Sanchez, Juan         80     99       102
## 13  cuarto 1/1/2015     Perez, Roberto         60     60       105
## 14  cuarto 1/2/2015     Perez, Roberto         76     66       105
## 15  cuarto 1/3/2015     Perez, Roberto         66     62       105
## 16  cuarto 1/4/2015     Perez, Roberto         74     70       105
## 17  cuarto 1/5/2015     Perez, Roberto         66     63       105
## 18  cuarto 1/6/2015     Perez, Roberto         60     64       105
## 19  cuarto 1/1/2015   Ramirez, Alberto         50     60        99
## 20  cuarto 1/2/2015   Ramirez, Alberto         55     65        99
## 21  cuarto 1/3/2015   Ramirez, Alberto         60     64        99
## 22  cuarto 1/4/2015   Ramirez, Alberto         55     63        99
## 23  cuarto 1/5/2015   Ramirez, Alberto         50     66        99
## 24  cuarto 1/6/2015   Ramirez, Alberto         62     70        99
## 25  cuarto 1/1/2015      Lopez, Ingrid         90     80       103
## 26  cuarto 1/2/2015      Lopez, Ingrid         95     85       103
## 27  cuarto 1/3/2015      Lopez, Ingrid         90     84       103
## 28  cuarto 1/4/2015      Lopez, Ingrid         95     93       103
## 29  cuarto 1/5/2015      Lopez, Ingrid         90     86       103
## 30  cuarto 1/6/2015      Lopez, Ingrid         92     80       103
## 31  cuarto 1/1/2015   Alvarez, Cecilia         92     71       109
## 32  cuarto 1/2/2015   Alvarez, Cecilia         81     72       109
## 33  cuarto 1/3/2015   Alvarez, Cecilia         82     73       109
## 34  cuarto 1/4/2015   Alvarez, Cecilia         74     84       109
## 35  cuarto 1/5/2015   Alvarez, Cecilia         86     73       109
## 36  cuarto 1/6/2015   Alvarez, Cecilia         82     71       109
## 37  cuarto 1/1/2015     Jimenez, Elena         92     91        98
## 38  cuarto 1/2/2015     Jimenez, Elena         91     92        98
## 39  cuarto 1/3/2015     Jimenez, Elena         92     93        98
## 40  cuarto 1/4/2015     Jimenez, Elena         94     94        98
## 41  cuarto 1/5/2015     Jimenez, Elena         96     93        98
## 42  cuarto 1/6/2015     Jimenez, Elena         82     99        98
## 43  cuarto 1/1/2015       Paz, Beatriz         74     81       110
## 44  cuarto 1/2/2015       Paz, Beatriz         85     82       110
## 45  cuarto 1/3/2015       Paz, Beatriz         77     83       110
## 46  cuarto 1/4/2015       Paz, Beatriz         83     84       110
## 47  cuarto 1/5/2015       Paz, Beatriz         72     83       110
## 48  cuarto 1/6/2015       Paz, Beatriz         81     89       110
## 49  quinto 1/1/2015        Díaz, Bruno         72     68       120
## 50  quinto 1/2/2015        Díaz, Bruno         70     72       120
## 51  quinto 1/3/2015        Díaz, Bruno         82     73       120
## 52  quinto 1/4/2015        Díaz, Bruno         62     68       120
## 53  quinto 1/5/2015        Díaz, Bruno         79     69       120
## 54  quinto 1/6/2015        Díaz, Bruno         79     71       120
## 55  quinto 1/1/2015  Fernández, Gudiel         93     84       122
## 56  quinto 1/2/2015  Fernández, Gudiel         75     70       122
## 57  quinto 1/3/2015  Fernández, Gudiel         82     65       122
## 58  quinto 1/4/2015  Fernández, Gudiel         66     69       122
## 59  quinto 1/5/2015  Fernández, Gudiel         85     84       122
## 60  quinto 1/6/2015  Fernández, Gudiel         96     68       122
## 61  quinto 1/1/2015    Sosa, Guillermo         71     79       125
## 62  quinto 1/2/2015    Sosa, Guillermo         76     86       125
## 63  quinto 1/3/2015    Sosa, Guillermo         91     99       125
## 64  quinto 1/4/2015    Sosa, Guillermo         87     87       125
## 65  quinto 1/5/2015    Sosa, Guillermo         68     78       125
## 66  quinto 1/6/2015    Sosa, Guillermo         74     74       125
## 67  quinto 1/1/2015  Aguirre, Benjamin         74     78       119
## 68  quinto 1/2/2015  Aguirre, Benjamin         88     78       119
## 69  quinto 1/3/2015  Aguirre, Benjamin         81     70       119
## 70  quinto 1/4/2015  Aguirre, Benjamin         89     88       119
## 71  quinto 1/5/2015  Aguirre, Benjamin         82     76       119
## 72  quinto 1/6/2015  Aguirre, Benjamin         90     73       119
## 73  quinto 1/1/2015    Medina, Paulina         77     83       123
## 74  quinto 1/2/2015    Medina, Paulina         67     99       123
## 75  quinto 1/3/2015    Medina, Paulina         91     87       123
## 76  quinto 1/4/2015    Medina, Paulina         96     85       123
## 77  quinto 1/5/2015    Medina, Paulina         82     95       123
## 78  quinto 1/6/2015    Medina, Paulina         71     91       123
## 79  quinto 1/1/2015   Torres, Gabriela         80     94       129
## 80  quinto 1/2/2015   Torres, Gabriela         98     63       129
## 81  quinto 1/3/2015   Torres, Gabriela         74     78       129
## 82  quinto 1/4/2015   Torres, Gabriela         99     84       129
## 83  quinto 1/5/2015   Torres, Gabriela         88     97       129
## 84  quinto 1/6/2015   Torres, Gabriela         96    100       129
## 85  quinto 1/1/2015   Flores, Patricia         67     61       118
## 86  quinto 1/2/2015   Flores, Patricia         85     83       118
## 87  quinto 1/3/2015   Flores, Patricia        100     90       118
## 88  quinto 1/4/2015   Flores, Patricia         65     70       118
## 89  quinto 1/5/2015   Flores, Patricia         73     72       118
## 90  quinto 1/6/2015   Flores, Patricia         95     90       118
## 91  quinto 1/1/2015      Aragón, Maria         91     97       130
## 92  quinto 1/2/2015      Aragón, Maria         93     68       130
## 93  quinto 1/3/2015      Aragón, Maria         84     74       130
## 94  quinto 1/4/2015      Aragón, Maria         80     78       130
## 95  quinto 1/5/2015      Aragón, Maria         91     97       130
## 96  quinto 1/6/2015      Aragón, Maria         96     75       130
## 97  tercer 1/1/2015   Dominguez, Tomas         75     65       140
## 98  tercer 1/2/2015   Dominguez, Tomas         83     62       140
## 99  tercer 1/3/2015   Dominguez, Tomas         63     90       140
## 100 tercer 1/4/2015   Dominguez, Tomas         94     86       140
## 101 tercer 1/5/2015   Dominguez, Tomas         92     65       140
## 102 tercer 1/6/2015   Dominguez, Tomas         64     95       140
## 103 tercer 1/1/2015         Paz, Edwin         84     67       142
## 104 tercer 1/2/2015         Paz, Edwin         63     84       142
## 105 tercer 1/3/2015         Paz, Edwin         76     62       142
## 106 tercer 1/4/2015         Paz, Edwin         85     90       142
## 107 tercer 1/5/2015         Paz, Edwin         71     78       142
## 108 tercer 1/6/2015         Paz, Edwin         82     94       142
## 109 tercer 1/1/2015    Vasquez, Samuel         61    100       145
## 110 tercer 1/2/2015    Vasquez, Samuel        100     91       145
## 111 tercer 1/3/2015    Vasquez, Samuel         64     89       145
## 112 tercer 1/4/2015    Vasquez, Samuel         92     98       145
## 113 tercer 1/5/2015    Vasquez, Samuel         66     83       145
## 114 tercer 1/6/2015    Vasquez, Samuel         93     80       145
## 115 tercer 1/1/2015  Fuentes, Fernando         65     95       139
## 116 tercer 1/2/2015  Fuentes, Fernando         62     95       139
## 117 tercer 1/3/2015  Fuentes, Fernando         97     76       139
## 118 tercer 1/4/2015  Fuentes, Fernando         85     73       139
## 119 tercer 1/5/2015  Fuentes, Fernando         82     60       139
## 120 tercer 1/6/2015  Fuentes, Fernando         74     72       139
## 121 tercer 1/1/2015     Ayala, Antonio         93     74       143
## 122 tercer 1/2/2015     Ayala, Antonio         93     78       143
## 123 tercer 1/3/2015     Ayala, Antonio         83     88       143
## 124 tercer 1/4/2015     Ayala, Antonio         85     67       143
## 125 tercer 1/5/2015     Ayala, Antonio         94     88       143
## 126 tercer 1/6/2015     Ayala, Antonio         78     79       143
## 127 tercer 1/1/2015    Juarez, Roberto         63     63       149
## 128 tercer 1/2/2015    Juarez, Roberto         69     70       149
## 129 tercer 1/3/2015    Juarez, Roberto         81     72       149
## 130 tercer 1/4/2015    Juarez, Roberto         83     85       149
## 131 tercer 1/5/2015    Juarez, Roberto         82     80       149
## 132 tercer 1/6/2015    Juarez, Roberto         95     77       149
## 133 tercer 1/1/2015  Cifuentes, Melisa         93     64       138
## 134 tercer 1/2/2015  Cifuentes, Melisa         91     69       138
## 135 tercer 1/3/2015  Cifuentes, Melisa         60     65       138
## 136 tercer 1/4/2015  Cifuentes, Melisa         77     99       138
## 137 tercer 1/5/2015  Cifuentes, Melisa         64     85       138
## 138 tercer 1/6/2015  Cifuentes, Melisa         93     80       138
## 139 tercer 1/1/2015      Ventura, Juan         90     84       150
## 140 tercer 1/2/2015      Ventura, Juan         79     68       150
## 141 tercer 1/3/2015      Ventura, Juan         74     60       150
## 142 tercer 1/4/2015      Ventura, Juan         71     88       150
## 143 tercer 1/5/2015      Ventura, Juan         85     80       150
## 144 tercer 1/6/2015      Ventura, Juan         70     66       150
```

Por ultimo, en este caso tambien se puede separar la columna 'nombre' en 'apellido' y 'nombre', usando los mismos principios anteriores.


```r
archivos %>%
  map_dfr(import, .id = "archivo") %>% 
  separate(archivo, into = c(NA, 'grado', NA), sep = '_') %>% 
  separate(nombre, into = c('apellido', 'nombre'), sep = ', ')
```

```
##      grado    fecha  apellido    nombre matematica ingles matricula
## 1   cuarto 1/1/2015 Hernandez   Rodrigo         90     60       100
## 2   cuarto 1/2/2015 Hernandez   Rodrigo         85     70       100
## 3   cuarto 1/3/2015 Hernandez   Rodrigo         70     80       100
## 4   cuarto 1/4/2015 Hernandez   Rodrigo         75     85       100
## 5   cuarto 1/5/2015 Hernandez   Rodrigo         70     90       100
## 6   cuarto 1/6/2015 Hernandez   Rodrigo         66     90       100
## 7   cuarto 1/1/2015   Sanchez      Juan         60     80       102
## 8   cuarto 1/2/2015   Sanchez      Juan         70     80       102
## 9   cuarto 1/3/2015   Sanchez      Juan         80     90       102
## 10  cuarto 1/4/2015   Sanchez      Juan         85     85       102
## 11  cuarto 1/5/2015   Sanchez      Juan         60     90       102
## 12  cuarto 1/6/2015   Sanchez      Juan         80     99       102
## 13  cuarto 1/1/2015     Perez   Roberto         60     60       105
## 14  cuarto 1/2/2015     Perez   Roberto         76     66       105
## 15  cuarto 1/3/2015     Perez   Roberto         66     62       105
## 16  cuarto 1/4/2015     Perez   Roberto         74     70       105
## 17  cuarto 1/5/2015     Perez   Roberto         66     63       105
## 18  cuarto 1/6/2015     Perez   Roberto         60     64       105
## 19  cuarto 1/1/2015   Ramirez   Alberto         50     60        99
## 20  cuarto 1/2/2015   Ramirez   Alberto         55     65        99
## 21  cuarto 1/3/2015   Ramirez   Alberto         60     64        99
## 22  cuarto 1/4/2015   Ramirez   Alberto         55     63        99
## 23  cuarto 1/5/2015   Ramirez   Alberto         50     66        99
## 24  cuarto 1/6/2015   Ramirez   Alberto         62     70        99
## 25  cuarto 1/1/2015     Lopez    Ingrid         90     80       103
## 26  cuarto 1/2/2015     Lopez    Ingrid         95     85       103
## 27  cuarto 1/3/2015     Lopez    Ingrid         90     84       103
## 28  cuarto 1/4/2015     Lopez    Ingrid         95     93       103
## 29  cuarto 1/5/2015     Lopez    Ingrid         90     86       103
## 30  cuarto 1/6/2015     Lopez    Ingrid         92     80       103
## 31  cuarto 1/1/2015   Alvarez   Cecilia         92     71       109
## 32  cuarto 1/2/2015   Alvarez   Cecilia         81     72       109
## 33  cuarto 1/3/2015   Alvarez   Cecilia         82     73       109
## 34  cuarto 1/4/2015   Alvarez   Cecilia         74     84       109
## 35  cuarto 1/5/2015   Alvarez   Cecilia         86     73       109
## 36  cuarto 1/6/2015   Alvarez   Cecilia         82     71       109
## 37  cuarto 1/1/2015   Jimenez     Elena         92     91        98
## 38  cuarto 1/2/2015   Jimenez     Elena         91     92        98
## 39  cuarto 1/3/2015   Jimenez     Elena         92     93        98
## 40  cuarto 1/4/2015   Jimenez     Elena         94     94        98
## 41  cuarto 1/5/2015   Jimenez     Elena         96     93        98
## 42  cuarto 1/6/2015   Jimenez     Elena         82     99        98
## 43  cuarto 1/1/2015       Paz   Beatriz         74     81       110
## 44  cuarto 1/2/2015       Paz   Beatriz         85     82       110
## 45  cuarto 1/3/2015       Paz   Beatriz         77     83       110
## 46  cuarto 1/4/2015       Paz   Beatriz         83     84       110
## 47  cuarto 1/5/2015       Paz   Beatriz         72     83       110
## 48  cuarto 1/6/2015       Paz   Beatriz         81     89       110
## 49  quinto 1/1/2015      Díaz     Bruno         72     68       120
## 50  quinto 1/2/2015      Díaz     Bruno         70     72       120
## 51  quinto 1/3/2015      Díaz     Bruno         82     73       120
## 52  quinto 1/4/2015      Díaz     Bruno         62     68       120
## 53  quinto 1/5/2015      Díaz     Bruno         79     69       120
## 54  quinto 1/6/2015      Díaz     Bruno         79     71       120
## 55  quinto 1/1/2015 Fernández    Gudiel         93     84       122
## 56  quinto 1/2/2015 Fernández    Gudiel         75     70       122
## 57  quinto 1/3/2015 Fernández    Gudiel         82     65       122
## 58  quinto 1/4/2015 Fernández    Gudiel         66     69       122
## 59  quinto 1/5/2015 Fernández    Gudiel         85     84       122
## 60  quinto 1/6/2015 Fernández    Gudiel         96     68       122
## 61  quinto 1/1/2015      Sosa Guillermo         71     79       125
## 62  quinto 1/2/2015      Sosa Guillermo         76     86       125
## 63  quinto 1/3/2015      Sosa Guillermo         91     99       125
## 64  quinto 1/4/2015      Sosa Guillermo         87     87       125
## 65  quinto 1/5/2015      Sosa Guillermo         68     78       125
## 66  quinto 1/6/2015      Sosa Guillermo         74     74       125
## 67  quinto 1/1/2015   Aguirre  Benjamin         74     78       119
## 68  quinto 1/2/2015   Aguirre  Benjamin         88     78       119
## 69  quinto 1/3/2015   Aguirre  Benjamin         81     70       119
## 70  quinto 1/4/2015   Aguirre  Benjamin         89     88       119
## 71  quinto 1/5/2015   Aguirre  Benjamin         82     76       119
## 72  quinto 1/6/2015   Aguirre  Benjamin         90     73       119
## 73  quinto 1/1/2015    Medina   Paulina         77     83       123
## 74  quinto 1/2/2015    Medina   Paulina         67     99       123
## 75  quinto 1/3/2015    Medina   Paulina         91     87       123
## 76  quinto 1/4/2015    Medina   Paulina         96     85       123
## 77  quinto 1/5/2015    Medina   Paulina         82     95       123
## 78  quinto 1/6/2015    Medina   Paulina         71     91       123
## 79  quinto 1/1/2015    Torres  Gabriela         80     94       129
## 80  quinto 1/2/2015    Torres  Gabriela         98     63       129
## 81  quinto 1/3/2015    Torres  Gabriela         74     78       129
## 82  quinto 1/4/2015    Torres  Gabriela         99     84       129
## 83  quinto 1/5/2015    Torres  Gabriela         88     97       129
## 84  quinto 1/6/2015    Torres  Gabriela         96    100       129
## 85  quinto 1/1/2015    Flores  Patricia         67     61       118
## 86  quinto 1/2/2015    Flores  Patricia         85     83       118
## 87  quinto 1/3/2015    Flores  Patricia        100     90       118
## 88  quinto 1/4/2015    Flores  Patricia         65     70       118
## 89  quinto 1/5/2015    Flores  Patricia         73     72       118
## 90  quinto 1/6/2015    Flores  Patricia         95     90       118
## 91  quinto 1/1/2015    Aragón     Maria         91     97       130
## 92  quinto 1/2/2015    Aragón     Maria         93     68       130
## 93  quinto 1/3/2015    Aragón     Maria         84     74       130
## 94  quinto 1/4/2015    Aragón     Maria         80     78       130
## 95  quinto 1/5/2015    Aragón     Maria         91     97       130
## 96  quinto 1/6/2015    Aragón     Maria         96     75       130
## 97  tercer 1/1/2015 Dominguez     Tomas         75     65       140
## 98  tercer 1/2/2015 Dominguez     Tomas         83     62       140
## 99  tercer 1/3/2015 Dominguez     Tomas         63     90       140
## 100 tercer 1/4/2015 Dominguez     Tomas         94     86       140
## 101 tercer 1/5/2015 Dominguez     Tomas         92     65       140
## 102 tercer 1/6/2015 Dominguez     Tomas         64     95       140
## 103 tercer 1/1/2015       Paz     Edwin         84     67       142
## 104 tercer 1/2/2015       Paz     Edwin         63     84       142
## 105 tercer 1/3/2015       Paz     Edwin         76     62       142
## 106 tercer 1/4/2015       Paz     Edwin         85     90       142
## 107 tercer 1/5/2015       Paz     Edwin         71     78       142
## 108 tercer 1/6/2015       Paz     Edwin         82     94       142
## 109 tercer 1/1/2015   Vasquez    Samuel         61    100       145
## 110 tercer 1/2/2015   Vasquez    Samuel        100     91       145
## 111 tercer 1/3/2015   Vasquez    Samuel         64     89       145
## 112 tercer 1/4/2015   Vasquez    Samuel         92     98       145
## 113 tercer 1/5/2015   Vasquez    Samuel         66     83       145
## 114 tercer 1/6/2015   Vasquez    Samuel         93     80       145
## 115 tercer 1/1/2015   Fuentes  Fernando         65     95       139
## 116 tercer 1/2/2015   Fuentes  Fernando         62     95       139
## 117 tercer 1/3/2015   Fuentes  Fernando         97     76       139
## 118 tercer 1/4/2015   Fuentes  Fernando         85     73       139
## 119 tercer 1/5/2015   Fuentes  Fernando         82     60       139
## 120 tercer 1/6/2015   Fuentes  Fernando         74     72       139
## 121 tercer 1/1/2015     Ayala   Antonio         93     74       143
## 122 tercer 1/2/2015     Ayala   Antonio         93     78       143
## 123 tercer 1/3/2015     Ayala   Antonio         83     88       143
## 124 tercer 1/4/2015     Ayala   Antonio         85     67       143
## 125 tercer 1/5/2015     Ayala   Antonio         94     88       143
## 126 tercer 1/6/2015     Ayala   Antonio         78     79       143
## 127 tercer 1/1/2015    Juarez   Roberto         63     63       149
## 128 tercer 1/2/2015    Juarez   Roberto         69     70       149
## 129 tercer 1/3/2015    Juarez   Roberto         81     72       149
## 130 tercer 1/4/2015    Juarez   Roberto         83     85       149
## 131 tercer 1/5/2015    Juarez   Roberto         82     80       149
## 132 tercer 1/6/2015    Juarez   Roberto         95     77       149
## 133 tercer 1/1/2015 Cifuentes    Melisa         93     64       138
## 134 tercer 1/2/2015 Cifuentes    Melisa         91     69       138
## 135 tercer 1/3/2015 Cifuentes    Melisa         60     65       138
## 136 tercer 1/4/2015 Cifuentes    Melisa         77     99       138
## 137 tercer 1/5/2015 Cifuentes    Melisa         64     85       138
## 138 tercer 1/6/2015 Cifuentes    Melisa         93     80       138
## 139 tercer 1/1/2015   Ventura      Juan         90     84       150
## 140 tercer 1/2/2015   Ventura      Juan         79     68       150
## 141 tercer 1/3/2015   Ventura      Juan         74     60       150
## 142 tercer 1/4/2015   Ventura      Juan         71     88       150
## 143 tercer 1/5/2015   Ventura      Juan         85     80       150
## 144 tercer 1/6/2015   Ventura      Juan         70     66       150
```

## Datos anidados, caso 1

Como se habia mencionado en la Seccion \@ref(nest) del Capitulo [Funcionamiento avanzado de R], una de las ventajas de los tibbles es que permiten tener columnas tipo lista, las cuales son muy utilies para iterar y realizar calculos de manera expedita.

En este caso 1 se trabaja con los datos de 'airq', que era la tabla modificada de 'airquality'. Un caso tipico de datos anidados es el agrupar la tabla de acuerdo a una variable categorica y aplicar la funcion `nest` de *tidyr*. Esto genera una columna 'data', del tipo lista, donde se almacena una tabla para cada nivel de la variable agrupadora.


```r
airq_nest = airq %>% 
  group_by(Month) %>% 
  nest()
```

El poder de los datos anidados es la combinacion de `mutate` (*dplyr*) para generar nuevas columnas, y de las funciones `map` (*purrr*) para iterar sobre una columan tipo lista. De forma general esta combinacion se plasma de la siguiente forma: `mutate(nueva_columna = map(columna_lista, ~ .f(.x)))`, donde 'nueva_columna' es el nombre de la columna a crear, 'columna_lista' es el nombre de la columna tipo lista sobre la cual se va a iterar, y `~ .f(.x)` es la funcion o secuencia de funciones a realizar sobre cada elemento (`.x`) de la 'columna_lista'.

Aplicano lo mencionado anteriormente sobre la tabla anidada 'airq_nest' se tienen los siguientes pasos, en diferentes `mutate`:

* `mod = map(data, ~lm(Wind ~ Temp, data = .x))`: Crea una nueva columna 'mod', que va a ser el resultado de un modelo lineal para cada mes (iterando sobre 'data'), en funcion del viento ('Wind') y la temperatura ('Temp'). La funcion para modelos lineales es `lm` y el primer argumento es la `formula` que lleva la estructura `y ~ x`, el argumento `data` se pone de forma explicita y aqui es donde se le indica los elementos sobre los cuales iterar (`.x`). El resultado es una lista, de ahi que se usara `map` y no una de sus versiones.
* `slope = map_dbl(mod, ~tidy(.) %>% filter(term == 'Temp') %>% pull(estimate))`: Crea una nueva columna 'slope', que va a almacenar la pendiente del modelo lineal ('mod') anteriormente calculado, como se sabe que es un numero se usa `map_dbl`.
* `r2 = map_dbl(mod, ~glance(.) %>% pull(r.squared))`: Crea una nueva columna 'r2', donde se va a almacenar el valor del coeficiente de determinacion ($R^2$), como se sabe que es un numero se usa `map_dbl`.
* `plot = map2(data,Month, ~ggplot(.x, aes(Temp, Wind)) + geom_point() + geom_smooth(method = 'lm') + labs(title = .y) + theme_bw(base_size = 12))`: Crea una nueva columna, donde se va a almacenar el grafico de dispersion para cada mes, y se le agrega un titulo para saber a que mes corresponde. En este caso se esta iterando sobre dos objetos, por lo que se usa `map2`: la columna tipo lista donde estan los datos a graficar ('data'), y la columna tipo factor (vector) donde esta la variable agrupadora ('Month') para poder poner el titulo correspondiente.


```r
airq_nest = airq_nest %>% 
  mutate(mod = map(data, ~lm(Wind ~ Temp, data = .x))) %>% 
  mutate(slope = map_dbl(mod, ~tidy(.) %>% 
                           filter(term == 'Temp') %>% 
                           pull(estimate)),
         r2 = map_dbl(mod, ~glance(.) %>% pull(r.squared)),
         plot = map2(data,Month, ~ggplot(.x, aes(Temp, Wind)) + 
                       geom_point() + 
                       geom_smooth(method = 'lm') + 
                       labs(title = .y) + 
                       theme_bw(base_size = 12)))
airq_nest
```

```
## # A tibble: 5 x 6
## # Groups:   Month [5]
##   Month     data              mod      slope     r2 plot  
##   <fct>     <list>            <list>   <dbl>  <dbl> <list>
## 1 Mayo      <tibble [31 × 6]> <lm>   -0.192  0.139  <gg>  
## 2 Junio     <tibble [30 × 6]> <lm>   -0.0691 0.0146 <gg>  
## 3 Julio     <tibble [31 × 6]> <lm>   -0.215  0.0932 <gg>  
## 4 Agosto    <tibble [31 × 6]> <lm>   -0.249  0.258  <gg>  
## 5 Setiembre <tibble [30 × 6]> <lm>   -0.236  0.325  <gg>
```

### Efectos secundarios

En algunas ocasiones el resultado de una iteracion no corresponde con un vector, tabla o lista, sino que puede ser la creacion de graficos o el exportar objetos (lo que se conoce en ingles como 'side effect'); para estos casos existe la funcion `walk` y sus variantes.

En el primer ejemplo se quiere imprimir cada grafico en la columna 'plot', por lo que se itera sobre la columna deseada, y se llama a la funcion `~ print(.)` para que despliegue cada uno de los elementos.


```r
walk(airq_nest$plot, ~print(.))
```

```
## `geom_smooth()` using formula 'y ~ x'
```

<img src="05-iteracion_files/figure-html/iter-15-1.png" width="672" />

```
## `geom_smooth()` using formula 'y ~ x'
```

<img src="05-iteracion_files/figure-html/iter-15-2.png" width="672" />

```
## `geom_smooth()` using formula 'y ~ x'
```

<img src="05-iteracion_files/figure-html/iter-15-3.png" width="672" />

```
## `geom_smooth()` using formula 'y ~ x'
```

<img src="05-iteracion_files/figure-html/iter-15-4.png" width="672" />

```
## `geom_smooth()` using formula 'y ~ x'
```

<img src="05-iteracion_files/figure-html/iter-15-5.png" width="672" />

Un resultado similar se puede obtener usando `pull`, donde se jala como vector los elementos de la columna deseada.


```r
airq_nest %>% pull(plot)
```

```
## [[1]]
```

```
## `geom_smooth()` using formula 'y ~ x'
```

<img src="05-iteracion_files/figure-html/iter-16-1.png" width="672" />

```
## 
## [[2]]
```

```
## `geom_smooth()` using formula 'y ~ x'
```

<img src="05-iteracion_files/figure-html/iter-16-2.png" width="672" />

```
## 
## [[3]]
```

```
## `geom_smooth()` using formula 'y ~ x'
```

<img src="05-iteracion_files/figure-html/iter-16-3.png" width="672" />

```
## 
## [[4]]
```

```
## `geom_smooth()` using formula 'y ~ x'
```

<img src="05-iteracion_files/figure-html/iter-16-4.png" width="672" />

```
## 
## [[5]]
```

```
## `geom_smooth()` using formula 'y ~ x'
```

<img src="05-iteracion_files/figure-html/iter-16-5.png" width="672" />

El ultimo ejemplo hace uso de `walk2` ya que se desea iterar sobre dos objetos: la columna de graficos ('plot') y la columna agrupadora ('Month'). Lo que se desea realizar es exportar cada grafico por separado, de ahi la necesidad de usar ambos objetos, el grafico a exportar y la variable agrupadora para incluirla en el nombre del archivo. Para esto ultimo se usa la funcion `str_glue` de *stringr* que lo que ahce es crear una linea de texto donde se pueden ingresar variables usando `{variable}`. En el ejemplo especificamente, se guarda cada grafico en la carpeta 'figs', con el nombre 'regresion_{.y}.png', donde '{.y}' corresponde con el segundo objeto a iterar, en este caso el mes ('Month').


```r
walk2(airq_nest$plot, 
      airq_nest$Month, 
      ~ggsave(filename = str_glue("figs/regresion_{.y}.png"),
       plot = .x, dpi = 300,
       width = 7, height = 4, units = "in",
       type = "cairo"))
```

```
## `geom_smooth()` using formula 'y ~ x'
## `geom_smooth()` using formula 'y ~ x'
## `geom_smooth()` using formula 'y ~ x'
## `geom_smooth()` using formula 'y ~ x'
## `geom_smooth()` using formula 'y ~ x'
```

## Datos anidados, caso 2

En este caso 2 se trabaja con los datos de 'gapminder', donde se agrupa por pais ('country'), y se crea una tabla para cada pais. Este caso es bastante ilustrativo del poder de los tibbles y la iteracion, ya que la tabla anidada cuenta con 142 filas (1 por pais), y si se quisiera realizar una tarea por pais a pie, seria muy tedioso y poco eficiente.


```r
gap_nest = gapminder %>% 
  group_by(country) %>% 
  nest()
```

De manera similar al caso 1, se genera un modelo lineal para cada pais en funcion de la expectativa de vida ('lifeExp') por anho ('year'), y adicionalmente se calcula el coeficiente de determinacion ($R^2$) para cada modelo lineal.


```r
gap_nest = gap_nest %>% 
  mutate(mod = map(data, ~lm(lifeExp ~ year, data = .x))) %>% 
  mutate(r2 = map_dbl(mod, ~glance(.) %>% pull(r.squared)))
gap_nest
```

```
## # A tibble: 142 x 4
## # Groups:   country [142]
##    country     data              mod       r2
##    <fct>       <list>            <list> <dbl>
##  1 Afghanistan <tibble [12 × 5]> <lm>   0.948
##  2 Albania     <tibble [12 × 5]> <lm>   0.911
##  3 Algeria     <tibble [12 × 5]> <lm>   0.985
##  4 Angola      <tibble [12 × 5]> <lm>   0.888
##  5 Argentina   <tibble [12 × 5]> <lm>   0.996
##  6 Australia   <tibble [12 × 5]> <lm>   0.980
##  7 Austria     <tibble [12 × 5]> <lm>   0.992
##  8 Bahrain     <tibble [12 × 5]> <lm>   0.967
##  9 Bangladesh  <tibble [12 × 5]> <lm>   0.989
## 10 Belgium     <tibble [12 × 5]> <lm>   0.995
## # … with 132 more rows
```

Con los datos anteriores se pueden filtrar los paises que hayan tenido un $R^2$ por debajo de 0.25, lo que seria indicio de un comportamiento no lineal, lo que podria estar asociado a problemas de desarrollo en esos paises. Para poder graficar los datos es necesario desanidarlos (`unnest`) para volver a contar con las columnas a como estaban en la tabla original, pero ahora con las columnas calculadas en las iteraciones.


```r
gap_nest %>%
  # ungroup() %>%
  # arrange(r2) %>%
  # slice(1:10) %>%
  filter(r2 < .25) %>%
  unnest(data) %>%
  ggplot() + 
  geom_line(aes(year, lifeExp, col = country, group = country))
```

<img src="05-iteracion_files/figure-html/iter-20-1.png" width="672" />


