# Практическая работа 004
trembochev@yandex.ru

## Цель работы

1.  Зекрепить практические навыки использования языка программирования R
    для обработки данны
2.  Закрепить знания основных функций обработки данных экосистемы языка
    R
3.  Закрепить навыки исследования метаданных DNS трафика

## Исходные данные

1.  Программное обеспечение Windows 11
2.  Rstudio Desktop
3.  Интерпретатор языка R 4.5.2
4.  Данные о DNS трафике в внутренней сети Доброй Организации

## Задание

Используя программный пакет dplyr, освоить анализ DNS логов с помощью
языка программирования R.

## Ход работы

1.  Подготовка данных  
    1.1. Импортируйте данные DNS –
    https://storage.yandexcloud.net/dataset.ctfsec/dns.zip  
    1.2. Добавьте пропущенные данные о структуре данных (назначении
    столбцов)  
    1.3. Преобразуйте данные в столбцах в нужный формат  
    1.4. Просмотрите общую структуру данных с помощью функции
    glimpse()  

2.  Анализ  
    2.1. Сколько участников информационного обмена в сети Доброй
    Организации?  
    2.2. Какое соотношение участников обмена внутри сети и участников
    обращений к внешним ресурсам?  
    2.3. Найдите топ-10 участников сети, проявляющих наибольшую сетевую
    активность.  
    2.4. Найдите топ-10 доменов, к которым обращаются пользователи сети
    и соответственное количество обращений.  
    2.5. Опеределите базовые статистические характеристики (функция
    summary() ) интервала времени между последовательными обращениями к
    топ-10 доменам.  
    2.6. Часто вредоносное программное обеспечение использует DNS канал
    в качестве канала управления, периодически отправляя запросы на
    подконтрольный злоумышленникам DNS сервер. По периодическим запросам
    на один и тот же домен можно выявить скрытый DNS канал. Есть ли
    такие IP адреса в исследуемом датасете?  

