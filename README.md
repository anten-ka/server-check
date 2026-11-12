# server-check — проверка сервера по шкале Kaskad

[Русский](#русский) · [English](#english)

## Русский

Один файл: запускаете на сервере и видите, насколько он хорош — процессор, память, диск и сеть. Баллы 0–1000 по
группам считаются по той же шкале, что и в Kaskad (`lib/kaskad/score_abs.py`, версия `abs-0.5-k1`): те же кривые,
опорные точки, веса и средние по группам. Скорость сети — iperf3 до публичных точек в России, Европе и США (быстрая
проверка — 3 точки, `--full` — 13, `--all` — 29): скачивание и отдача, пинг, jitter и повторы TCP. Без выводов
«годится для…», без цены, тарифа и общего итога.

### Как запустить

```bash
wget -qO- https://bench.anten-ka.com|bash                    # по-русски, быстрая проверка
wget -qO- https://bench.anten-ka.com/en|bash                 # по-английски
curl -fsSL https://bench.anten-ka.com|bash -s -- --full      # подробно
bash server-check.sh [--lang ru|en] [--full|--all] [--json] [--no-net] [--ascii] [--no-color]
```

Адрес — всегда с `https://`: без него скрипт идёт по открытому HTTP, и его можно подменить по дороге.

| Параметр | Переменная | Что делает |
|---|---|---|
| (без ключей) | | быстрая проверка, ≈ 1,5 мин: iperf3 до Москвы, Франкфурта и Нью-Йорка, без Speedtest; тесты системы укорочены (≈) |
| `--full` | `SC_FULL=1` | подробно, ≈ 6 мин: 13 точек iperf3 (Россия, Европа, США), Speedtest by Ookla, тесты системы целиком |
| `--all` | `SC_ALL=1` | все 29 точек iperf3 (ещё города России, Европа, США, Азия) и Speedtest, ≈ 11 мин |
| `--quick` | `SC_QUICK=1` | то же, что без ключей (старые команды работают) |
| `--lang ru\|en` | `SC_LANG=ru\|en` | язык; по умолчанию русский, у `https://bench.anten-ka.com/en` — английский |
| `--json` | `SC_JSON=1` | ответ в JSON для программ |
| `--no-net` | `SC_NO_NET=1` | без замеров сети |
| | `SC_NO_FETCH=1` | ничего не скачивать: только то, что уже есть на сервере |
| | `SC_NO_IPERF=1` | без iperf3 (тогда скорость — скачиванием по HTTP) |
| | `SC_NO_SPEEDTEST=1` | без Speedtest by Ookla |
| | `SC_NET_BUDGET_GB=N` | бюджет трафика iperf3, ГБ: по умолчанию 5, у `--full` — 30, у `--all` — 60; 0 — без предела |
| `--ascii` | `SC_ASCII=1` | только ASCII: полосы `#` и `.`, рисунок jitter `_.-=+*#@` (так же при локали без UTF-8) |
| `--no-color` | `NO_COLOR=1` | без цвета |

Можно от root и от обычного пользователя. Нужны Linux и bash ≥ 4.2 (Ubuntu 20.04–24.04, Debian 11–12; ARM64 тоже).
Вопросов скрипт не задаёт. Весь код — функции, вызов — в последней строке: если загрузка оборвалась, ничего не
выполнится.

### Что ничего не ломает

- **Ничего не устанавливает в систему**: ни пакетов, ни `apt-get update`, ничего в `/etc`, `/usr`, `/usr/local` и в
  базе dpkg. Чего нет на сервере — скачивается только в свою временную папку (`mktemp -d` в `/var/tmp`, иначе в `/tmp`)
  и удаляется вместе с ней при выходе, Ctrl+C и `kill`:
  - **iperf3** — свой системный, если есть; иначе статическая сборка закреплённого выпуска
    (`github.com/userdocs/iperf3-static`, 3.22) для x86_64, ARM64, ARMv7 и i686, сверяется по SHA-256, не совпала —
    удаляется, не запускаясь;
  - **Speedtest by Ookla** — официальная программа 1.2.0 с `install.speedtest.net`, тоже по SHA-256;
  - **sysbench и fio** (точные способы Kaskad) — только на Debian/Ubuntu со списками пакетов apt: `apt-get download`
    пакета и его недостающих библиотек из подписанного репозитория системы, `dpkg-deb -x` во временную папку, запуск с
    `LD_LIBRARY_PATH` оттуда. Без блокировки apt, кэш и журналы apt — во временной папке или нигде; больше 50 МБ — не
    скачивается. Списков пакетов нет или они устарели — используются встроенные способы (≈), причина — в конце отчёта.
- Сетевые программы (iperf3, Speedtest) под root запускаются от `nobody` (`setpriv`).
- Тестовый файл диска — 1 ГиБ при свободных ≥ 3 ГиБ, иначе 256 МиБ при ≥ 1,25 ГиБ, иначе диск не проверяется. Без
  sysbench память читается из своей папки в `/dev/shm` (до 1 ГиБ, не больше ¼ доступной памяти), она удаляется сразу.
- IP сервера и имя хоста не печатаются и не попадают в JSON. Своей телеметрии нет. Точки iperf3 видят адрес сервера, как
  любой сервер скорости. **Speedtest by Ookla** запускается с `--accept-license --accept-gdpr` — это принятие условий
  Ookla (speedtest.net/about/eula, speedtest.net/about/privacy); результат замера Ookla сохраняет у себя. Об этом —
  строка в отчёте; отключить — `SC_NO_SPEEDTEST=1`.
- **Трафик** iperf3 ограничен бюджетом (ниже, «Бюджет трафика»): быстрая проверка — до 5 ГБ, `--full` — до 30 ГБ,
  `--all` — до 60 ГБ (ГБ = 10⁹ байт). Speedtest by Ookla (`--full`, `--all`) — сверх бюджета, ещё до ≈ 10 ГБ. Замеры
  12.11 на VPS в Москве (канал ≈ 10 Гбит/с): быстрая проверка — iperf3 4,9–5,0 ГБ, `--full` — 28,7 ГБ, `--all` — 57,5
  ГБ; Speedtest — ≈ 9,4 ГБ. Без бюджета быстрая проверка на нём же взяла ≈ 10,7 ГБ. Перед сетью в отчёте — строка о
  трафике и о том, как его уменьшить. На сервере с лимитом трафика — `SC_NO_IPERF=1` (скорость — только скачиванием по
  HTTP) или `--no-net`.

### Скорость сети

Главная таблица — **iperf3**: на каждую точку скачивание (`-R`, точка отдаёт) и отдача, 8 параллельных потоков
(`-P 8`), 7 с, первые 2 из них не считаются (`-t 5 -O 2`; в быстрой проверке `-t 4 -O 1`), жёсткий предел времени на
каждый запуск. 5 с вместо прежних 6: 13 точек `--full` укладываются в ≈ 3 мин 40 с, а 8 потоков выходят на скорость
за 2 отброшенные секунды. Число — сумма получателя из `--json` (`end.sum_received`: при скачивании — своя, при отдаче —
точки).

Пинг точки — **медиана 20 ICMP-эхо** (по 0,2 с; все точки разом перед таблицей, ≈ 4 с, не больше 8 с); без `ping` или
без ответа на ICMP (Hostkey его закрывает) — наименьший `min_rtt` своих TCP-потоков при отдаче. **Jitter** — среднее
|Δ| соседних ответов (как в RFC 3550). Рисунок jitter — до 8 столбиков: 20 ответов делятся на 8 равных частей, в
каждой берётся наибольший; высота — ⌊(ответ − наименьший ответ точки) / 2 мс⌋, 0…7. Шаг постоянный, ровный пинг
остаётся ровным. **Повт.** — повторные передачи TCP за скачивание и отдачу вместе (у отправителя iperf3: при
скачивании — у точки, при отдаче — свои).

8 потоков, а не 4: на пути 57 мс (Москва — Красноярск) один поток упирается в TCP-окно; 4 потока дали 2084 / 1446
Мбит/с, 8 — 3460 / 2828 (12.11). На коротких путях 8 потоков не мешают.

Публичный сервер iperf3 обслуживает одного клиента за раз. Занят — следующий порт из опубликованного оператором
диапазона (начало — случайный порт) через 1 с, до 4 попыток на направление (на экране — «занят · попытка 2 из 4»);
сбой (оборвался канал управления) или запуск, прерванный по пределу времени (порт точки завис на старте), — ещё одна
попытка на следующем порту; у точки с одним портом зависание не повторяют. Зависает обычно один порт: 12.11 у
ЭР-Телекома в Самаре 2 попытки из 4 не начались, на других портах всё прошло. Нет соединения дважды — точка не
отвечает. Если скачивание с точки не удалось (не отвечает, занята, сбой), строка пробует **запасную точку** того же
города (или ближайшего), и в строке — та, что ответила (в JSON — `fallback: true`); точку, которая уже есть в другой
строке, запасной не берут. На всю таблицу — 60 с в быстрой проверке, 260 с в `--full`, 600 с в `--all`. Строка не
пропадает: «занят · идёт чужой тест», «нет связи · нет ответа», «сбой», «нет времени».

| Набор | Точки (оператор; запасная) |
|---|---|
| быстрая проверка (3) | Москва (Hostkey; ЭР-Телеком), Франкфурт (Leaseweb; Clouvider), Нью-Йорк (Leaseweb; Ашберн, Clouvider) |
| `--full` (13) | **Россия**: Москва (Hostkey; ЭР-Телеком), Москва (МТС; ЭР-Телеком), СПб, Казань, Ростов, Красноярск (ЭР-Телеком / Дом.ру); **Европа**: Амстердам (Eranium; Leaseweb), Франкфурт (Leaseweb; Clouvider), Хельсинки (Hostkey, порт 1 Гбит/с; Стокгольм, Fegis), Варшава (Hostkey), Лондон (Leaseweb; Hostkey); **США**: Нью-Йорк (Leaseweb; Ашберн, Clouvider), Лос-Анджелес (Leaseweb; Clouvider) |
| `--all` (29) | `--full` + Москва (ЭР-Телеком), Калининград (Билайн), Нижний Новгород, Самара, Волгоград, Челябинск, Омск, Иркутск (ЭР-Телеком), Стокгольм (Fegis; Kamel), Париж (Moji; Hostkey), Чикаго, Даллас (Leaseweb), **Азия**: Стамбул (Hostkey), Ташкент (Uztelecom), Сингапур, Токио (Leaseweb) |

Все точки и запасные ответили iperf3 в обе стороны с VPS в Москве 12.11. Не взяты: Билайн Воронеж, Clouvider
Нью-Йорк и Лондон (отдача), Bouygues Париж (не отвечает из России), Leaseweb Вашингтон и Сан-Франциско (скачивание 2
Мбит/с), Казахтелеком Астана (запуски зависают), Scaleway Париж (занят, зависает), Leaseweb Гонконг (279 Мбит/с). МТС
`mskst.st.mtsws.net:3333` — один порт: в 1 из 5 попыток занят, тогда строка берёт ЭР-Телеком Москвы. У точки с
известным портом (Hostkey Хельсинки, 1 Гбит/с) результат от 90 % порта помечен **≥** — «не меньше»: сервер может быть
быстрее.

**Бюджет трафика.** Байты считаются по JSON самого iperf3 (все интервалы, и отброшенные первые секунды тоже). Перед
каждым запуском — предел `-b` (на поток — 1/8): запуск может взять остаток бюджета, если каждому следующему запуску
таблицы оставить по 1 Гбит/с; ниже 1 Гбит/с предела нет. 1 % бюджета — в запасе: `-b` держит запуск в пределах 0,2 %
от плана (замеры 12.11 до Москвы и Нью-Йорка), а оборванная и повторённая попытка не планировалась. Больше всего
получают первые строки: ближняя Москва, которая обычно и даёт «Канал», идёт первой — в быстрой проверке до ≈ 2,9
Гбит/с, в `--full` до ≈ 8,9, в `--all` до ≈ 10,9 Гбит/с, дальше — по остатку. Сумма на стенде 12.11 — 4,9–5,0 ГБ из 5,
28,5–28,7 из 30, 57,5 из 60; один прогон быстрой проверки — 5,02 ГБ из 5 (скорее всего, такой повтор). Строка,
дошедшая до 90 % своего предела, — **≥** «не меньше», в легенде — «≥ — скорость держал бюджет трафика: iperf3 N ГБ из
M»: там всегда настоящий счёт. На канале до 1 Гбит/с предел ничего не меняет. Бюджет меньше, чем все запуски по 1
Гбит/с (быстрая ≈ 3,75 ГБ, `--full` ≈ 22,75, `--all` ≈ 50,75), не соблюдается: строки идут по 1 Гбит/с.
`SC_NET_BUDGET_GB=0` — без предела.

Полосы скорости — от нуля до **шкалы**: max(1 Гбит/с, лучший замер, округлённый вверх до 1 / 2,5 / 10 / 25 / 100
Гбит/с); шкала напечатана в строке «Сеть». В журнале (труба, файл) её задаёт первая строка — до неё ничего не
напечатано; полоса быстрее шкалы просто полная, число точное.

**Канал** (группа `net_bw` Kaskad) = среднее двух баллов по шкале Kaskad с весами 1 и 1: `net_down` — лучшее
скачивание из строк iperf3 (опорные 30 / 300 / 600 / 1500 Мбит/с → 100 / 500 / 700 / 900), `net_up` — лучшая отдача
(20 / 200 / 500 / 1200 Мбит/с). Как в Kaskad: «значение — лучшая из попыток»; в быстрой проверке балл каждого не выше
700 (`quick_cap`). Лучшая строка — «не меньше» — у числа стоит **≥**. Ни одна точка iperf3 не ответила или iperf3 нет —
скорость меряется прежним скачиванием по HTTP (Cloudflare, Hetzner, OVH, Clouvider, Selectel, зеркало Яндекса), таблица
так и подписана: «HTTP, только скачивание»; отдача тогда не измерена.

**Speedtest by Ookla** — только в `--full` и `--all`: отдельная строка под таблицей (скачивание, отдача, пинг и jitter
до ближайшего сервера по выбору Ookla), без балла. Не скачался или не запустился — строки просто нет. Его трафик — сверх
бюджета iperf3 (в JSON — `traffic.speedtest_bytes`).

### Что ещё меряется

| Группа | Тесты (как в Kaskad) | Чем |
|---|---|---|
| **Процессор** | 1 ядро и все ядра — sysbench (простые числа до 10 000); 1 ядро — ChaCha20-Poly1305 и AES-128-GCM по 1420 байт (как шифрует туннель); все ядра — ChaCha20 через `openssl speed -multi`; кража времени (steal) во время тестов | sysbench (свой или скачанный), openssl |
| **Память** | объём; скорость в один поток: с sysbench — запись блоками 1 МиБ, как в Kaskad; без него ≈ чтение файла из `/dev/shm`. Встроенное число в балле, только если оно правдоподобно (меньше 60 ГБ/с на поток) и проходы сходятся | sysbench, иначе ≈ dd |
| **Диск** | случайные чтение и запись 4K с очередью 32 (IOPS); запись 4K со сбросом на диск (мс); последовательные запись и чтение 1 МиБ (МБ/с, без балла) | fio, иначе ≈ dd |
| **Отклик и DNS** | отклик до 1.1.1.1, 8.8.8.8, 77.88.8.8 (ping по 0,2 с; без ping — TCP); jitter и потери; DNS — 10 случайных имён под example.com через системный резолвер | ping, getent |

В паспорте: ОС, виртуализация, модель процессора, ядра, ОЗУ, swap, диск; AES-NI, TCP-алгоритм, WireGuard в ядре,
IPv4 и IPv6 (интернет отвечает). Полностью (ядро ОС, заголовки, `/dev/net/tun`, загрузка) — в JSON.

### Что на экране

Сверху вниз (вид — Claude Design r2, принят владельцем 12.11):

1. **Паспорт** — 3 строки: «Проверка сервера · bench.anten-ka.com» и дата; ОС, виртуализация, процессор ×ядра, ОЗУ,
   swap, диск; AES-NI, TCP-алгоритм, WireGuard, IPv4 / IPv6.
2. **Оценки, 0–1000** — одна общая ось, 7 строк: процессор (1 ядро, все ядра), память, диск (4K случайный, 4K +
   сброс), сетевой канал, задержка. Точка ● на оси — балл Kaskad, справа — балл и сырое значение. Нет значения — почему
   (например «нет fio»). **≈** — приближённо: встроенный способ или укороченный замер.
3. **Сеть** — таблица iperf3 по группам **Россия / Европа / США** (в `--all` ещё **Азия**, у группы — число точек и
   медианы колонок): ↓ скачивание и ↑ отдача — полоса и число (Мбит/с, от 10 000 — «10,2Г»), пинг, рисунок jitter и
   его число, повт. Состояния — словами. Название «город · оператор» не сокращается никогда: колонка = самое длинное
   название + 1 (не меньше 24); название длиннее 30 — отдельной строкой над своими цифрами.
4. Строка **Ookla Speedtest** — в `--full` и `--all`, если он запускался; название сервера Ookla длиннее строки
   обрезается «…».
5. **Легенда** — 2 строки (Jitter, Повт.); ≥ и ≈ — строкой, если они есть на экране (≥ бюджета — со счётом трафика:
   «iperf3 N ГБ из M»); что не измерено и почему.
6. **Итог** — «✓ Готово, m:ss · сеть N из M · ничего не установлено · bench.anten-ka.com». В быстрой проверке над ним —
   «Быстрая проверка. Подробно: wget -qO- https://bench.anten-ka.com|bash -s -- --full» (в 80 колонок — две строки).

Перед сетью — **строка о трафике**: бюджет iperf3, Speedtest и принятие его условий флагом, как меньше (без
`--full` / `--all`, `SC_NO_IPERF=1`, `--no-net`). В журнале она идёт сразу под паспортом, на живом экране — до конца
замеров; на итоговом экране и с `--no-net` её нет. Она же — в `--help`.

Числа: разряды — обычный пробел, дробь — запятая (в `/en` — точка); пинг и jitter до 10 мс — с десятыми, дальше —
целые; повторы от 1000 — «1,9к», от 10 000 — «12к». Журнал на стенде 12.11 (от паспорта до итога) — 33 строки в
быстрой проверке, 44 в `--full`, 61 в `--all`.

**Живой экран** — если вывод в терминал с цветом и кадр помещается (≥ 80 колонок, ≥ 42 строк): один экран заполняется
на месте — ждущие строки серые, у текущей точки спиннер, внизу полоса хода «0:52 из ~5:20» и «Сейчас …»; в конце на
его месте — итоговый экран. Высота кадра от начала до конца одна: легенда появляется только на итоговом экране, а
строка о трафике держит её место. Курсор прячется на время и возвращается и по Ctrl+C. Иначе — **журнал**: каждая
строка печатается один раз, когда готова: паспорт, строка о трафике, строки сети по мере точек (шкала и заголовок —
с первой строкой, у которой есть число), затем оценки (две из семи — по сети), легенда, итог. В трубе и файле (и с
`NO_COLOR`, `--no-color`, `TERM=dumb`) — ни одного ESC-кода; в маленьком терминале журнал цветной, а под ним — строка
хода «› …» на stderr, она стирается перед каждой новой строкой журнала. Цвет — по глубине терминала: truecolor
(`COLORTERM=truecolor`), 256 (`TERM=*256color*`) или 16; фон не красится. `--ascii` или локаль без UTF-8 — полосы `#`
и `.`, рисунок `_.-=+*#@`, ось `. o :`. От 100 колонок полосы шире (до 20 клеток), ось длиннее (до 60).

Шкала Kaskad `abs-0.5-k1`, 0–1000, 1000 не достигается; словесных оценок нет. **Общего итога нет:** в Kaskad он
требует «Стабильности» за 72 ч. Остальные тесты групп (ChaCha20 и AES на ядро, все ядра — ChaCha20, кража времени,
объём памяти, последовательные запись и чтение диска, потери, DNS, баллы групп) — в JSON.

### Время

| | быстрая (без ключей) | `--full` | `--all` |
|---|---|---|---|
| всего | ≈ 1,5 мин | ≈ 5,5–6 мин | ≈ 10–11 мин |
| из них скорость: пинг точек, iperf3, Speedtest | ≈ 40 с (Speedtest нет) | ≈ 4 мин (9 с, 3 мин 40 с, 20–30 с) | ≈ 9 мин |
| трафик iperf3, предел | до 5 ГБ | до 30 ГБ + Speedtest | до 60 ГБ + Speedtest |

Замеры 12.11 на VPS в Москве (2 ядра): быстрая — 1:23 и 1:26 (3 из 3 точек), `--full` — 5:38 (13 из 13), `--all` —
10:05 (29 из 29). Раньше без бюджета: быстрая — 1:11 и 1:12, подробная (13 точек) — 5:27 и 5:44, 29 точек — 10:33 (27
из 29: два порта ЭР-Телекома зависли, тогда без повтора) и 10:38 (29 из 29). До сети (инструменты, процессор, память,
диск, отклик и DNS) — ≈ 1,5 мин, в быстрой проверке — ≈ 40 с.

### JSON

`--json` печатает один объект: `system` (без адресов), `groups`, `groups_approx`, `tests` (`value`, `score`, `method`,
`why`, `approx`, `n`), `extra`, `tools` (у sysbench, fio, iperf3, speedtest — `source`: `system` / `apt` / `pinned` /
`null` и `why`), **`speed`** — все строки таблицы: `id`, `method` (`iperf3` или `http`), `city`, `city_ru`, `operator`,
`operator_ru`, `region` (`ru`, `eu`, `us`, `asia`), `fallback` (строку дала запасная точка), `host`, `port`, `down`,
`up` (Мбит/с), `ping` (мс; медиана ICMP, иначе `min_rtt`), `ping_median`, `status` (`ok`, `partial`, `busy`,
`no_answer`, `error`, `time`), `down_status`, `up_status`, `down_at_least`, `up_at_least`, `ping_avg`, `ping_jitter`
(среднее |Δ| соседних откликов, мс), `ping_loss` (%), `ping_n`, `ping_samples` (отклики по порядку, мс — для
спарклайна), `min_rtt` (мс), `retr_down`, `retr_up` (повторные передачи TCP у отправителя iperf3: при скачивании — у
точки, при отдаче — свои; `null` — нет данных); **`traffic`** — `iperf3_bytes` (байты iperf3 за проверку),
`speedtest_bytes` (байты Speedtest, `null` — не было), `budget_bytes` (`null` — без предела), `floor_cap_mbit` (1000 —
нижний предел), `held_runs` (запусков, скорость которых держал бюджет); **`speedtest`** — `status`, `why`, `down`, `up`,
`ping`, `jitter`, `loss`, `server_id`, `name`, `city`, `country` (адресов нет); `download` — попытки HTTP, если
скачивание по HTTP было. С версии 1.1 нет `latency` (отклик по регионам: его заменил пинг строк iperf3).

### Для разработчиков

- `bash tools/server-check/test.sh` — проверки без замеров и без сети (на Mac можно): шкала и средние групп — с
  `score_abs.py`, разборщики — с `bench/parse.py` на фикстурах Kaskad, вывод iperf3 3.22 и Speedtest 1.2.0 — на своих
  фикстурах (`fixtures/iperf-*.json`, `fixtures/speedtest.jsonl`, адреса заменены), повторы при «занят», закреплённые
  загрузки и SHA-256, apt во временную папку (поддельные `apt-get` и `dpkg-deb`), отчёт на двух языках (≤ 80
  символов, в английском нет кириллицы), JSON, очистка, страницы, shellcheck.
- `bash tools/server-check/make-pages.sh [папка]` — файлы для `bench.anten-ka.com`: `index.html` (скрипт как есть,
  русский) и `en/index.html` (отличается одной строкой `SC_DEFAULT_LANG=en`). По умолчанию — в
  `tools/server-check/pages` (не в git).
- Новый выпуск iperf3 или Speedtest — поменять адрес и SHA-256 в `SC_PINS`; точки — в `SC_IPERF_POINTS`.
- `SC_BUILTIN=1` — не брать sysbench и fio: сравнить встроенные способы с точными на одном сервере.
- Сбор и вывод разделены: `sc_collect` заполняет данные, экран их только читает (`sc_screen final|live`). Строки
  скорости выводятся по одной через `SC_ON_ROW` — функцию скрипта, её вызывают с номером строки, как только строка
  готова (журнал — `sc_log_row`, живой экран — перерисовка кадра). Из окружения не берётся.

## English

One file: run it on a server and see how good it is — CPU, memory, disk and network. Scores 0–1000 per group use the
scale of Kaskad (`lib/kaskad/score_abs.py`, `abs-0.5-k1`): the same curves, anchors, weights and group means. The
network speed is iperf3 to public points in Russia, Europe and the USA (the quick check 3 points, `--full` 13, `--all`
29): download, upload, ping, jitter and TCP retransmits. No verdicts, no price, no overall total.

### How to run

```bash
wget -qO- https://bench.anten-ka.com/en|bash                 # in English, the quick check
wget -qO- https://bench.anten-ka.com|bash                    # in Russian
curl -fsSL https://bench.anten-ka.com/en|bash -s -- --full   # in full
bash server-check.sh [--lang ru|en] [--full|--all] [--json] [--no-net] [--ascii] [--no-color]
```

Always with `https://`: without it the script comes over plain HTTP and can be swapped on the way.

Modes: no options — the quick check (≈ 1.5 min: iperf3 to Moscow, Frankfurt and New York, no Speedtest, the system
tests shortened, ≈); `--full` / `SC_FULL=1` (≈ 6 min: 13 iperf3 points, Speedtest by Ookla, the system tests at
length); `--all` / `SC_ALL=1` (≈ 11 min: all 29 iperf3 points and Speedtest); `--quick` / `SC_QUICK=1` — the same as
no options (old commands still work). Options: `--lang` / `SC_LANG` (Russian by default, English at
`https://bench.anten-ka.com/en`), `--json` / `SC_JSON=1`, `--no-net` / `SC_NO_NET=1`, `SC_NO_FETCH=1` (fetch nothing),
`SC_NO_IPERF=1`, `SC_NO_SPEEDTEST=1`, `SC_NET_BUDGET_GB=N` (the iperf3 traffic budget, GB: 5 by default, 30 with
`--full`, 60 with `--all`; 0 — none), `--ascii` / `SC_ASCII=1`, `--no-color` / `NO_COLOR=1`. Root or a plain user;
Linux, bash ≥ 4.2.

### It changes nothing

Installs nothing into the system: no package, no `apt-get update`, nothing in `/etc`, `/usr`, `/usr/local` or the dpkg
database. What the server lacks goes into its own temporary directory only, removed with it on exit, Ctrl+C and kill:
iperf3 — the system one, else a static build of a pinned release (`userdocs/iperf3-static` 3.22) checked by SHA-256;
Speedtest by Ookla — the official CLI 1.2.0, checked the same way; sysbench and fio — on Debian/Ubuntu with apt
package lists only: `apt-get download` from the signed repository of the system, `dpkg-deb -x` into the temporary
directory, run with `LD_LIBRARY_PATH` from there (no apt lock, the cache and logs of apt in the temporary directory or
nowhere, at most 50 MB). No lists or stale ones — the built-in methods (≈), the reason at the end of the report.
Network tools run as `nobody` under root. The IP and host name of the server are never printed or put into the JSON.
**Speedtest by Ookla runs with `--accept-license --accept-gdpr`** — this accepts the terms of Ookla; Ookla keeps the
result. The report says so in one line; `SC_NO_SPEEDTEST=1` turns it off. **Traffic** of iperf3 has a budget (below):
up to 5 GB for the quick check, 30 GB for `--full`, 60 GB for `--all` (GB = 10⁹ bytes); Speedtest (`--full`, `--all`)
is on top of it, up to ≈ 10 GB more. Measured 12.11 on a VPS in Moscow (a ≈ 10 Gbit/s channel): the quick check —
iperf3 4.9–5.0 GB, `--full` — 28.7 GB, `--all` — 57.5 GB; Speedtest ≈ 9.4 GB; with no budget the quick check took ≈
10.7 GB there. A line before the network says so and how to use less. A server with a traffic limit — `SC_NO_IPERF=1`
(the speed by HTTP download only) or `--no-net`.

### The network speed

iperf3 per point: download (`-R`) and upload, 8 parallel streams, `-t 5 -O 2` (quick `-t 4 -O 1`; 5 s, not 6: the 13
points of `--full` in ≈ 3 min 40 s), a hard time limit per run; the number is the receiver's sum of `--json`
(`end.sum_received`).
The ping — the **median of 20 ICMP echoes** (0.2 s apart, all points at once before the table), else the least
`min_rtt` of our TCP streams in the upload (Hostkey drops ICMP). **Jitter** — the mean |Δ| of adjacent replies (as RFC
3550); its sparkline — up to 8 columns, the largest reply of each of 8 equal parts, height ⌊(reply − least reply) / 2
ms⌋, 0…7. **Retr.** — TCP retransmits of the iperf3 sender, download and upload together. 8 streams, not 4: on a 57 ms
path one stream is held by its TCP window (4 streams 2084 / 1446 Mbit/s, 8 — 3460 / 2828, 12.11).

A busy point — the next port of its published range after 1 s, up to 4 tries a direction («busy · try 2 of 4»); an
error or a run cut by the time limit (a port of the point hung at the start) — one more try on the next port (a point
of one port — none): usually one port hangs, 12.11 ER-Telecom Samara — 2 tries of 4 never started, the other ports
were fine. The download of a point gave nothing — the row tries its **fallback point** (same or nearest city) and
shows the one that answered (`fallback: true`); a point already shown by another row is never a fallback. The table
has 60 s in the quick check, 260 s in `--full`, 600 s in `--all`. A row is never dropped: «busy · another test», «no
link · no answer», «failed», «out of time».

The sets — the quick check (3): Moscow (Hostkey; ER-Telecom), Frankfurt (Leaseweb; Clouvider), New York (Leaseweb;
Ashburn, Clouvider). `--full` (13): **Russia** Moscow (Hostkey; ER-Telecom), Moscow (MTS; ER-Telecom), St Petersburg,
Kazan, Rostov-on-Don, Krasnoyarsk (ER-Telecom / Dom.ru); **Europe** Amsterdam (Eranium; Leaseweb), Frankfurt
(Leaseweb; Clouvider), Helsinki (Hostkey, a 1 Gbit/s port — 90 % of it and more is **≥** «not less than»; Stockholm,
Fegis), Warsaw (Hostkey), London (Leaseweb; Hostkey); **USA** New York (Leaseweb; Ashburn, Clouvider), Los Angeles
(Leaseweb; Clouvider). `--all` (29) — `--full` plus Moscow (ER-Telecom), Kaliningrad (Beeline), Nizhny Novgorod,
Samara, Volgograd, Chelyabinsk, Omsk, Irkutsk, Stockholm (Fegis; Kamel), Paris (Moji; Hostkey), Chicago, Dallas
(Leaseweb), **Asia** Istanbul (Hostkey), Tashkent (Uztelecom), Singapore, Tokyo (Leaseweb). All answered both ways
from a VPS in Moscow 12.11; left out: Beeline Voronezh, Clouvider New York and London, Bouygues Paris, Leaseweb
Washington and San Francisco, Kazakhtelecom Astana, Scaleway Paris, Leaseweb Hong Kong.

**The traffic budget.** Bytes are counted from the JSON of iperf3 itself (every interval, the omitted first seconds
too). Each run gets a `-b` cap (1/8 a stream): it may take what is left of the budget once every later run of the
table keeps 1 Gbit/s; no cap is below 1 Gbit/s. 1 % of the budget is kept back: `-b` holds a run within 0.2 % of its
plan (measured 12.11 to Moscow and New York), a try cut short and tried again is not planned. The first rows get the
most: the near Moscow, which usually gives «Channel», runs first — up to ≈ 2.9 Gbit/s in the quick check, ≈ 8.9 in
`--full`, ≈ 10.9 in `--all`, then what is left. The sum on the stand 12.11: 4.9–5.0 GB of 5, 28.5–28.7 of 30, 57.5 of
60; one quick run — 5.02 GB of 5 (most likely such a retry). A row at 90 % of its cap is **≥** «not less than»; the
legend says «≥ — held by the traffic budget: iperf3 N GB of M» — always the real count. A channel up to 1 Gbit/s is
not touched. A budget below all runs at 1 Gbit/s (quick ≈ 3.75 GB, `--full` ≈ 22.75, `--all` ≈ 50.75) is not kept: the
rows run at 1 Gbit/s. `SC_NET_BUDGET_GB=0` — no budget.

The bars run from zero to the **scale**: max(1 Gbit/s, the best measured rounded up to 1 / 2.5 / 10 / 25 / 100 Gbit/s),
printed in the «Network» line (in the line log — set by the first row).

**Channel** (`net_bw` of Kaskad) = the mean of two Kaskad scores, weights 1 and 1: `net_down` — the best iperf3 download
(anchors 30 / 300 / 600 / 1500 Mbit/s → 100 / 500 / 700 / 900), `net_up` — the best upload (20 / 200 / 500 / 1200);
quick caps each at 700. No iperf3 or no point answered — the old HTTP download, labelled «HTTP, download only», no
upload. Speedtest by Ookla — `--full` and `--all` only: one line under the table (the nearest server by its choice), no
score; not shown when it fails; its traffic is on top of the iperf3 budget (`traffic.speedtest_bytes`).

### The screen

Passport (3 lines) → «Scores, 0–1000» on one axis, 7 rows (CPU 1 core / all cores, memory, disk 4K random / 4K +
flush, network channel, latency: the point ● is the Kaskad score; the score and the raw value on the right; **≈** — a
built-in method or a shortened run) → «Network» by regions Russia / Europe / USA (Asia in `--all`, with the count and
the medians) → Ookla (`--full`, `--all`; a long Ookla server name is cut with «…») → a 2-line legend (≥ of the budget
with the traffic: «iperf3 N GB of M») → «✓ Done, m:ss · network N of M · nothing installed»; in the quick check «Quick
check. In full: wget -qO- https://bench.anten-ka.com/en|bash -s -- --full» comes just before it. Names «city ·
operator» in the table are never cut. Before the network — **a traffic line**: the iperf3 budget, Speedtest and its
terms accepted by flag, how to use less (no `--full` / `--all`, `SC_NO_IPERF=1`, `--no-net`); in the line log under
the passport, on the live screen until the end, not on the final screen nor with `--no-net`; `--help` says the same.
The log on the stand 12.11 (passport to footer): 33 lines in the quick check, 44 in `--full`, 61 in `--all`.

On a terminal with colour where the frame fits (≥ 80 × 42) — a **live screen** filled in place (waiting rows grey, a
spinner on the current point, a progress bar and «Now»); the frame keeps one height to the end (the legend comes with
the final screen only, the traffic line holds its place). Otherwise — a **line log**, each line printed once when
ready: the passport, the traffic line, the network rows as they finish (the scale and the header come with the first
row that has a number), then the scores (two of the seven come from the network), the legend and the footer. A pipe or
a file (and `NO_COLOR`, `--no-color`, `TERM=dumb`) gets not a single escape code; a small terminal gets the log in
colour with a progress line «› …» on stderr under it, cleared before each new line of the log. Colours by the depth of
the terminal (truecolor / 256 / 16); `--ascii` or a non-UTF-8 locale — ASCII only; 100+ columns — wider bars and axis.

### Time

The quick check (no options) ≈ 1.5 min (the speed ≈ 40 s of it, no Speedtest), `--full` ≈ 5.5–6 min (the speed ≈ 4
min: the ping of the points 9 s, iperf3 3 min 40 s, Speedtest 20–30 s), `--all` ≈ 10–11 min. Measured 12.11 on a VPS
in Moscow (2 cores): quick 1:23 and 1:26 (3 of 3 points), `--full` 5:38 (13 of 13), `--all` 10:05 (29 of 29). Before
the budget: quick 1:11 and 1:12, 13 points 5:27 and 5:44, 29 points 10:33 (27 of 29: two ER-Telecom ports hung, no
retry then) and 10:38 (29 of 29); before the network (tools, CPU, memory, disk, latency and DNS) ≈ 1.5 min, in the
quick check ≈ 40 s.

### JSON

`system`, `groups`, `groups_approx`, `tests`, `extra`, `tools` (`source` system / apt / pinned / null and `why`),
`speed` — every row of the table (`id`, `method`, `city`, `city_ru`, `operator`, `operator_ru`, `region`, `fallback`,
`host`, `port`, `down`, `ping_median`,
`up`, `ping`, `status`, `down_status`, `up_status`, `down_at_least`, `up_at_least`, `ping_avg`, `ping_jitter` — the
mean |Δ| of adjacent replies, `ping_loss` %, `ping_n`, `ping_samples` — the replies in order for a sparkline,
`min_rtt`, `retr_down` / `retr_up` — TCP retransmits of the iperf3 sender, `null` when unknown), `traffic`
(`iperf3_bytes`, `speedtest_bytes` — `null` when it did not run, `budget_bytes` — `null` with no budget,
`floor_cap_mbit` 1000, `held_runs` — runs held by the budget), `speedtest` (no addresses),
`download` (the HTTP tries when HTTP was used). `latency` is gone since 1.1.

### For developers

`bash tools/server-check/test.sh` — no measurements, no network (fine on a Mac). `bash tools/server-check/make-pages.sh
[dir]` — `index.html` (Russian) and `en/index.html` (one line apart: `SC_DEFAULT_LANG=en`) for `bench.anten-ka.com`.
A new iperf3 or Speedtest release — the URL and SHA-256 in `SC_PINS`; the points — `SC_IPERF_POINTS` (a «~» line is the
fallback of the row above). Collection and the screen are apart: `sc_collect` fills the data, `sc_screen final|live`
only reads it; `SC_ON_ROW` (a function of the script, not taken from the environment) gets each speed row as it is done.
