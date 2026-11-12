# server-check — проверка сервера по шкале Kaskad

[Русский](#русский) · [English](#english)

## Русский

Один файл: запускаете на сервере и видите, насколько он хорош — процессор, память, диск, сеть и система. Баллы 0–1000
по группам считаются по той же шкале, что и в Kaskad (`lib/kaskad/score_abs.py`, версия `abs-0.5-k1`): те же кривые,
опорные точки, веса и средние по группам. Без выводов «годится для…», без цены, тарифа и общего итога.

### Как запустить

```bash
curl -fsSL bench.anten-ka.com | bash
wget -qO- bench.anten-ka.com | bash
curl -fsSL bench.anten-ka.com | bash -s -- --quick --lang ru     # с параметрами
bash server-check.sh [--lang ru|en] [--quick] [--json] [--no-net] [--ascii] [--no-color]
```

| Параметр | Переменная | Что делает |
|---|---|---|
| `--lang ru\|en` | `SC_LANG=ru\|en` | язык отчёта; по умолчанию — из `LANG` (`ru*` → русский) |
| `--quick` | `SC_QUICK=1` | быстрая проверка, ≈ 1 мин |
| `--json` | `SC_JSON=1` | ответ в JSON для программ |
| `--no-net` | `SC_NO_NET=1` | без замеров сети |
| `--ascii` | `SC_ASCII=1` | полосы символами `#` и `.` |
| `--no-color` | `NO_COLOR=1` | без цвета |

Можно от root и от обычного пользователя. Нужны Linux и bash ≥ 4.2 (Ubuntu 20.04–24.04, Debian 11–12; ARM64 тоже).
Вопросов скрипт не задаёт: при запуске через `| bash` его вход — сам скрипт. Весь код — функции, вызов — в последней
строке: если загрузка оборвалась, ничего не выполнится.

### Что ничего не ломает

- **Ничего не устанавливает** (нет apt) и не трогает службы и настройки.
- Пишет только в свою временную папку (`mktemp -d` в `/var/tmp`, иначе в `/tmp`; если там tmpfs — в домашней папке)
  и удаляет её при выходе, Ctrl+C и `kill`. Без sysbench ещё одна своя папка — в `/dev/shm` для проверки памяти
  (файл до 1 ГиБ, не больше ¼ доступной памяти и половины свободного `/dev/shm`); удаляется сразу после замера.
- Тестовый файл диска — 1 ГиБ при свободных ≥ 3 ГиБ, иначе 256 МиБ при ≥ 1,25 ГиБ, иначе диск не проверяется.
- Без телеметрии: только GET-запросы к публичным серверам замера скорости и TCP-подключения к ним без данных.
  IP сервера и имя хоста не печатаются и не попадают в JSON.
- Трафик: полная проверка — до ≈ 2,5 ГБ на быстром канале (1 ГБ ближайшая точка + по 500 МБ на регион; попытки
  нескольких точек региона делят его 500 МБ), быстрая — до 250 МБ.

### Что меряется

| Группа | Тесты (как в Kaskad) | Чем |
|---|---|---|
| **Процессор** | 1 ядро и все ядра — sysbench (простые числа до 10 000); 1 ядро — ChaCha20-Poly1305 и AES-128-GCM по 1420 байт (как шифрует туннель); все ядра — ChaCha20 через `openssl speed -multi`; кража времени (steal) во время тестов | sysbench, если есть; openssl |
| **Память** | объём (ГБ); скорость в один поток (МиБ/с): с sysbench — запись блоками 1 МиБ, как в Kaskad; без него ≈ чтение файла из `/dev/shm` (настоящие страницы памяти, не «нулевая страница» `/dev/zero`). Встроенное число в балле, только если оно правдоподобно (меньше 60 ГБ/с на поток) и проходы сходятся (разброс не больше 2×); иначе оно показано как ориентир без балла, а балл группы — только по объёму, с причиной | sysbench, если есть; иначе ≈ dd |
| **Диск** | случайное чтение и запись 4K с очередью 32 (IOPS); запись 4K со сбросом на диск после каждой (мс); последовательные запись и чтение 1 МиБ (МБ/с, без балла) | fio, если есть; иначе ≈ dd |
| **Канал** | скачивание в 4 потока, первые 2 с разгона не считаются; лучшая из точек (Мбит/с). Отдача не меряется: проверка только скачивает | curl или wget + dd |
| **Отклик и DNS** | отклик до 1.1.1.1, 8.8.8.8, 77.88.8.8 (ping по 0,2 с; без ping — TCP-рукопожатие); дрожание и потери — числом; DNS — 10 случайных имён под example.com через системный резолвер: имена с точкой на конце (иначе несуществующее имя переспрашивается с каждым доменом `search`), одно имя сначала не в счёт (холодный кэш), время запуска getent вычитается | ping, getent, bash `/dev/tcp` |
| **Система** | ОС, ядро, архитектура, виртуализация, модель и число ядер, AES/AVX2, память и подкачка, корневой диск, IPv4/IPv6 (маршрут и доступ), TCP-алгоритм, загрузка; для WireGuard/AmneziaWG — модуль в ядре, заголовки ядра (установлены / есть в apt), `/dev/net/tun`, контейнер ли это | только чтение |

Дополнительно, без балла: скачивание по регионам (ближайшая точка Cloudflare; Европа — Hetzner Фалькенштайн, OVH
Рубе, Clouvider Амстердам; Россия — Selectel, зеркало Яндекса; США — Hetzner Эшберн, Clouvider Нью-Йорк, OVH США) и
отклик по регионам (TCP-рукопожатие к точкам во Франкфурте, Амстердаме, Хельсинки, Стокгольме, Варшаве, у Яндекса и
Selectel, в Нью-Джерси и Лос-Анджелесе). Регион пробует свои точки по порядку и оставляет лучшую; следующая — только
если точка не ответила, к концу 2 с разгона дала меньше 128 КБ (страница ошибки или соединение, придержанное на
нескольких КБ) или меньше четверти лучшего результата проверки (файл ограничен на стороне точки). Не ответила ни
одна — «—». Точки Cloudflare, Selectel, Яндекса и OVH — из `lib/kaskad/bench/channel-targets.json`.

### Как читать баллы

Шкала Kaskad 0–1000, 1000 не достигается. Балл теста — логистическая кривая через четыре опорные точки шкалы
(100/500/700/900, у каждого теста свои значения), за крайними точками продолжается; словесных оценок нет.
Балл группы — среднее баллов её измеренных тестов по весам из `score_abs.py`. На экране сначала полоса группы, потом
числа. «Канал» и «Отклик и DNS» — это группы `net_bw` («Канал») и `net_q` («Сеть») Kaskad.

Знак **≈** — число получено встроенным способом в тех же единицах (чтение `/dev/shm` вместо sysbench, dd вместо fio, getent вместо своего
DNS-клиента, TCP вместо ping) или по выведенной шкале; балл группы с таким тестом тоже помечен ≈. Выведенная шкала одна: «все ядра — ChaCha20» стоит в балле
вместо sysbench «все ядра», когда sysbench нет. Её опорные точки (310 / 1900 / 3600 / 11000 МБ/с) получены правилом
калибровки K1 для `cpu_mt` (500 — эталон 1 vCPU, 700 — эталон 2 vCPU, 900 = 700 × 3, 100 = 500 / 6) из чисел ChaCha20
тех же эталонных серверов (`tests/fixtures/score_abs_calib_k1.json`); `test.sh` выводит их заново и сверяет.

**Общего итога нет:** в Kaskad он требует группы «Стабильность» (steal p95 за 72 ч, «канарейка»), а за минуты её не
измерить. Дрожание и потери показаны числом без балла: Kaskad оценивает их от 1000 и 10 000 замеров.

### Время

| | без sysbench и fio | с sysbench и fio |
|---|---|---|
| полная | ≈ 1–1,5 мин | ≈ 2–2,5 мин |
| `--quick` | ≈ 35–45 с | ≈ 55–60 с |

Медленный диск и недоступные точки добавляют время, но каждый шаг ограничен по времени (не дольше ≈ 3 мин всего):
точка, которая не отвечает, отнимает ≈ 2 с, медленная — до 6 с, в регионе не больше трёх точек.

### Что не проверяется

Задержка памяти (нужна скомпилированная программа), отдача (нужен POST), стабильность за сутки, потери пакетов под
нагрузкой, iperf3; случайный ввод-вывод диска без fio; Secure Boot для модуля AmneziaWG.

### JSON

`--json` печатает один объект: `system` (без адресов), `groups` (баллы групп), `groups_approx` (в группе есть ≈), `tests` (у каждого `value`, `score`,
`method`, `why` — почему нет значения или балла, `approx`, `n`), `extra` (масштабирование ядер, steal, подкачка,
последовательные скорости диска), `download` (по регионам: лучшая точка и `tries` — все попытки, `mbps` — `null`, если
точка не ответила) и `latency` по точкам. У `mem_bw` без балла `why` — `mem_implausible` (больше 60 ГБ/с на поток),
`mem_unstable` (проходы расходятся), `no_shm` или `mem_room`.

### Для разработчиков

- `bash tools/server-check/test.sh` — проверки без замеров (на Mac можно: скрипт там отказывается запускаться):
  шкала и средние групп сверяются с `score_abs.py` (≈ 1600 значений), разборщики — с `bench/parse.py` на фикстурах
  Kaskad (`tests/fixtures/bench`, в том числе настоящие выводы 4 систем), отчёт на двух языках (≤ 80 символов, в
  английском нет кириллицы), JSON, очистка, защита от обрыва загрузки, shellcheck.
- `SC_BUILTIN=1` — не брать sysbench и fio: чтобы на одном сервере сравнить встроенные способы с точными.
- Шкала меняется в `score_abs.py` → поменять таблицу `SC_SCALE` в `server-check.sh`; `test.sh` покажет расхождение.

## English

One file: run it on a server and see how good it is — CPU, memory, disk, network and the system. Scores 0–1000 per
group use the scale of Kaskad (`lib/kaskad/score_abs.py`, version `abs-0.5-k1`): the same curves, anchors, weights and
group means. No «good for…» verdicts, no price, tariff or overall total.

### How to run

```bash
curl -fsSL bench.anten-ka.com | bash
wget -qO- bench.anten-ka.com | bash
curl -fsSL bench.anten-ka.com | bash -s -- --quick --lang en     # with options
bash server-check.sh [--lang ru|en] [--quick] [--json] [--no-net] [--ascii] [--no-color]
```

Options and their environment variables: `--lang` / `SC_LANG`, `--quick` / `SC_QUICK=1` (≈ 1 min), `--json` /
`SC_JSON=1`, `--no-net` / `SC_NO_NET=1`, `--ascii` / `SC_ASCII=1`, `--no-color` / `NO_COLOR=1`. The language defaults
to `LANG` (`ru*` → Russian, otherwise English). Root or a plain user; Linux with bash ≥ 4.2 (Ubuntu 20.04–24.04,
Debian 11–12, ARM64 too). It asks nothing (piped into bash, its input is the script itself). All the code is functions
called on the last line, so a cut-short download runs nothing.

### It changes nothing

Installs nothing (no apt), touches no service or config; writes only into its own temporary directory (`mktemp -d` in
`/var/tmp`, else `/tmp`, else the home directory when those are tmpfs) and removes it on exit, Ctrl+C and kill; without
sysbench one more of its own in `/dev/shm` for the memory test (a file of up to 1 GiB, at most ¼ of the available memory
and half of the free `/dev/shm`), removed right after the test. The disk
test file is 1 GiB with ≥ 3 GiB free, 256 MiB with ≥ 1.25 GiB, otherwise the disk is not tested. No telemetry: only GETs
to public speed-test points and bare TCP connections to them; the IP and the host name of the server are never printed
or put into the JSON. Traffic: a full check up to ≈ 2.5 GB on a fast link (the tries of a region share its 500 MB), a
quick one up to 250 MB.

### What is measured

- **CPU** — 1 core and all cores with sysbench (primes up to 10,000) when installed; 1 core ChaCha20-Poly1305 and
  AES-128-GCM on 1420-byte blocks (as a tunnel encrypts); all cores ChaCha20 via `openssl speed -multi`; CPU steal.
- **Memory** — size (GB); the speed of one thread (MiB/s): with sysbench — writes in 1 MiB blocks, as Kaskad; without
  it ≈ a read of a file in `/dev/shm` (real RAM pages, not the zero page of `/dev/zero`). The built-in number is scored
  only when plausible (under 60 GB/s for one thread) and steady (passes within 2×); otherwise it is shown as a guide
  without a score and the group is the RAM size alone, with the reason.
- **Disk** — 4K random read and write at queue depth 32 (IOPS), 4K writes each flushed to the disk (ms), sequential
  1 MiB write and read (MB/s, no score) — fio, else ≈ dd (no random I/O without fio).
- **Channel** — download, 4 streams, the first 2 s of warm-up dropped, the best point (Mbit/s). No upload: GET only.
- **Latency and DNS** — to 1.1.1.1, 8.8.8.8, 77.88.8.8 (ping every 0.2 s; without ping — the TCP handshake); jitter and
  loss as numbers; DNS — 10 random names under example.com through the system resolver: absolute names (a trailing dot,
  else a missing name is asked again with every `search` domain), one name first and not counted (a cold cache), the
  start of getent itself taken off.
- **System** — OS, kernel, arch, virtualization, CPU model and cores, AES/AVX2, memory and swap, root disk, IPv4/IPv6
  (route and access), TCP congestion control, load; for WireGuard/AmneziaWG — the module, kernel headers (installed /
  in apt), `/dev/net/tun`, a container or not.

Also, without a score: download by region (Cloudflare nearest; Europe — Hetzner Falkenstein, OVH Roubaix, Clouvider
Amsterdam; Russia — Selectel, the Yandex mirror; USA — Hetzner Ashburn, Clouvider New York, OVH US) and latency by
region (TCP handshakes to points in Frankfurt, Amsterdam, Helsinki, Stockholm, Warsaw, Yandex, Selectel, New Jersey,
Los Angeles). A region tries its points in order and keeps the best; it goes on only when a point gave nothing, under
128 KB by the end of the 2 s warm-up (an error page, a connection held at a few KB), or under a quarter of the best so
far (a file throttled on its side). None answered — «—».

### Reading the scores

The Kaskad scale 0–1000, 1000 is never reached. A test score is a logistic curve through the four anchors of the scale
(100/500/700/900, values of its own for each test), extended past the outer ones, with no words attached; a group score is the weighted mean
of its measured tests (weights of `score_abs.py`). Each section shows its bar first, then the numbers. «Channel» and
«Latency and DNS» are the Kaskad groups `net_bw` («Канал») and `net_q` («Сеть»).

**≈** marks a built-in method in the same units (a `/dev/shm` read for sysbench, dd for fio, getent for the DNS client of Kaskad, TCP for
ping) or the one derived scale (a group with such a test is marked ≈ too): «all cores — ChaCha20» stands in for sysbench all cores when sysbench is absent. Its
anchors (310 / 1900 / 3600 / 11000 MB/s) follow the K1 calibration rule of `cpu_mt` (500 — the 1 vCPU reference,
700 — the 2 vCPU one, 900 = 700 × 3, 100 = 500 / 6) applied to the ChaCha20 numbers of the same reference servers
(`tests/fixtures/score_abs_calib_k1.json`); `test.sh` derives them again and compares.

**No total:** Kaskad needs the «Stability» group for it (steal p95 over 72 h, the canary) — not measurable in minutes.
Jitter and loss are shown without a score: Kaskad scores them from 1,000 and 10,000 samples.

### Time

Full: ≈ 1–1.5 min without sysbench and fio, ≈ 2–2.5 min with them. `--quick`: ≈ 35–45 s / ≈ 55–60 s. A slow disk and
unreachable points add time; every step has a time limit (≈ 3 min at most in all): a point that does not answer takes
≈ 2 s, a slow one up to 6 s, at most three points a region.

### Not checked

Memory latency (needs a compiled program), upload (needs POST), stability over a day, loss under load, iperf3, random
disk I/O without fio, Secure Boot for the AmneziaWG module.

### JSON

`--json` prints one object: `system` (no addresses), `groups`, `groups_approx`, `tests` (each with `value`, `score`, `method`, `why`,
`approx`, `n`), `extra` (core scaling, steal, swap, sequential disk speeds), `download` (by region: the best point and
`tries` — every try, `mbps` `null` when a point gave nothing) and `latency` by point. `mem_bw` without a score has `why`
`mem_implausible` (over 60 GB/s for one thread), `mem_unstable` (the passes disagree), `no_shm` or `mem_room`.

### For developers

`bash tools/server-check/test.sh` — no measurements (fine on a Mac: the script refuses to run there): the scale and the
group means against `score_abs.py` (≈ 1,600 values), the parsers against `bench/parse.py` on the Kaskad fixtures (real
outputs of 4 systems included), the report in both languages (≤ 80 characters, no Cyrillic in English), the JSON, the
cleanup, cut-short downloads, shellcheck. `SC_BUILTIN=1` ignores sysbench and fio — to compare the built-in methods with
the exact ones on one server. When `score_abs.py` changes, update `SC_SCALE` in `server-check.sh`; `test.sh` shows the
difference.