3.  Обогащение данных  
    3.1. Определите местоположение (страну, город) и
    организацию-провайдера для топ-10 доменов. Для этого можно
    использовать сторонние сервисы, например http://ip-api.com
    (API-эндпоинт – http://ip-api.com/json).  

4.  Оформить отчет в соответствии с шаблоном  

### Шаг 1

импортируем пакет `dplyr`:

``` r
library(dplyr)
```


    Присоединяю пакет: 'dplyr'

    Следующие объекты скрыты от 'package:stats':

        filter, lag

    Следующие объекты скрыты от 'package:base':

        intersect, setdiff, setequal, union

Считаем наш лог:

``` r
dns_data <- read.table(
  "dns.log",
  sep = "\t",
  header = FALSE,
  fill = TRUE,
  quote = "",
  stringsAsFactors = FALSE
)
```

Добавим названия столбцов:

``` r
colnames(dns_data) <- c(
  "ts","uid","id.orig_h","id.orig_p","id.resp_h","id.resp_p",
  "proto","trans_id","query","qclass","qclass_name","qtype",
  "qtype_name","rcode","rcode_name","AA","TC","RD","RA","Z",
  "answers","TTLs","rejected"
)[1:ncol(dns_data)]
```

Преобразуем данные в столбцах в нужный формат:

``` r
dns_data$ts <- as.POSIXct(as.numeric(dns_data$ts), origin="1970-01-01", tz="UTC")
dns_data$id.orig_p <- as.integer(dns_data$id.orig_p)
dns_data$id.resp_p <- as.integer(dns_data$id.resp_p)
```

Посмотрим общую структуру данных повторно:

``` r
glimpse(dns_data)
```

    Rows: 427,935
    Columns: 23
    $ ts          <dttm> 2012-03-16 12:30:05, 2012-03-16 12:30:15, 2012-03-16 12:3…
    $ uid         <chr> "CWGtK431H9XuaTN4fi", "C36a282Jljz7BsbGH", "C36a282Jljz7Bs…
    $ id.orig_h   <chr> "192.168.202.100", "192.168.202.76", "192.168.202.76", "19…
    $ id.orig_p   <int> 45658, 137, 137, 137, 137, 137, 137, 137, 137, 137, 137, 1…
    $ id.resp_h   <chr> "192.168.27.203", "192.168.202.255", "192.168.202.255", "1…
    $ id.resp_p   <int> 137, 137, 137, 137, 137, 137, 137, 137, 137, 137, 137, 137…
    $ proto       <chr> "udp", "udp", "udp", "udp", "udp", "udp", "udp", "udp", "u…
    $ trans_id    <int> 33008, 57402, 57402, 57402, 57398, 57398, 57398, 62187, 62…
    $ query       <chr> "*\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\…
    $ qclass      <chr> "1", "1", "1", "1", "1", "1", "1", "1", "1", "1", "1", "1"…
    $ qclass_name <chr> "C_INTERNET", "C_INTERNET", "C_INTERNET", "C_INTERNET", "C…
    $ qtype       <chr> "33", "32", "32", "32", "32", "32", "32", "32", "32", "32"…
    $ qtype_name  <chr> "SRV", "NB", "NB", "NB", "NB", "NB", "NB", "NB", "NB", "NB…
    $ rcode       <chr> "0", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-"…
    $ rcode_name  <chr> "NOERROR", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-…
    $ AA          <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FA…
    $ TC          <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FA…
    $ RD          <lgl> FALSE, TRUE, TRUE, TRUE, TRUE, TRUE, TRUE, TRUE, TRUE, TRU…
    $ RA          <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FA…
    $ Z           <int> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 1, 1, 1, 1, 0…
    $ answers     <chr> "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-"…
    $ TTLs        <chr> "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-"…
    $ rejected    <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FA…

### Шаг 2

#### Сколько участников информационного обмена в сети Доброй Организации?

Найдем все уникальные IP-адреса в данных DNS (и источников, и
получателей) и считам их количество:

``` r
all_ips <- dns_data %>% select(id.orig_h, id.resp_h  ) %>%unlist() %>% unique()

length(all_ips)
```

    [1] 1359

#### 5. Какое соотношение участников обмена внутри сети и участников обращений к внешним ресурсам?

Ip-адреса можно разделить на серые и белые (публичные), cерые адреса:

-   192.168.0.0/16
-   10.0.0.0/8,
-   172.16.0.0/12

Воспользуемся пакетом `ipaddress` :

``` r
library(ipaddress)
```

Функция `is_private`, позволи понять, является ли адрес внутренним:

``` r
ip_addresses <- ip_address(dns_data %>% distinct(id.resp_h) %>% pull(id.resp_h))
internal <- sum(is_private(ip_addresses), na.rm = TRUE)
external <- sum(!is_private(ip_addresses), na.rm = TRUE)
internal / (internal + external)
```

    [1] 0.9658537

#### Найдите топ-10 участников сети, проявляющих наибольшую сетевую активность.

Будем считать за активность, если участник отправляет и получает
запросы.

``` r
top10_act <- bind_rows(
  dns_data %>%select(ip = id.orig_h),
  dns_data %>% select(ip = id.resp_h)
) %>%
  filter(!is.na(ip), ip != "-") %>%
  count(ip, name = "act") %>%arrange(desc(act)) %>%  head(10)

top10_act
```

                    ip    act
    1    192.168.207.4 266627
    2    10.10.117.210  75943
    3  192.168.202.255  68720
    4   192.168.202.93  26522
    5     172.19.1.100  25481
    6  192.168.202.103  18121
    7   192.168.202.76  16978
    8   192.168.202.97  16176
    9  192.168.202.141  14976
    10 192.168.202.110  14493

#### Найдите топ-10 доменов, к которым обращаются пользователи сети и соответственное количество обращений

``` r
top_dmn <- dns_data %>%
  count(query, sort = TRUE, name = "hits") %>% head(10)

top_dmn
```

                                                                         query
    1                                                teredo.ipv6.microsoft.com
    2                                                         tools.google.com
    3                                                            www.apple.com
    4                                                           time.apple.com
    5                                          safebrowsing.clients.google.com
    6  *\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00
    7                                                                     WPAD
    8                                              44.206.168.192.in-addr.arpa
    9                                                                 HPE8AA67
    10                                                                  ISATAP
        hits
    1  39273
    2  14057
    3  13390
    4  13109
    5  11658
    6  10401
    7   9134
    8   7248
    9   6929
    10  6569

#### Опеределите базовые статистические характеристики (функция summary() ) интервала времени между последовательными обращениями к топ-10 доменам.

``` r
time_int <- dns_data %>%
  filter(query %in% top_dmn$query) %>% arrange(query, ts) %>%group_by(query) %>% mutate(time_diff = ts - lag(ts)) %>%
  ungroup()

summary(time_int$time_diff)
```

    Time differences in secs
         Min.   1st Qu.    Median      Mean   3rd Qu.      Max.      NA's 
        0.000     0.000     0.750     8.758     1.740 52723.500        10 

#### Поиск возможного скрытого DNS-канала

Для каждого домена вычисляет время между последовательными запросами, и
найдем домены с большим колличесвом запросов и низким отклонением

``` r
periodic_sus <- dns_data %>%
  filter(!is.na(query), query != "-") %>% arrange(query, ts) %>%group_by(query) %>%
  mutate(
    interval = as.numeric(ts - lag(ts), units = "secs")
  ) %>%
  summarise(
    mean_int = mean(interval, na.rm = TRUE),
    sd_int  = sd(interval, na.rm = TRUE),
    n             = sum(!is.na(interval))
  ) %>%
  filter(n > 30, sd_int < 1)

periodic_sus
```

    # A tibble: 6 × 4
      query             mean_int  sd_int     n
      <chr>                <dbl>   <dbl> <int>
    1 hq.h               0.00265 0.0515    377
    2 httphq.hec.net     0.00990 0.100     102
    3 input.mozilla.com  5.00    0.00475    31
    4 lifehacker.com     0.00612 0.0171     67
    5 www.h              0.0156  0.125      64
    6 www.hkparts.net    0.00371 0.0124     35

Наиболее подозрительными выглядят домены `hq.h` и `httphq.hec.net`,
генерирующие большое количество обращений с очень малыми интервалами.
Домен `input.mozilla.com` сохраняет периодичность в 5 секунд, скорее
всего сбор телеметрии с браузера.

### Шаг 3

#### Определить местоположение и организацию-провайдера для топ-10 доменов

``` r
library(httr)
library(jsonlite)

resolve <- function(domain) {
  url <- paste0("http://ip-api.com/json/", domain)
  response <- httr::GET(url)
  data <- jsonlite::fromJSON(rawToChar(response$content))
  tibble(
    domain = domain,
    country = data$country,
    city = data$city,
    isp = data$isp,
    org = data$org,
    as = data$as
  )
}

enr <- purrr::map_df(top_dmn$query, resolve)

enr
```

    # A tibble: 10 × 6
       domain                                        country city  isp   org   as   
       <chr>                                         <chr>   <chr> <chr> <chr> <chr>
     1 "teredo.ipv6.microsoft.com"                   <NA>    <NA>  <NA>  <NA>  <NA> 
     2 "tools.google.com"                            United… Wash… Goog… Goog… AS15…
     3 "www.apple.com"                               United… Ashb… Akam… Akam… AS20…
     4 "time.apple.com"                              United… Sant… Appl… Appl… AS61…
     5 "safebrowsing.clients.google.com"             United… Moun… Goog… Goog… AS15…
     6 "*\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\… <NA>    <NA>  <NA>  <NA>  <NA> 
     7 "WPAD"                                        <NA>    <NA>  <NA>  <NA>  <NA> 
     8 "44.206.168.192.in-addr.arpa"                 <NA>    <NA>  <NA>  <NA>  <NA> 
     9 "HPE8AA67"                                    <NA>    <NA>  <NA>  <NA>  <NA> 
    10 "ISATAP"                                      <NA>    <NA>  <NA>  <NA>  <NA> 

### Шаг 4

Отчёт написан и оформлен

## Оценка результатов

Задача решена с использованием языка программирования R и пакетами
`dplyr`, `httr`, `jsonlite`, `ipaddress`. Я научился использовать эти
инструмента для анализа DNS логов.

## Вывод

В данной работе я используя программный пакет dplyr, освоить анализ DNS
логов с помощью языка программирования R.
