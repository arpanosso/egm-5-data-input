
<!-- README.md is generated from README.Rmd. Please edit that file -->

# Script Para Leitura e manipulação dos dados da EGM-5

Todos os dados provenientes da máquina EGM-5 devem ser colocados na
pasta **data-raw**, posteriormente, os dados pré-processados são salvos
na pasta **data**.

## Carregando os pacotes

``` r
library(tidyverse)
library(readr)
library(stringr)
library(janitor)
library(lubridate)
library(writexl)
```

## Definindo os caminhos dos arquivos txt

``` r
caminhos_arquivos <- list.files(path = "data-raw",
           pattern = ".TXT|.txt",
           full.names = TRUE)
```

## Estrutura do arquivo original

As
![17](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;17 "17")
primeiras colunas são listadas abaixo, após a leitura do arquivos,
deve-se adicionar, organizar,
![5](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;5 "5")
parâmetros restantes.

<img src="https://raw.githubusercontent.com/arpanosso/egm-5-data-input/master/img/data_format_1.png" width="600px" style="display: block; margin: auto;" />

``` r
df <- read_csv(caminhos_arquivos[8]) %>% clean_names() %>% 
  drop_na() %>% 
  separate(msoil,c("msoil","process", "dc", "dt", "srl_rate", "srq_rate"),",") %>% 
  mutate(
    across(
      .cols = c("msoil","process", "dc", "dt", "srl_rate", "srq_rate"),
      .fns = as.numeric
    ),
   date = as.Date(date, format="%d/%m/%y")
  ) 

point_count <- 0
df$point <- NA
for(i in 1:nrow(df)){
  if(df$dt[i] == 1) point_count = point_count + 1 
  df$point[i] <- point_count
}
```

Abaixo segue os
![5](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;5 "5")
parâmetros a serem manipulados

<img src="https://raw.githubusercontent.com/arpanosso/egm-5-data-input/master/img/data_format_2.png" width="600px" style="display: block; margin: auto;" />

``` r
head(df) 
#> # A tibble: 6 x 23
#>   tag_m3 date       time     plot_no rec_no   co2 pressure  flow   h2o  tsen
#>   <chr>  <date>     <time>     <dbl>  <dbl> <dbl>    <dbl> <dbl> <dbl> <dbl>
#> 1 M5     2022-07-13 07:34:02       1   1447   477     988.   329     0     0
#> 2 M5     2022-07-13 07:34:03       1   1448   477     988.   329     0     0
#> 3 M5     2022-07-13 07:34:04       1   1449   476     988.   328     0     0
#> 4 M5     2022-07-13 07:34:05       1   1450   475     988.   328     0     0
#> 5 M5     2022-07-13 07:34:06       1   1451   475     988.   329     0     0
#> 6 M5     2022-07-13 07:34:07       1   1452   474     988.   330     0     0
#> # ... with 13 more variables: o2 <dbl>, error <dbl>, aux_v <dbl>, par <dbl>,
#> #   tsoil <dbl>, tair <dbl>, msoil <dbl>, process <dbl>, dc <dbl>, dt <dbl>,
#> #   srl_rate <dbl>, srq_rate <dbl>, point <dbl>
```

``` r
pt <- df %>% 
  ggplot(aes(x=dt, y=o2)) +
  geom_point() +
  facet_wrap(~point)
ggsave(str_replace(caminhos_arquivos[8],".TXT|.txt", ".png"),pt)
#> Saving 7 x 5 in image
```

## Salvar o Banco de dados em .xlsx

``` r
caminho_saida <- str_replace(caminhos_arquivos[26],".TXT|.txt",".xlsx")
write_xlsx(df,caminho_saida) 
```

## Calcular a emissão de CO<sub>2</sub> do solo para cada ponto em cada arquivo.

Será utilizada a abordagem de **Parkinson (1981)**.

<img src="https://raw.githubusercontent.com/arpanosso/egm-5-data-input/master/img/data_format_3.png" width="600px" style="display: block; margin: auto;" />

