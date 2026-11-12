# server-check — проверка сервера по шкале Kaskad

[Русский](#русский) · [English](#english)

## Русский

Один файл: запускаете на сервере и видите, насколько он хорош — процессор, память, диск и сеть. Баллы 0–1000 по
группам считаются по шкале Kaskad (`lib/kaskad/score_abs.py`, версия `abs-0.5-k1`): те же кривые, опорные точки, веса
и средние по группам — кроме сетевого канала:
выше 1 Гбит/с — своя кривая bench, ниже оценки в панели Kaskad, до 1000 при 10 Гбит/с (шкала `abs-0.5-k1+net10g`,
ниже — «Канал»). Скорость сети — iperf3 до публичных точек в России, Европе и США (быстрая
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
| | `SC_NET_BUDGET_GB=N` | свой бюджет трафика iperf3, ГБ, — не меняется с портом; 0 — без предела. Без неё: 5, у `--full` — 30, у `--all` — 60 на порту 1 Гбит/с; на быстром порту больше — до 15 / 100 / 200 |
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
- **Трафик** iperf3 ограничен бюджетом (ниже, «Бюджет трафика»): на порту 1 Гбит/с быстрая проверка — до 5 ГБ,
  `--full` — до 30 ГБ, `--all` — до 60 ГБ (ГБ = 10⁹ байт); на быстром порту бюджет растёт с портом, но не выше 15 / 100
  / 200 ГБ. Speedtest by Ookla (`--full`, `--all`) — сверх бюджета, ≈ 2 ГБ на каждый Гбит/с порта (на 10 Гбит/с — ≈ 20
  ГБ). Замеры 12.11 на VPS в Москве (порт ≈ 10 Гбит/с), bench 1.2: быстрая проверка — iperf3 7,8 ГБ из 15, `--full` —
  42,0 ГБ из 100; Speedtest — 9,4 и 18,2 ГБ. В 1.1 (бюджет 5 / 30 / 60 ГБ на любом порту) — 4,9–5,0 / 28,7 / 57,5 ГБ,
  без бюджета быстрая проверка брала ≈ 10,7 ГБ. Перед сетью в отчёте — строка о трафике и о том, как его уменьшить;
  после пробы она называет порт и его бюджет. На сервере с лимитом трафика — `SC_NO_IPERF=1` (скорость — только
  скачиванием по HTTP), свой бюджет `SC_NET_BUDGET_GB=N` или `--no-net`.

### Скорость сети

Главная таблица — **iperf3**: на каждую точку скачивание (`-R`, точка отдаёт) и отдача, 8 параллельных потоков
(`-P 8`), 7 с, первые 2 из них не считаются (`-t 5 -O 2`; в быстрой проверке `-t 4 -O 1`), жёсткий предел времени на
каждый запуск. 5 с вместо прежних 6: 13 точек `--full` укладываются в ≈ 3 мин 40 с, а 8 потоков выходят на скорость
за 2 отброшенные секунды. Число — сумма получателя из `--json` (`end.sum_received`: при скачивании — своя, при отдаче —
точки).

**Быстрый порт (1.2).** Первое удачное скачивание таблицы (обычно ближняя Москва) — **проба**: 3 с, первая отброшена
(`-t 2 -O 1`), без предела `-b` (его держит только потолок режима, ниже). Проба — скорость до одной точки, а не порт: в
строке о трафике она так и названа, «проба ≈ 5,2 Гбит/с». До 1,2 Гбит/с — порт 1 Гбит/с, всё как раньше: строка тут же
меряется обычным запуском на том же порту (не вышел — в строке остаётся проба), бюджеты 5 / 30 / 60 ГБ. Быстрее — каждый
запуск таблицы тоже `-t 2 -O 1` (проба и есть скачивание первой строки), а бюджет растёт с **классом порта** — ступенью
2,5 / 10 / 25 / 40 / 100 Гбит/с не ниже пробы: бюджет режима × класс / 1 Гбит/с × 3 с / обычный запуск (5 с в быстрой, 7
с в `--full` и `--all`), с округлением вверх до целого ГБ, но не больше **потолка**: 15 ГБ в быстрой проверке, 100 в
`--full`, 200 в `--all`. Выходит: класс 2,5 Гбит/с (проба 1,2–2,5) — 8 / 33 / 65 ГБ, от 10 Гбит/с — потолок. Бюджет
режима рассчитан на его обычные запуски по 1 Гбит/с; запуски на быстром порту короче, поэтому «× 3 с / обычный запуск»
— тот же запас на секунду замера. Класс, а не сама проба: проба видит одну точку, порт бывает быстрее — 12.11 на стенде
Москва дала 3,5 Гбит/с, Ростов — 8,2, и бюджет по 3,5 (45 ГБ) едва не придержал последние строки `--full`. **Класс
только растёт:** запуск быстрее своего класса (у класса 1 Гбит/с — быстрее 1,2 Гбит/с) поднимает класс, а с ним бюджет
и шкалу; если этот запуск держал бюджет, он повторяется один раз уже с новым. Так проба 1,1 или 2,4 Гбит/с до медленной
точки не оставляет порт 9 Гбит/с на малом бюджете со строками «≥»; строка о трафике тогда — «порт ≥ 9,0 Гбит/с»:
столько дал запуск, поднявший класс. **Дальние точки** — пинг выше 150 мс (Лос-Анджелес, Азия) — и на быстром порту
меряются обычным запуском: TCP выходит на скорость за ≈ 10 обменов, при 250 мс это 2,5 с — весь короткий запуск; если
дальней оказалась и точка пробы (сервер далеко от Москвы), её строка меряется ещё раз обычным запуском. Скорость в
3-секундном запуске ограничивает и трафик: 3,75 ГБ на запуск при 10 Гбит/с. Свой `SC_NET_BUDGET_GB` не растёт (проба
тогда тоже в его пределах). Числа пробы, запусков и пинга идут в арифметику только как цифры (≤ 7 до точки); иначе
порт считается неизвестным и всё идёт как на 1 Гбит/с, а точка — не дальней.

2 с после отброшенной 1 с — потому что этого хватает: 12.11 на стенде (≈ 4–6 Гбит/с, 8 потоков; Москва, Франкфурт,
Красноярск, Нью-Йорк, в обе стороны) короткий запуск отличался от 5-секундного после 2 отброшенных на 0–14 %, медиана
6 %, в 7 случаях из 10 — не больше 10 %. Скачивание из Франкфурта не в счёт: там сам 5-секундный запуск просел с 6,2 до
3,3 Гбит/с посреди замера. Два 2-секундных запуска подряд расходятся на 3–7 %, и уже первая посчитанная секунда идёт на
полной скорости, даже до Нью-Йорка (≈ 120 мс). Дальше — нет: 13.11 там же короткий запуск до Лос-Анджелеса (176 мс)
был ниже 5-секундного на 4–15 %, до Сингапура (314 мс) — вдвое (первая посчитанная секунда ещё разгоняется), поэтому
точки с пингом выше 150 мс меряются обычным запуском.

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
получают первые строки: ближняя Москва, которая обычно и даёт «Канал», идёт первой, дальше — по остатку. Так шло в 1.1
на порту ≈ 10 Гбит/с (12.11): Москва до ≈ 2,9 Гбит/с в быстрой проверке, до ≈ 8,9 в `--full`, до ≈ 10,9 в `--all`;
сумма — 4,9–5,0 ГБ из 5, 28,5–28,7 из 30, 57,5 из 60, один прогон быстрой — 5,02 ГБ из 5 (скорее всего, такой повтор).
В 1.2 на таком порту бюджет растёт, и ни одна строка не придержана: быстрая — 7,8 ГБ из 15, `--full` — 42,0 ГБ из 100.
Строка, дошедшая до 90 % своего предела, — **≥** «не меньше», в легенде — «≥ — скорость держал бюджет трафика: iperf3 N
ГБ из M»: там всегда настоящий счёт. На канале до 1 Гбит/с предел ничего не меняет. Бюджет меньше, чем все запуски по 1
Гбит/с с пробой (быстрая ≈ 4,1 ГБ, `--full` ≈ 23,1, `--all` ≈ 51,1), не соблюдается: строки идут по 1 Гбит/с, и строка
о трафике говорит это сразу — «Сеть: iperf3 до ≈ 4,1 ГБ трафика (SC_NET_BUDGET_GB — не меньше ≈ 4,1 ГБ).» (после пробы
на быстром порту — меньше: запуски короче).
`SC_NET_BUDGET_GB=0` — без предела. С 1.2 на быстром порту бюджет растёт (выше), и `-b` с **≥** остаются последним
средством — когда не хватает потолка режима.

Полосы скорости — от нуля до **шкалы**: класс порта или лучший замер, что больше, округлённый вверх до 1 / 2,5 / 10 /
25 / 40 / 100 Гбит/с — одна шкала на все строки; она напечатана в строке «Сеть» («шкала — 10 Гбит/с»), там же — сколько
длятся запуски: «по 2 с», а если по-разному (дальние точки, строка до пробы) — «по 2–5 с». В журнале (труба, файл)
заголовок печатается, когда порт известен (после пробы), — строки до него ждут; если порт потом оказался быстрее и
запуски или шкала изменились, над следующей строкой — снова строка о трафике и заголовок: над каждой строкой — верный.
Полоса быстрее шкалы журнала (обычно Ookla) — полная, с **▸** на конце, число точное; в легенде — «▸ — быстрее шкалы
полос».

**Канал** (группа `net_bw` Kaskad) = среднее двух баллов с весами 1 и 1: `net_down` — лучшее скачивание из строк
iperf3 (опорные Kaskad 30 / 300 / 600 / 1500 Мбит/с → 100 / 500 / 700 / 900), `net_up` — лучшая отдача (20 / 200 /
500 / 1200 Мбит/с). Как в Kaskad: «значение — лучшая из попыток». Лучшая строка — «не меньше» — у числа стоит **≥**,
балл тогда тоже «не меньше». Ни одна точка iperf3 не ответила или iperf3 нет — скорость меряется прежним скачиванием по
HTTP (Cloudflare, Hetzner, OVH, Clouvider, Selectel, зеркало Яндекса), таблица так и подписана: «HTTP, только
скачивание»; отдача тогда не измерена.

**Шкала сети bench 1.2 — расширена выше 1 Гбит/с** (`abs-0.5-k1+net10g`: так она названа в JSON и в `--version`).
Кривая Kaskad после 1,5 Гбит/с почти упирается в 1000 (2,5 Гбит/с — 950, 5 — 981, 10 — 993): на оси сервер 5 и 10
Гбит/с не отличить. Поэтому до 1 Гбит/с балл — точно кривая Kaskad, а выше он растёт дальше прямо по логарифму
скорости — от балла 1 Гбит/с до 1000 при 10 Гбит/с (каждое удвоение — около +50 скачивание, +40 отдача); больше 1000 не
бывает, на экране — не больше 999. Значит, между 1 и 10 Гбит/с балл ниже, чем в панели Kaskad (2,5 Гбит/с — 898
вместо 950): быстрые серверы так различимы. В `SC_SCALE` это ключи `ext_from=1000 ext_top=10000` у `net_down` и
`net_up`; `parity.py` называет это единственным отличием и сверяет с `score_abs.py` всё остальное. Панель Kaskad
пока считает канал по своей кривой — выровнять её — отдельная задача. Быстрая проверка больше не держит
балл iperf3 на 700 и не ставит ему ≈: запуски iperf3 в ней те же, по времени (у Kaskad `quick_cap` — для его
короткого скачивания до 250 МБ); прежний предел 700 и ≈ остались только у скачивания по HTTP в быстрой проверке.

| Скорость | 0,3 Гбит/с | 0,6 | 1 | 1,5 | 2,5 | 5 | 10 | 25 |
|---|---|---|---|---|---|---|---|---|
| `net_down`, bench 1.2 | 500 | 700 | 832 | 861 | 898 | 949 | 999 | 999 |
| `net_down`, Kaskad | 500 | 700 | 832 | 900 | 950 | 981 | 993 | 998 |
| `net_up`, bench 1.2 | 592 | 755 | 871 | 894 | 922 | 961 | 999 | 999 |
| `net_up`, Kaskad | 592 | 755 | 871 | 926 | 965 | 987 | 995 | 998 |

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
   сброс), сетевой канал, задержка. Точка ● на оси — балл (шкала Kaskad), справа — балл и сырое значение. Нет значения —
   почему (например «нет fio»). **≈** — приближённо: встроенный способ или укороченный замер.
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

Перед сетью — **строка о трафике**: бюджет iperf3 («Сеть: iperf3 до 5 ГБ трафика при 1 Гбит/с, на быстром порту — до
15 ГБ.»), Speedtest и принятие его условий флагом, как меньше (без `--full` / `--all`, `SC_NO_IPERF=1`, `--no-net`).
В журнале она идёт сразу под паспортом, на живом экране — до конца замеров; на итоговом экране и с `--no-net` её нет.
Она же — в `--help`. После пробы её первая строка говорит, что вышло: «Сеть: проба ≈ 2,4 Гбит/с — iperf3 до 8 ГБ
трафика.» — это скорость до одной точки, не порт; если бюджет упёрся в потолок — «… до 15 ГБ (предел): быстрее —
«≥».»; если потом запуск оказался быстрее класса — «Сеть: порт ≥ 9,0 Гбит/с — iperf3 до 15 ГБ (предел): быстрее —
«≥».». На живом экране она меняется на месте (это всегда одна строка, высота кадра та же), в журнале печатается над
таблицей и снова — над строкой, после которой порт вырос. Свой `SC_NET_BUDGET_GB` — своя строка; ниже минимума замеров
— «Сеть: iperf3 до ≈ 4,1 ГБ трафика (SC_NET_BUDGET_GB — не меньше ≈ 4,1 ГБ).».

Числа: разряды — обычный пробел, дробь — запятая (в `/en` — точка); пинг и jitter до 10 мс — с десятыми, дальше —
целые; повторы от 1000 — «1,9к», от 10 000 — «12к». Журнал на стенде 12.11 (от паспорта до итога) — 33 строки в
быстрой проверке, 44 в `--full`, 61 в `--all`.

**Живой экран** — если вывод в терминал с цветом и кадр помещается (≥ 80 колонок, ≥ 42 строк): один экран заполняется
на месте — ждущие строки серые, у текущей точки спиннер, внизу полоса хода «0:52 из ~5:20» и «Сейчас …»; в конце на
его месте — итоговый экран. Высота кадра от начала до конца одна: легенда появляется только на итоговом экране, а
строка о трафике держит её место. Курсор прячется на время и возвращается и по Ctrl+C. Иначе — **журнал**: каждая
строка печатается один раз, когда готова: паспорт, строка о трафике, строки сети по мере точек (шкала и заголовок —
когда порт известен; вырос порт — заголовок снова), затем оценки (две из семи — по сети), легенда, итог. В трубе и файле
(и с `NO_COLOR`, `--no-color`, `TERM=dumb`) — ни одного ESC-кода; в маленьком терминале журнал цветной, а под ним —
строка хода «› …» на stderr, она стирается перед каждой новой строкой журнала. Цвет — по глубине терминала: truecolor
(`COLORTERM=truecolor`), 256 (`TERM=*256color*`) или 16; фон не красится. `--ascii` или локаль без UTF-8 — полосы `#`
и `.`, рисунок `_.-=+*#@`, ось `. o :`. От 100 колонок полосы шире (до 20 клеток), ось длиннее (до 60).

Шкала `abs-0.5-k1+net10g` (Kaskad `abs-0.5-k1`, у сетевого канала выше 1 Гбит/с — своя кривая), 0–1000, 1000 не
достигается (сетевой канал — 1000 при 10 Гбит/с, на экране 999); словесных
оценок нет. **Общего итога нет:** в Kaskad он
требует «Стабильности» за 72 ч. Остальные тесты групп (ChaCha20 и AES на ядро, все ядра — ChaCha20, кража времени,
объём памяти, последовательные запись и чтение диска, потери, DNS, баллы групп) — в JSON.

### Время

| | быстрая (без ключей) | `--full` | `--all` |
|---|---|---|---|
| всего, порт 1 Гбит/с | ≈ 1,5 мин | ≈ 5,5–6 мин | ≈ 10–11 мин |
| из них скорость: пинг точек, iperf3, Speedtest | ≈ 40 с (Speedtest нет) | ≈ 4 мин (9 с, 3 мин 40 с, 20–30 с) | ≈ 9 мин |
| всего, быстрый порт (1.2, запуски по 2 с, дальние точки — как обычно) | ≈ 1,2 мин | ≈ 4 мин | ≈ 6–7 мин (оценка) |
| трафик iperf3, предел: порт 1 Гбит/с / быстрый | 5 / 15 ГБ | 30 / 100 ГБ + Speedtest | 60 / 200 ГБ + Speedtest |

Замеры 12.11 на VPS в Москве (2 ядра, порт ≈ 10 Гбит/с), bench 1.2: быстрая — 1:12 и 1:13 (3 из 3 точек), `--full` —
3:44 и 3:52 (13 из 13); 13.11 после проверки (класс растёт, дальние точки по 5 с): быстрая — 1:12 и 68 с, iperf3 8,6 ГБ
из 15, `--full` — 3:48 (≈ 49 ГБ из 100 по строкам); `--all` в 1.2 не мерили. В 1.1 там же: быстрая — 1:23 и 1:26,
`--full` — 5:38 (13 из 13), `--all` — 10:05 (29 из 29). Раньше без бюджета: быстрая — 1:11 и 1:12, подробная (13 точек)
— 5:27 и 5:44, 29 точек — 10:33 (27 из 29: два порта ЭР-Телекома зависли, тогда без повтора) и 10:38 (29 из 29). До сети
(инструменты, процессор, память, диск, отклик и DNS) — ≈ 1,5 мин, в быстрой проверке — ≈ 40 с.

### JSON

`--json` печатает один объект: `scale` (`kaskad abs-0.5-k1+net10g`), `system` (без адресов), `groups`, `groups_approx`,
`tests` (`value`, `score`, `method`, `why`, `approx`, `n`), `extra`, `tools` (у sysbench, fio, iperf3, speedtest —
`source`: `system` / `apt` / `pinned` / `null` и `why`), **`speed`** — все строки таблицы: `id`, `method` (`iperf3` или
`http`), `city`, `city_ru`, `operator`, `operator_ru`, `region` (`ru`, `eu`, `us`, `asia`), `fallback` (строку дала
запасная точка), `host`, `port`, `down`, `up` (Мбит/с), `ping` (мс; медиана ICMP, иначе `min_rtt`), `ping_median`,
`status` (`ok`, `partial`, `busy`, `no_answer`, `error`, `time`), `down_status`, `up_status`, `down_at_least`,
`up_at_least`, `ping_avg`, `ping_jitter` (среднее |Δ| соседних откликов, мс), `ping_loss` (%), `ping_n`, `ping_samples`
(отклики по порядку, мс — для спарклайна), `min_rtt` (мс), `retr_down`, `retr_up` (повторные передачи TCP у отправителя
iperf3: при скачивании — у точки, при отдаче — свои; `null` — нет данных), `down_seconds`, `up_seconds` (посчитанные
секунды запуска, давшего число: 2 на быстром порту, у дальних точек — как обычно); **`traffic`** — `iperf3_bytes` (байты
iperf3 за проверку, с пробой), `speedtest_bytes` (байты Speedtest, `null` — не было), `budget_bytes` (действующий
бюджет; `null` — без предела), `budget_start_bytes` (до пробы: режима или свой), `budget_ceiling_bytes` (потолок режима
на быстром порту; `null` при своём бюджете), `budget_user` (бюджет задан `SC_NET_BUDGET_GB`), `probe_mbit` (проба —
скорость до одной точки, Мбит/с; `null` — не было), `port_mbit` (порт **не меньше** этого: наибольшее из пробы, запуска,
поднявшего класс, и строк без «≥», Мбит/с), `port_class_mbit`
(1000 / 2500 / 10000 / 25000 / 40000 / 100000 — шкала полос), `port_fast`, `run_seconds` и `omit_seconds` (запуски после
пробы), `probe_seconds`, `probe_omit_seconds`, `probe_bytes`, `far_ping_ms` (150: точки дальше меряются обычным
запуском), `floor_cap_mbit` (1000 — нижний предел), `held_runs`
(запусков, скорость которых держал бюджет); **`speedtest`** — `status`, `why`, `down`, `up`,
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
scale of Kaskad (`lib/kaskad/score_abs.py`, `abs-0.5-k1`): the same curves, anchors, weights and group means — but the
network channel:
above 1 Gbit/s — a curve of bench, lower than the Kaskad panel, to 1000 at 10 Gbit/s (the scale `abs-0.5-k1+net10g`,
«Channel» below). The network speed is
iperf3 to public points in Russia, Europe and the USA (the quick check 3 points, `--full` 13, `--all` 29): download,
upload, ping, jitter and TCP retransmits. No verdicts, no price, no overall total.

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
`SC_NO_IPERF=1`, `SC_NO_SPEEDTEST=1`, `SC_NET_BUDGET_GB=N` (your own iperf3 traffic budget, GB, kept whatever the
port; 0 — none; without it 5, 30 with `--full`, 60 with `--all` on a 1 Gbit/s port, more on a faster one, up to 15 /
100 / 200), `--ascii` / `SC_ASCII=1`, `--no-color` / `NO_COLOR=1`. Root or a plain user;
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
on a 1 Gbit/s port up to 5 GB for the quick check, 30 GB for `--full`, 60 GB for `--all` (GB = 10⁹ bytes); on a faster
port it grows with the port, at most 15 / 100 / 200 GB. Speedtest (`--full`, `--all`) is on top of it, ≈ 2 GB per
Gbit/s of the port (≈ 20 GB at 10 Gbit/s). Measured 12.11 on a VPS in Moscow (a ≈ 10 Gbit/s port), bench 1.2: the quick
check — iperf3 7.8 GB of 15, `--full` — 42.0 GB of 100; Speedtest 9.4 and 18.2 GB. In 1.1 (5 / 30 / 60 GB on any port) —
4.9–5.0 / 28.7 / 57.5 GB; with no budget the quick check took ≈ 10.7 GB. A line before the network says so and how to
use less; after the probe it names the port and its budget. A server with a traffic limit — `SC_NO_IPERF=1` (the speed
by HTTP download only), your own `SC_NET_BUDGET_GB=N` or `--no-net`.

### The network speed

iperf3 per point: download (`-R`) and upload, 8 parallel streams, `-t 5 -O 2` (quick `-t 4 -O 1`; 5 s, not 6: the 13
points of `--full` in ≈ 3 min 40 s), a hard time limit per run; the number is the receiver's sum of `--json`
(`end.sum_received`).

**A fast port (1.2).** The first download of the table that answers (usually the near Moscow) is **the probe**: 3 s,
the first omitted (`-t 2 -O 1`), no `-b` (only the ceiling of the mode holds it, below). The probe is the speed to one
point, not the port: the traffic line calls it so, «probe ≈ 5.2 Gbit/s». Up to 1.2 Gbit/s it is a 1 Gbit/s port and all
is as before: the row is measured right away by its usual run on the same port (if that fails, the probe stays), the
budgets 5 / 30 / 60 GB. Faster — every run of the table is `-t 2 -O 1` too (the probe is the download of the first
row), and the budget grows with the **class of the port** — the step of 2.5 / 10 / 25 / 40 / 100 Gbit/s not below the
probe: the budget of the mode × the class / 1 Gbit/s × 3 s / the usual run (5 s quick, 7 s `--full` and `--all`), whole
GB up, at most the **ceiling**: 15 GB quick, 100 `--full`, 200 `--all`. That is 8 / 33 / 65 GB for the class of 2.5
Gbit/s (a probe of 1.2–2.5), the ceiling from 10 Gbit/s on. The budget of a mode is for its usual runs at 1 Gbit/s;
runs on a fast port are shorter, so «× 3 s / the usual run» keeps the same room per second of measuring. The class, not
the probe itself: the probe sees one point and the port may be faster — 12.11 on the stand Moscow gave 3.5 Gbit/s,
Rostov 8.2, and a budget by 3.5 (45 GB) nearly held the last rows of `--full`. **The class only grows:** a run faster
than its class (above 1.2 Gbit/s for the class of 1 Gbit/s) raises the class, and the budget and the scale with it; a
run so held by the budget runs once more with the new one. So a probe of 1.1 or 2.4 Gbit/s to a slow point leaves no 9
Gbit/s port on a small budget with «≥» rows; the traffic line then says «port ≥ 9.0 Gbit/s» — what the run that
raised the class gave. **Far points** — a ping above 150 ms (Los Angeles, Asia) — keep the usual run on a fast port
too: TCP gets to speed in ≈ 10 round trips, 2.5 s at 250 ms — the whole of a short run; a far probe point (a server far
from Moscow) has its row run again at the usual length. The 3-s run bounds the traffic too:
3.75 GB a run at 10 Gbit/s. Your own `SC_NET_BUDGET_GB` does not grow (the probe keeps within it then). The numbers of
the probe, the runs and the ping enter arithmetic only as plain digits (≤ 7 before the point); anything else — the port
is unknown and all runs as on 1 Gbit/s, the point is not far.

Why 2 s after 1 omitted: it is enough. On the stand 12.11 (≈ 4–6 Gbit/s, 8 streams; Moscow, Frankfurt, Krasnoyarsk,
New York, both ways) the short run was within 0–14 % of the 5-s run after 2 omitted, median 6 %, 7 of 10 within 10 %.
The Frankfurt download is left out: the 5-s run itself fell from 6.2 to 3.3 Gbit/s mid-run. Two 2-s runs in a row
differ by 3–7 %, and the first counted second is at full speed already, New York (≈ 120 ms) too. Not farther: 13.11
there the short run to Los Angeles (176 ms) read 4–15 % below the 5-s one, to Singapore (314 ms) half of it (its first
counted second still ramps up) — so points with a ping above 150 ms keep the usual run.

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
most: the near Moscow, which usually gives «Channel», runs first, then what is left. So it went in 1.1 on a ≈ 10 Gbit/s
port (12.11): Moscow up to ≈ 2.9 Gbit/s in the quick check, ≈ 8.9 in `--full`, ≈ 10.9 in `--all`; the sum 4.9–5.0 GB
of 5, 28.5–28.7 of 30, 57.5 of 60, one quick run 5.02 GB of 5 (most likely such a retry). In 1.2 the budget grows on
such a port and no row was held: quick 7.8 GB of 15, `--full` 42.0 GB of 100. A row at 90 % of its cap is **≥** «not
less than»; the legend says «≥ — held by the traffic budget: iperf3 N GB of M» — always the real count. A channel up to
1 Gbit/s is not touched. A budget below all runs at 1 Gbit/s with the probe (quick ≈ 4.1 GB, `--full` ≈ 23.1, `--all` ≈
51.1) is not kept: the rows run at 1 Gbit/s, and the traffic line says so at once — «Network: iperf3 up to ≈ 4.1 GB
(SC_NET_BUDGET_GB — at least ≈ 4.1 GB).» (less after the probe on a fast port: shorter runs). `SC_NET_BUDGET_GB=0` — no
budget. Since 1.2 the budget grows on a fast port
(above): `-b` and **≥** are the last resort, when the ceiling of the mode is not enough.

The bars run from zero to the **scale**: the class of the port or the best measured, whichever is more, rounded up to
1 / 2.5 / 10 / 25 / 40 / 100 Gbit/s — one scale for every row, printed in the «Network» line («scale — 10 Gbit/s»)
with the length of the runs: «for 2 s», or «for 2–5 s» when they differ (far points, a row before the probe). The line
log prints this head once the port is known (after the probe; rows before it wait); when the port turns out faster and
the runs or the scale change, the traffic line and the head come again above the next row — every row has a true head
above it. A bar faster than the scale of the log (Ookla, usually) is full with **▸** at its end, the number exact; the
legend says «▸ — faster than the bar scale».

**Channel** (`net_bw` of Kaskad) = the mean of two scores, weights 1 and 1: `net_down` — the best iperf3 download
(the Kaskad anchors 30 / 300 / 600 / 1500 Mbit/s → 100 / 500 / 700 / 900), `net_up` — the best upload (20 / 200 / 500 /
1200); a best row «not less than» has **≥** at the number, its score is «not less» too. No iperf3 or no point
answered — the old HTTP download, labelled «HTTP, download only», no upload.

**The network scale of bench 1.2 — extended above 1 Gbit/s** (`abs-0.5-k1+net10g`, so named in the JSON and
`--version`). The Kaskad curve nearly reaches 1000 after 1.5 Gbit/s (2.5 Gbit/s — 950, 5 — 981, 10 — 993): servers of 5
and 10 Gbit/s look the same on the axis. So up to 1 Gbit/s the score is exactly the Kaskad curve, and above it goes on
straight in the log of the speed — from the score of 1 Gbit/s to 1000 at 10 Gbit/s (a doubling adds about
+50 the download, +40 the upload); never above 1000, at most 999 on the screen. Between 1 and 10 Gbit/s the score is so
lower than in the Kaskad panel (2.5 Gbit/s — 898, not 950): fast servers are told apart. In `SC_SCALE` these are
`ext_from=1000 ext_top=10000` of `net_down` and `net_up`; `parity.py` names it the one difference and checks
everything else against `score_abs.py`. The Kaskad panel still scores the channel by its own curve — aligning it is a
separate task. The quick check no longer holds the iperf3 score at 700 nor marks it ≈: its iperf3 runs are the same,
timed (the `quick_cap` of Kaskad is for its short download of at most 250 MB); the old cap of 700 and ≈ stay with the
HTTP download of the quick check only.

| Speed | 0.3 Gbit/s | 0.6 | 1 | 1.5 | 2.5 | 5 | 10 | 25 |
|---|---|---|---|---|---|---|---|---|
| `net_down`, bench 1.2 | 500 | 700 | 832 | 861 | 898 | 949 | 999 | 999 |
| `net_down`, Kaskad | 500 | 700 | 832 | 900 | 950 | 981 | 993 | 998 |
| `net_up`, bench 1.2 | 592 | 755 | 871 | 894 | 922 | 961 | 999 | 999 |
| `net_up`, Kaskad | 592 | 755 | 871 | 926 | 965 | 987 | 995 | 998 |

Speedtest by Ookla — `--full` and `--all` only: one line under the table (the nearest server by its choice), no
score; not shown when it fails; its traffic is on top of the iperf3 budget (`traffic.speedtest_bytes`).

### The screen

Passport (3 lines) → «Scores, 0–1000» on one axis, 7 rows (CPU 1 core / all cores, memory, disk 4K random / 4K +
flush, network channel, latency: the point ● is the Kaskad score; the score and the raw value on the right; **≈** — a
built-in method or a shortened run) → «Network» by regions Russia / Europe / USA (Asia in `--all`, with the count and
the medians) → Ookla (`--full`, `--all`; a long Ookla server name is cut with «…») → a 2-line legend (≥ of the budget
with the traffic: «iperf3 N GB of M») → «✓ Done, m:ss · network N of M · nothing installed»; in the quick check «Quick
check. In full: wget -qO- https://bench.anten-ka.com/en|bash -s -- --full» comes just before it. Names «city ·
operator» in the table are never cut. Before the network — **a traffic line**: the iperf3 budget («Network: iperf3 up
to 5 GB of traffic at 1 Gbit/s, up to 15 GB if faster.»), Speedtest and its terms accepted by flag, how to use less (no
`--full` / `--all`, `SC_NO_IPERF=1`, `--no-net`); in the line log under the passport, on the live screen until the end,
not on the final screen nor with `--no-net`; `--help` says the same. After the probe its first line says what came
out: «Network: probe ≈ 2.4 Gbit/s — iperf3 up to 8 GB of traffic.» — the speed to one point, not the port; at the
ceiling «… up to 15 GB (the limit): faster «≥».»; once a run beats the class — «Network: port ≥ 9.0 Gbit/s — iperf3 up
to 15 GB (the limit): faster «≥».» — in place on the live screen (always one line, the frame keeps its height), above
the table in the line log and again above the row after which the port grew. Your own `SC_NET_BUDGET_GB` has its own
line; below the minimum of the runs — «Network: iperf3 up to ≈ 4.1 GB (SC_NET_BUDGET_GB — at least ≈ 4.1 GB).».
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

On a 1 Gbit/s port: the quick check (no options) ≈ 1.5 min (the speed ≈ 40 s of it, no Speedtest), `--full` ≈
5.5–6 min (the speed ≈ 4 min: the ping of the points 9 s, iperf3 3 min 40 s, Speedtest 20–30 s), `--all` ≈ 10–11 min.
On a fast port (1.2, runs of 2 s): quick ≈ 1.2 min, `--full` ≈ 4 min, `--all` ≈ 6–7 min (an estimate); the iperf3
traffic at most 5 / 30 / 60 GB on a 1 Gbit/s port, 15 / 100 / 200 GB on a fast one, Speedtest on top. Measured 12.11
on a VPS in Moscow (2 cores, a ≈ 10 Gbit/s port), bench 1.2: quick 1:12 and 1:13 (3 of 3 points), `--full` 3:44 and
3:52 (13 of 13); 13.11 after the review (the class grows, far points 5 s): quick 1:12 and 68 s, iperf3 8.6 GB of 15,
`--full` 3:48 (≈ 49 GB of 100 by the rows); `--all` not measured in 1.2. In 1.1 there: quick 1:23 and 1:26, `--full`
5:38 (13 of 13), `--all` 10:05 (29 of 29). Before the budget: quick 1:11 and 1:12, 13 points 5:27 and 5:44, 29 points
10:33 (27 of 29: two ER-Telecom ports hung, no retry then) and 10:38 (29 of 29); before the network (tools, CPU, memory,
disk, latency and DNS) ≈ 1.5 min, in the quick check ≈ 40 s.

### JSON

`scale` (`kaskad abs-0.5-k1+net10g`), `system`, `groups`, `groups_approx`, `tests`, `extra`, `tools` (`source` system /
apt / pinned / null and `why`),
`speed` — every row of the table (`id`, `method`, `city`, `city_ru`, `operator`, `operator_ru`, `region`, `fallback`,
`host`, `port`, `down`, `ping_median`,
`up`, `ping`, `status`, `down_status`, `up_status`, `down_at_least`, `up_at_least`, `ping_avg`, `ping_jitter` — the
mean |Δ| of adjacent replies, `ping_loss` %, `ping_n`, `ping_samples` — the replies in order for a sparkline,
`min_rtt`, `retr_down` / `retr_up` — TCP retransmits of the iperf3 sender, `null` when unknown, `down_seconds` /
`up_seconds` — the counted seconds of the run that gave the number, 2 on a fast port, the usual at far points),
`traffic` (`iperf3_bytes` — the probe too, `speedtest_bytes` — `null` when it did not run, `budget_bytes` — the budget
in force, `null` with none, `budget_start_bytes` — before the probe (the mode's or your own), `budget_ceiling_bytes` —
the ceiling of the mode on a fast port (`null` with your own), `budget_user`, `probe_mbit` — the probe, the speed to one
point, `null` when none, `port_mbit` — the port is **at least** this: the most of the probe, the run that raised the
class and the rows not «≥», `port_class_mbit` — 1000 / 2500 / 10000 / 25000 / 40000 / 100000, the scale of the bars,
`port_fast`, `run_seconds` and `omit_seconds` — the runs after the probe, `probe_seconds`, `probe_omit_seconds`,
`probe_bytes`, `far_ping_ms` 150 — farther points run as usual, `floor_cap_mbit` 1000, `held_runs` — runs held
by the budget), `speedtest` (no addresses),
`download` (the HTTP tries when HTTP was used). `latency` is gone since 1.1.

### For developers

`bash tools/server-check/test.sh` — no measurements, no network (fine on a Mac). `bash tools/server-check/make-pages.sh
[dir]` — `index.html` (Russian) and `en/index.html` (one line apart: `SC_DEFAULT_LANG=en`) for `bench.anten-ka.com`.
A new iperf3 or Speedtest release — the URL and SHA-256 in `SC_PINS`; the points — `SC_IPERF_POINTS` (a «~» line is the
fallback of the row above). Collection and the screen are apart: `sc_collect` fills the data, `sc_screen final|live`
only reads it; `SC_ON_ROW` (a function of the script, not taken from the environment) gets each speed row as it is done.
