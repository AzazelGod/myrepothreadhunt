# Практическая работа 003
trembochev@yandex.ru

## Цель работы

1.  Развить практические навыки использования языка программирования R
    для обработки данных
2.  Закрепить знания базовых типов данных языка R
3.  Развить практические навыки использования функций обработки данных
    пакета `dplyr` – функции
    `select(), filter(), mutate(), arrange(), group_by()`

## Исходные данные

1.  Программное обеспечение Windows 11
2.  Rstudio Desktop
3.  Интерпретатор языка R 4.5.2

## Задание

Проанализировать встроенный в пакет nycflights13 набор данных языка R и
ответить на вопросы.

## Ход работы

1.  Установить nycflight13
2.  Подключить библиотеку dplyr и nycflights13
3.  Проанализировать встроенные в пакет nycflights13 наборы данных с
    помощью языка R и ответить на вопросы:
    -   Сколько встроенных в пакет nycflights13 датафреймов?
    -   Сколько строк в каждом датафрейме?
    -   Сколько столбцов в каждом датафрейме?
    -   Как просмотреть примерный вид датафрейма?
    -   Сколько компаний-перевозчиков (carrier) учитывают эти наборы
        данных (представлено в наборах данных)?
    -   Сколько рейсов принял аэропорт John F Kennedy Intl в мае?
    -   Какой самый северный аэропорт?
    -   Какой аэропорт самый высокогорный (находится выше всех над
        уровнем моря)?
    -   Какие бортовые номера у самых старых самолетов?
    -   Какая средняя температура воздуха была в сентябре в аэропорту
        John FKennedy Intl (в градусах Цельсия).
    -   Самолеты какой авиакомпании совершили больше всего вылетов в
        июне?
    -   Самолеты какой авиакомпании задерживались чаще других в 2013
        году?
4.  Оформить отчет в соответствии с шаблоном

### Шаг 1

Установим пакет с помощью install.packages(‘nycflights13’)

    > install.packages('nycflights13')
    WARNING: Rtools is required to build R packages but is not currently installed. Please download and install the appropriate version of Rtools before proceeding:

    https://cran.rstudio.com/bin/windows/Rtools/
    Устанавливаю пакет в ‘C:/Users/Trembachevs/AppData/Local/R/win-library/4.5’
    (потому что ‘lib’ не определено)
    пробую URL 'https://cran.rstudio.com/bin/windows/contrib/4.5/nycflights13_1.0.2.zip'
    Content type 'application/zip' length 4511564 bytes (4.3 MB)
    downloaded 4.3 MB

    пакет ‘nycflights13’ успешно распакован, MD5-суммы проверены

    Скачанные бинарные пакеты находятся в
        C:\Users\Trembachevs\AppData\Local\Temp\RtmpItuDfG\downloaded_packages
    > 

### Шаг 2

Подключим необходимые библиотеки

``` r
library(nycflights13)
```

``` r
library(dplyr)
```


    Присоединяю пакет: 'dplyr'

    Следующие объекты скрыты от 'package:stats':

        filter, lag

    Следующие объекты скрыты от 'package:base':

        intersect, setdiff, setequal, union

### Шаг 3

#### Сколько встроенных в пакет nycflights13 датафреймов?

``` r
length(data(package = "nycflights13")$results[, "Item"])
```

    [1] 5

#### Сколько строк в каждом датафрейме?

``` r
airports %>% nrow()
```

    [1] 1458

``` r
airlines %>% nrow()
```

    [1] 16

``` r
flights %>% nrow()
```

    [1] 336776

``` r
weather %>% nrow()
```

    [1] 26115

``` r
planes %>% nrow()
```

    [1] 3322

#### Сколько столбцов в каждом датафрейме?

``` r
airports %>% ncol()
```

    [1] 8

``` r
airlines %>% ncol()
```

    [1] 2

``` r
flights %>% ncol()
```

    [1] 19

``` r
weather %>% ncol()
```

    [1] 15

``` r
planes %>% ncol()
```

    [1] 9

#### Как просмотреть примерный вид датафрейма?