``` r
df
#> # A tibble: 3,270 x 23
#>    tag_m3 date       time     plot_no rec_no   co2 pressure  flow   h2o  tsen
#>    <chr>  <date>     <time>     <dbl>  <dbl> <dbl>    <dbl> <dbl> <dbl> <dbl>
#>  1 M5     2022-07-13 07:34:02       1   1447   477     988.   329     0     0
#>  2 M5     2022-07-13 07:34:03       1   1448   477     988.   329     0     0
#>  3 M5     2022-07-13 07:34:04       1   1449   476     988.   328     0     0
#>  4 M5     2022-07-13 07:34:05       1   1450   475     988.   328     0     0
#>  5 M5     2022-07-13 07:34:06       1   1451   475     988.   329     0     0
#>  6 M5     2022-07-13 07:34:07       1   1452   474     988.   330     0     0
#>  7 M5     2022-07-13 07:34:08       1   1453   474     988.   328     0     0
#>  8 M5     2022-07-13 07:34:09       1   1454   474     988.   328     0     0
#>  9 M5     2022-07-13 07:34:10       1   1455   474     988.   329     0     0
#> 10 M5     2022-07-13 07:34:11       1   1456   474     988.   329     0     0
#> # ... with 3,260 more rows, and 13 more variables: o2 <dbl>, error <dbl>,
#> #   aux_v <dbl>, par <dbl>, tsoil <dbl>, tair <dbl>, msoil <dbl>,
#> #   process <dbl>, dc <dbl>, dt <dbl>, srl_rate <dbl>, srq_rate <dbl>,
#> #   point <dbl>
points <- df %>% pull(point) %>%  unique()
for( i in seq_along(points)){
  V = 0.001678
  A = pi*(.1)^2/4
  dff <- df %>% filter(point == points[i])
  Tair = dff %>% pull(tair) %>% mean(na.rm=TRUE)
  Tsoil = dff %>% pull(tsoil) %>% mean(na.rm=TRUE)
  Msoil = dff %>% pull(msoil) %>% mean(na.rm=TRUE)
  Pressure = dff %>% pull(pressure) %>% mean(na.rm=TRUE)
  srl_rate_m = dff %>% filter(dt > 60) %>% pull(srl_rate) %>% mean(na.rm=TRUE)
  srq_rate_m = dff %>% filter(dt > 60) %>% pull(srq_rate) %>% mean(na.rm=TRUE)
 
  dC <- dff %>% filter(dt > 60) %>% pull(dc)
  dT <- dff %>% filter(dt > 60) %>% pull(dt)
  
  fco2 = dC %>% range() %>%  diff() / dT %>% range() %>%  diff() * Pressure/1013 * 273/(273+Tair) * 1/22.414 * V/A * 1e3
  
  o2 <- dff %>% pull(o2)
  time <- dff %>% pull(dt)
  reg_lim <- lm(o2 ~ time)
  taxa_o2 <- reg_lim$coefficients[2]
  
  dim_cam <- 0.1
  alt_cam <- 0.215
  R <- 8.314462
  
  Dv <- (taxa_o2*10000*V*0.000001)
  Dn <- Pressure *100* Dv/R/(Tair+273.105)
  fo2 <- (-Dn)*32*1000*A
}
#> Warning in min(x): nenhum argumento não faltante para min; retornando Inf
#> Warning in max(x): nenhum argumento não faltante para max; retornando -Inf
#> Warning in min(x): nenhum argumento não faltante para min; retornando Inf
#> Warning in max(x): nenhum argumento não faltante para max; retornando -Inf
c(Tair_m=Tair, Presurre_m = Pressure, Taxa_Linear = srl_rate_m, Taxa_quad = srq_rate_m, FCO2= fco2, FO2 = fo2 ,Ts=Tsoil, Ms = Msoil)
#>        Tair_m    Presurre_m   Taxa_Linear     Taxa_quad          FCO2 
#>  2.756519e+01  9.896608e+02  3.737207e-01  4.092099e-01  2.132347e+00 
#>      FO2.time            Ts            Ms 
#> -1.077616e-04  2.210000e+01  7.759669e+00
```

### Criando a função para ler vários arquivos, faz os cálculos das estatísticas

dos arquivos.

