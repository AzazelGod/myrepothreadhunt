# Практическая работа 006
trembochev@yandex.ru

## Цель работы

1.  Закрепить навыки исследования данных журнала Windows Active
    Directory
2.  Изучить структуру журнала системы Windows Active Directory
3.  Зекрепить практические навыки использования языка программирования R
    для обработки данных
4.  Закрепить знания основных функций обработки данных экосистемы
    tidyverse языка R

## Исходные данные

1.  Программное обеспечение Windows 11
2.  Rstudio Desktop
3.  Интерпретатор языка R 4.5.2
4.  Выгрузки данных журнала Windows Active Directory

## Задание

Используя программный пакет dplyr, освоить анализ DNS логов с помощью
языка программирования R.

## Ход работы

1.  Подготовка данных  
    1.1. Импортировать данные
    (https://storage.yandexcloud.net/iamcth-data/dataset.tar.gz)  
    1.2. Привести датасеты в вид “аккуратных данных”, преобразовать типы
    столбцов в соответствии с типом данных  
    1.3. Просмотрите общую структуру данных с помощью функции
    glimpse()  

2.  Анализ  
    2.1. Раскройте датафрейм избавившись от вложенных датафреймов. Для
    обнаружения таких можно использовать функцию dplyr::glimpse() , а
    для раскрытия вложенности – tidyr::unnest() . Обратите внимание, что
    при раскрытии теряются внешние названия колонок – это можно
    предотвратить если использовать параметр tidyr::unnest(…, names_sep
    = )  
    2.2. Минимизируйте количество колонок в датафрейме – уберите колоки
    с единственным значением параметра.  
    2.3. Какое количество хостов представлено в данном датасете?  
    2.4. Подготовьте датафрейм с расшифровкой Windows Event_ID,
    приведите типы данных к типу их значений.  
    2.5. Есть ли в логе события с высоким и средним уровнем значимости?
    Сколько их?

3.  Оформить отчет в соответствии с шаблоном  

### Шаг 1.

#### Импортировать данные

Установим:

    > download.file("https://storage.yandexcloud.net/iamcth-data/dataset.tar.gz", "dataset.tar.gz")
    пробую URL 'https://storage.yandexcloud.net/iamcth-data/dataset.tar.gz'
    Content type 'application/gzip' length 12608123 bytes (12.0 MB)
    downloaded 12.0 MB

    > untar("dataset.tar.gz")

Для начала импортируем необходимые пакеты:

``` r
library(jsonlite)
library(tidyverse)
```

    ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ✔ dplyr     1.1.4     ✔ readr     2.1.6
    ✔ forcats   1.0.1     ✔ stringr   1.6.0
    ✔ ggplot2   4.0.1     ✔ tibble    3.3.0
    ✔ lubridate 1.9.4     ✔ tidyr     1.3.1
    ✔ purrr     1.2.0     
    ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ✖ dplyr::filter()  masks stats::filter()
    ✖ purrr::flatten() masks jsonlite::flatten()
    ✖ dplyr::lag()     masks stats::lag()
    ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

Cделаем датафрейм:

``` r
json_path <- file.path("caldera_attack_evals_round1_day1_2019-10-20201108.json")

events_data <- stream_in(file(json_path, open = "r"), verbose = FALSE)

glimpse(events_data)
```

    Rows: 101,904
    Columns: 9
    $ `@timestamp` <chr> "2019-10-20T20:11:06.937Z", "2019-10-20T20:11:07.101Z", "…
    $ `@metadata`  <df[,4]> <data.frame[26 x 4]>
    $ event        <df[,4]> <data.frame[26 x 4]>
    $ log          <df[,1]> <data.frame[26 x 1]>
    $ message      <chr> "A token right was adjusted.\n\nSubject:\n\tSecurity I…
    $ winlog       <df[,16]> <data.frame[26 x 16]>
    $ ecs          <df[,1]> <data.frame[26 x 1]>
    $ host         <df[,1]> <data.frame[26 x 1]>
    $ agent        <df[,5]> <data.frame[26 x 5]>

Cкачаем справочник о условным кодам журнала Windows это будет event_df.

``` r
library(xml2)
library(rvest)
```


    Присоединяю пакет: 'rvest'

    Следующий объект скрыт от 'package:readr':

        guess_encoding

``` r
webpage_url <- "https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/appendix-l--events-to-monitor"
webpage <- xml2::read_html(webpage_url)
event_df <- rvest::html_table(webpage)[[1]]
```

``` r
glimpse(event_df)
```

    Rows: 381
    Columns: 4
    $ `Current Windows Event ID` <chr> "4618", "4649", "4719", "4765", "4766", "47…
    $ `Legacy Windows Event ID`  <chr> "N/A", "N/A", "612", "N/A", "N/A", "N/A", "…
    $ `Potential Criticality`    <chr> "High", "High", "High", "High", "High", "Hi…
    $ `Event Summary`            <chr> "A monitored security event pattern has occ…

### Шаг 2. Анализ

#### Раскройте датафрейм избавившись от вложенных датафреймов. Для обнаружения таких можно использовать функцию dplyr::glimpse() , а для раскрытия вложенности – tidyr::unnest() . Обратите внимание, что при раскрытии теряются внешние названия колонок – это можно предотвратить если использовать параметр tidyr::unnest(…, names_sep = ).

Распакуем все вложенные датафреймы в исходном датафрейме

``` r
library(tidyr)
library(dplyr)

events_flat <- events_data %>%
  unnest(c(`@metadata`, event, log, winlog, ecs, host, agent), names_sep = "_")

glimpse(events_flat)
```

    Rows: 101,904
    Columns: 34
    $ `@timestamp`         <chr> "2019-10-20T20:11:06.937Z", "2019-10-20T20:11:07.…
    $ `@metadata_beat`     <chr> "winlogbeat", "winlogbeat", "winlogbeat", "winlog…
    $ `@metadata_type`     <chr> "_doc", "_doc", "_doc", "_doc", "_doc", "_doc", "…
    $ `@metadata_version`  <chr> "7.4.0", "7.4.0", "7.4.0", "7.4.0", "7.4.0", "7.4…
    $ `@metadata_topic`    <chr> "winlogbeat", "winlogbeat", "winlogbeat", "winlog…
    $ event_created        <chr> "2019-10-20T20:11:09.988Z", "2019-10-20T20:11:09.…
    $ event_kind           <chr> "event", "event", "event", "event", "event", "eve…
    $ event_code           <int> 4703, 4673, 10, 10, 10, 10, 11, 10, 10, 10, 10, 7…
    $ event_action         <chr> "Token Right Adjusted Events", "Sensitive Privile…
    $ log_level            <chr> "information", "information", "information", "inf…
    $ message              <chr> "A token right was adjusted.\n\nSubject:\n\tSecur…
    $ winlog_event_data    <df[,234]> <data.frame[26 x 234]>
    $ winlog_event_id      <int> 4703, 4673, 10, 10, 10, 10, 11, 10, 10, 10, …
    $ winlog_provider_name <chr> "Microsoft-Windows-Security-Auditing", "Microsoft…
    $ winlog_api           <chr> "wineventlog", "wineventlog", "wineventlog", "win…
    $ winlog_record_id     <int> 50588, 104875, 226649, 153525, 163488, 153526, 13…
    $ winlog_computer_name <chr> "HR001.shire.com", "HFDC01.shire.com", "IT001.shi…
    $ winlog_process       <df[,2]> <data.frame[26 x 2]>
    $ winlog_keywords      <list> "Audit Success", "Audit Failure", <NULL>, <NULL>,…
    $ winlog_provider_guid <chr> "{54849625-5478-4994-a5ba-3e3b0328c30d}", "{54…
    $ winlog_channel       <chr> "security", "Security", "Microsoft-Windows-Sysmo…
    $ winlog_task          <chr> "Token Right Adjusted Events", "Sensitive Privile…
    $ winlog_opcode        <chr> "Info", "Info", "Info", "Info", "Info", "Info", "…
    $ winlog_version       <int> NA, NA, 3, 3, 3, 3, 2, 3, 3, 3, 3, 3, 3, 3, NA, 3…
    $ winlog_user          <df[,4]> <data.frame[26 x 4]>
    $ winlog_activity_id   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog_user_data     <df[,30]> <data.frame[26 x 30]>
    $ ecs_version          <chr> "1.1.0", "1.1.0", "1.1.0", "1.1.0", "1.1.0", "1.1…
    $ host_name            <chr> "WECServer", "WECServer", "WECServer", "WECSer…
    $ agent_ephemeral_id   <chr> "b372be1f-ba0a-4d7e-b4df-79eac86e1fde", "b372be1f…
    $ agent_hostname       <chr> "WECServer", "WECServer", "WECServer", "WECSe…
    $ agent_id             <chr> "d347d9a4-bff4-476c-b5a4-d51119f78250", "d347d9a4…
    $ agent_version        <chr> "7.4.0", "7.4.0", "7.4.0", "7.4.0", "7.4.0", "7.4…
    $ agent_type           <chr> "winlogbeat", "winlogbeat", "winlogbeat", "winlog…

``` r
ncol(events_flat)
```

    [1] 34

#### 2. Минимизируйте количество колонок в датафрейме – уберите колоки с единственным значением параметра.

``` r
events_clean <- events_flat %>%
  select(where(~n_distinct(.) > 1))
glimpse(events_clean)
```

    Rows: 101,904
    Columns: 21
    $ `@timestamp`         <chr> "2019-10-20T20:11:06.937Z", "2019-10-20T20:11:07.…
    $ event_created        <chr> "2019-10-20T20:11:09.988Z", "2019-10-20T20:11:09.…
    $ event_code           <int> 4703, 4673, 10, 10, 10, 10, 11, 10, 10, 10, 10, 7…
    $ event_action         <chr> "Token Right Adjusted Events", "Sensitive Privile…
    $ log_level            <chr> "information", "information", "information", "inf…
    $ message              <chr> "A token right was adjusted.\n\nSubject:\n\tSecur…
    $ winlog_event_data    <df[,234]> <data.frame[26 x 234]>
    $ winlog_event_id      <int> 4703, 4673, 10, 10, 10, 10, 11, 10, 10, 10, …
    $ winlog_provider_name <chr> "Microsoft-Windows-Security-Auditing", "Microsoft…
    $ winlog_record_id     <int> 50588, 104875, 226649, 153525, 163488, 153526, 13…
    $ winlog_computer_name <chr> "HR001.shire.com", "HFDC01.shire.com", "IT001.shi…
    $ winlog_process       <df[,2]> <data.frame[26 x 2]>
    $ winlog_keywords      <list> "Audit Success", "Audit Failure", <NULL>, <NULL>,…
    $ winlog_provider_guid <chr> "{54849625-5478-4994-a5ba-3e3b0328c30d}", "{54…
    $ winlog_channel       <chr> "security", "Security", "Microsoft-Windows-Sysmo…
    $ winlog_task          <chr> "Token Right Adjusted Events", "Sensitive Privile…
    $ winlog_opcode        <chr> "Info", "Info", "Info", "Info", "Info", "Info", "…
    $ winlog_version       <int> NA, NA, 3, 3, 3, 3, 2, 3, 3, 3, 3, 3, 3, 3, NA, 3…
    $ winlog_user          <df[,4]> <data.frame[26 x 4]>
    $ winlog_activity_id   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog_user_data     <df[,30]> <data.frame[26 x 30]>

``` r
ncol(events_clean)
```

    [1] 21

#### 3. Какое количество хостов представлено в данном датасете?

``` r
events_clean %>% select(winlog_computer_name) %>% distinct() %>% count()
```

    # A tibble: 1 × 1
          n
      <int>
    1     5

#### 4. Подготовьте датафрейм с расшифровкой Windows Event_ID, приведите типы данных к типу их значений

Подготовим справочник и соеденим его с нашем логом
(датафреймов).Добавили к каждому событию из лога информацию об его
важности.

``` r
event_dict <- event_df %>%
  rename(
    Event_ID = `Current Windows Event ID`,
    Legacy_ID = `Legacy Windows Event ID`,
    Criticality = `Potential Criticality`,
    Summary = `Event Summary`
  ) %>%

  mutate(
    Event_ID = as.integer(Event_ID),
    Criticality = factor(Criticality)
  )
```

    Warning: There was 1 warning in `mutate()`.
    ℹ In argument: `Event_ID = as.integer(Event_ID)`.
    Caused by warning:
    ! в результате преобразования созданы NA

``` r
events_final <- events_clean %>%
  left_join(event_dict, by = c("winlog_event_id" = "Event_ID"))

glimpse(events_final)
```

    Rows: 101,904
    Columns: 24
    $ `@timestamp`         <chr> "2019-10-20T20:11:06.937Z", "2019-10-20T20:11:07.…
    $ event_created        <chr> "2019-10-20T20:11:09.988Z", "2019-10-20T20:11:09.…
    $ event_code           <int> 4703, 4673, 10, 10, 10, 10, 11, 10, 10, 10, 10, 7…
    $ event_action         <chr> "Token Right Adjusted Events", "Sensitive Privile…
    $ log_level            <chr> "information", "information", "information", "inf…
    $ message              <chr> "A token right was adjusted.\n\nSubject:\n\tSecur…
    $ winlog_event_data    <df[,234]> <data.frame[26 x 234]>
    $ winlog_event_id      <int> 4703, 4673, 10, 10, 10, 10, 11, 10, 10, 10, …
    $ winlog_provider_name <chr> "Microsoft-Windows-Security-Auditing", "Microsoft…
    $ winlog_record_id     <int> 50588, 104875, 226649, 153525, 163488, 153526, 13…
    $ winlog_computer_name <chr> "HR001.shire.com", "HFDC01.shire.com", "IT001.shi…
    $ winlog_process       <df[,2]> <data.frame[26 x 2]>
    $ winlog_keywords      <list> "Audit Success", "Audit Failure", <NULL>, <NULL>,…
    $ winlog_provider_guid <chr> "{54849625-5478-4994-a5ba-3e3b0328c30d}", "{54…
    $ winlog_channel       <chr> "security", "Security", "Microsoft-Windows-Sysmo…
    $ winlog_task          <chr> "Token Right Adjusted Events", "Sensitive Privile…
    $ winlog_opcode        <chr> "Info", "Info", "Info", "Info", "Info", "Info", "…
    $ winlog_version       <int> NA, NA, 3, 3, 3, 3, 2, 3, 3, 3, 3, 3, 3, 3, NA, 3…
    $ winlog_user          <df[,4]> <data.frame[26 x 4]>
    $ winlog_activity_id   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog_user_data     <df[,30]> <data.frame[26 x 30]>
    $ Legacy_ID            <chr> NA, "577", NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ Criticality          <fct> NA, Low, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ Summary              <chr> NA, "A privileged service was called.", NA, NA, N…

#### 5. Есть ли в логе события с высоким и средним уровнем значимости? Сколько их?

Считаем, сколько событий каждого уровня важности есть в логе.

``` r
summary_counts <- events_final %>%
  filter(!is.na(Criticality)) %>% group_by(Criticality) %>%summarise(count = n()) %>% arrange(desc(count))

print(summary_counts)
```

    # A tibble: 1 × 2
      Criticality count
      <fct>       <int>
    1 Low          2436

Как оказалось событий с критичностью Medium/High в наших событиях нет,
есть только события с критичностью Low

### Шаг 3

Отчёт написан и оформлен

## Оценка результатов

Задача решена с использованием языка программирования R и пакетами
`dplyr`, `jsonlite`, `tidyverse`.Я научился использовать эти инструмента
для данных журнала Windows Active Directory.

## Вывод

В данной работе я используя программный пакет dplyr, освоить анализ
данных журнала Windows Active Directory с помощью языка программирования
R.