``` r
airports %>% glimpse()
```

    Rows: 1,458
    Columns: 8
    $ faa   <chr> "04G", "06A", "06C", "06N", "09J", "0A9", "0G6", "0G7", "0P2", "…
    $ name  <chr> "Lansdowne Airport", "Moton Field Municipal Airport", "Schaumbur…
    $ lat   <dbl> 41.13047, 32.46057, 41.98934, 41.43191, 31.07447, 36.37122, 41.4…
    $ lon   <dbl> -80.61958, -85.68003, -88.10124, -74.39156, -81.42778, -82.17342…
    $ alt   <dbl> 1044, 264, 801, 523, 11, 1593, 730, 492, 1000, 108, 409, 875, 10…
    $ tz    <dbl> -5, -6, -6, -5, -5, -5, -5, -5, -5, -8, -5, -6, -5, -5, -5, -5, …
    $ dst   <chr> "A", "A", "A", "A", "A", "A", "A", "A", "U", "A", "A", "U", "A",…
    $ tzone <chr> "America/New_York", "America/Chicago", "America/Chicago", "Ameri…

``` r
airlines %>% glimpse()
```

    Rows: 16
    Columns: 2
    $ carrier <chr> "9E", "AA", "AS", "B6", "DL", "EV", "F9", "FL", "HA", "MQ", "O…
    $ name    <chr> "Endeavor Air Inc.", "American Airlines Inc.", "Alaska Airline…

``` r
flights %>% glimpse()
```

    Rows: 336,776
    Columns: 19
    $ year           <int> 2013, 2013, 2013, 2013, 2013, 2013, 2013, 2013, 2013, 2…
    $ month          <int> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1…
    $ day            <int> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1…
    $ dep_time       <int> 517, 533, 542, 544, 554, 554, 555, 557, 557, 558, 558, …
    $ sched_dep_time <int> 515, 529, 540, 545, 600, 558, 600, 600, 600, 600, 600, …
    $ dep_delay      <dbl> 2, 4, 2, -1, -6, -4, -5, -3, -3, -2, -2, -2, -2, -2, -1…
    $ arr_time       <int> 830, 850, 923, 1004, 812, 740, 913, 709, 838, 753, 849,…
    $ sched_arr_time <int> 819, 830, 850, 1022, 837, 728, 854, 723, 846, 745, 851,…
    $ arr_delay      <dbl> 11, 20, 33, -18, -25, 12, 19, -14, -8, 8, -2, -3, 7, -1…
    $ carrier        <chr> "UA", "UA", "AA", "B6", "DL", "UA", "B6", "EV", "B6", "…
    $ flight         <int> 1545, 1714, 1141, 725, 461, 1696, 507, 5708, 79, 301, 4…
    $ tailnum        <chr> "N14228", "N24211", "N619AA", "N804JB", "N668DN", "N394…
    $ origin         <chr> "EWR", "LGA", "JFK", "JFK", "LGA", "EWR", "EWR", "LGA",…
    $ dest           <chr> "IAH", "IAH", "MIA", "BQN", "ATL", "ORD", "FLL", "IAD",…
    $ air_time       <dbl> 227, 227, 160, 183, 116, 150, 158, 53, 140, 138, 149, 1…
    $ distance       <dbl> 1400, 1416, 1089, 1576, 762, 719, 1065, 229, 944, 733, …
    $ hour           <dbl> 5, 5, 5, 5, 6, 5, 6, 6, 6, 6, 6, 6, 6, 6, 6, 5, 6, 6, 6…
    $ minute         <dbl> 15, 29, 40, 45, 0, 58, 0, 0, 0, 0, 0, 0, 0, 0, 0, 59, 0…
    $ time_hour      <dttm> 2013-01-01 05:00:00, 2013-01-01 05:00:00, 2013-01-01 0…

``` r
weather %>% glimpse()
```

    Rows: 26,115
    Columns: 15
    $ origin     <chr> "EWR", "EWR", "EWR", "EWR", "EWR", "EWR", "EWR", "EWR", "EW…
    $ year       <int> 2013, 2013, 2013, 2013, 2013, 2013, 2013, 2013, 2013, 2013,…
    $ month      <int> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,…
    $ day        <int> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,…
    $ hour       <int> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 13, 14, 15, 16, 17, 18, …
    $ temp       <dbl> 39.02, 39.02, 39.02, 39.92, 39.02, 37.94, 39.02, 39.92, 39.…
    $ dewp       <dbl> 26.06, 26.96, 28.04, 28.04, 28.04, 28.04, 28.04, 28.04, 28.…
    $ humid      <dbl> 59.37, 61.63, 64.43, 62.21, 64.43, 67.21, 64.43, 62.21, 62.…
    $ wind_dir   <dbl> 270, 250, 240, 250, 260, 240, 240, 250, 260, 260, 260, 330,…
    $ wind_speed <dbl> 10.35702, 8.05546, 11.50780, 12.65858, 12.65858, 11.50780, …
    $ wind_gust  <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, 20.…
    $ precip     <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,…
    $ pressure   <dbl> 1012.0, 1012.3, 1012.5, 1012.2, 1011.9, 1012.4, 1012.2, 101…
    $ visib      <dbl> 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10,…
    $ time_hour  <dttm> 2013-01-01 01:00:00, 2013-01-01 02:00:00, 2013-01-01 03:00…

