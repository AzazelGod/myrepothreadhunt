# Практическая работа 002
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

Проанализировать встроенный в пакет dplyr набор данных языка R и
ответить на вопросы.

## Ход работы

1.  Подключаем библиотеку dplyr
2.  Проанализировать встроенный в пакет dplyr набор данных starwars с
    помощью языка R и ответить на вопросы:
    -   Сколько строк в датафрейме?
    -   Сколько столбцов в датафрейме?
    -   Как просмотреть примерный вид датафрейма?
    -   Сколько уникальных рас персонажей (species) представлено в
        данных?
    -   Найти самого высокого персонажа.
    -   Найти всех персонажей ниже 170
    -   Подсчитать ИМТ (индекс массы тела) для всех персонажей.
    -   Найти 10 самых “вытянутых” персонажей. “Вытянутость” оценить по
        отношению массы (mass) к росту (height) персонажей
    -   Найти средний возраст персонажей каждой расы вселенной Звездных
        войн
    -   Найти самый распространенный цвет глаз персонажей вселенной
        Звездных войн.
    -   Подсчитать среднюю длину имени в каждой расе вселенной Звездных
        войн.
3.  Оформить отчет в соответствии с шаблоном

### Шаг 1

Подключаем библиотеку dplyr

``` r
library(dplyr)
```


    Присоединяю пакет: 'dplyr'

    Следующие объекты скрыты от 'package:stats':

        filter, lag

    Следующие объекты скрыты от 'package:base':

        intersect, setdiff, setequal, union

### Шаг 2

#### Сколько строк в датафрейме?

``` r
starwars %>% nrow()
```

    [1] 87

#### Сколько столбцов в датафрейме?

``` r
starwars %>% ncol()
```

    [1] 14

#### Как просмотреть примерный вид датафрейма?

``` r
starwars %>% glimpse()
```

    Rows: 87
    Columns: 14
    $ name       <chr> "Luke Skywalker", "C-3PO", "R2-D2", "Darth Vader", "Leia Or…
    $ height     <int> 172, 167, 96, 202, 150, 178, 165, 97, 183, 182, 188, 180, 2…
    $ mass       <dbl> 77.0, 75.0, 32.0, 136.0, 49.0, 120.0, 75.0, 32.0, 84.0, 77.…
    $ hair_color <chr> "blond", NA, NA, "none", "brown", "brown, grey", "brown", N…
    $ skin_color <chr> "fair", "gold", "white, blue", "white", "light", "light", "…
    $ eye_color  <chr> "blue", "yellow", "red", "yellow", "brown", "blue", "blue",…
    $ birth_year <dbl> 19.0, 112.0, 33.0, 41.9, 19.0, 52.0, 47.0, NA, 24.0, 57.0, …
    $ sex        <chr> "male", "none", "none", "male", "female", "male", "female",…
    $ gender     <chr> "masculine", "masculine", "masculine", "masculine", "femini…
    $ homeworld  <chr> "Tatooine", "Tatooine", "Naboo", "Tatooine", "Alderaan", "T…
    $ species    <chr> "Human", "Droid", "Droid", "Human", "Human", "Human", "Huma…
    $ films      <list> <"A New Hope", "The Empire Strikes Back", "Return of the J…
    $ vehicles   <list> <"Snowspeeder", "Imperial Speeder Bike">, <>, <>, <>, "Imp…
    $ starships  <list> <"X-wing", "Imperial shuttle">, <>, <>, "TIE Advanced x1",…

#### Сколько уникальных рас персонажей (species) представлено в данных?

``` r
starwars["species"] %>% unique() %>% nrow() 
```

    [1] 38

#### Найти самого высокого персонажа.

``` r
starwars %>% filter(height == max(height, na.rm = TRUE)) %>%
  select(name, height)
```

    # A tibble: 1 × 2
      name        height
      <chr>        <int>
    1 Yarael Poof    264

#### Найти всех персонажей ниже 170

``` r
starwars %>% filter(height < 170) %>% select(name, height)
```

    # A tibble: 22 × 2
       name                  height
       <chr>                  <int>
     1 C-3PO                    167
     2 R2-D2                     96
     3 Leia Organa              150
     4 Beru Whitesun Lars       165
     5 R5-D4                     97
     6 Yoda                      66
     7 Mon Mothma               150
     8 Wicket Systri Warrick     88
     9 Nien Nunb                160
    10 Watto                    137
    # ℹ 12 more rows

