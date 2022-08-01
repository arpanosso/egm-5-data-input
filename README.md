
<!-- README.md is generated from README.Rmd. Please edit that file -->

# Script Para Leitura e manipulação dos dados da EGM-5

Todos os dados provenientes da máquina EGM-5 devem ser colocados na
pasta **data-raw**, posteriormente, os dados pré-processados são salvos
na pasta **data**.

## Carregando os pacotes

``` r
library(tidyverse)
#> -- Attaching packages --------------------------------------- tidyverse 1.3.1 --
#> v ggplot2 3.3.5     v purrr   0.3.4
#> v tibble  3.1.6     v dplyr   1.0.8
#> v tidyr   1.2.0     v stringr 1.4.0
#> v readr   2.1.2     v forcats 0.5.1
#> -- Conflicts ------------------------------------------ tidyverse_conflicts() --
#> x dplyr::filter() masks stats::filter()
#> x dplyr::lag()    masks stats::lag()
library(readr)
```

## Definindo os caminhos dos arquivos txt

``` r
caminhos_arquivos <- list.files(path = "data-raw",
           pattern = ".txt",
           full.names = TRUE)
```

## 

``` r
df <- read_csv(caminhos_arquivos)
#> Warning: One or more parsing issues, see `problems()` for details
#> Rows: 1106 Columns: 17
#> -- Column specification --------------------------------------------------------
#> Delimiter: ","
#> chr   (3): Tag(M3), Date, Msoil
#> dbl  (13): Plot_No, Rec_No, CO2, Pressure, Flow, H2O, Tsen, O2, Error, Aux_V...
#> time  (1): Time
#> 
#> i Use `spec()` to retrieve the full column specification for this data.
#> i Specify the column types or set `show_col_types = FALSE` to quiet this message.
```