``` r
planes %>% glimpse()
```

    Rows: 3,322
    Columns: 9
    $ tailnum      <chr> "N10156", "N102UW", "N103US", "N104UW", "N10575", "N105UW…
    $ year         <int> 2004, 1998, 1999, 1999, 2002, 1999, 1999, 1999, 1999, 199…
    $ type         <chr> "Fixed wing multi engine", "Fixed wing multi engine", "Fi…
    $ manufacturer <chr> "EMBRAER", "AIRBUS INDUSTRIE", "AIRBUS INDUSTRIE", "AIRBU…
    $ model        <chr> "EMB-145XR", "A320-214", "A320-214", "A320-214", "EMB-145…
    $ engines      <int> 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, …
    $ seats        <int> 55, 182, 182, 182, 55, 182, 182, 182, 182, 182, 55, 55, 5…
    $ speed        <int> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ engine       <chr> "Turbo-fan", "Turbo-fan", "Turbo-fan", "Turbo-fan", "Turb…

#### Сколько компаний-перевозчиков (carrier) учитывают эти наборы данных (представлено в наборах данных)?

``` r
airlines$carrier %>% n_distinct()
```

    [1] 16

#### Сколько рейсов принял аэропорт John F Kennedy Intl в мае?

``` r
flights %>% filter(origin == "JFK", month == 5) %>%  nrow()
```

    [1] 9397

#### Какой самый северный аэропорт?

``` r
airports %>% filter(airports$lon==max(airports$lon)) %>% pull("name")
```

    [1] "Eareckson As"

#### Какой аэропорт самый высокогорный (находится выше всех над уровнем моря)?

``` r
airports %>% filter(airports$alt==max(airports$alt)) %>% pull("name")
```

    [1] "Telluride"

#### Какие бортовые номера у самых старых самолетов?

``` r
planes %>%arrange(year) %>% head(10) %>% select(tailnum)
```

    # A tibble: 10 × 1
       tailnum
       <chr>  
     1 N381AA 
     2 N201AA 
     3 N567AA 
     4 N378AA 
     5 N575AA 
     6 N14629 
     7 N615AA 
     8 N425AA 
     9 N383AA 
    10 N364AA 

#### Какая средняя температура воздуха была в сентябре в аэропорту John F Kennedy Intl (в градусах Цельсия).

``` r
weather %>%  filter(origin == "JFK", month == 9) %>%  summarise(temp_c = mean((temp - 32) * 5/9, na.rm = TRUE))
```

    # A tibble: 1 × 1
      temp_c
       <dbl>
    1   19.4

#### Самолеты какой авиакомпании совершили больше всего вылетов в июне?

``` r
flights %>% left_join(., airlines, join_by(carrier))  %>% filter(month == 6) %>%count(name, sort = TRUE) %>% head(1)
```

    # A tibble: 1 × 2
      name                      n
      <chr>                 <int>
    1 United Air Lines Inc.  4975

#### Самолеты какой авиакомпании задерживались чаще других в 2013 году?

``` r
flights %>% left_join(., airlines, join_by(carrier)) %>% group_by(., name) %>% filter(., arr_delay > 0 & year == 2013) %>% summarise(., amount = n()) %>% arrange(., desc(amount)) %>% select(., name) %>% head(1)
```

    # A tibble: 1 × 1
      name                    
      <chr>                   
    1 ExpressJet Airlines Inc.

### Шаг 4

Отчёт написан и оформлен

## Оценка результатов

Задача решена с использованием языка программирования R и пакета
`dplyr`. Я научился использовать эти два инструмента для анализа данных
на более продвинутом уровне.

## Вывод

В данной работе я ответил на вопросы путем анализа и обработки
датафрейфов встроенные в пакет `nycflights13` и развил практические
навыки
