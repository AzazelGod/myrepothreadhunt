# Практическая работа 005
trembochev@yandex.ru

## Цель работы

1.  Получить знания о методах исследования радиоэлектронной обстановки.
2.  Составить представление о механизмах работы Wi-Fi сетей на канальном
    и сетевом уровне модели OSI.
3.  Зекрепить практические навыки использования языка программирования R
    для обработки данных
4.  Закрепить знания основных функций обработки данных экосистемы
    tidyverse языка R

## Исходные данные

1.  Программное обеспечение Windows 11
2.  Rstudio Desktop
3.  Интерпретатор языка R 4.5.2
4.  Данные wi-fi снифера

## Задание

Используя программный пакет dplyr, освоить анализ DNS логов с помощью
языка программирования R.

## Ход работы

1.  Подготовка данных  
    1.1. Импортировать данные
    (https://storage.yandexcloud.net/dataset.ctfsec/P2_wi_data.csv)  
    1.2. Привести датасеты в вид “аккуратных данных”, преобразовать типы
    столбцов в соответствии с типом данных 1.3. Просмотрите общую
    структуру данных с помощью функции `glimpse()`  
2.  Анализ  
    2.1 Анализ точек доступа  
    2.1.1 Определить небезопасные точки доступа (без шифрования – OPN)  
    2.1.2. Определить производителя для каждого обнаруженного
    устройства  
    2.1.3. Выявить устройства, использующие последнюю версию протокола
    шифрования WPA3, и названия точек доступа, реализованных на этих
    устройствах  
    2.1.4. Отсортировать точки доступа по интервалу времени, в течение
    которого они находились на связи, по убыванию.  
    2.1.5. Обнаружить топ-10 самых быстрых точек доступа.  
    2.1.6. Отсортировать точки доступа по частоте отправки запросов
    (beacons) в единицу времени по их убыванию.  
    2.2 Анализ клиентов  
    2.2.1 Определить производителя для каждого обнаруженного
    устройства.  
    2.2.2 Обнаружить устройства, которые НЕ рандомизируют свой MAC
    адрес  
    2.2.3. Кластеризовать запросы от устройств к точкам доступа по их
    именам. Определить время появления устройства в зоне радиовидимости
    и время выхода его из нее.  
    2.2.4. Оценить стабильность уровня сигнала внури кластера во
    времени. Выявить наиболее стабильный кластер.  
3.  Оформить отчет в соответствии с шаблоном  

### Шаг 1.

Для начала импортируем необходимые пакеты:

``` r
library(tidyverse)
```

    ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ✔ dplyr     1.1.4     ✔ readr     2.1.6
    ✔ forcats   1.0.1     ✔ stringr   1.6.0
    ✔ ggplot2   4.0.1     ✔ tibble    3.3.0
    ✔ lubridate 1.9.4     ✔ tidyr     1.3.1
    ✔ purrr     1.2.0     
    ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ✖ dplyr::filter() masks stats::filter()
    ✖ dplyr::lag()    masks stats::lag()
    ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

Скачаем файл:

``` r
tmp <- tempfile()
download.file(
  "https://storage.yandexcloud.net/dataset.ctfsec/P2_wifi_data.csv",
  tmp,
  mode = "wb"
)

wifi_ap <- read_csv(tmp, n_max = 167)
```

    Rows: 167 Columns: 15
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr  (6): BSSID, Privacy, Cipher, Authentication, LAN IP, ESSID
    dbl  (6): channel, Speed, Power, # beacons, # IV, ID-length
    lgl  (1): Key
    dttm (2): First time seen, Last time seen

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
clients <- read_csv(tmp, skip = 169)
```

    Warning: One or more parsing issues, call `problems()` on your data frame for details,
    e.g.:
      dat <- vroom(...)
      problems(dat)

    Rows: 12081 Columns: 7
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr  (3): Station MAC, BSSID, Probed ESSIDs
    dbl  (2): Power, # packets
    dttm (2): First time seen, Last time seen

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

Приведем аттрибуты к нужному типу данных для датасета точек доступа:

``` r
names(wifi_ap) <- trimws(names(wifi_ap))

wifi_ap_clean <- wifi_ap %>%
  rename(
    bssid      = BSSID,
    first_seen = `First time seen`,
    last_seen  = `Last time seen`,
    speed      = Speed,
    privacy    = Privacy,
    cipher     = Cipher,
    auth       = Authentication,
    power      = Power,
    beacons    = `# beacons`,
    iv_count   = `# IV`,
    lan_ip     = `LAN IP`,
    id_length  = `ID-length`,
    essid      = ESSID
  ) %>%
  mutate(
    across(where(is.character), trimws),
    first_seen = as.POSIXct(first_seen),
    last_seen  = as.POSIXct(last_seen),
    across(c(channel, speed, power, beacons, iv_count, id_length), as.numeric)
  )
```

Проделаем операцию с клиентами:

``` r
names(clients) <- trimws(names(clients))

wifi_clients_clean <- clients %>%
  rename(
    station_mac = `Station MAC`,
    first_seen  = `First time seen`,
    last_seen   = `Last time seen`,
    power       = Power,
    packets     = `# packets`,
    bssid       = BSSID,
    probed_essids = `Probed ESSIDs`
  ) %>%
  mutate(
    across(where(is.character), trimws),
    first_seen = as.POSIXct(first_seen),
    last_seen  = as.POSIXct(last_seen),
    power      = as.numeric(power),
    packets    = as.numeric(packets),
    station_mac = toupper(station_mac),
    bssid = case_when(
      grepl("(?i)not associated", bssid) ~ NA_character_,
      TRUE ~ toupper(bssid)
    )
  )
```

Теперь посмотрим общую структуру файлов:

``` r
glimpse(wifi_ap_clean)
```

    Rows: 167
    Columns: 15
    $ bssid      <chr> "BE:F1:71:D5:17:8B", "6E:C7:EC:16:DA:1A", "9A:75:A8:B9:04:1…
    $ first_seen <dttm> 2023-07-28 09:13:03, 2023-07-28 09:13:03, 2023-07-28 09:13…
    $ last_seen  <dttm> 2023-07-28 11:50:50, 2023-07-28 11:55:12, 2023-07-28 11:53…
    $ channel    <dbl> 1, 1, 1, 7, 6, 6, 11, 11, 11, 1, 6, 14, 11, 11, 6, 6, 6, 6,…
    $ speed      <dbl> 195, 130, 360, 360, 130, 130, 195, 130, 130, 195, 180, 65, …
    $ privacy    <chr> "WPA2", "WPA2", "WPA2", "WPA2", "WPA2", "OPN", "WPA2", "WPA…
    $ cipher     <chr> "CCMP", "CCMP", "CCMP", "CCMP", "CCMP", NA, "CCMP", "CCMP",…
    $ auth       <chr> "PSK", "PSK", "PSK", "PSK", "PSK", NA, "PSK", "PSK", "PSK",…
    $ power      <dbl> -30, -30, -68, -37, -57, -63, -27, -38, -38, -66, -42, -62,…
    $ beacons    <dbl> 846, 750, 694, 510, 647, 251, 1647, 1251, 704, 617, 1390, 1…
    $ iv_count   <dbl> 504, 116, 26, 21, 6, 3430, 80, 11, 0, 0, 86, 0, 0, 0, 907, …
    $ lan_ip     <chr> "0.  0.  0.  0", "0.  0.  0.  0", "0.  0.  0.  0", "0.  0. …
    $ id_length  <dbl> 12, 4, 2, 14, 25, 13, 12, 13, 24, 12, 10, 0, 24, 24, 12, 0,…
    $ essid      <chr> "C322U13 3965", "Cnet", "KC", "POCO X5 Pro 5G", NA, "MIREA_…
    $ Key        <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA,…

``` r
glimpse(wifi_clients_clean)
```

    Rows: 12,081
    Columns: 7
    $ station_mac   <chr> "CA:66:3B:8F:56:DD", "96:35:2D:3D:85:E6", "5C:3A:45:9E:1…
    $ first_seen    <dttm> 2023-07-28 09:13:03, 2023-07-28 09:13:03, 2023-07-28 09…
    $ last_seen     <dttm> 2023-07-28 10:59:44, 2023-07-28 09:13:03, 2023-07-28 11…
    $ power         <dbl> -33, -65, -39, -61, -53, -43, -31, -71, -74, -65, -45, -…
    $ packets       <dbl> 858, 4, 432, 958, 1, 344, 163, 3, 115, 437, 265, 77, 7, …
    $ bssid         <chr> "BE:F1:71:D5:17:8B", NA, "BE:F1:71:D6:10:D7", "BE:F1:71:…
    $ probed_essids <chr> "C322U13 3965", "IT2 Wireless", "C322U21 0566", "C322U13…

### Шаг 2.

#### 1. Определить небезопасные точки доступа (без шифрования – OPN)

``` r
wifi_ap_clean %>% filter(privacy == "OPN")
```

    # A tibble: 42 × 15
       bssid    first_seen          last_seen           channel speed privacy cipher
       <chr>    <dttm>              <dttm>                <dbl> <dbl> <chr>   <chr> 
     1 E8:28:C… 2023-07-28 09:13:03 2023-07-28 11:55:38       6   130 OPN     <NA>  
     2 E8:28:C… 2023-07-28 09:13:06 2023-07-28 11:55:12       6   130 OPN     <NA>  
     3 E8:28:C… 2023-07-28 09:13:06 2023-07-28 11:55:11       6   130 OPN     <NA>  
     4 E8:28:C… 2023-07-28 09:13:06 2023-07-28 11:55:10       6    -1 OPN     <NA>  
     5 00:25:0… 2023-07-28 09:13:06 2023-07-28 11:56:21      44    -1 OPN     <NA>  
     6 E8:28:C… 2023-07-28 09:13:09 2023-07-28 11:56:05      11   130 OPN     <NA>  
     7 E8:28:C… 2023-07-28 09:13:13 2023-07-28 10:27:06       6   130 OPN     <NA>  
     8 E8:28:C… 2023-07-28 09:13:13 2023-07-28 10:39:43       6   130 OPN     <NA>  
     9 E8:28:C… 2023-07-28 09:13:17 2023-07-28 11:52:32       1   130 OPN     <NA>  
    10 E8:28:C… 2023-07-28 09:13:50 2023-07-28 11:43:39      11   130 OPN     <NA>  
    # ℹ 32 more rows
    # ℹ 8 more variables: auth <chr>, power <dbl>, beacons <dbl>, iv_count <dbl>,
    #   lan_ip <chr>, id_length <dbl>, essid <chr>, Key <lgl>

### 2. Определить производителя для каждого обнаруженного устройства

``` r
oui_url <- "https://standards-oui.ieee.org/oui/oui.txt"
oui_text <- httr::content(httr::GET(oui_url), "text", encoding = "UTF-8")

oui_lines <- strsplit(oui_text, "\n")[[1]]
oui_db <- data.frame(
  oui = substr(grep("^[0-9A-F]{6}", oui_lines, value = TRUE), 1, 8),
  vendor = substr(grep("^[0-9A-F]{6}", oui_lines, value = TRUE), 22, 100)
)

oui_db$oui <- paste0(
  substr(oui_db$oui, 1, 2), ":",
  substr(oui_db$oui, 3, 4), ":",
  substr(oui_db$oui, 5, 6)
)


wifi_ap_clean$oui <- toupper(substr(wifi_ap_clean$bssid, 1, 8))
wifi_ap_clean$vendor <- oui_db$vendor[match(wifi_ap_clean$oui, oui_db$oui)]

result <- wifi_ap_clean %>%
  filter(!is.na(vendor)) %>%
  select(bssid, vendor) %>%
  as.data.frame()

print(result)
```

                   bssid                                                  vendor
    1  E8:28:C1:DC:B2:52                               \tEltex Enterprise Ltd.\r
    2  38:1A:52:0D:84:D7                             \tSeiko Epson Corporation\r
    3  1C:7E:E5:8E:B7:DE                                \tD-Link International\r
    4  38:1A:52:0D:97:60                             \tSeiko Epson Corporation\r
    5  38:1A:52:0D:90:A1                             \tSeiko Epson Corporation\r
    6  E8:28:C1:DC:B2:50                               \tEltex Enterprise Ltd.\r
    7  E8:28:C1:DC:B2:51                               \tEltex Enterprise Ltd.\r
    8  E8:28:C1:DC:FF:F2                               \tEltex Enterprise Ltd.\r
    9  00:25:00:FF:94:73                                         \tApple, Inc.\r
    10 00:26:99:F2:7A:E2                                  \tCisco Systems, Inc\r
    11 48:5B:39:F9:7A:48                               \tASUSTek COMPUTER INC.\r
    12 00:26:99:F2:7A:E1                                  \tCisco Systems, Inc\r
    13 0C:80:63:A9:6E:EE                       \tTP-LINK TECHNOLOGIES CO.,LTD.\r
    14 E8:28:C1:DD:04:52                               \tEltex Enterprise Ltd.\r
    15 E8:28:C1:DE:74:31                               \tEltex Enterprise Ltd.\r
    16 E8:28:C1:DE:74:32                               \tEltex Enterprise Ltd.\r
    17 E8:28:C1:DC:C8:32                               \tEltex Enterprise Ltd.\r
    18 00:26:99:BA:75:80                                  \tCisco Systems, Inc\r
    19 08:3A:2F:56:35:FE \tGuangzhou Juan Intelligent Tech Joint Stock Co.,Ltd\r
    20 00:23:EB:E3:81:F2                                  \tCisco Systems, Inc\r
    21 E8:28:C1:DD:04:50                               \tEltex Enterprise Ltd.\r
    22 E8:28:C1:DD:04:51                               \tEltex Enterprise Ltd.\r
    23 E8:28:C1:DC:C8:30                               \tEltex Enterprise Ltd.\r
    24 E8:28:C1:DE:74:30                               \tEltex Enterprise Ltd.\r
    25 00:23:EB:E3:81:F1                                  \tCisco Systems, Inc\r
    26 E0:D9:E3:48:FF:D2                               \tEltex Enterprise Ltd.\r
    27 E8:28:C1:DC:B2:41                               \tEltex Enterprise Ltd.\r
    28 E8:28:C1:DC:B2:40                               \tEltex Enterprise Ltd.\r
    29 00:26:99:F2:7A:E0                                  \tCisco Systems, Inc\r
    30 00:23:EB:E3:81:FE                                  \tCisco Systems, Inc\r
    31 00:23:EB:E3:81:FD                                  \tCisco Systems, Inc\r
    32 E8:28:C1:DC:B2:42                               \tEltex Enterprise Ltd.\r
    33 E8:28:C1:DD:04:40                               \tEltex Enterprise Ltd.\r
    34 E8:28:C1:DD:04:41                               \tEltex Enterprise Ltd.\r
    35 00:26:CB:AA:62:71                                  \tCisco Systems, Inc\r
    36 E8:28:C1:DE:47:D2                               \tEltex Enterprise Ltd.\r
    37 E8:28:C1:DC:3C:92                               \tEltex Enterprise Ltd.\r
    38 10:50:72:00:11:08                                \tSercomm Corporation.\r
    39 14:EB:B6:6A:76:37                                 \tTP-Link Systems Inc\r
    40 E0:D9:E3:48:FF:D0                               \tEltex Enterprise Ltd.\r
    41 E8:28:C1:DC:C6:B1                               \tEltex Enterprise Ltd.\r
    42 E8:28:C1:DD:04:42                               \tEltex Enterprise Ltd.\r
    43 00:03:7A:1A:18:56                               \tTaiyo Yuden Co., Ltd.\r
    44 E8:28:C1:DC:C8:31                               \tEltex Enterprise Ltd.\r
    45 E8:28:C1:DC:54:72                               \tEltex Enterprise Ltd.\r
    46 E8:28:C1:DE:47:D1                               \tEltex Enterprise Ltd.\r
    47 00:09:9A:12:55:04                               \tELMO COMPANY, LIMITED\r
    48 38:1A:52:0D:85:1D                             \tSeiko Epson Corporation\r
    49 E8:28:C1:DC:3A:B0                               \tEltex Enterprise Ltd.\r
    50 E8:28:C1:DC:C6:B0                               \tEltex Enterprise Ltd.\r
    51 E8:28:C1:DC:C6:B2                               \tEltex Enterprise Ltd.\r
    52 E8:28:C1:DB:F5:F2                               \tEltex Enterprise Ltd.\r
    53 E8:28:C1:DC:BD:50                               \tEltex Enterprise Ltd.\r
    54 E8:28:C1:DC:0B:B2                               \tEltex Enterprise Ltd.\r
    55 E8:28:C1:DC:3C:80                               \tEltex Enterprise Ltd.\r
    56 00:23:EB:E3:49:31                                  \tCisco Systems, Inc\r
    57 00:23:EB:E3:44:31                                  \tCisco Systems, Inc\r
    58 E8:28:C1:DC:33:12                               \tEltex Enterprise Ltd.\r
    59 E8:28:C1:DB:FC:F2                               \tEltex Enterprise Ltd.\r
    60 00:03:7A:1A:03:56                               \tTaiyo Yuden Co., Ltd.\r
    61 00:26:99:BA:75:8F                                  \tCisco Systems, Inc\r
    62 DC:09:4C:32:34:9B                         \tHUAWEI TECHNOLOGIES CO.,LTD\r
    63 E8:28:C1:DC:F0:90                               \tEltex Enterprise Ltd.\r
    64 E0:D9:E3:49:04:52                               \tEltex Enterprise Ltd.\r
    65 00:03:7F:12:34:56                        \tAtheros Communications, Inc.\r
    66 E0:D9:E3:49:04:50                               \tEltex Enterprise Ltd.\r
    67 E8:28:C1:DC:03:30                               \tEltex Enterprise Ltd.\r
    68 E0:D9:E3:49:04:40                               \tEltex Enterprise Ltd.\r
    69 E0:D9:E3:49:00:B1                               \tEltex Enterprise Ltd.\r
    70 E8:28:C1:DC:BD:52                               \tEltex Enterprise Ltd.\r
    71 E8:28:C1:DC:54:B0                               \tEltex Enterprise Ltd.\r
    72 E0:D9:E3:49:00:B0                               \tEltex Enterprise Ltd.\r
    73 E8:28:C1:DC:33:10                               \tEltex Enterprise Ltd.\r
    74 E8:28:C1:DB:F5:F0                               \tEltex Enterprise Ltd.\r
    75 E8:28:C1:DE:72:D0                               \tEltex Enterprise Ltd.\r
    76 E0:D9:E3:49:04:41                               \tEltex Enterprise Ltd.\r
    77 38:1A:52:0D:8F:EC                             \tSeiko Epson Corporation\r
    78 00:26:99:F1:1A:E1                                  \tCisco Systems, Inc\r
    79 38:1A:52:0D:90:5D                             \tSeiko Epson Corporation\r
    80 00:23:EB:E3:44:32                                  \tCisco Systems, Inc\r
    81 00:26:CB:AA:62:72                                  \tCisco Systems, Inc\r
    82 E0:D9:E3:48:B4:D2                               \tEltex Enterprise Ltd.\r
    83 00:00:00:00:00:00                                   \tXEROX CORPORATION\r
    84 E8:28:C1:DC:3C:90                               \tEltex Enterprise Ltd.\r
    85 30:B4:B8:11:C0:90                                      \tLG Electronics\r
    86 00:26:99:F2:7A:EF                                  \tCisco Systems, Inc\r
    87 E8:28:C1:DC:0B:B0                               \tEltex Enterprise Ltd.\r
    88 E8:28:C1:DC:03:32                               \tEltex Enterprise Ltd.\r
    89 E8:28:C1:DC:54:B2                               \tEltex Enterprise Ltd.\r
    90 00:03:7F:10:17:56                        \tAtheros Communications, Inc.\r
    91 00:0D:97:6B:93:DF                             \tHitachi Energy USA Inc.\r
    92 9C:A5:13:28:D5:89                         \tSamsung Electronics Co.,Ltd\r
    93 E8:28:C1:DE:47:D0                               \tEltex Enterprise Ltd.\r

#### 3. Выявить устройства, использующие последнюю версию протокола шифрования WPA3, и названия точек доступа, реализованных на этих устройствах?

``` r
wpa3_aps <- wifi_ap_clean %>%
  filter(grepl("WPA3", privacy, ignore.case = TRUE) | grepl("SAE", auth, ignore.case = TRUE)) %>%
  select(bssid, essid, auth)

wpa3_aps
```

    # A tibble: 8 × 3
      bssid             essid                                          auth   
      <chr>             <chr>                                          <chr>  
    1 26:20:53:0C:98:E8  <NA>                                          SAE PSK
    2 A2:FE:FF:B8:9B:C9 "Christie’s"                                   SAE PSK
    3 96:FF:FC:91:EF:64  <NA>                                          SAE PSK
    4 CE:48:E7:86:4E:33 "iPhone (Анастасия)"                           SAE PSK
    5 8E:1F:94:96:DA:FD "iPhone (Анастасия)"                           SAE PSK
    6 BE:FD:EF:18:92:44 "Димасик"                                      SAE PSK
    7 3A:DA:00:F9:0C:02 "iPhone XS Max \U0001f98a\U0001f431\U0001f98a" SAE PSK
    8 76:C5:A0:70:08:96  <NA>                                          SAE PSK

#### 4. Отсортировать точки доступа по интервалу времени, в течение которого они находились на связи, по убыванию

``` r
wifi_ap_clean <- wifi_ap_clean %>%
  mutate(duration_sec = as.numeric(difftime(last_seen, first_seen, units = "secs")))


sorted_ap <- wifi_ap_clean %>%
  arrange(desc(duration_sec))


result <- sorted_ap %>%
  mutate(duration = paste(
    floor(duration_sec/3600), "h",
    floor((duration_sec %% 3600)/60), "m",
    round(duration_sec %% 60), "s"
  )) %>%
  select(bssid, essid, first_seen, last_seen, duration)

print(result)
```

    # A tibble: 167 × 5
       bssid             essid      first_seen          last_seen           duration
       <chr>             <chr>      <dttm>              <dttm>              <chr>   
     1 00:25:00:FF:94:73 <NA>       2023-07-28 09:13:06 2023-07-28 11:56:21 2 h 43 …
     2 E8:28:C1:DD:04:52 MIREA_HOT… 2023-07-28 09:13:09 2023-07-28 11:56:05 2 h 42 …
     3 E8:28:C1:DC:B2:52 MIREA_HOT… 2023-07-28 09:13:03 2023-07-28 11:55:38 2 h 42 …
     4 08:3A:2F:56:35:FE <NA>       2023-07-28 09:13:27 2023-07-28 11:55:53 2 h 42 …
     5 6E:C7:EC:16:DA:1A Cnet       2023-07-28 09:13:03 2023-07-28 11:55:12 2 h 42 …
     6 E8:28:C1:DC:B2:50 MIREA_GUE… 2023-07-28 09:13:06 2023-07-28 11:55:12 2 h 42 …
     7 E8:28:C1:DC:B2:51 <NA>       2023-07-28 09:13:06 2023-07-28 11:55:11 2 h 42 …
     8 48:5B:39:F9:7A:48 <NA>       2023-07-28 09:13:06 2023-07-28 11:55:11 2 h 42 …
     9 E8:28:C1:DC:FF:F2 <NA>       2023-07-28 09:13:06 2023-07-28 11:55:10 2 h 42 …
    10 8E:55:4A:85:5B:01 Vladimir   2023-07-28 09:13:06 2023-07-28 11:55:09 2 h 42 …
    # ℹ 157 more rows

#### 5. Обнаружить топ-10 самых быстрых точек доступа

``` r
top_fastest <- wifi_ap_clean %>%
  arrange(desc(speed)) %>%
  head(10) %>%
  select(bssid, essid, speed, channel, power)

print(top_fastest)
```

    # A tibble: 10 × 5
       bssid             essid              speed channel power
       <chr>             <chr>              <dbl>   <dbl> <dbl>
     1 26:20:53:0C:98:E8 <NA>                 866      44   -85
     2 96:FF:FC:91:EF:64 <NA>                 866      44   -85
     3 CE:48:E7:86:4E:33 iPhone (Анастасия)   866      44   -65
     4 8E:1F:94:96:DA:FD iPhone (Анастасия)   866      44   -67
     5 9A:75:A8:B9:04:1E KC                   360       1   -68
     6 4A:EC:1E:DB:BF:95 POCO X5 Pro 5G       360       7   -37
     7 56:C5:2B:9F:84:90 OnePlus 6T           360       1   -64
     8 E8:28:C1:DC:B2:41 MIREA_GUESTS         360      48   -89
     9 E8:28:C1:DC:B2:40 MIREA_HOTSPOT        360      48   -88
    10 E8:28:C1:DC:B2:42 <NA>                 360      48   -87

#### 6. Отсортировать точки доступа по частоте отправки запросов (beacons) в единицу времени по их убыванию.

``` r
sorted_rate <- wifi_ap_clean %>%
  mutate(
    duration_sec = as.numeric(difftime(last_seen, first_seen, units = "secs")),
    beacon_rate = ifelse(duration_sec > 0, beacons / duration_sec, 0)
  ) %>%
  arrange(desc(beacon_rate)) %>%
  select(bssid, essid, beacons, duration_sec, beacon_rate, first_seen, last_seen)

print(sorted_rate)
```

    # A tibble: 167 × 7
       bssid             essid  beacons duration_sec beacon_rate first_seen         
       <chr>             <chr>    <dbl>        <dbl>       <dbl> <dttm>             
     1 F2:30:AB:E9:03:ED "iPho…       6            7       0.857 2023-07-28 10:27:02
     2 B2:CF:C0:00:4A:60 "Миха…       4            5       0.8   2023-07-28 10:40:54
     3 3A:DA:00:F9:0C:02 "iPho…       5            9       0.556 2023-07-28 10:27:01
     4 02:BC:15:7E:D5:DC "MT_F…       1            2       0.5   2023-07-28 09:24:46
     5 00:3E:1A:5D:14:45 "MT_F…       1            2       0.5   2023-07-28 10:34:03
     6 76:C5:A0:70:08:96  <NA>        1            2       0.5   2023-07-28 11:16:36
     7 D2:25:91:F6:6C:D8 "Саня"       5           13       0.385 2023-07-28 09:45:29
     8 BE:F1:71:D6:10:D7 "C322…    1647         9461       0.174 2023-07-28 09:13:03
     9 00:03:7A:1A:03:56 "MT_F…       1            6       0.167 2023-07-28 10:29:13
    10 38:1A:52:0D:84:D7 "EBFC…     704         4319       0.163 2023-07-28 09:13:03
    # ℹ 157 more rows
    # ℹ 1 more variable: last_seen <dttm>

#### 1. Определить производителя для каждого обнаруженного устройства

``` r
wifi_clients_clean$oui <- toupper(substr(wifi_clients_clean$station_mac, 1, 8))
wifi_clients_clean$vendor <- oui_db$vendor[match(wifi_clients_clean$oui, oui_db$oui)]

result2 <- wifi_clients_clean %>%
  filter(!is.na(vendor)) %>%
  select(station_mac, vendor) %>%
  as.data.frame()

print(result2)
```

              station_mac
    1   5C:3A:45:9E:1A:7B
    2   C0:E4:34:D8:E7:E5
    3   10:51:07:CB:33:E7
    4   68:54:5A:40:35:9E
    5   74:4C:A1:70:CE:F7
    6   BC:F1:71:D4:DB:04
    7   4C:44:5B:14:76:E3
    8   A0:E7:0B:AE:D5:44
    9   00:95:69:E7:7F:35
    10  00:95:69:E7:7C:ED
    11  14:13:33:59:9F:AB
    12  10:51:07:FE:77:BB
    13  10:51:07:CB:33:BF
    14  BC:F1:71:D5:17:8B
    15  48:68:4A:93:DF:B4
    16  28:7F:CF:23:25:53
    17  00:95:69:E7:7D:21
    18  BC:F1:71:D5:0E:53
    19  8C:55:4A:DE:F2:38
    20  BC:F1:71:D6:10:D7
    21  88:D8:2E:4F:9B:1A
    22  34:7D:F6:95:25:E5
    23  BC:F1:71:D4:CC:7C
    24  BC:F1:71:D5:13:08
    25  BC:F1:71:D5:B3:5D
    26  BC:F1:71:D5:3F:C7
    27  10:51:07:FF:23:B4
    28  10:51:07:FE:77:C0
    29  34:E1:2D:3C:C8:2D
    30  BC:F1:71:D5:19:2A
    31  BC:F1:71:D5:11:DC
    32  BC:F1:71:D5:48:00
    33  B8:27:EB:59:FA:0E
    34  BC:F1:71:D5:0E:71
    35  50:3E:AA:33:F5:5B
    36  5C:10:C5:EE:AB:7F
    37  BC:F1:71:D5:47:B5
    38  10:51:07:FE:77:B6
    39  50:3E:AA:34:2F:10
    40  BC:F1:71:D5:12:7C
    41  BC:F1:71:D5:13:53
    42  8C:55:4A:85:5B:01
    43  9C:A3:A9:D6:28:3C
    44  10:51:07:FF:1E:CD
    45  AC:B5:7D:B4:92:05
    46  BC:F1:71:D5:13:85
    47  BC:F1:71:D5:E0:35
    48  8C:55:4A:DF:0D:A4
    49  10:51:07:CB:33:C9
    50  3C:13:5A:B5:DA:C0
    51  BC:F1:71:D5:3F:77
    52  AC:D1:B8:FE:25:87
    53  70:66:55:D0:B6:C7
    54  BC:F1:71:D5:E0:26
    55  3C:13:5A:2D:6A:B1
    56  BC:F1:71:D4:DA:C3
    57  BC:F1:71:D5:14:66
    58  10:51:07:FF:A4:A6
    59  BC:F1:71:D4:DA:C8
    60  BC:F1:71:D5:47:B0
    61  04:10:6B:9E:83:05
    62  1C:BF:C0:2D:7D:C9
    63  BC:F1:71:D5:47:C4
    64  BC:F1:71:D5:3F:E5
    65  BC:F1:71:D5:19:2F
    66  50:3E:AA:34:2F:0F
    67  E4:8D:8C:4D:06:A1
    68  10:51:07:FF:29:0E
    69  8C:55:4A:E2:25:ED
    70  00:90:4C:E6:54:54
    71  10:51:07:FF:1F:5E
    72  D4:54:8B:8F:56:E7
    73  BC:F1:71:D5:0D:22
    74  F0:67:28:61:26:6B
    75  EC:55:F9:A1:4C:6B
    76  20:F4:78:D2:34:FF
    77  C0:E7:BF:06:FC:9E
    78  A4:02:B9:77:C4:28
    79  1C:BF:C0:2B:EA:97
    80  A4:02:B9:73:2C:6F
    81  BC:F1:71:D5:47:C9
    82  78:2B:46:FF:46:0A
    83  48:74:12:82:D7:91
    84  38:1A:52:0D:97:60
    85  10:51:07:FE:77:B1
    86  58:6C:25:9E:61:DC
    87  8C:55:4A:DF:74:E2
    88  10:51:07:FF:1E:BE
    89  DC:CC:E6:7B:9B:FC
    90  BC:F1:71:D5:12:BD
    91  28:CD:C4:96:D0:E9
    92  48:A4:72:33:DC:C7
    93  1C:E6:1D:3B:A2:C6
    94  0C:E4:41:E8:C3:6E
    95  10:51:07:FF:1E:46
    96  50:98:39:4A:84:FE
    97  DC:16:B2:6C:D0:95
    98  10:51:07:FF:1E:69
    99  40:74:E0:99:49:EF
    100 50:3E:AA:5F:AB:23
    101 9C:A3:A9:DB:7E:01
    102 70:1A:B8:FC:DA:23
    103 4C:44:5B:36:96:C9
    104 74:DF:BF:7B:00:19
    105 00:04:35:22:4F:75
    106 A4:02:B9:73:2D:A0
    107 50:3E:AA:33:52:EC
    108 1C:BF:C0:2D:7D:B5
    109 50:DA:D6:D1:03:AD
    110 38:1A:52:0D:85:1D
    111 A4:02:B9:76:34:46
    112 70:1A:B8:DF:26:2C
    113 F8:32:E4:3F:7D:5C
    114 10:51:07:FF:1E:7D
    115 68:E7:C2:F0:C9:D9
    116 10:51:07:FF:1E:4B
    117 A4:02:B9:73:0B:FE
    118 BC:F1:71:D4:DA:91
    119 10:51:07:FF:1E:91
    120 50:8F:4C:71:25:96
    121 48:74:12:04:D7:91
    122 D4:54:8B:5E:35:C6
    123 98:F6:21:30:3D:31
    124 00:E9:3A:67:93:E9
    125 70:1A:B8:DF:26:9F
    126 00:E9:3A:F8:10:C7
    127 94:08:53:3E:7A:09
    128 A4:02:B9:73:2D:C8
    129 70:1A:B8:DE:20:1A
    130 00:0C:E7:A8:D6:73
    131 3C:13:5A:BD:93:F5
    132 EC:2E:98:51:03:9B
    133 88:BF:E4:63:61:43
    134 4C:E0:DB:0B:F8:C2
    135 48:2C:A0:74:B7:12
    136 54:71:DD:FE:12:68
    137 04:8C:9A:0B:40:EA
    138 BC:F1:71:4A:20:8B
    139 88:28:B3:54:FD:2B
    140 70:1A:B8:FC:CB:41
    141 E0:75:26:64:09:FE
    142 BC:F1:71:D4:D6:6D
    143 BC:F1:71:D4:DA:DC
    144 BC:F1:71:D4:DA:A0
    145 BC:F1:71:D6:10:CD
    146 BC:F1:71:D5:E0:3F
    147 C4:75:AB:E5:6E:EC
    148 D4:9C:DD:EA:B9:1E
    149 D0:7F:A0:A2:20:23
    150 BC:F1:71:D5:47:BA
    151 BC:F1:71:D5:47:D3
    152 E0:D4:E8:EF:87:EC
    153 C4:D0:E3:5B:B4:6D
    154 BC:F1:71:D5:47:FB
    155 F4:02:28:FD:36:F9
    156 04:33:85:71:86:B2
    157 50:8F:4C:4D:21:13
    158 BC:F1:71:D5:3F:6D
    159 BC:F1:71:D4:DB:09
    160 BC:F1:71:D5:47:CE
    161 BC:F1:71:D5:0D:45
    162 04:B1:A1:F4:18:DF
    163 C0:E4:34:FA:1D:1B
    164 B0:BE:83:81:8E:68
    165 D8:9B:3B:FA:6D:50
    166 A4:02:B9:73:81:60
    167 BC:F1:71:D5:13:58
    168 BC:F1:71:D4:DA:CD
    169 10:51:07:FF:A4:9C
    170 A4:02:B9:73:2D:A5
    171 BC:F1:71:D5:12:72
    172 A4:02:B9:73:83:18
    173 38:88:A4:58:C3:08
    174 A4:02:B9:72:9A:07
    175 BC:F1:71:D6:10:C8
    176 BC:F1:71:D5:12:8B
    177 BC:F1:71:D4:DA:D7
    178 BC:F1:71:D5:23:2F
    179 BC:F1:71:D5:13:5D
    180 38:1A:52:0D:90:A1
    181 A4:02:B9:73:2F:76
    182 98:F6:21:72:9E:D6
    183 A4:02:B9:73:81:47
    184 E4:0E:EE:E6:67:D1
    185 38:1A:52:0D:8F:EC
    186 E8:88:43:C8:37:EC
    187 A4:02:B9:73:2D:F0
    188 10:32:7E:8A:8C:D9
    189 A4:02:B9:73:81:24
    190 8C:55:4A:DF:86:94
    191 00:F4:8D:F7:C5:19
    192 10:51:07:FE:77:CA
    193 10:51:07:FF:42:68
    194 A4:02:B9:73:81:92
    195 50:ED:3C:3B:BC:A2
    196 10:51:07:FF:28:69
    197 10:51:07:FF:A4:97
    198 A4:02:B9:73:2C:83
    199 AC:5F:3E:2F:D7:8D
    200 8C:55:4A:84:F8:AF
    201 10:51:07:FF:29:D6
    202 A4:02:B9:73:81:7E
    203 28:C6:3F:F0:AC:D5
    204 10:51:07:FF:1E:78
    205 30:05:05:9A:53:30
    206 70:66:55:D0:B7:0F
    207 30:B1:B5:81:B1:B0
    208 50:C2:E8:0A:D9:5F
    209 04:BA:8D:AB:72:D7
    210 5C:51:81:37:EA:25
    211 F4:30:8B:D5:BF:32
    212 BC:F1:71:D4:E5:2C
    213 BC:7A:BF:0A:78:5E
    214 70:3A:51:3B:C2:53
                                                                       vendor
    1                                \tCHONGQING FUGUI ELECTRONICS CO.,LTD.\r
    2                                           \tAzureWave Technology Inc.\r
    3                                                     \tIntel Corporate\r
    4                                                     \tIntel Corporate\r
    5                                       \tLiteon Technology Corporation\r
    6                                                     \tIntel Corporate\r
    7                                                     \tIntel Corporate\r
    8                                                     \tIntel Corporate\r
    9                                 \tLSD Science and Technology Co.,Ltd.\r
    10                                \tLSD Science and Technology Co.,Ltd.\r
    11                                          \tAzureWave Technology Inc.\r
    12                                                    \tIntel Corporate\r
    13                                                    \tIntel Corporate\r
    14                                                    \tIntel Corporate\r
    15                                                    \tIntel Corporate\r
    16                                                    \tIntel Corporate\r
    17                                \tLSD Science and Technology Co.,Ltd.\r
    18                                                    \tIntel Corporate\r
    19                                                    \tIntel Corporate\r
    20                                                    \tIntel Corporate\r
    21                                                    \tIntel Corporate\r
    22                                                    \tIntel Corporate\r
    23                                                    \tIntel Corporate\r
    24                                                    \tIntel Corporate\r
    25                                                    \tIntel Corporate\r
    26                                                    \tIntel Corporate\r
    27                                                    \tIntel Corporate\r
    28                                                    \tIntel Corporate\r
    29                                                    \tIntel Corporate\r
    30                                                    \tIntel Corporate\r
    31                                                    \tIntel Corporate\r
    32                                                    \tIntel Corporate\r
    33                                            \tRaspberry Pi Foundation\r
    34                                                    \tIntel Corporate\r
    35                                      \tTP-LINK TECHNOLOGIES CO.,LTD.\r
    36                                        \tSamsung Electronics Co.,Ltd\r
    37                                                    \tIntel Corporate\r
    38                                                    \tIntel Corporate\r
    39                                      \tTP-LINK TECHNOLOGIES CO.,LTD.\r
    40                                                    \tIntel Corporate\r
    41                                                    \tIntel Corporate\r
    42                                                    \tIntel Corporate\r
    43  \tGuangzhou Juan Optical and Electronical Tech Joint Stock Co., Ltd\r
    44                                                    \tIntel Corporate\r
    45                                      \tLiteon Technology Corporation\r
    46                                                    \tIntel Corporate\r
    47                                                    \tIntel Corporate\r
    48                                                    \tIntel Corporate\r
    49                                                    \tIntel Corporate\r
    50                                       \tXiaomi Communications Co Ltd\r
    51                                                    \tIntel Corporate\r
    52                                    \tHon Hai Precision Ind. Co.,Ltd.\r
    53                                          \tAzureWave Technology Inc.\r
    54                                                    \tIntel Corporate\r
    55                                       \tXiaomi Communications Co Ltd\r
    56                                                    \tIntel Corporate\r
    57                                                    \tIntel Corporate\r
    58                                                    \tIntel Corporate\r
    59                                                    \tIntel Corporate\r
    60                                                    \tIntel Corporate\r
    61                                       \tXiaomi Communications Co Ltd\r
    62                               \tCHONGQING FUGUI ELECTRONICS CO.,LTD.\r
    63                                                    \tIntel Corporate\r
    64                                                    \tIntel Corporate\r
    65                                                    \tIntel Corporate\r
    66                                      \tTP-LINK TECHNOLOGIES CO.,LTD.\r
    67                                                    \tRouterboard.com\r
    68                                                    \tIntel Corporate\r
    69                                                    \tIntel Corporate\r
    70                                                      \tEpigram, Inc.\r
    71                                                    \tIntel Corporate\r
    72                                                    \tIntel Corporate\r
    73                                                    \tIntel Corporate\r
    74                 \tGUANGDONG OPPO MOBILE TELECOMMUNICATIONS CORP.,LTD\r
    75                                    \tHon Hai Precision Ind. Co.,Ltd.\r
    76                                       \tXiaomi Communications Co Ltd\r
    77                               \tSichuan AI-Link Technology Co., Ltd.\r
    78                                                    \tIntel Corporate\r
    79                               \tCHONGQING FUGUI ELECTRONICS CO.,LTD.\r
    80                                                    \tIntel Corporate\r
    81                                                    \tIntel Corporate\r
    82                                                    \tIntel Corporate\r
    83                             \tOnePlus Technology (Shenzhen) Co., Ltd\r
    84                                            \tSeiko Epson Corporation\r
    85                                                    \tIntel Corporate\r
    86                                                    \tIntel Corporate\r
    87                                                    \tIntel Corporate\r
    88                                                    \tIntel Corporate\r
    89                                        \tSamsung Electronics Co.,Ltd\r
    90                                                    \tIntel Corporate\r
    91                               \tCHONGQING FUGUI ELECTRONICS CO.,LTD.\r
    92                                                    \tIntel Corporate\r
    93                                        \tSamsung Electronics Co.,Ltd\r
    94                                                        \tApple, Inc.\r
    95                                                    \tIntel Corporate\r
    96                                       \tXiaomi Communications Co Ltd\r
    97                                        \tHUAWEI TECHNOLOGIES CO.,LTD\r
    98                                                    \tIntel Corporate\r
    99                                                    \tIntel Corporate\r
    100                                     \tTP-LINK TECHNOLOGIES CO.,LTD.\r
    101 \tGuangzhou Juan Optical and Electronical Tech Joint Stock Co., Ltd\r
    102                                                   \tIntel Corporate\r
    103                                                   \tIntel Corporate\r
    104                                     \tLiteon Technology Corporation\r
    105                                                       \tInfiNet LLC\r
    106                                                   \tIntel Corporate\r
    107                                     \tTP-LINK TECHNOLOGIES CO.,LTD.\r
    108                              \tCHONGQING FUGUI ELECTRONICS CO.,LTD.\r
    109                                      \tXiaomi Communications Co Ltd\r
    110                                           \tSeiko Epson Corporation\r
    111                                                   \tIntel Corporate\r
    112                                                   \tIntel Corporate\r
    113                                             \tASUSTek COMPUTER INC.\r
    114                                                   \tIntel Corporate\r
    115                                       \tSamsung Electronics Co.,Ltd\r
    116                                                   \tIntel Corporate\r
    117                                                   \tIntel Corporate\r
    118                                                   \tIntel Corporate\r
    119                                                   \tIntel Corporate\r
    120                                      \tXiaomi Communications Co Ltd\r
    121                            \tOnePlus Technology (Shenzhen) Co., Ltd\r
    122                                                   \tIntel Corporate\r
    123                                      \tXiaomi Communications Co Ltd\r
    124                                         \tAzureWave Technology Inc.\r
    125                                                   \tIntel Corporate\r
    126                                         \tAzureWave Technology Inc.\r
    127                                     \tLiteon Technology Corporation\r
    128                                                   \tIntel Corporate\r
    129                                                   \tIntel Corporate\r
    130                                                      \tMediaTek Inc\r
    131                                      \tXiaomi Communications Co Ltd\r
    132                                         \tAzureWave Technology Inc.\r
    133                                       \tHUAWEI TECHNOLOGIES CO.,LTD\r
    134                                      \tXiaomi Communications Co Ltd\r
    135                                      \tXiaomi Communications Co Ltd\r
    136                                           \tHuawei Device Co., Ltd.\r
    137                                           \tHuawei Device Co., Ltd.\r
    138                                                   \tIntel Corporate\r
    139                                       \tHUAWEI TECHNOLOGIES CO.,LTD\r
    140                                                   \tIntel Corporate\r
    141                                   \tChina Dragon Technology Limited\r
    142                                                   \tIntel Corporate\r
    143                                                   \tIntel Corporate\r
    144                                                   \tIntel Corporate\r
    145                                                   \tIntel Corporate\r
    146                                                   \tIntel Corporate\r
    147                                                   \tIntel Corporate\r
    148                                             \tAMPAK Technology,Inc.\r
    149                                       \tSamsung Electronics Co.,Ltd\r
    150                                                   \tIntel Corporate\r
    151                                                   \tIntel Corporate\r
    152                                                   \tIntel Corporate\r
    153                                                   \tIntel Corporate\r
    154                                                   \tIntel Corporate\r
    155                               \tSAMSUNG ELECTRO-MECHANICS(THAILAND)\r
    156                                      \tNanchang BlackShark Co.,Ltd.\r
    157                                      \tXiaomi Communications Co Ltd\r
    158                                                   \tIntel Corporate\r
    159                                                   \tIntel Corporate\r
    160                                                   \tIntel Corporate\r
    161                                                   \tIntel Corporate\r
    162                                       \tSamsung Electronics Co.,Ltd\r
    163                                         \tAzureWave Technology Inc.\r
    164                                                       \tApple, Inc.\r
    165                                       \tHUAWEI TECHNOLOGIES CO.,LTD\r
    166                                                   \tIntel Corporate\r
    167                                                   \tIntel Corporate\r
    168                                                   \tIntel Corporate\r
    169                                                   \tIntel Corporate\r
    170                                                   \tIntel Corporate\r
    171                                                   \tIntel Corporate\r
    172                                                   \tIntel Corporate\r
    173                                                       \tApple, Inc.\r
    174                                                   \tIntel Corporate\r
    175                                                   \tIntel Corporate\r
    176                                                   \tIntel Corporate\r
    177                                                   \tIntel Corporate\r
    178                                                   \tIntel Corporate\r
    179                                                   \tIntel Corporate\r
    180                                           \tSeiko Epson Corporation\r
    181                                                   \tIntel Corporate\r
    182                                      \tXiaomi Communications Co Ltd\r
    183                                                   \tIntel Corporate\r
    184                                       \tHUAWEI TECHNOLOGIES CO.,LTD\r
    185                                           \tSeiko Epson Corporation\r
    186                                      \tXiaomi Communications Co Ltd\r
    187                                                   \tIntel Corporate\r
    188                                           \tHuawei Device Co., Ltd.\r
    189                                                   \tIntel Corporate\r
    190                                                   \tIntel Corporate\r
    191                                     \tLiteon Technology Corporation\r
    192                                                   \tIntel Corporate\r
    193                                                   \tIntel Corporate\r
    194                                                   \tIntel Corporate\r
    195                                                       \tApple, Inc.\r
    196                                                   \tIntel Corporate\r
    197                                                   \tIntel Corporate\r
    198                                                   \tIntel Corporate\r
    199                               \tSAMSUNG ELECTRO-MECHANICS(THAILAND)\r
    200                                                   \tIntel Corporate\r
    201                                                   \tIntel Corporate\r
    202                                                   \tIntel Corporate\r
    203                                                   \tIntel Corporate\r
    204                                                   \tIntel Corporate\r
    205                                                   \tIntel Corporate\r
    206                                         \tAzureWave Technology Inc.\r
    207                                              \tArcadyan Corporation\r
    208                      \tCLOUD NETWORK TECHNOLOGY SINGAPORE PTE. LTD.\r
    209                                       \tSamsung Electronics Co.,Ltd\r
    210                                       \tSamsung Electronics Co.,Ltd\r
    211                                      \tXiaomi Communications Co Ltd\r
    212                                                   \tIntel Corporate\r
    213                                       \tSamsung Electronics Co.,Ltd\r
    214                                      \tXiaomi Communications Co Ltd\r

#### 2. Обнаружить устройства, которые НЕ рандомизируют свой MAC адрес

Нерандомизированный MAC — это глобально назначенный адрес (бит LAA = 0).
Проверяем 2 бита первого октета:

``` r
is_laa <- function(mac) {
  mac_hex <- toupper(gsub("[^0-9A-F]", "", mac))
  if (nchar(mac_hex) < 2) return(NA)
  
  b1 <- strtoi(substr(mac_hex, 1, 2), base = 16)
  if (is.na(b1)) return(NA)
  
  if (bitwAnd(b1, 0x01) != 0) return(NA)
  
  laa <- bitwAnd(b1, 0x02) != 0
  return(laa)
}

clients_nr <- wifi_clients_clean %>%
  mutate(is_randomized = vapply(station_mac, is_laa, logical(1))) %>%
  filter(is_randomized == FALSE) %>%
  select(station_mac, bssid, first_seen, last_seen, power, packets)

clients_nr
```

    # A tibble: 217 × 6
       station_mac       bssid first_seen          last_seen           power packets
       <chr>             <chr> <dttm>              <dttm>              <dbl>   <dbl>
     1 5C:3A:45:9E:1A:7B BE:F… 2023-07-28 09:13:03 2023-07-28 11:51:54   -39     432
     2 C0:E4:34:D8:E7:E5 BE:F… 2023-07-28 09:13:03 2023-07-28 11:53:16   -61     958
     3 10:51:07:CB:33:E7 <NA>  2023-07-28 09:13:05 2023-07-28 11:56:06   -43     344
     4 68:54:5A:40:35:9E 1E:9… 2023-07-28 09:13:06 2023-07-28 11:50:50   -31     163
     5 74:4C:A1:70:CE:F7 E8:2… 2023-07-28 09:13:06 2023-07-28 09:20:01   -71       3
     6 BC:F1:71:D4:DB:04 <NA>  2023-07-28 09:13:07 2023-07-28 10:57:52   -45     265
     7 4C:44:5B:14:76:E3 E8:2… 2023-07-28 09:13:09 2023-07-28 09:47:44    -1      71
     8 A0:E7:0B:AE:D5:44 0A:C… 2023-07-28 09:13:09 2023-07-28 11:34:42   -37     125
     9 00:95:69:E7:7F:35 <NA>  2023-07-28 09:13:11 2023-07-28 11:56:07   -69    2245
    10 00:95:69:E7:7C:ED <NA>  2023-07-28 09:13:11 2023-07-28 11:56:13   -55    4096
    # ℹ 207 more rows

#### Кластеризовать запросы от устройств к точкам доступа по их именам. Определить время появления устройства в зоне радиовидимости и время выхода его из нее

``` r
ap_essid <- wifi_ap_clean %>%
  transmute(bssid, essid_norm = na_if(str_squish(essid), ""))

device_essid_presence <- wifi_clients_clean %>%
  inner_join(ap_essid, by = "bssid") %>%  
  group_by(station_mac, essid_norm) %>% 
  summarise(
    first_seen   = min(first_seen),
    last_seen    = max(last_seen),
    duration_sec = as.numeric(last_seen - first_seen, units = "secs"),
    n_records    = n(),
    .groups = "drop"
  ) %>%
  arrange(desc(duration_sec), station_mac)

device_essid_presence
```

    # A tibble: 186 × 6
       station_mac   essid_norm first_seen          last_seen           duration_sec
       <chr>         <chr>      <dttm>              <dttm>                     <dbl>
     1 8C:55:4A:DE:… Galaxy A3… 2023-07-28 09:13:17 2023-07-28 11:56:16         9779
     2 70:66:55:D0:… <NA>       2023-07-28 09:14:09 2023-07-28 11:56:21         9732
     3 CA:54:C4:8B:… GIVC       2023-07-28 09:13:06 2023-07-28 11:55:04         9718
     4 9E:A3:A9:D6:… <NA>       2023-07-28 09:14:37 2023-07-28 11:55:53         9676
     5 F6:4D:98:98:… GIVC       2023-07-28 09:14:37 2023-07-28 11:55:29         9652
     6 C0:E4:34:D8:… C322U13 3… 2023-07-28 09:13:03 2023-07-28 11:53:16         9613
     7 5C:3A:45:9E:… C322U21 0… 2023-07-28 09:13:03 2023-07-28 11:51:54         9531
     8 9C:A3:A9:D6:… <NA>       2023-07-28 09:13:45 2023-07-28 11:52:30         9525
     9 28:7F:CF:23:… KC         2023-07-28 09:13:14 2023-07-28 11:51:50         9516
    10 34:E1:2D:3C:… Cnet       2023-07-28 09:13:29 2023-07-28 11:51:50         9501
    # ℹ 176 more rows
    # ℹ 1 more variable: n_records <int>

#### Оценить стабильность уровня сигнала внури кластера во времени. Выявить наиболее стабильный кластер.

``` r
client_clusters <- wifi_clients_clean %>%
  filter(!is.na(bssid)) %>%
  left_join(
    ap_essid,
    by = "bssid"
  ) %>%
  filter(!is.na(essid_norm), essid_norm != "") %>%
  group_by(station_mac, essid_norm)
stability <- client_clusters %>%
  summarise(
    n_obs = n(),
    span_m = as.numeric(difftime(max(last_seen), min(first_seen), units = "mins")),
    sd_rssi   = sd(power,   na.rm = TRUE),
    .groups   = "drop"
  ) %>%
  arrange(sd_rssi)

most_stable <- slice_head(stability, n = 1)

stability
```

    # A tibble: 99 × 5
       station_mac       essid_norm    n_obs  span_m sd_rssi
       <chr>             <chr>         <int>   <dbl>   <dbl>
     1 00:E9:3A:67:93:E9 POCO C40          1  99.9        NA
     2 00:F4:8D:F7:C5:19 Redmi 12          1  58.4        NA
     3 02:69:A5:29:F1:3E Galaxy A71        1  32.3        NA
     4 02:B3:4E:24:2A:00 Димасик           1   0          NA
     5 04:8C:9A:0B:40:EA MIREA_HOTSPOT     1  88.1        NA
     6 06:15:2E:12:C8:A6 MIREA_HOTSPOT     1   2.9        NA
     7 06:7A:BA:E6:9F:FA Vladimir          1  18.1        NA
     8 06:F2:A9:C1:8D:09 MIREA_HOTSPOT     1 139.         NA
     9 0A:AB:49:39:BB:29 MIREA_HOTSPOT     1  67.1        NA
    10 0A:C2:C3:08:9E:F8 MIREA_HOTSPOT     1   0.133      NA
    # ℹ 89 more rows

``` r
most_stable
```

    # A tibble: 1 × 5
      station_mac       essid_norm n_obs span_m sd_rssi
      <chr>             <chr>      <int>  <dbl>   <dbl>
    1 00:E9:3A:67:93:E9 POCO C40       1   99.9      NA

### Шаг 3

Отчёт написан и оформлен

## Оценка результатов

Задача решена с использованием языка программирования R и пакетами
`tidyverse`. Я научился использовать эти инструмента для анализа данных
полученных с wi-fi снифера.

## Вывод

В данной работе я используя программный пакет dplyr, освоить анализ
данных полученных с wi-fi снифера, с помощью языка программирования R.