#### Подсчитать ИМТ (индекс массы тела) для всех персонажей.

``` r
starwars <- starwars %>% mutate(IMT = mass / ( (height / 100) ^ 2 ))
starwars 
```

    # A tibble: 87 × 15
       name     height  mass hair_color skin_color eye_color birth_year sex   gender
       <chr>     <int> <dbl> <chr>      <chr>      <chr>          <dbl> <chr> <chr> 
     1 Luke Sk…    172    77 blond      fair       blue            19   male  mascu…
     2 C-3PO       167    75 <NA>       gold       yellow         112   none  mascu…
     3 R2-D2        96    32 <NA>       white, bl… red             33   none  mascu…
     4 Darth V…    202   136 none       white      yellow          41.9 male  mascu…
     5 Leia Or…    150    49 brown      light      brown           19   fema… femin…
     6 Owen La…    178   120 brown, gr… light      blue            52   male  mascu…
     7 Beru Wh…    165    75 brown      light      blue            47   fema… femin…
     8 R5-D4        97    32 <NA>       white, red red             NA   none  mascu…
     9 Biggs D…    183    84 black      light      brown           24   male  mascu…
    10 Obi-Wan…    182    77 auburn, w… fair       blue-gray       57   male  mascu…
    # ℹ 77 more rows
    # ℹ 6 more variables: homeworld <chr>, species <chr>, films <list>,
    #   vehicles <list>, starships <list>, IMT <dbl>

#### Найти 10 самых “вытянутых” персонажей. “Вытянутость” оценить по отношению массы (mass) к росту (height) персонажей

``` r
starwars %>%
  mutate(stretch = mass / height) %>% arrange(desc(stretch)) %>%
  head(10) %>% select(name, stretch)
```

    # A tibble: 10 × 2
       name                  stretch
       <chr>                   <dbl>
     1 Jabba Desilijic Tiure   7.76 
     2 Grievous                0.736
     3 IG-88                   0.7  
     4 Owen Lars               0.674
     5 Darth Vader             0.673
     6 Jek Tono Porkins        0.611
     7 Bossk                   0.595
     8 Tarfful                 0.581
     9 Dexter Jettster         0.515
    10 Chewbacca               0.491

#### Найти средний возраст персонажей каждой расы вселенной Звездных войн

``` r
starwars %>%
  filter(!is.na(species), !is.na(birth_year)) %>%
  group_by(species) %>%
  summarise(avg_age = mean(birth_year, na.rm = TRUE)) %>%
  arrange(desc(avg_age)) 
```

    # A tibble: 15 × 2
       species        avg_age
       <chr>            <dbl>
     1 Yoda's species   896  
     2 Hutt             600  
     3 Wookiee          200  
     4 Cerean            92  
     5 Zabrak            54  
     6 Human             53.7
     7 Droid             53.3
     8 Trandoshan        53  
     9 Gungan            52  
    10 Mirialan          49  
    11 Twi'lek           48  
    12 Rodian            44  
    13 Mon Calamari      41  
    14 Kel Dor           22  
    15 Ewok               8  

#### Найти самый распространенный цвет глаз персонажей вселенной Звездных войн.

``` r
starwars %>%
  count(eye_color, sort = TRUE) %>% head(1)
```

    # A tibble: 1 × 2
      eye_color     n
      <chr>     <int>
    1 brown        21

#### Подсчитать среднюю длину имени в каждой расе вселенной Звездных войн.

``` r
starwars %>%
  mutate(name_length = nchar(name)) %>% group_by(species) %>%
  summarise(avg_name_length = mean(name_length, na.rm = TRUE))
```

    # A tibble: 38 × 2
       species   avg_name_length
       <chr>               <dbl>
     1 Aleena              12   
     2 Besalisk            15   
     3 Cerean              12   
     4 Chagrian            10   
     5 Clawdite            10   
     6 Droid                4.83
     7 Dug                  7   
     8 Ewok                21   
     9 Geonosian           17   
    10 Gungan              11.7 
    # ℹ 28 more rows

### Шаг 2

Отчет оформлен в соответствии с шаблоном

## Оценка результатов

Задача решена с использованием языка программирования R и пакета
`dplyr`. Я научился использовать эти два инструмента для анализа данных
на более продвинутом уровне.

## Вывод

В данной работе я ответил на вопросы путем анализа и обработки
датафрейма `starwars` и развил практические навыки