``` r
egm_5_reader <- function(pasta){
 
  df = read_csv(pasta) %>% clean_names() %>% 
    drop_na() %>% 
    separate(msoil,c("msoil","process", "dc", "dt", "srl_rate", "srq_rate"),",") %>% 
    mutate(
      across(
        .cols = c("msoil","process", "dc", "dt", "srl_rate", "srq_rate"),
        .fns = as.numeric
      ),
      date = as.Date(date, format="%d/%m/%y")
    ) 
  
  if(nrow(df)!=0){
    point_count = 0
    df$point = NA
    for(i in 1:nrow(df)){
      if(df$dt[i] == 1) point_count = point_count + 1 
      df$point[i] = point_count
    }
    caminho_saida <- str_replace(pasta,".TXT|.txt",".xlsx")
    write_xlsx(df,caminho_saida)
    
    # co2_graph <-  df %>% 
    #   ggplot(aes(x=dt, y=co2)) +
    #   geom_point() +
    #   facet_wrap(~point) +
    #   theme_bw()
    # ggsave(str_replace(pasta,".TXT|.txt", "_co2.png"),co2_graph)
    # 
    # o2_graph <-  df %>% 
    #   ggplot(aes(x=dt, y=o2)) +
    #   geom_point() +
    #   facet_wrap(~point)+
    #   theme_bw()
    # ggsave(str_replace(pasta,".TXT|.txt", "_o2.png"),o2_graph)
    
    points <- df %>% pull(point) %>%  unique()
    V = 0.001678
    A = pi*(.1)^2/4
    R <- 8.314462
    alt_cam <- 0.215
    Vol <- A * alt_cam
    
    for(i in seq_along(points)){
      dff <- df %>% filter(point == points[i])
      Tair = dff %>% pull(tair) %>% mean(na.rm=TRUE)
      Pressure = dff %>% pull(pressure) %>% mean(na.rm=TRUE)
      Tsoil = dff %>% pull(tsoil) %>% mean(na.rm=TRUE)
      Msoil = dff %>% pull(msoil) %>% mean(na.rm=TRUE)
      srl_rate_m = dff %>% filter(dt > 60) %>% pull(srl_rate) %>% mean(na.rm=TRUE)
      srq_rate_m = dff %>% filter(dt > 60) %>% pull(srq_rate) %>% mean(na.rm=TRUE)
      dC <- dff %>% filter(dt > 60) %>% pull(dc)
      dT <- dff %>% filter(dt > 60) %>% pull(dt)
      fco2 = dC %>% range() %>%  diff() / dT %>% range() %>%  diff() * Pressure/1013 * 273/(273+Tair) * 1/22.414 * V/A * 1e3
      o2 <- dff %>% pull(o2)
      time <- dff %>% pull(dt)
      reg_lim <- lm(o2 ~ time)
      taxa_o2 <- reg_lim$coefficients[[2]]
      
      Dv <- (taxa_o2*10000*Vol*0.000001)
      Dn <- Pressure*100 * Dv/R/(Tair+273.105)
      fo2 <- (-Dn)*32*1000*A
      
      tb <- c(Tair_m=Tair, Presurre_m = Pressure, Taxa_Linear = srl_rate_m, Taxa_quad = srq_rate_m, FCO2= fco2, FO2 = fo2,Ts=Tsoil, Ms = Msoil)
      if(i == 1 ){ 
        tb_final <- tb
      }else{
        tb_final <- rbind(tb_final, tb)
      }
    }
    tb_final<-as.data.frame(tb_final)
    
    saida<-paste0("data/summary_",
                  str_remove(str_split(pasta,"/",simplify = TRUE)[2],".TXT|.txt"),
                  ".xlsx")
    write_xlsx(tb_final,saida)
  }
  ## calculando as estatisticas para cada ponto
  # return(tb_final)
} 
map(caminhos_arquivos,~egm_5_reader(.x))
#> [[1]]
#> [1] "C:\\GitHub\\egm-5-data-input\\data\\summary_22062010.xlsx"
#> 
#> [[2]]
#> [1] "C:\\GitHub\\egm-5-data-input\\data\\summary_22062110.xlsx"
#> 
#> [[3]]
#> NULL
#> 
#> [[4]]
#> NULL
#> 
#> [[5]]
#> [1] "C:\\GitHub\\egm-5-data-input\\data\\summary_22070210.xlsx"
#> 
#> [[6]]
#> NULL
#> 
#> [[7]]
#> NULL
#> 
#> [[8]]
#> [1] "C:\\GitHub\\egm-5-data-input\\data\\summary_22071310.xlsx"
#> 
#> [[9]]
#> [1] "C:\\GitHub\\egm-5-data-input\\data\\summary_22071311.xlsx"
#> 
#> [[10]]
#> NULL
#> 
#> [[11]]
#> [1] "C:\\GitHub\\egm-5-data-input\\data\\summary_22071313.xlsx"
#> 
#> [[12]]
#> [1] "C:\\GitHub\\egm-5-data-input\\data\\summary_22071410.xlsx"
#> 
#> [[13]]
#> [1] "C:\\GitHub\\egm-5-data-input\\data\\summary_22071411.xlsx"
#> 
#> [[14]]
#> [1] "C:\\GitHub\\egm-5-data-input\\data\\summary_22071412.xlsx"
#> 
#> [[15]]
#> [1] "C:\\GitHub\\egm-5-data-input\\data\\summary_22071510.xlsx"
#> 
#> [[16]]
#> [1] "C:\\GitHub\\egm-5-data-input\\data\\summary_22071511.xlsx"
#> 
#> [[17]]
#> [1] "C:\\GitHub\\egm-5-data-input\\data\\summary_22071512.xlsx"
#> 
#> [[18]]
#> [1] "C:\\GitHub\\egm-5-data-input\\data\\summary_22071810.xlsx"
#> 
#> [[19]]
#> [1] "C:\\GitHub\\egm-5-data-input\\data\\summary_22071811.xlsx"
#> 
#> [[20]]
#> [1] "C:\\GitHub\\egm-5-data-input\\data\\summary_22071812.xlsx"
#> 
#> [[21]]
#> [1] "C:\\GitHub\\egm-5-data-input\\data\\summary_22071910.xlsx"
#> 
#> [[22]]
#> [1] "C:\\GitHub\\egm-5-data-input\\data\\summary_22071911.xlsx"
#> 
#> [[23]]
#> [1] "C:\\GitHub\\egm-5-data-input\\data\\summary_22071912.xlsx"
#> 
#> [[24]]
#> [1] "C:\\GitHub\\egm-5-data-input\\data\\summary_22072010.xlsx"
#> 
#> [[25]]
#> [1] "C:\\GitHub\\egm-5-data-input\\data\\summary_22072011.xlsx"
#> 
#> [[26]]
#> [1] "C:\\GitHub\\egm-5-data-input\\data\\summary_Arquivo original.xlsx"
```
