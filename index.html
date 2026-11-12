#!/usr/bin/env bash
# server-check.sh — how good is this server: CPU, memory, disk, network and the system, in one file (Kaskad PRO).
#
#   wget -qO- https://bench.anten-ka.com|bash           English: wget -qO- https://bench.anten-ka.com/en|bash
#   curl -fsSL https://bench.anten-ka.com|bash -s -- --full
#   bash server-check.sh [--lang ru|en] [--full|--all] [--json] [--no-net] [--ascii] [--no-color]
#
# Without options — the quick check (≈ 1.5 min, iperf3 to 3 points, no Speedtest; --quick is the same); --full — 13
# points and Speedtest by Ookla, the system tests at full length (≈ 6 min); --all — all 29 points (≈ 11 min).
#
# Installs nothing into the system: no package is installed, no apt-get update, nothing in /etc, /usr or the database of
# dpkg. The tools it lacks go into its own «mktemp -d» only and are removed with it (on exit, Ctrl+C, kill): iperf3 — a
# static build of a pinned release, checked by its SHA-256; Speedtest by Ookla — the official CLI of a pinned version,
# checked the same way; sysbench and fio — «apt-get download» of the signed distribution packages, unpacked by «dpkg-deb
# -x» (Debian/Ubuntu with package lists only). Network tools run as «nobody» when the check runs as root. The IP of the
# server is never printed. Scores 0–1000 are the scale of Kaskad (lib/kaskad/score_abs.py, abs-0.5-k1): the same curves,
# anchors, weights and group means; no total and no verdicts (owner 23.10 / 27.10). With sysbench / fio — the exact
# methods of Kaskad (lib/kaskad/bench/suite.py); without them built-in ones (openssl, dd, /dev/shm, getent) — «≈».
# The speed: iperf3 to the public points of Russian operators and of Europe, download and upload; the HTTP download of
# the old versions only when iperf3 is unavailable. Environment: SC_LANG=ru|en, SC_FULL=1, SC_ALL=1, SC_JSON=1,
# SC_NO_NET=1, SC_NO_FETCH=1 (fetch nothing), SC_NO_IPERF=1, SC_NO_SPEEDTEST=1, SC_NET_BUDGET_GB=N (the traffic of
# iperf3, 0 — no limit), SC_ASCII=1, NO_COLOR=1; SC_BUILTIN=1 — ignore sysbench and fio (to compare the built-in methods
# with them on one server).
# Everything below is functions; the last line calls them: a download cut short runs nothing.
# Tests: tools/server-check/test.sh (parsers, scores, the report on fixtures; no benchmark, no network).

SC_DEFAULT_LANG=ru     # the language without --lang / SC_LANG; make-pages.sh writes «en» here for /en (this line only)

if [ -z "${BASH_VERSION:-}" ]; then
    echo "server-check: run it with bash:  curl -fsSL URL | bash" >&2
    exit 1
fi

sc_init() {
    SC_VERSION="1.1"
    SC_SCALE_VERSION="abs-0.5-k1"
    # = lib/kaskad/score_abs.py GROUPS (abs-0.5-k1), a test a line: id better weight anchors 100/500/700/900 [key=value];
    # test.sh checks it against score_abs.py field by field.
    SC_SCALE='net_down higher 1 30 300 600 1500 floor=0.01 quick_cap=700
net_up higher 1 20 200 500 1200 floor=0.01 quick_cap=700
rtt lower 1 120 25 10 3 floor=0.05
loss lower 1.5 5 0.5 0.1 0.01 floor=0.005 per_sample=1 min_n=10000
jitter lower 1 30 3 1 0.3 floor=0.05 min_n=1000
dns lower 0.5 300 40 15 5 floor=0.05
cpu_1t higher 1 600 2400 3600 6000
cpu_mt higher 1 910 5500 11000 33000
chacha_1t higher 1.5 150 630 1100 2000
aes_1t higher 0.5 300 2300 4000 6900
steal_p95 lower 1.5 20 3 1 0.2 floor=0.05
cpu_cv lower 1 30 8 4 1.5 floor=0.3
cpu_sustain higher 1 0.5 0.9 0.95 0.98 clamp_max=1.0
cpu_canary_cv lower 1 40 10 5 2 floor=0.5
net_cv lower 1 60 25 12 5 floor=1
mem_gb higher 1 0.5 2 4 16
mem_bw higher 0.5 3000 12000 20000 35000
disk_rr higher 1 1000 15000 40000 120000
disk_rw higher 1 500 8000 25000 80000
disk_fsync lower 1 20 2 0.8 0.2 floor=0.02'
    # Not in Kaskad: all cores by openssl when sysbench is absent, in the place of cpu_mt (its weight). The anchors are
    # the K1 rule of cpu_mt (500 = the 1 vCPU reference, 700 = the 2 vCPU one, 900 = 700 × 3, 100 = 500 / 6) applied
    # to the chacha20 numbers of the same reference servers (tests/fixtures/score_abs_calib_k1.json; test.sh derives it)
    SC_DERIVED='chacha_mt higher 1 310 1900 3600 11000'
    # The groups of score_abs in their order (net_bw — «Канал», net_q — «Сеть»); «a|b» — b when a is not measured.
    # «stab» needs days of probes (steal p95 of 72 h, the canary) — not here, hence no total either.
    SC_GROUPS='cpu:cpu_1t,cpu_mt|chacha_mt,chacha_1t,aes_1t mem:mem_gb,mem_bw disk:disk_rr,disk_rw,disk_fsync net_bw:net_down,net_up net_q:rtt,loss,jitter,dns'
    SC_BAD_FS=" tmpfs ramfs overlay zfs aufs squashfs devtmpfs "        # = bench/suite.py BAD_FS
    SC_CONTAINERS=" openvz lxc lxc-libvirt systemd-nspawn docker podman rkt wsl proot pouch "   # = score.CONTAINER_VIRT
    SC_ANCHORS="1.1.1.1 8.8.8.8 77.88.8.8"                              # = gw/hostping.py DEFAULT_ANCHORS
    # The iperf3 points (the speed table): id|sets|region|city en|city ru|operator en|operator ru|host|ports|cap. Sets: q —
    # the quick check (the default), b — --full, l — the long list (--all); «~» — a fallback point of the row above (same id):
    # tried in order when the download there gives nothing (down, busy, an error). Region: ru | eu | us | asia (the groups
    # of the table). Ports: «5201-5209» — the range the operator publishes (a public iperf3 server serves one client at a
    # time: the tries start at a random port of it and go round), or one port. Cap: Mbit/s of the port of the point
    # (0 — unknown): a result at 90 % of it is «not less than». Every point and fallback answered both directions from a
    # VPS in Moscow 12.11 (iperf3 3.22, 8 streams). Left out: Beeline Voronezh, Clouvider New York and London (upload),
    # Bouygues Paris (no answer from Russia), Leaseweb Washington and San Francisco (download 2 Mbit/s), Kazakhtelecom
    # Astana (runs hang), Scaleway Paris (busy, hangs), Leaseweb Hong Kong (279 Mbit/s).
    SC_IPERF_POINTS='msk-hostkey|qbl|ru|Moscow|Москва|Hostkey|Hostkey|spd-rudp.hostkey.ru|5200-5209|0
msk-hostkey|~|ru|Moscow|Москва|ER-Telecom|ЭР-Телеком|st.msk.ertelecom.ru|5201-5209|0
msk-mts|bl|ru|Moscow|Москва|MTS|МТС|mskst.st.mtsws.net|3333|0
msk-mts|~|ru|Moscow|Москва|ER-Telecom|ЭР-Телеком|st.msk.ertelecom.ru|5201-5209|0
msk-ertelecom|l|ru|Moscow|Москва|ER-Telecom|ЭР-Телеком|st.msk.ertelecom.ru|5201-5209|0
spb-ertelecom|bl|ru|St Petersburg|СПб|ER-Telecom|ЭР-Телеком|st.spb.ertelecom.ru|5201-5209|0
kgd-beeline|l|ru|Kaliningrad|Калининград|Beeline|Билайн|speedtest-kaliningrad-01.corbina.net|5201|0
nn-ertelecom|l|ru|Nizhny Novgorod|Нижний Новгород|ER-Telecom|ЭР-Телеком|st.nn.ertelecom.ru|5201-5209|0
kzn-ertelecom|bl|ru|Kazan|Казань|ER-Telecom|ЭР-Телеком|st.kzn.ertelecom.ru|5201-5209|0
samara-ertelecom|l|ru|Samara|Самара|ER-Telecom|ЭР-Телеком|st.samara.ertelecom.ru|5201-5209|0
volgograd-ertelecom|l|ru|Volgograd|Волгоград|ER-Telecom|ЭР-Телеком|st.volgograd.ertelecom.ru|5201-5209|0
rostov-ertelecom|bl|ru|Rostov-on-Don|Ростов|ER-Telecom|ЭР-Телеком|st.rostov.ertelecom.ru|5201-5209|0
chel-ertelecom|l|ru|Chelyabinsk|Челябинск|ER-Telecom|ЭР-Телеком|st.chel.ertelecom.ru|5201-5209|0
omsk-ertelecom|l|ru|Omsk|Омск|ER-Telecom|ЭР-Телеком|st.omsk.ertelecom.ru|5201-5209|0
krsk-ertelecom|bl|ru|Krasnoyarsk|Красноярск|ER-Telecom|ЭР-Телеком|st.krsk.ertelecom.ru|5201-5209|0
irkutsk-ertelecom|l|ru|Irkutsk|Иркутск|ER-Telecom|ЭР-Телеком|st.irkutsk.ertelecom.ru|5201-5209|0
ams-eranium|bl|eu|Amsterdam|Амстердам|Eranium|Eranium|iperf-ams-nl.eranium.net|5201-5210|0
ams-eranium|~|eu|Amsterdam|Амстердам|Leaseweb|Leaseweb|speedtest.ams1.nl.leaseweb.net|5201-5210|0
fra-leaseweb|qbl|eu|Frankfurt|Франкфурт|Leaseweb|Leaseweb|speedtest.fra1.de.leaseweb.net|5201-5210|0
fra-leaseweb|~|eu|Frankfurt|Франкфурт|Clouvider|Clouvider|fra.speedtest.clouvider.net|5200-5209|0
hel-hostkey|bl|eu|Helsinki|Хельсинки|Hostkey|Hostkey|spd-fisrv.hostkey.com|5201-5209|1000
hel-hostkey|~|eu|Stockholm|Стокгольм|Fegis|Fegis|iperf.fegis.nu|5201-5207|0
waw-hostkey|bl|eu|Warsaw|Варшава|Hostkey|Hostkey|spd-plsrv.hostkey.com|5201-5209|0
lon-leaseweb|bl|eu|London|Лондон|Leaseweb|Leaseweb|speedtest.lon12.uk.leaseweb.net|5201-5210|0
lon-leaseweb|~|eu|London|Лондон|Hostkey|Hostkey|spd-uksrv.hostkey.com|5201-5209|0
sto-fegis|l|eu|Stockholm|Стокгольм|Fegis|Fegis|iperf.fegis.nu|5201-5207|0
sto-fegis|~|eu|Stockholm|Стокгольм|Kamel|Kamel|speedtest.kamel.network|5201-5205|0
par-moji|l|eu|Paris|Париж|Moji|Moji|iperf3.moji.fr|5200-5240|0
par-moji|~|eu|Paris|Париж|Hostkey|Hostkey|spd-frsrv.hostkey.com|5201-5209|0
nyc-leaseweb|qbl|us|New York|Нью-Йорк|Leaseweb|Leaseweb|speedtest.nyc1.us.leaseweb.net|5201-5210|0
nyc-leaseweb|~|us|Ashburn|Ашберн|Clouvider|Clouvider|ash.speedtest.clouvider.net|5200-5209|0
chi-leaseweb|l|us|Chicago|Чикаго|Leaseweb|Leaseweb|speedtest.chi11.us.leaseweb.net|5201-5210|0
dal-leaseweb|l|us|Dallas|Даллас|Leaseweb|Leaseweb|speedtest.dal13.us.leaseweb.net|5201-5210|0
lax-leaseweb|bl|us|Los Angeles|Лос-Анджелес|Leaseweb|Leaseweb|speedtest.lax12.us.leaseweb.net|5201-5210|0
lax-leaseweb|~|us|Los Angeles|Лос-Анджелес|Clouvider|Clouvider|la.speedtest.clouvider.net|5200-5209|0
ist-hostkey|l|asia|Istanbul|Стамбул|Hostkey|Hostkey|spd-tr.hostkey.com|5201-5209|0
tas-uztelecom|l|asia|Tashkent|Ташкент|Uztelecom|Uztelecom|speedtest.uztelecom.uz|5200-5209|0
sin-leaseweb|l|asia|Singapore|Сингапур|Leaseweb|Leaseweb|speedtest.sin1.sg.leaseweb.net|5201-5210|0
tyo-leaseweb|l|asia|Tokyo|Токио|Leaseweb|Leaseweb|speedtest.tyo11.jp.leaseweb.net|5201-5210|0'
    # iperf3: 8 parallel streams. On a path of 57 ms (Moscow — Krasnoyarsk, 12.11) one TCP stream is held by its window:
    # 4 streams gave 2084 / 1446 Mbit/s, 8 — 3460 / 2828 (+66 % / +96 %); on short paths 8 cost nothing
    SC_STREAMS=8
    # The traffic of iperf3 (S2 of REV-BENCH-3): a budget a mode — the quick check 5 GB, --full 30, --all 60 (GB = 10^9 B;
    # SC_NET_BUDGET_GB=N changes it, 0 — none). A run is capped (-b) so that the runs after it still fit at 1 Gbit/s
    # (sc_net_cap_check); all of them at 1 Gbit/s fit every budget: 6 × 0.625, 26 × 0.875, 58 × 0.875 GB.
    SC_NET_CAPMBIT=1000
    # The tools fetched into the temporary directory: tool|arch (uname -m, i?86 -> i686, armv*l -> armv7l)|url|sha256.
    # iperf3 — the static builds of github.com/userdocs/iperf3-static 3.22 (the digests of the release, checked by a
    # download 12.11); Speedtest — the official CLI 1.2.0 of Ookla (the digests of the downloads 12.11). A file that does
    # not match is removed unrun.
    SC_PINS='iperf3|x86_64|https://github.com/userdocs/iperf3-static/releases/download/3.22/iperf3-amd64|ef4c89f3fc01c56cf00d34b900024c74f68e540dab452eef4fd902544d9c1c64
iperf3|aarch64|https://github.com/userdocs/iperf3-static/releases/download/3.22/iperf3-arm64v8|34364466812357fe021a8647cb433c8006b0360e3b690ecda470a979a2601031
iperf3|armv7l|https://github.com/userdocs/iperf3-static/releases/download/3.22/iperf3-arm32v7|ebac45408e994c86d78f20dafb1ec7bfd0a5d4cf6acf05283f79c07b2d4647c4
iperf3|i686|https://github.com/userdocs/iperf3-static/releases/download/3.22/iperf3-i386|241effdef6b0a170043a2de94b69b6213f81bb10d6c6adf790b5c183cf195cd5
speedtest|x86_64|https://install.speedtest.net/app/cli/ookla-speedtest-1.2.0-linux-x86_64.tgz|5690596c54ff9bed63fa3732f818a05dbc2db19ad36ed68f21ca5f64d5cfeeb7
speedtest|aarch64|https://install.speedtest.net/app/cli/ookla-speedtest-1.2.0-linux-aarch64.tgz|3953d231da3783e2bf8904b6dd72767c5c6e533e163d3742fd0437affa431bd3
speedtest|armv7l|https://install.speedtest.net/app/cli/ookla-speedtest-1.2.0-linux-armhf.tgz|e45fcdebbd8a185553535533dd032d6b10bc8c64eee4139b1147b9c09835d08d
speedtest|i686|https://install.speedtest.net/app/cli/ookla-speedtest-1.2.0-linux-i386.tgz|9ff7e18dbae7ee0e03c66108445a2fb6ceea6c86f66482e1392f55881b772fe8'
    SC_APT_MAX=50000000       # bytes: sysbench / fio with their missing libraries above this — not fetched
    # The HTTP download points — only when iperf3 is unavailable or no iperf3 point answered (download only):
    # id|region|English|Russian|url|bytes of one request ({bytes}; 0 — a file)|size of the file.
    # Cloudflare, Selectel, Yandex and OVH — bench/channel-targets.json of Kaskad; Hetzner and Clouvider — the 1 GB
    # files of hosting speed-test sites (HTTPS, 12.11: Vultr dropped — its 1000MB.bin gave 7–28 Mbit/s on a 2.8 Gbit/s
    # server and a 503 page from elsewhere). A region tries its points in order and keeps the best; the next point is
    # tried only when the last one gave nothing or less than a quarter of the best so far (sc_net_down).
    SC_DL_POINTS='cf|any|Cloudflare|Cloudflare|https://speed.cloudflare.com/__down?bytes={bytes}|50000000|0
hetzner-fsn|eu|Falkenstein (Hetzner)|Фалькенштайн (Hetzner)|https://fsn1-speed.hetzner.com/1GB.bin|0|1073741824
ovh|eu|Roubaix (OVH)|Рубе (OVH)|https://proof.ovh.net/files/1Gb.dat|0|1073741824
clouvider-ams|eu|Amsterdam (Clouvider)|Амстердам (Clouvider)|https://ams.speedtest.clouvider.net/1g.bin|0|1048576000
selectel|ru|Selectel|Selectel|https://speedtest.selectel.ru/1GB|0|1073741824
yandex|ru|Yandex mirror|Зеркало Яндекса|https://mirror.yandex.ru/ubuntu/ls-lR.gz|0|38000000
hetzner-ash|us|Ashburn (Hetzner)|Эшберн (Hetzner)|https://ash-speed.hetzner.com/1GB.bin|0|1073741824
clouvider-nyc|us|New York (Clouvider)|Нью-Йорк (Clouvider)|https://nyc.speedtest.clouvider.net/1g.bin|0|1048576000
ovh-us|us|OVH US|OVH США|https://proof.ovh.us/files/1Gb.dat|0|1073741824'
    SC_DL_MIN=128000          # bytes: less in all streams by the end of the warm-up (or in all) — an error page or a
                              # connection held at a few KB, not a file: the point counts as «—» and the next one goes
    SC_DNS_ZONE="example.com"                                           # = bench/suite.py DNS_ZONE
    # Memory without sysbench: one thread cannot read RAM faster than this (≈ 60 GB/s, in MiB/s) — a number above it is a
    # cache or a zero-page effect, shown as a guide without a score
    SC_MEM_MAX_MIBS=57220
}

# ── small helpers ───────────────────────────────────────────────────────────────────────────────────────────────────

sc_have() { command -v "$1" >/dev/null 2>&1; }

# sc_t EN RU -> the text of the language of the report
sc_t() { if [ "$SC_LANG" = ru ]; then printf '%s' "$2"; else printf '%s' "$1"; fi; }

sc_isnum() { [[ ${1:-} =~ ^[0-9]+([.][0-9]+)?([eE][-+]?[0-9]+)?$ ]]; }

# sc_clock -> SC_NOW (microseconds)
sc_clock() {
    if [ -n "${EPOCHREALTIME:-}" ]; then SC_NOW=${EPOCHREALTIME/[.,]/}; else SC_NOW=$(date +%s%6N); fi
}

# sc_to SECS CMD… — a command with a time limit (coreutils timeout; without it — as is: every tool here has its own)
sc_to() {
    local s=$1; shift
    if [ "$SC_TIMEOUT" = 1 ]; then timeout --foreground -k 3 "$s" "$@"; else "$@"; fi
}

# sc_to_int SECS CMD… — the same with SIGINT: dd prints what it has copied and ends
sc_to_int() {
    local s=$1; shift
    if [ "$SC_TIMEOUT" = 1 ]; then timeout --foreground -k 3 -s INT "$s" "$@"; else "$@"; fi
}

# sc_clean TEXT -> REPLY: a line of the system made safe for a screen and JSON (no control characters, single spaces,
# at most 80 characters) — as bench/parse.py _clean
sc_clean() {
    local s=${1//[$'\001'-$'\037'$'\177']/ } words
    s=${s//\"/}; s=${s//\\/}
    read -r -a words <<<"$s"
    s="${words[*]}"
    if [ ${#s} -gt 80 ]; then          # bytes (LC_ALL=C): a cut must not leave half of a UTF-8 character
        s=${s:0:80}
        local k=0 need=0
        while [ "$k" -lt 3 ] && [ "$k" -lt ${#s} ] && [[ ${s:${#s}-1-k:1} == [$'\x80'-$'\xbf'] ]]; do k=$((k + 1)); done
        case "${s:${#s}-1-k:1}" in
            [$'\xc0'-$'\xdf']) need=1 ;; [$'\xe0'-$'\xef']) need=2 ;; [$'\xf0'-$'\xf7']) need=3 ;;
        esac
        [ "$need" -gt "$k" ] && s=${s:0:${#s}-1-k}
    fi
    REPLY=$s
}

# sc_len TEXT -> REPLY: characters (UTF-8) whatever the locale; the script runs with LC_ALL=C, ${#s} counts bytes
sc_len() {
    local s=$1 c w
    c=${s//[^$'\x80'-$'\xbf']/}
    REPLY=$(( ${#s} - ${#c} ))
    # East Asian wide characters take two columns (roughly as wcwidth): U+3000–U+DFFF (lead bytes E3–ED), U+F900–U+FAFF
    # and U+FF00–U+FF3F (EF A4–AB, EF BC), U+10000 and up (F0–F4: CJK extensions, emoji). Ours are all narrow (E2 and less).
    if [[ $s == *[$'\xe3'-$'\xf4']* ]]; then
        w=${s//[^$'\xe3'-$'\xed'$'\xf0'-$'\xf4']/}
        c=${s//$'\xef'[$'\xa4'-$'\xab'$'\xbc']/}
        REPLY=$((REPLY + ${#w} + (${#s} - ${#c}) / 2))
    fi
}

# sc_plural N ONE FEW MANY -> REPLY (Russian: 1 ядро, 2 ядра, 5 ядер, 21 ядро)
sc_plural() {
    local n=$(( $1 % 100 )) m=$(( $1 % 10 ))
    if [ "$n" -ge 11 ] && [ "$n" -le 14 ]; then REPLY=$4
    elif [ "$m" -eq 1 ]; then REPLY=$2
    elif [ "$m" -ge 2 ] && [ "$m" -le 4 ]; then REPLY=$3
    else REPLY=$4; fi
}

# sc_json_str TEXT -> REPLY: a JSON string (the text is clean already: sc_clean)
sc_json_str() {
    local s=${1//\\/\\\\}
    s=${s//\"/\\\"}
    s=${s//[$'\001'-$'\037']/ }
    REPLY="\"$s\""
}

sc_json_num() { if sc_isnum "${1:-}"; then REPLY=$1; else REPLY=null; fi; }

# ── progress (stderr, a terminal only) ──────────────────────────────────────────────────────────────────────────────

sc_prog() {
    local el
    if [ "${SC_LIVE:-0}" = 1 ]; then          # the «Now» line of the frame: one line, a column to spare (no wrap)
        sc_len "$(sc_t "Now" "Сейчас")"; sc_cut "$*" $(( ${SC_W:-80} - REPLY - 3 ))
        sc_paint "$REPLY"; printf '%s' "$REPLY" >"$SC_WORK/live.n" 2>/dev/null; return 0
    fi
    [ "$SC_PROGRESS" = 1 ] || return 0
    sc_clock; el=$(( (SC_NOW - SC_T0) / 1000000 ))
    sc_cut "$*" $(( ${SC_COLS:-80} - 11 )); sc_vis "$REPLY"
    printf '\r\033[K %s %d:%02d  %s' "›" $((el / 60)) $((el % 60)) "$REPLY" >&2
}

sc_prog_clear() { [ "$SC_PROGRESS" = 1 ] && printf '\r\033[K' >&2; return 0; }

# ── parsers (stdin -> stdout; test.sh runs them on fixtures) ────────────────────────────────────────────────────────

# PRETTY_NAME of /etc/os-release
sc_parse_osrel() {
    awk -F= '$1 == "PRETTY_NAME" { v = substr($0, index($0, "=") + 1); gsub(/^["'\'']|["'\'']$/, "", v); print v; exit }'
}

# «NAME VERSION_ID» of /etc/os-release, the first word of NAME («Debian 12», «Ubuntu 24.04»); nothing when one is missing
sc_parse_osshort() {
    awk -F= '$1 == "NAME" || $1 == "VERSION_ID" { v = substr($0, index($0, "=") + 1); gsub(/^["'\'']|["'\'']$/, "", v); a[$1] = v }
             END { split(a["NAME"], w, " "); if (w[1] != "" && a["VERSION_ID"] != "") print w[1], a["VERSION_ID"] }'
}

# «model name» (x86) of /proc/cpuinfo
sc_parse_cpu_model() {
    awk -F: 'tolower($1) ~ /^model name[ \t]*$/ { sub(/^[ \t]+/, "", $2); print $2; exit }'
}

# the flags of Kaskad (bench/parse.py FLAGS: aes avx2 vaes) of the first «flags» / «Features» line of /proc/cpuinfo
sc_parse_cpu_flags() {
    awk -F: 'tolower($1) ~ /^(flags|features)[ \t]*$/ { n = split($2, f, " "); for (i = 1; i <= n; i++) h[f[i]] = 1
             out = ""; if ("aes" in h) out = out " aes"; if ("avx2" in h) out = out " avx2"; if ("vaes" in h) out = out " vaes"
             sub(/^ /, "", out); print out; exit }'
}

# lscpu «Model name» / «Thread(s) per core»: sc_parse_lscpu model|tpc
sc_parse_lscpu() {
    awk -F: -v k="$1" '
        k == "model" && $1 == "Model name" { sub(/^[ \t]+/, "", $2); print $2; exit }
        k == "tpc" && $1 == "Thread(s) per core" { gsub(/[ \t]/, "", $2); print $2; exit }'
}

# sc_parse_meminfo KEY -> kB
sc_parse_meminfo() {
    awk -v k="$1:" '$1 == k && $3 == "kB" && $2 ~ /^[0-9]+$/ { print $2; exit }'
}

# «+H:<size>…» and «+F:<n>:<name>:<bytes/s>…» of «openssl speed … -mr» -> bytes/s of the block SIZE — as bench/parse.py
# openssl_speed (the last «+F»). With -multi the parent prints that line as the sum of its children; without it, the sum
# of the children's lines «Got: +F:… from N» is taken.
sc_parse_openssl() {
    awk -v size="$1" '
        function pick(line,    m, F, i, h, v) {
            m = split(line, F, ":")
            if (!hs) { if (m - 3 == 1) { H[1] = size; n = 1 } else return "" }
            for (i = 1; i <= n; i++) {
                h = H[i]; gsub(/[ \t]/, "", h)
                if (h == size "") {
                    v = F[i + 3]
                    if (i + 3 <= m && v ~ /^[0-9]+(\.[0-9]+)?$/ && v + 0 > 0) return v
                    return ""
                }
            }
            return ""
        }
        { sub(/\r$/, ""); sub(/^[ \t]+/, "") }
        /^\+H:/ { n = split(substr($0, 4), H, ":"); hs = 1 }
        /^\+F:/ { last = $0 }
        /^Got: \+F:/ { g = $0; sub(/^Got: /, "", g); sub(/ from [0-9]+$/, "", g); kids[++nk] = g }
        /^Got: \+H:/ && !hs { g = $0; sub(/^Got: \+H:/, "", g); sub(/ from [0-9]+$/, "", g); n = split(g, H, ":"); hs = 1 }
        END {
            if (last != "") { v = pick(last); if (v != "") print v; exit }
            for (i = 1; i <= nk; i++) { v = pick(kids[i]); if (v == "") exit; s += v }
            if (nk && s > 0) printf "%.2f\n", s
        }'
}

# «events per second: N» of «sysbench cpu … run»
sc_parse_sysbench_cpu() {
    awk '/^[ \t]*events per second:[ \t]*[0-9]+(\.[0-9]+)?[ \t]*$/ { v = $NF + 0; if (v > 0) print $NF; exit }'
}

# «N MiB transferred (X MiB/sec)» of «sysbench memory … run» -> X
sc_parse_sysbench_mem() {
    awk 'match($0, /[0-9]+(\.[0-9]+)? MiB transferred \([0-9]+(\.[0-9]+)? MiB\/sec\)/) {
             s = substr($0, RSTART, RLENGTH); sub(/.*\(/, "", s); sub(/ MiB\/sec\)/, "", s); if (s + 0 > 0) print s; exit }'
}

# fio --output-format=json, the first job: sc_parse_fio read|write iops|bw -> IOPS | MB/s (bw_bytes / 1e6); a job with
# an error or no I/O — nothing (as bench/parse.py fio_iops / fio_bw_mbs)
sc_parse_fio() {
    awk -v side="$1" -v what="$2" '
        function val(line) { sub(/^[^:]*:[ \t]*/, "", line); sub(/,[ \t]*$/, "", line); return line }
        /"jobname"[ \t]*:/ { jobs++; if (jobs > 1) exit }
        jobs == 1 && !errseen && /"error"[ \t]*:/ { err = val($0); errseen = 1 }
        /^[ \t]*"(read|write|trim|sync|mixed)"[ \t]*:[ \t]*\{/ { cur = $0; gsub(/[ \t":{]/, "", cur); next }
        jobs == 1 && cur == side && /"iops"[ \t]*:/ && iops == "" { iops = val($0) }
        jobs == 1 && cur == side && /"total_ios"[ \t]*:/ && tot == "" { tot = val($0) }
        jobs == 1 && cur == side && /"bw_bytes"[ \t]*:/ && bwb == "" { bwb = val($0) }
        jobs == 1 && cur == side && /"bw"[ \t]*:/ && bw == "" { bw = val($0) }
        END {
            if (err != "" && err + 0 != 0) exit
            if (tot == "" || tot + 0 <= 0) exit
            if (what == "iops") { if (iops + 0 > 0) printf "%.6f\n", iops + 0; exit }
            b = (bwb != "") ? bwb + 0 : bw * 1024
            if (b > 0) printf "%.6f\n", b / 1e6
        }'
}

# the statistics lines of GNU dd -> «bytes seconds» a line, in their order (SIGUSR1 prints one more on the way)
sc_parse_dd() {
    awk '/^[0-9]+ bytes/ && match($0, /copied, [0-9]+(\.[0-9]+)?(e[-+]?[0-9]+)? s/) {
             t = substr($0, RSTART + 8, RLENGTH - 10); print $1, t }'
}

# «N+M records out» of dd -> N+M
sc_parse_dd_records() {
    awk '/records out/ { split($1, r, "+"); n = r[1] + r[2] } END { if (n != "") print n }'
}

# ping (iputils / busybox) -> «sent received rtt rtt …» (DUP! replies left out)
sc_parse_ping() {
    awk '/bytes from/ && !/DUP!/ && match($0, /time[=<][0-9]+(\.[0-9]+)?/) { r = r " " substr($0, RSTART + 5, RLENGTH - 5) }
         /packets transmitted/ { sent = $1; rec = $4 }
         END { if (sent != "") print sent, rec r }'
}

# sc_fs_type PATH < mountinfo -> the type of the file system of the longest mount point holding PATH (suite.fs_type)
sc_fs_type() {
    awk -v p="$1" '
        { i = index($0, " - "); if (!i) next; split(substr($0, 1, i - 1), f, " "); split(substr($0, i + 3), g, " ")
          mp = f[5]; gsub(/\\040/, " ", mp); m = mp; sub(/\/$/, "", m)
          if ((p == mp || index(p, m "/") == 1) && length(mp) >= length(best)) { best = mp; typ = g[1] } }
        END { if (typ != "") print typ }'
}

# ── statistics (awk; numbers as arguments) ──────────────────────────────────────────────────────────────────────────

# sc_median X… -> REPLY (statistics.median: the mean of the two middle ones for an even count)
sc_median() {
    REPLY=$(printf '%s\n' "$@" | awk 'NF && $1 ~ /^[0-9.eE+-]+$/ { v[n++] = $1 + 0 }
        END { if (!n) exit; for (i = 1; i < n; i++) { x = v[i]; for (j = i - 1; j >= 0 && v[j] > x; j--) v[j + 1] = v[j]; v[j + 1] = x }
              if (n % 2) printf "%.10g\n", v[int(n / 2)]; else printf "%.10g\n", (v[n / 2 - 1] + v[n / 2]) / 2 }')
}

# sc_reduce X… -> REPLY: the node from k points (gw/hostmon.py _reduce): sorted, the worst ⌊(k−1)/2⌋ dropped, the mean
# of the rest; k = 2 — the less
sc_reduce() {
    REPLY=$(printf '%s\n' "$@" | awk 'NF && $1 ~ /^[0-9.eE+-]+$/ { v[n++] = $1 + 0 }
        END { if (!n) exit; for (i = 1; i < n; i++) { x = v[i]; for (j = i - 1; j >= 0 && v[j] > x; j--) v[j + 1] = v[j]; v[j + 1] = x }
              if (n == 2) { printf "%.10g\n", v[0]; exit }
              keep = n - int((n - 1) / 2); s = 0; for (i = 0; i < keep; i++) s += v[i]; printf "%.10g\n", s / keep }')
}

# sc_jitter RTT… -> REPLY «mean |Δ| of neighbour replies» and REPLY2 the number of pairs (gw/hostmon.py ping_rows)
sc_jitter() {
    local out
    out=$(printf '%s\n' "$@" | awk 'NF { v[n++] = $1 + 0 } END { for (i = 1; i < n; i++) { d = v[i] - v[i - 1]; s += (d < 0 ? -d : d) }
        if (n > 1) printf "%.10g %d\n", s / (n - 1), n - 1 }')
    REPLY=${out% *}; REPLY2=${out#* }
    [ -n "$out" ] || { REPLY=""; REPLY2=0; }
}

# ── the scale of Kaskad ─────────────────────────────────────────────────────────────────────────────────────────────

# sc_score TEST VALUE [N] -> REPLY: the score 0–1000, a float (score_abs.test_score / score_of): a logistic curve through
# the anchors 100/500/700/900 in (ln x, logit s), the ends extend the outer segments; the floor, clamp_max, quick_cap
# of a quick check, «0 of n» = 100/n % for loss; fewer samples than min_n (N given) — "".
sc_score() {
    REPLY=""
    sc_isnum "${2:-}" || return 0
    REPLY=$(SC_T="$SC_SCALE
$SC_DERIVED" awk -v t="$1" -v x="$2" -v n="${3:-}" -v mode="$SC_MODE" 'BEGIN {
        k = split(ENVIRON["SC_T"], rows, "\n")
        for (r = 1; r <= k; r++) {
            m = split(rows[r], f, " ")
            if (f[1] != t) continue
            found = 1; better = f[2]; a[1] = f[4] + 0; a[2] = f[5] + 0; a[3] = f[6] + 0; a[4] = f[7] + 0
            for (j = 8; j <= m; j++) { split(f[j], kv, "="); o[kv[1]] = kv[2] }
        }
        if (!found) exit
        if (n != "" && ("min_n" in o) && n + 0 < o["min_n"] + 0) exit
        x += 0
        lo = a[1]; for (j = 2; j <= 4; j++) if (a[j] < lo) lo = a[j]
        lo = (("floor" in o) && o["floor"] + 0 > 0) ? o["floor"] + 0 : 1e-6 * lo
        if (("per_sample" in o) && n + 0 > 0 && 100.0 / n > lo) lo = 100.0 / n
        if (x < lo) x = lo
        if (("clamp_max" in o) && x > o["clamp_max"] + 0) x = o["clamp_max"] + 0
        A[1] = 100; A[2] = 500; A[3] = 700; A[4] = 900
        for (j = 1; j <= 4; j++) { u[j] = log(a[j]); if (better == "lower") u[j] = -u[j]; L[j] = log(A[j] / (1000 - A[j])) }
        q = log(x); if (better == "lower") q = -q
        i = (q <= u[2]) ? 1 : ((q <= u[3]) ? 2 : 3)
        s = 1000 / (1 + exp(-(L[i] + (L[i + 1] - L[i]) * (q - u[i]) / (u[i + 1] - u[i]))))
        if (mode == "quick" && ("quick_cap" in o) && s > o["quick_cap"] + 0) s = o["quick_cap"] + 0
        printf "%.17g\n", s
    }')
}

# sc_shown SCORE -> REPLY: on screen — floor, at most 999 (score_abs._shown)
sc_shown() {
    REPLY=""
    sc_isnum "${1:-}" || return 0
    REPLY=$(awk -v s="$1" 'BEGIN { v = int(s + 1e-9); if (v > 999) v = 999; print v }')
}

# sc_weight TEST -> REPLY
sc_weight() {
    REPLY=$(printf '%s\n%s\n' "$SC_SCALE" "$SC_DERIVED" | awk -v t="$1" '$1 == t { print $3; exit }')
}

# sc_group_score G -> GS[G] (shown) and GSF[G] (float): the weighted mean of the scores of its measured tests (the tests
# with a score; «a|b» — b only when a has none), as score_abs.total «groups»
sc_group_score() {
    local g=$1 spec tests t alt pairs="" ap=0
    for spec in $SC_GROUPS; do [ "${spec%%:*}" = "$g" ] && tests=${spec#*:}; done
    local IFS=,
    for t in $tests; do
        alt=""
        case "$t" in *"|"*) alt=${t#*|}; t=${t%%|*} ;; esac
        if [ -z "${SF[$t]:-}" ] && [ -n "$alt" ]; then t=$alt; fi
        if [ -n "${SF[$t]:-}" ]; then
            sc_weight "$t"; pairs+=" $REPLY ${SF[$t]}"
            [ "${AP[$t]:-0}" = 1 ] && ap=1
        fi
    done
    unset IFS
    GSF[$g]=""; GS[$g]=""; GA[$g]=$ap
    [ -n "$pairs" ] || return 0
    # shellcheck disable=SC2086
    GSF[$g]=$(awk 'BEGIN { for (i = 1; i < ARGC; i += 2) { num += ARGV[i] * ARGV[i + 1]; den += ARGV[i] } if (den > 0) printf "%.17g\n", num / den }' $pairs)
    sc_shown "${GSF[$g]}"; GS[$g]=$REPLY
}

# sc_set TEST VALUE METHOD [WHY] [APPROX] [N] -> V/SF/SS/M/WHY/AP of the test (a score only with a value)
sc_set() {
    local t=$1 v=${2:-} n=${6:-}
    V[$t]=""; SF[$t]=""; SS[$t]=""; M[$t]=${3:-}; WHY[$t]=${4:-}; AP[$t]=${5:-0}; NN[$t]=$n
    if sc_isnum "$v"; then
        V[$t]=$v
        sc_score "$t" "$v" "$n"; SF[$t]=$REPLY
        sc_shown "$REPLY"; SS[$t]=$REPLY
        if [ -z "${SF[$t]}" ] && [ -z "${WHY[$t]}" ]; then WHY[$t]="few_samples"; fi
    elif [ -z "${WHY[$t]}" ]; then
        WHY[$t]="tool_failed"
    fi
    [ "${SC_LIVE:-0}" = 1 ] && sc_live_frame
    return 0
}

# ── the system ──────────────────────────────────────────────────────────────────────────────────────────────────────

sc_read() { [ -r "$1" ] && cat -- "$1" 2>/dev/null; }

sc_virt() {
    local v=""
    sc_have systemd-detect-virt && v=$(systemd-detect-virt 2>/dev/null)
    if [ -z "$v" ]; then
        if [ -e /proc/vz ] && [ ! -e /proc/bc ]; then v=openvz
        elif [ -e /.dockerenv ]; then v=docker
        elif [ -e /run/.containerenv ]; then v=podman
        elif grep -qw hypervisor /proc/cpuinfo 2>/dev/null; then v=vm
        else v=unknown; fi
    fi
    sc_clean "$v"; REPLY=${REPLY:0:32}
}

sc_sysinfo() {
    local kr v ok
    sc_prog "$(sc_t "system" "система")"
    kr=$(uname -r 2>/dev/null); sc_clean "$kr"; SYS[kernel]=$REPLY
    sc_clean "$(uname -m 2>/dev/null)"; SYS[arch]=$REPLY
    sc_clean "$(sc_read /etc/os-release | sc_parse_osrel)"; SYS[os]=${REPLY:-Linux}
    sc_clean "$(sc_read /etc/os-release | sc_parse_osshort)"; SYS[os_short]=${REPLY:-${SYS[os]}}
    sc_virt; SYS[virt]=$REPLY
    case "$SC_CONTAINERS" in *" ${SYS[virt]} "*) SYS[container]=1 ;; *) SYS[container]=0 ;; esac
    v=$(sc_read /proc/cpuinfo | sc_parse_cpu_model)
    [ -z "$v" ] && sc_have lscpu && v=$(lscpu 2>/dev/null | sc_parse_lscpu model)
    sc_clean "$v"; SYS[cpu]=${REPLY:-?}
    SYS[flags]=$(sc_read /proc/cpuinfo | sc_parse_cpu_flags)
    v=$(nproc 2>/dev/null); [[ $v =~ ^[0-9]+$ ]] || v=$(grep -c '^processor' /proc/cpuinfo 2>/dev/null)
    [[ $v =~ ^[0-9]+$ ]] && [ "$v" -ge 1 ] || v=1
    SYS[nproc]=$v
    v=""; sc_have lscpu && v=$(lscpu 2>/dev/null | sc_parse_lscpu tpc)
    [[ $v =~ ^[0-9]+$ ]] || v=""
    SYS[tpc]=$v
    local mi; mi=$(sc_read /proc/meminfo)
    SYS[mem_kb]=$(sc_parse_meminfo MemTotal <<<"$mi")
    SYS[avail_kb]=$(sc_parse_meminfo MemAvailable <<<"$mi")
    SYS[swap_kb]=$(sc_parse_meminfo SwapTotal <<<"$mi")
    SYS[swapfree_kb]=$(sc_parse_meminfo SwapFree <<<"$mi")
    # the root file system
    v=$(df -Pk / 2>/dev/null | awk 'NR == 2 { print $2, $4 }')
    SYS[root_kb]=${v% *}; SYS[root_free_kb]=${v#* }
    [[ ${SYS[root_kb]} =~ ^[0-9]+$ ]] || { SYS[root_kb]=""; SYS[root_free_kb]=""; }
    sc_clean "$(sc_read /proc/self/mountinfo | sc_fs_type /)"; SYS[root_fs]=${REPLY:0:24}
    # the network: routes and addresses only (no address is printed)
    SYS[ipv4]=0; SYS[ipv6]=0
    awk 'NR > 1 && $2 == "00000000" && $1 != "lo" { f = 1 } END { exit !f }' /proc/net/route 2>/dev/null && SYS[ipv4]=1
    if awk '$1 == "00000000000000000000000000000000" && $2 == "00" && $10 != "lo" { f = 1 } END { exit !f }' \
            /proc/net/ipv6_route 2>/dev/null \
        && awk '$4 == "00" && $1 !~ /^f[cd]/ { f = 1 } END { exit !f }' /proc/net/if_inet6 2>/dev/null; then
        SYS[ipv6]=1
    fi
    sc_clean "$(sc_read /proc/sys/net/ipv4/tcp_congestion_control)"; SYS[tcp_cc]=${REPLY:0:16}
    sc_clean "$(sc_read /proc/sys/net/core/default_qdisc)"; SYS[qdisc]=${REPLY:0:16}
    v=$(awk '{ print $1; exit }' /proc/loadavg 2>/dev/null); sc_isnum "$v" || v=""; SYS[load]=$v
    # WireGuard / AmneziaWG — informational
    if [ -d /sys/module/wireguard ]; then SYS[wg]=loaded
    elif [ -n "$kr" ] && grep -qs '/wireguard\.ko' "/lib/modules/$kr/modules.builtin"; then SYS[wg]=builtin
    elif [ -n "$kr" ] && [ -d "/lib/modules/$kr" ] \
        && [ -n "$(find "/lib/modules/$kr" -name 'wireguard.ko*' -print -quit 2>/dev/null)" ]; then SYS[wg]=module
    else SYS[wg]=no; fi
    if [ -n "$kr" ] && [ -e "/lib/modules/$kr/build/Makefile" ]; then SYS[headers]=yes
    elif [ -n "$kr" ] && sc_have apt-cache; then
        v=$(sc_to 10 apt-cache policy "linux-headers-$kr" 2>/dev/null | awk '/Candidate:/ { print $2; exit }')
        if [ -n "$v" ] && [ "$v" != "(none)" ]; then SYS[headers]=apt; else SYS[headers]=no; fi
    else SYS[headers]=no; fi
    if [ -c /dev/net/tun ]; then SYS[tun]=yes; else SYS[tun]=no; fi
    # does the Internet answer over IPv4 / IPv6 (one GET of 0 bytes to Cloudflare each)
    SYS[ipv4_ok]=""; SYS[ipv6_ok]=""
    if [ "$SC_NONET" != 1 ] && [ -n "$SC_FETCH" ]; then
        for v in 4 6; do
            if [ "$SC_FETCH" = curl ]; then
                curl -"$v" -sgf --proto '=https' -o /dev/null --connect-timeout 4 -m 6 \
                    "https://speed.cloudflare.com/__down?bytes=0" 2>/dev/null && ok=1 || ok=0
            else
                sc_to 7 wget -"$v" -q -O /dev/null --https-only --timeout=4 --tries=1 --max-redirect=0 \
                    "https://speed.cloudflare.com/__down?bytes=0" 2>/dev/null && ok=1 || ok=0
            fi
            SYS[ipv${v}_ok]=$ok
        done
    fi
}

sc_gb() { awk -v k="${1:-0}" -v d="${2:-1}" 'BEGIN { printf "%." d "f\n", k * 1024 / 1e9 }'; }

# ── tools: fetched into the temporary directory, never installed ────────────────────────────────────────────────────

# sc_arch -> REPLY: the key of the machine in SC_PINS (x86_64, aarch64, armv7l, i686; anything else as uname -m says)
sc_arch() {
    case "${SYS[arch]:-}" in
        i?86) REPLY=i686 ;; armv7*|armv8l|armhf) REPLY=armv7l ;; arm64) REPLY=aarch64 ;; *) REPLY=${SYS[arch]:-} ;;
    esac
}

# sc_pin TOOL ARCH -> REPLY the url, REPLY2 the sha256 (false: no build for this machine)
sc_pin() {
    local t a u h
    REPLY=""; REPLY2=""
    while IFS='|' read -r t a u h; do
        if [ "$t" = "$1" ] && [ "$a" = "$2" ]; then REPLY=$u; REPLY2=$h; return 0; fi
    done <<<"$SC_PINS"
    return 1
}

# sc_sha256 FILE -> REPLY: the SHA-256 of the file in hex ("" — nothing here can count it)
sc_sha256() {
    REPLY=""
    if sc_have sha256sum; then REPLY=$(sha256sum "$1" 2>/dev/null | awk '{ print $1 }')
    elif sc_have openssl; then REPLY=$(openssl dgst -sha256 -r "$1" 2>/dev/null | awk '{ print $1 }'); fi
    [[ $REPLY =~ ^[0-9a-f]{64}$ ]] || REPLY=""
}

# sc_get URL FILE SECS — a download into FILE (redirects to HTTPS only: GitHub sends its files from another host)
sc_get() {
    if [ "$SC_FETCH" = curl ]; then
        curl -fsSL --proto '=https' --proto-redir '=https' --connect-timeout 5 --max-time "$3" -o "$2" "$1" 2>/dev/null
    else
        sc_to "$3" wget -q -O "$2" --https-only --timeout=5 --tries=1 "$1" 2>/dev/null
    fi
}

# sc_fetch_pinned TOOL -> REPLY: the path of the tool in SC_BIN, fetched by SC_PINS and checked by its SHA-256 (a .tgz —
# only the member TOOL is unpacked); false and TWHY[TOOL]: arch | no_fetch_tool | download | no_sha | checksum | extract
sc_fetch_pinned() {
    local tool=$1 url sha f
    REPLY=""
    sc_arch
    if ! sc_pin "$tool" "$REPLY"; then TWHY[$tool]=arch; return 1; fi
    url=$REPLY; sha=$REPLY2; REPLY=""
    [ -n "$SC_FETCH" ] || { TWHY[$tool]=no_fetch_tool; return 1; }
    f="$SC_BIN/$tool.download"
    sc_prog "$(sc_t "tools · download" "инструменты · загрузка") $tool"
    if ! sc_get "$url" "$f" 60 || [ ! -s "$f" ]; then TWHY[$tool]=download; rm -f -- "$f"; return 1; fi
    sc_sha256 "$f"
    if [ -z "$REPLY" ]; then TWHY[$tool]=no_sha; rm -f -- "$f"; return 1; fi
    if [ "$REPLY" != "$sha" ]; then TWHY[$tool]=checksum; rm -f -- "$f"; REPLY=""; return 1; fi
    case "$url" in
        *.tgz)
            if ! tar -xzf "$f" -C "$SC_BIN" "$tool" 2>/dev/null; then TWHY[$tool]=extract; rm -f -- "$f"; REPLY=""; return 1; fi
            rm -f -- "$f" ;;
        *) mv -f -- "$f" "$SC_BIN/$tool" ;;
    esac
    chmod 755 "$SC_BIN/$tool" 2>/dev/null
    REPLY="$SC_BIN/$tool"
}

# sc_runas_init -> SC_RUNAS: the prefix that runs the network tools as nobody when the check runs as root (setpriv of
# util-linux; it must work here — a user namespace may not map nobody) with a clean environment (env -i: HOME, PATH,
# LANG only — root's variables are not for nobody's /proc/PID/environ); empty otherwise. SC_WORK becomes 0711 (others
# pass through, list nothing) for nobody to reach SC_BIN (0755) and SC_HOME (0700, nobody's)
sc_runas_init() {
    local u g
    SC_RUNAS=()
    [ "$(id -u 2>/dev/null)" = 0 ] && sc_have setpriv || return 0
    u=$(id -u nobody 2>/dev/null); g=$(id -g nobody 2>/dev/null)
    [[ $u =~ ^[0-9]+$ ]] && [[ $g =~ ^[0-9]+$ ]] && [ "$u" != 0 ] || return 0
    setpriv --reuid="$u" --regid="$g" --clear-groups --no-new-privs true >/dev/null 2>&1 || return 0
    chmod 711 "$SC_WORK" && chmod 755 "$SC_BIN" && chown "$u" "$SC_HOME" 2>/dev/null || return 0
    SC_RUNAS=(env -i HOME="$SC_HOME" XDG_CONFIG_HOME="$SC_HOME/.config" PATH=/usr/sbin:/usr/bin:/sbin:/bin LANG=C
        setpriv --reuid="$u" --regid="$g" --clear-groups --no-new-privs)
}

# sc_tool_apt TOOL… — sysbench / fio that the server lacks: «apt-get download» of the package and of what «apt-get -s
# install» says is missing (an upgrade of an installed package is left out) from the signed repository of the system,
# into SC_WORK/apt; «dpkg-deb -x» there; the tool runs with LD_LIBRARY_PATH of the unpacked libraries. No dpkg -i, no
# apt-get update (no lists or old ones — the tool is skipped), no lock; the package cache and the logs of apt go to the
# temporary directory or nowhere. TOOL[t] / TSRC[t]=apt, or TWHY[t]: no_apt | no_package | too_big | download | broken
sc_tool_apt() {
    local d o=() t pkgs all=() need=() size f b libs
    if ! sc_have apt-get || ! sc_have dpkg-deb; then for t in "$@"; do TWHY[$t]=no_apt; done; return 0; fi
    d="$SC_WORK/apt"; mkdir -p "$d/deb" "$d/root" "$d/log" || return 0
    o=(-q -o Debug::NoLocking=1 -o Dir::Cache::pkgcache= -o Dir::Cache::srcpkgcache= -o "Dir::Log=$d/log"
       -o "Dir::Cache::archives=$d/deb")
    [ "$(id -u 2>/dev/null)" = 0 ] && o+=(-o APT::Sandbox::User=root)       # _apt cannot write into our 0700 directory
    for t in "$@"; do
        sc_prog "$(sc_t "tools · apt" "инструменты · apt") $t"
        pkgs=$(sc_to 30 apt-get -s "${o[@]}" install --no-install-recommends "$t" 2>/dev/null \
            | awk '/^Inst / && $3 !~ /^\[/ && $2 ~ /^[a-z0-9][a-z0-9.+:-]*$/ { print $2 }')
        if [ -z "$pkgs" ]; then TWHY[$t]=no_package; continue; fi
        need+=("$t")
        # shellcheck disable=SC2206
        all+=($pkgs)
    done
    [ ${#need[@]} -gt 0 ] || return 0
    size=$(cd "$d/deb" && sc_to 30 apt-get "${o[@]}" download --print-uris "${all[@]}" 2>/dev/null \
        | awk '$3 ~ /^[0-9]+$/ { s += $3 } END { printf "%d\n", s }')
    if [ "${size:-0}" -gt "$SC_APT_MAX" ]; then for t in "${need[@]}"; do TWHY[$t]=too_big; done; return 0; fi
    sc_prog "$(sc_t "tools · apt download" "инструменты · загрузка из apt") ${need[*]}"
    (cd "$d/deb" && sc_to 120 apt-get "${o[@]}" download "${all[@]}") >/dev/null 2>&1
    for f in "$d"/deb/*.deb; do [ -f "$f" ] && dpkg-deb -x "$f" "$d/root" 2>/dev/null; done
    libs=$(find "$d/root" \( -type f -o -type l \) -name '*.so*' 2>/dev/null | sed 's|/[^/]*$||' | sort -u | tr '\n' ':')
    SC_LDPATH=${libs%:}
    for t in "${need[@]}"; do
        b="$d/root/usr/bin/$t"
        if [ ! -x "$b" ]; then TWHY[$t]=download; continue; fi
        if sc_to 10 env LD_LIBRARY_PATH="$SC_LDPATH" "$b" --version >/dev/null 2>&1; then
            TOOL[$t]=$b; TSRC[$t]=apt
        else
            TWHY[$t]=broken
        fi
    done
}

# sc_tools — the tools of the check: TOOL / TSRC (system | apt | pinned) / TWHY per tool; the commands of sysbench and fio
# (SC_SYSBENCH_CMD, SC_FIO_CMD) with the libraries of their own; SC_IPERF_CT — iperf3 knows --connect-timeout
sc_tools() {
    local t lack=()
    for t in sysbench fio iperf3 speedtest; do TOOL[$t]=""; TSRC[$t]=""; TWHY[$t]=""; done
    SC_LDPATH=""; SC_IPERF_CT=0
    # sysbench and fio — the exact methods of Kaskad
    for t in sysbench fio; do
        if [ "${SC_BUILTIN:-0}" = 1 ]; then TWHY[$t]=builtin
        elif sc_have "$t"; then TOOL[$t]=$(command -v "$t"); TSRC[$t]=system
        elif [ "$SC_NOFETCH" = 1 ]; then TWHY[$t]=no_fetch
        else lack+=("$t"); fi
    done
    [ ${#lack[@]} -gt 0 ] && sc_tool_apt "${lack[@]}"
    SC_SYSBENCH=0; SC_FIO=0; SC_SYSBENCH_CMD=(); SC_FIO_CMD=()
    if [ -n "${TOOL[sysbench]}" ]; then
        SC_SYSBENCH=1; SC_SYSBENCH_CMD=("${TOOL[sysbench]}")
        [ "${TSRC[sysbench]}" = apt ] && SC_SYSBENCH_CMD=(env LD_LIBRARY_PATH="$SC_LDPATH" "${TOOL[sysbench]}")
    fi
    if [ -n "${TOOL[fio]}" ]; then
        SC_FIO=1; SC_FIO_CMD=("${TOOL[fio]}")
        [ "${TSRC[fio]}" = apt ] && SC_FIO_CMD=(env LD_LIBRARY_PATH="$SC_LDPATH" "${TOOL[fio]}")
    fi
    [ "$SC_NONET" = 1 ] && { TWHY[iperf3]=skipped; TWHY[speedtest]=skipped; return 0; }
    # iperf3: the system one, else the pinned static build
    if [ "$SC_NOIPERF" = 1 ]; then TWHY[iperf3]=off
    elif sc_have iperf3; then TOOL[iperf3]=$(command -v iperf3); TSRC[iperf3]=system
    elif [ "$SC_NOFETCH" = 1 ]; then TWHY[iperf3]=no_fetch
    elif sc_fetch_pinned iperf3; then TOOL[iperf3]=$REPLY; TSRC[iperf3]=pinned; fi
    if [ -n "${TOOL[iperf3]}" ]; then
        if sc_to 10 "${SC_RUNAS[@]}" "${TOOL[iperf3]}" --version >/dev/null 2>&1; then
            sc_to 10 "${SC_RUNAS[@]}" "${TOOL[iperf3]}" --help 2>&1 | grep -q -- '--connect-timeout' && SC_IPERF_CT=1
        else
            TOOL[iperf3]=""; TSRC[iperf3]=""; TWHY[iperf3]=broken
        fi
    fi
    # Speedtest by Ookla: the pinned CLI only (a «speedtest» of the system may be another program with that name)
    if [ "$SC_NOSPEEDTEST" = 1 ]; then TWHY[speedtest]=off
    elif [ "$SC_MODE" = quick ]; then TWHY[speedtest]=quick
    elif [ "$SC_NOFETCH" = 1 ]; then TWHY[speedtest]=no_fetch
    elif sc_fetch_pinned speedtest; then
        if sc_to 10 "${SC_RUNAS[@]}" "$REPLY" --version >/dev/null 2>&1; then TOOL[speedtest]=$REPLY; TSRC[speedtest]=pinned
        else TWHY[speedtest]=broken; fi
    fi
    return 0
}

# ── CPU ─────────────────────────────────────────────────────────────────────────────────────────────────────────────

# sc_stat -> REPLY «total steal» of the «cpu» line of /proc/stat (user … steal)
sc_stat() {
    REPLY=$(awk '$1 == "cpu" { t = 0; for (i = 2; i <= 9 && i <= NF; i++) t += $i; print t, (NF >= 9 ? $9 : 0); exit }' /proc/stat 2>/dev/null)
}

sc_openssl_runs() {   # ALG RUNS LABEL -> REPLY: the median MB/s of RUNS × «openssl speed -aead -evp ALG -bytes 1420 -seconds 1 -mr»
    local alg=$1 runs=$2 i v vals=()
    for ((i = 1; i <= runs; i++)); do
        sc_prog "$3 $i/$runs"
        v=$(sc_to 30 openssl speed -aead -evp "$alg" -bytes 1420 -seconds 1 -mr 2>/dev/null | sc_parse_openssl 1420)
        sc_isnum "$v" && vals+=("$(awk -v b="$v" 'BEGIN { printf "%.6f\n", b / 1e6 }')")
    done
    REPLY=""
    [ ${#vals[@]} -gt 0 ] && sc_median "${vals[@]}"
}

sc_cpu() {
    local n=${SYS[nproc]} runs secs mt i v vals s0 s1 ossl=0
    sc_stat; s0=$REPLY
    # sysbench — the methods of Kaskad (bench/suite.py cpu_a / cpu_mt): prime 10000, 2 s × 5, median; all threads 10 s
    if [ "$SC_SYSBENCH" = 1 ]; then
        if [ "$SC_MODE" = quick ]; then runs=3; secs=5; else runs=5; secs=10; fi
        vals=()
        for ((i = 1; i <= runs; i++)); do
            sc_prog "$(sc_t "CPU · sysbench, 1 core" "процессор · sysbench, 1 ядро") $i/$runs"
            v=$(sc_to 22 "${SC_SYSBENCH_CMD[@]}" cpu --cpu-max-prime=10000 --threads=1 --time=2 run 2>/dev/null | sc_parse_sysbench_cpu)
            sc_isnum "$v" && vals+=("$v")
        done
        REPLY=""; [ ${#vals[@]} -gt 0 ] && sc_median "${vals[@]}"
        sc_set cpu_1t "$REPLY" sysbench
        sc_prog "$(sc_t "CPU · sysbench, all cores" "процессор · sysbench, все ядра") ($n) ${secs} s"
        v=$(sc_to $((secs + 20)) "${SC_SYSBENCH_CMD[@]}" cpu --cpu-max-prime=10000 --threads="$n" --time="$secs" run 2>/dev/null \
            | sc_parse_sysbench_cpu)
        sc_set cpu_mt "$v" sysbench
    else
        sc_set cpu_1t "" "" no_sysbench
        sc_set cpu_mt "" "" no_sysbench
    fi
    # openssl — chacha20-poly1305 and aes-128-gcm as a tunnel runs them (suite.crypto), 3 runs, the median
    if sc_have openssl; then
        ossl=1
        sc_openssl_runs chacha20-poly1305 3 "$(sc_t "CPU · ChaCha20-Poly1305, 1 core" "процессор · ChaCha20-Poly1305, 1 ядро")"
        sc_set chacha_1t "$REPLY" openssl
        sc_openssl_runs aes-128-gcm 3 "$(sc_t "CPU · AES-128-GCM, 1 core" "процессор · AES-128-GCM, 1 ядро")"
        sc_set aes_1t "$REPLY" openssl
        # all cores: openssl -multi (1 core — the single-core number)
        if [ "$n" -le 1 ]; then
            mt=${V[chacha_1t]}
        else
            sc_prog "$(sc_t "CPU · ChaCha20-Poly1305, all cores" "процессор · ChaCha20-Poly1305, все ядра") ($n)"
            v=$(sc_to 40 openssl speed -multi "$n" -aead -evp chacha20-poly1305 -bytes 1420 -seconds 2 -mr 2>/dev/null \
                | sc_parse_openssl 1420)
            mt=""; sc_isnum "$v" && mt=$(awk -v b="$v" 'BEGIN { printf "%.6f\n", b / 1e6 }')
        fi
    else
        sc_set chacha_1t "" "" no_openssl
        sc_set aes_1t "" "" no_openssl
        mt=""
    fi
    # the derived scale only stands in for cpu_mt: with sysbench it is a raw number
    if [ -n "${V[cpu_mt]}" ]; then
        sc_set chacha_mt "$mt" openssl "" 1
        SF[chacha_mt]=""; SS[chacha_mt]=""; WHY[chacha_mt]=""
    else
        sc_set chacha_mt "$mt" openssl "$([ "$ossl" = 1 ] || echo no_openssl)" 1
    fi
    X[chacha_scale]=""
    if sc_isnum "$mt" && sc_isnum "${V[chacha_1t]}"; then
        X[chacha_scale]=$(awk -v a="$mt" -v b="${V[chacha_1t]}" 'BEGIN { if (b > 0) printf "%.2f\n", a / b }')
    fi
    sc_stat; s1=$REPLY
    X[steal]=$(awk -v a="$s0" -v b="$s1" 'BEGIN { split(a, x, " "); split(b, y, " "); dt = y[1] - x[1]
        if (dt > 0) printf "%.2f\n", 100 * (y[2] - x[2]) / dt }')
    sc_group_score cpu
}

# ── memory ──────────────────────────────────────────────────────────────────────────────────────────────────────────

# sc_mem_judge RATE… -> REPLY: the value, MiB/s — the median of the warm passes (all but the first), and REPLY2: ok |
# implausible (the value is at or above SC_MEM_MAX_MIBS: no single thread reads RAM that fast — a cache effect) |
# unstable (the warm passes differ more than 2×, or the cold first pass is off their median more than 3×); fewer than 3
# passes — both "". Only «ok» is scored.
sc_mem_judge() {
    local out
    out=$(awk -v max="$SC_MEM_MAX_MIBS" 'BEGIN {
        n = ARGC - 1; if (n < 3) exit
        cold = ARGV[1] + 0; m = 0
        for (i = 2; i <= n; i++) v[m++] = ARGV[i] + 0
        for (i = 1; i < m; i++) { x = v[i]; for (j = i - 1; j >= 0 && v[j] > x; j--) v[j + 1] = v[j]; v[j + 1] = x }
        med = (m % 2) ? v[int(m / 2)] : (v[m / 2 - 1] + v[m / 2]) / 2
        verdict = "ok"
        if (med >= max) verdict = "implausible"
        else if (v[0] <= 0 || v[m - 1] / v[0] > 2 || cold <= 0 || cold / med > 3 || med / cold > 3) verdict = "unstable"
        printf "%.2f %s\n", med, verdict
    }' "$@")
    REPLY=${out% *}; REPLY2=${out#* }
    [ -n "$out" ] || { REPLY=""; REPLY2=""; }
}

# sc_mem_tmpfs -> REPLY: the read speed of RAM, MiB/s of every pass, in order (dd of a file in /dev/shm -> /dev/null,
# 1 MiB blocks, one thread: the kernel copies real RAM pages into the buffer of dd); "" and SC_MEMWHY when not measured.
# The file is written data, not a hole (a hole reads as the zero page — the 112 GB/s of dd /dev/zero, 12.11): copies of
# 16 MiB from /dev/urandom. Its size: 1 GiB (quick 512 MiB), at most 1/4 of MemAvailable and of what the cgroup limit
# leaves, at most half of the free /dev/shm, at least 256 MiB (past the L3 cache of any CPU); removed right after.
# Passes until ≥ 1 s of reading (quick 0.3 s) and ≥ 5 passes (quick 3), at most 40.
sc_mem_tmpfs() {
    local avail cg cur shm size want d f seed i passes=0 min tgt sum=0 r out=""
    REPLY=""; SC_MEMWHY=no_shm
    want=1024; [ "$SC_MODE" = quick ] && want=512
    avail=${SYS[avail_kb]:-}; [[ $avail =~ ^[0-9]+$ ]] || { SC_MEMWHY=tool_failed; return 0; }
    avail=$((avail / 1024))
    cg=$(sc_read /sys/fs/cgroup/memory.max); cur=$(sc_read /sys/fs/cgroup/memory.current)
    if [[ $cg =~ ^[0-9]+$ ]] && [[ $cur =~ ^[0-9]+$ ]] && [ "$cg" -gt "$cur" ] && [ $(((cg - cur) / 1048576)) -lt "$avail" ]; then
        avail=$(((cg - cur) / 1048576))
    fi
    size=$((avail / 4)); [ "$size" -gt "$want" ] && size=$want
    [ "$size" -ge 256 ] || { SC_MEMWHY=mem_room; return 0; }
    [ -d /dev/shm ] && [ -w /dev/shm ] || return 0
    [ "$(sc_read /proc/self/mountinfo | sc_fs_type /dev/shm)" = tmpfs ] || return 0
    shm=$(df -Pk /dev/shm 2>/dev/null | awk 'NR == 2 { print $4 }')
    [[ $shm =~ ^[0-9]+$ ]] || return 0
    [ $((shm / 2048)) -lt "$size" ] && size=$((shm / 2048))
    size=$((size / 16 * 16))
    [ "$size" -ge 256 ] || { SC_MEMWHY=mem_room; return 0; }
    d=$(mktemp -d /dev/shm/server-check.XXXXXX 2>/dev/null) || return 0
    SC_WORK3=$d; f="$d/mem"; seed="$d/seed"
    SC_MEMWHY=tool_failed
    head -c 16777216 /dev/urandom >"$seed" 2>/dev/null
    if [ "$(wc -c <"$seed" 2>/dev/null | tr -d ' ')" = 16777216 ]; then
        for ((i = 0; i < size / 16; i++)); do cat -- "$seed" || break; done >"$f" 2>/dev/null
    fi
    rm -f -- "$seed"
    min=5; tgt=1000000; [ "$SC_MODE" = quick ] && { min=3; tgt=300000; }
    while [ -s "$f" ] && [ "$passes" -lt 40 ]; do
        r=$(sc_to 20 dd if="$f" of=/dev/null bs=1M 2>&1 | sc_parse_dd | tail -n 1)
        # a whole pass only (a short file — /dev/shm filled up); «rate µs»
        r=$(awk -v r="$r" -v want=$((size * 1048576)) 'BEGIN { split(r, x, " ")
            if (x[1] == want && x[2] > 0) printf "%.2f %d\n", x[1] / 1048576 / x[2], x[2] * 1e6 }')
        [ -n "$r" ] || break
        out+="${out:+ }${r% *}"; sum=$((sum + ${r#* })); passes=$((passes + 1))
        [ "$passes" -ge "$min" ] && [ "$sum" -ge "$tgt" ] && break
    done
    rm -rf -- "$d"; SC_WORK3=""
    [ "$passes" -ge 3 ] || return 0
    REPLY=$out; SC_MEMWHY=""
}

# sc_mem_set VALUE VERDICT — the built-in memory test: «ok» is scored (≈); otherwise the value stays on screen as a guide
# without a score, and the group is the RAM size alone (sc_report_mem says why)
sc_mem_set() {
    sc_set mem_bw "$1" tmpfs "" 1
    if [ -n "${V[mem_bw]}" ] && [ "$2" != ok ]; then
        SF[mem_bw]=""; SS[mem_bw]=""; WHY[mem_bw]="mem_${2:-unstable}"
    fi
}

sc_mem() {
    local v gb t verdict
    gb=""; sc_isnum "${SYS[mem_kb]}" && gb=$(awk -v k="${SYS[mem_kb]}" 'BEGIN { printf "%.3f\n", k * 1024 / 1e9 }')
    sc_set mem_gb "$gb" meminfo
    if [ "$SC_SYSBENCH" = 1 ]; then
        # = suite.memory: 1 MiB blocks, 1 thread, by the time
        t=5; [ "$SC_MODE" = quick ] && t=3
        sc_prog "$(sc_t "memory · sysbench" "память · sysbench") ${t} s"
        v=$(sc_to $((t + 20)) "${SC_SYSBENCH_CMD[@]}" memory --memory-block-size=1M --memory-total-size=1T --threads=1 --time="$t" run \
            2>/dev/null | sc_parse_sysbench_mem)
        sc_set mem_bw "$v" sysbench
    else
        # ≈ without sysbench: the read of RAM pages by one thread (sc_mem_tmpfs), scored only when plausible and steady
        sc_prog "$(sc_t "memory · read from /dev/shm" "память · чтение из /dev/shm")"
        sc_mem_tmpfs
        if [ -n "$REPLY" ]; then
            # shellcheck disable=SC2086
            sc_mem_judge $REPLY
            v=$REPLY; verdict=$REPLY2
            sc_mem_set "$v" "$verdict"
        else
            sc_set mem_bw "" "" "$SC_MEMWHY"
        fi
    fi
    X[swap_gb]=""; sc_isnum "${SYS[swap_kb]}" && X[swap_gb]=$(sc_gb "${SYS[swap_kb]}" 2)
    sc_group_score mem
}

# ── disk ────────────────────────────────────────────────────────────────────────────────────────────────────────────

# sc_disk_dir -> SC_DDIR: a directory of our own for the test file on a real file system ("" — none); SC_DFS its type
sc_disk_dir() {
    local d typ
    SC_DDIR=""; SC_DFS=""
    for d in "$SC_WORK" "${HOME:-}"; do
        [ -n "$d" ] && [ -d "$d" ] || continue
        if [ "$d" != "$SC_WORK" ]; then
            [ -w "$d" ] || continue
            d=$(mktemp -d "$d/.server-check.XXXXXX" 2>/dev/null) || continue
            SC_WORK2=$d
        fi
        typ=$(sc_read /proc/self/mountinfo | sc_fs_type "$(cd "$d" && pwd -P)")
        SC_DFS=$typ
        case "$SC_BAD_FS" in *" $typ "*) continue ;; esac
        [ -n "$typ" ] || continue
        SC_DDIR=$d; return 0
    done
    return 1
}

sc_fio() {   # NAME FILE SIZE TIMEOUT ARGS… -> REPLY: the JSON of fio («--direct=1 --ioengine=libaio» as suite._fio)
    local f=$2 size=$3 to=$4; shift 4
    REPLY=$(sc_to "$to" "${SC_FIO_CMD[@]}" --name=kbench --filename="$f" --size="$size" --direct=1 --ioengine=libaio \
        --output-format=json "$@" 2>/dev/null)
}

sc_dd_rate() {   # SECS DD-ARGS… -> REPLY: MB/s of what dd copied (dd prints its statistics on SIGINT too); 0 bytes — ""
    local s=$1 r; shift
    r=$(sc_to_int "$s" dd "$@" 2>&1 | sc_parse_dd | tail -n 1)
    REPLY=$(awk -v r="$r" 'BEGIN { split(r, x, " "); if (x[1] > 0 && x[2] > 0) printf "%.2f\n", x[1] / 1e6 / x[2] }')
}

sc_disk() {
    local why="" free size rt f v approx=0 out
    X[disk_seq_r]=""; X[disk_seq_w]=""; X[disk_file_mb]=""; X[disk_dir_fs]=""
    if ! sc_disk_dir; then
        why=disk_fs
    else
        X[disk_dir_fs]=$SC_DFS
        free=$(df -Pk "$SC_DDIR" 2>/dev/null | awk 'NR == 2 { print $4 }')
        [[ $free =~ ^[0-9]+$ ]] || free=0
        # the file: 1 GiB with ≥ 3 GiB free (as Kaskad), 256 MiB with ≥ 1.25 GiB (and always in a quick check), else none
        if [ "$SC_MODE" != quick ] && [ "$free" -ge $((3 * 1024 * 1024)) ]; then size=1024
        elif [ "$free" -ge $((1280 * 1024)) ]; then size=256; approx=1
        else why=disk_space; fi
    fi
    if [ -n "$why" ]; then
        for v in disk_rr disk_rw disk_fsync; do sc_set "$v" "" "" "$why"; done
        sc_group_score disk
        return 0
    fi
    X[disk_file_mb]=$size
    f="$SC_DDIR/server-check.tmp"
    if [ "$SC_MODE" = quick ]; then rt=3; else rt=10; fi
    local fio_ok=0
    if [ "$SC_FIO" = 1 ]; then
        # = suite.disk: 4k random read / write at queue depth 32, write 4k with fsync after each, read 1 MiB
        sc_prog "$(sc_t "disk · fio random read 4K" "диск · fio, случайное чтение 4K") (${size} MiB)"
        sc_fio rr "$f" "${size}M" $((rt + 60)) --rw=randread --bs=4k --iodepth=32 --runtime="$rt" --time_based
        v=$(sc_parse_fio read iops <<<"$REPLY")
        if sc_isnum "$v"; then
            fio_ok=1
            sc_set disk_rr "$v" fio "" "$approx"
            sc_prog "$(sc_t "disk · fio random write 4K" "диск · fio, случайная запись 4K")"
            sc_fio rw "$f" "${size}M" $((rt + 25)) --rw=randwrite --bs=4k --iodepth=32 --io_size="${size}M" --runtime="$rt"
            sc_set disk_rw "$(sc_parse_fio write iops <<<"$REPLY")" fio "" "$approx"
            sc_prog "$(sc_t "disk · fio write + fsync" "диск · fio, запись + fsync")"
            sc_fio fs "$f" 16M 35 --rw=write --bs=4k --fsync=1 --runtime="$([ "$SC_MODE" = quick ] && echo 3 || echo 5)"
            v=$(sc_parse_fio write iops <<<"$REPLY")
            sc_set disk_fsync "$(awk -v i="$v" 'BEGIN { if (i > 0) printf "%.6f\n", 1000 / i }')" fio
            if [ "$SC_MODE" != quick ]; then        # as Kaskad: the sequential read in a full check only
                sc_prog "$(sc_t "disk · fio sequential 1M" "диск · fio, последовательно 1M")"
                sc_fio sr "$f" "${size}M" 35 --rw=read --bs=1M --runtime=5 --time_based
                X[disk_seq_r]=$(sc_parse_fio read bw <<<"$REPLY")
                sc_fio sw "$f" "${size}M" 35 --rw=write --bs=1M --runtime=5 --time_based
                X[disk_seq_w]=$(sc_parse_fio write bw <<<"$REPLY")
            fi
        fi
        rm -f -- "$f"
    fi
    if [ "$fio_ok" != 1 ]; then
        local nofio=no_fio; [ "$SC_FIO" = 1 ] && nofio=tool_failed
        sc_set disk_rr "" "" "$nofio"; sc_set disk_rw "" "" "$nofio"
        # dd: sequential write and read of the file with direct I/O, then 4K writes over it with O_DSYNC (each write on
        # the disk before the next — what fio --fsync=1 measures on a laid-out file)
        sc_prog "$(sc_t "disk · dd sequential write" "диск · dd, последовательная запись") (${size} MiB)"
        X[disk_direct]=1
        sc_dd_rate 30 if=/dev/zero of="$f" bs=1M count="$size" oflag=direct conv=fdatasync
        if [ -z "$REPLY" ]; then          # no O_DIRECT here (dd copied nothing): the page cache, fdatasync at the end
            X[disk_direct]=0
            sc_dd_rate 30 if=/dev/zero of="$f" bs=1M count="$size" conv=fdatasync
        fi
        X[disk_seq_w]=$REPLY
        if [ "${X[disk_direct]}" = 1 ] && [ -s "$f" ]; then
            sc_prog "$(sc_t "disk · dd sequential read" "диск · dd, последовательное чтение")"
            sc_dd_rate 30 if="$f" of=/dev/null bs=1M iflag=direct
            X[disk_seq_r]=$REPLY
        fi
        local fsz=0; [ -f "$f" ] && fsz=$(wc -c <"$f"); fsz=${fsz//[!0-9]/}; fsz=${fsz:-0}
        if [ "$fsz" -ge 4096 ]; then
            # over the written file only (conv=notrunc, no growth: a growing file adds the journal to every write)
            sc_prog "$(sc_t "disk · dd write + sync 4K" "диск · dd, запись + sync 4K")"
            local flag=dsync; [ "${X[disk_direct]}" = 1 ] && flag=direct,dsync
            out=$(sc_to_int "$([ "$SC_MODE" = quick ] && echo 3 || echo 5)" dd if=/dev/zero of="$f" bs=4k \
                count=$((fsz / 4096)) oflag="$flag" conv=notrunc 2>&1)
            v=$(sc_parse_dd <<<"$out" | tail -n 1)
            local recs; recs=$(sc_parse_dd_records <<<"$out")
            sc_set disk_fsync "$(awk -v r="$v" -v n="$recs" 'BEGIN { split(r, x, " ")
                if (n > 0 && x[1] > 0 && x[2] > 0) printf "%.6f\n", x[2] * 1000 / n }')" dd "" 1
        else
            sc_set disk_fsync "" "" tool_failed
        fi
        rm -f -- "$f"
    fi
    sc_group_score disk
}

# ── network ─────────────────────────────────────────────────────────────────────────────────────────────────────────

# sc_tcp_rtt HOST PORT N -> stdout: the time of the TCP handshake, ms, a line per connection that came up (bash /dev/tcp
# in a child of its own under timeout: nothing is sent; the address is the first the system resolver prefers)
sc_tcp_rtt() {
    local host=$1 port=$2 n=$3 ip i us fam out=""
    for fam in ahosts ahostsv4; do
        ip=$(sc_to 5 getent "$fam" "$host" 2>/dev/null | awk 'NR == 1 { print $1 }')
        [ -n "$ip" ] || continue
        for ((i = 0; i < n; i++)); do
            # the child measures itself: its start is not in the time; the output — microseconds
            # shellcheck disable=SC2016
            us=$(sc_to 3 bash -c 'c() { if [ -n "${EPOCHREALTIME:-}" ]; then r=${EPOCHREALTIME/[.,]/}; else r=$(date +%s%6N); fi; }
                c; a=$r; exec 3<>"/dev/tcp/$0/$1" || exit 1; c; exec 3>&-; echo $((r - a))' "$ip" "$port" 2>/dev/null)
            [[ $us =~ ^[0-9]+$ ]] && out+="$us"$'\n'
            sleep 0.2
        done
        if [ -n "$out" ]; then
            printf '%s' "$out" | awk '{ printf "%.3f\n", $1 / 1000 }'
            return 0
        fi
        { [ "$fam" = ahostsv4 ] || [ "$ip" = "${ip#*:}" ]; } && return 1      # IPv4 already — no second try
    done
    return 1
}

# the latency of the server (score_abs «rtt», «jitter», «loss»): ICMP echo to the anchors of Kaskad, 0.2 s apart (as
# gw/hostping); per anchor the median, the mean |Δ|, the loss; the node — _reduce over the anchors. No ping — TCP 53.
sc_net_rtt() {
    local n a i out rtts meds=() jits=() jn=0 losses=() sent=0 via=icmp pids=()
    if [ "$SC_MODE" = quick ]; then n=10; else n=20; fi
    sc_prog "$(sc_t "network · latency to 1.1.1.1, 8.8.8.8, 77.88.8.8" "сеть · отклик до 1.1.1.1, 8.8.8.8, 77.88.8.8")"
    i=0
    if sc_have ping; then
        for a in $SC_ANCHORS; do
            ping -n -c "$n" -i 0.2 -W 2 -w $((n / 5 + 4)) "$a" >"$SC_WORK/ping.$i" 2>&1 &
            pids+=($!); i=$((i + 1))
        done
        # its own jobs only (none — no wait: a bare «wait» would wait for the ticker of the live screen too)
        [ ${#pids[@]} -eq 0 ] || wait "${pids[@]}"
        i=0
        for a in $SC_ANCHORS; do
            out=$(sc_parse_ping <"$SC_WORK/ping.$i"); i=$((i + 1))
            read -r -a rtts <<<"$out"
            [ ${#rtts[@]} -ge 2 ] || continue
            sent=$((sent + rtts[0]))
            losses+=("$(awk -v s="${rtts[0]}" -v r="${rtts[1]}" 'BEGIN { if (s > 0) printf "%.4f\n", 100 * (s - r) / s }')")
            [ ${#rtts[@]} -ge 3 ] || continue
            sc_median "${rtts[@]:2}"; meds+=("$REPLY")
            sc_jitter "${rtts[@]:2}"; [ -n "$REPLY" ] && { jits+=("$REPLY"); jn=$((jn + REPLY2)); }
        done
    fi
    if [ ${#meds[@]} -eq 0 ]; then         # no ping, no rights or ICMP closed: the TCP handshake to port 53 of the anchors
        via=tcp; losses=(); jits=(); jn=0; sent=0; i=0
        pids=()
        for a in $SC_ANCHORS; do sc_tcp_rtt "$a" 53 "$((n / 2))" >"$SC_WORK/tcp53.$i" & pids+=($!); i=$((i + 1)); done
        [ ${#pids[@]} -eq 0 ] || wait "${pids[@]}"
        i=0
        for a in $SC_ANCHORS; do
            mapfile -t rtts <"$SC_WORK/tcp53.$i"; i=$((i + 1))
            [ ${#rtts[@]} -ge 1 ] || continue
            sc_median "${rtts[@]}"; meds+=("$REPLY")
            sc_jitter "${rtts[@]}"; [ -n "$REPLY" ] && { jits+=("$REPLY"); jn=$((jn + REPLY2)); }
        done
    fi
    local ap=0; [ "$via" = tcp ] && ap=1
    if [ ${#meds[@]} -gt 0 ]; then sc_reduce "${meds[@]}"; sc_set rtt "$REPLY" "$via" "" "$ap"
    else sc_set rtt "" "" no_answer; fi
    if [ ${#jits[@]} -gt 0 ]; then sc_reduce "${jits[@]}"; sc_set jitter "$REPLY" "$via" "" "$ap" "$jn"
    else sc_set jitter "" "" no_answer; fi
    if [ ${#losses[@]} -gt 0 ]; then sc_reduce "${losses[@]}"; sc_set loss "$REPLY" icmp "" 0 "$sent"
    else sc_set loss "" "" "$([ "$via" = tcp ] && echo no_icmp || echo no_answer)"; fi
}

# DNS (score_abs «dns», = suite.dns): 10 random names under example.com through the system resolver, the median, ms; an
# unanswered name counts as 2 s; fewer than 5 answers — not measured. Closer to the one UDP query of suite.dns_once than
# a bare getent: the names end with a dot (a name that does not exist is otherwise asked again with every «search»
# domain of resolv.conf — 2–3 queries in one sample); one more name goes first and is not counted (the resolver learns
# the servers of the zone: a cold cache adds their lookups — suite.dns has no warm-up, its median of 10 absorbs one cold
# sample; here getent also loads its libraries on that call); the start of getent itself (the least of 3 runs on a
# numeric address — no lookup) is taken off every sample, at least 0.05 ms (the floor of the scale) stays.
sc_net_dns() {
    local i name rc ms base vals=() got=0 t0
    vals=()
    for ((i = 1; i <= 3; i++)); do
        sc_clock; t0=$SC_NOW
        sc_to 3 getent ahostsv4 127.0.0.1 >/dev/null 2>&1
        sc_clock; vals+=("$((SC_NOW - t0))")
    done
    base=$(printf '%s\n' "${vals[@]}" | awk 'NR == 1 || $1 < m { m = $1 } END { printf "%.3f\n", (m > 0 && m < 50000) ? m / 1000 : 0 }')
    vals=()
    for ((i = 0; i <= 10; i++)); do
        sc_prog "$(sc_t "network · DNS" "сеть · DNS") $i/10"
        name=$(printf 'sc-%04x%04x%04x.%s.' "$RANDOM" "$RANDOM" "$RANDOM" "$SC_DNS_ZONE")
        sc_clock; t0=$SC_NOW
        sc_to 3 getent ahostsv4 "$name" >/dev/null 2>&1; rc=$?
        sc_clock
        [ "$i" = 0 ] && continue
        ms=$(awk -v d=$((SC_NOW - t0)) -v b="$base" 'BEGIN { m = d / 1000; if (m <= 2000) { m -= b; if (m < 0.05) m = 0.05 }
            printf "%.3f\n", m }')
        if { [ "$rc" = 0 ] || [ "$rc" = 2 ]; } && awk -v m="$ms" 'BEGIN { exit !(m <= 2000) }'; then
            vals+=("$ms"); got=$((got + 1))
        else
            vals+=(2000)
        fi
    done
    X[dns_base_ms]=$base
    if [ "$got" -ge 5 ]; then sc_median "${vals[@]}"; sc_set dns "$REPLY" getent "" 1
    else sc_set dns "" "" no_answer; fi
}

# ── the speed: iperf3 to public points, Speedtest by Ookla ──────────────────────────────────────────────────────────

# iperf3 --json (stdin) -> «ok MBIT/S RTT RETR» — the sum of the receiver (end.sum_received: our own count in a download,
# the server's in an upload; the omitted seconds of -O are not in it), the least min_rtt of the sending streams, ms (ours —
# an upload; «-» when there is none) and the retransmits of the sender (end.sum_sent: ours in an upload, the server's in a
# download; «-» when not given); or «busy» (the server runs another test), «no_answer» (no connection, no name), «cut»
# (iperf3 ended by the time limit of sc_iperf: the point hung — the end of the test never came), «error» (anything
# else: the control channel closed, an empty output)
sc_parse_iperf() {
    awk '
        function val(s) { sub(/^[^:]*:[ \t]*/, "", s); sub(/[ \t]*,?[ \t\r]*$/, "", s); gsub(/^"|"$/, "", s); return s }
        /^[ \t]*"error"[ \t]*:/ { err = val($0) }
        /"sum_received"[ \t]*:/ { sr = 1 }
        sr && /"bits_per_second"[ \t]*:/ { bps = val($0); sr = 0 }
        /"sum_sent"[ \t]*:/ { ss = 1 }
        ss && /"retransmits"[ \t]*:/ { rt = val($0); ss = 0 }
        ss && /^[ \t]*\}/ { ss = 0 }
        /"min_rtt"[ \t]*:/ { v = val($0) + 0; if (v > 0 && (mr == "" || v < mr)) mr = v }
        END {
            if (err != "") {
                e = tolower(err)
                if (e ~ /busy/) print "busy"
                else if (e ~ /^interrupt|terminated by signal/) print "cut"
                else if (e ~ /unable to connect|refused|no route|timed out|unreachable|resolve|name or service|not known/) print "no_answer"
                else print "error"
                exit
            }
            if (bps ~ /^[0-9.eE+-]+$/ && bps + 0 > 0) {
                printf "ok %.3f %s %s\n", bps / 1e6, (mr != "" ? sprintf("%.3f", mr / 1000) : "-"), (rt ~ /^[0-9]+$/ ? rt : "-"); exit
            }
            print "error"
        }'
}

# sc_net_budget_init -> SC_NET_BUDGET (bytes; 0 — no limit) of the mode, SC_NET_BUDGET_GB when it is a number
sc_net_budget_init() {
    local gb
    case "$SC_POINTS" in quick) gb=5 ;; long) gb=60 ;; *) gb=30 ;; esac
    [[ ${SC_NET_BUDGET_GB:-} =~ ^[0-9]{1,6}$ ]] && gb=$((10#$SC_NET_BUDGET_GB))
    SC_NET_BUDGET=$((gb * 1000000000)); SC_NET_BYTES=0; SC_NET_RCAP=0; SC_NET_NCAP=0
}

# sc_net_cap_check I DIR -> SC_NET_RCAP: the cap of the next run, Mbit/s in all (0 — none). With a budget the run may
# take what is left of it once every later run of the table has its share at SC_NET_CAPMBIT (1 Gbit/s), never less
# than that: the sum stays within the budget (the first run too — nothing is known of the server yet), the first rows
# (the near points that give the Канал score) get the most. 100 Gbit/s and more — no cap. 1 % of the budget is kept
# back: -b holds a run within 0.2 % of its plan (12.11, probes to Moscow and New York: the far one is slow in the
# omitted seconds and catches up after them, the sum of the run is the plan), and a try that broke off mid-run and is
# tried again is not planned (one quick run on the stand: 5.02 GB of 5).
sc_net_cap_check() {
    local left secs avail r u=0
    SC_NET_RCAP=0
    [ "${SC_NET_BUDGET:-0}" -gt 0 ] || return 0
    [ "$2" = up ] && u=1
    left=$((2 * (SPN - $1) - u)); [ "$left" -ge 1 ] || left=1
    sc_iperf_len; secs=$((${REPLY% *} + ${REPLY#* }))
    avail=$((SC_NET_BUDGET - SC_NET_BUDGET / 100 - SC_NET_BYTES - (left - 1) * SC_NET_CAPMBIT * 125000 * secs))
    r=0; [ "$avail" -gt 0 ] && r=$((avail * 8 / secs / 1000000))
    [ "$r" -lt "$SC_NET_CAPMBIT" ] && r=$SC_NET_CAPMBIT
    [ "$r" -ge 100000 ] && r=0
    SC_NET_RCAP=$r
    return 0
}

# sc_net_count BYTES — the traffic of a run added (SC_NET_BYTES)
sc_net_count() {
    [[ ${1:-} =~ ^[0-9]{1,15}$ ]] || return 0
    SC_NET_BYTES=$((SC_NET_BYTES + 10#$1))
    return 0
}

# the bytes of an iperf3 --json run: the sums of its intervals, the omitted ones too (what went over the wire)
sc_parse_iperf_bytes() {
    awk '
        /^[ \t]*"intervals"[ \t]*:/ { iv = 1 }
        /^[ \t]*"end"[ \t]*:[ \t]*\{/ { iv = 0 }          # the end object (the «end» of an interval is a number)
        iv && /"sum"[ \t]*:/ { want = 1; next }
        iv && want && /"bytes"[ \t]*:/ { v = $0; sub(/^[^:]*:[ \t]*/, "", v); sub(/[ \t]*,?[ \t\r]*$/, "", v); if (v ~ /^[0-9]+$/) b += v; want = 0 }
        END { printf "%.0f\n", b + 0 }'
}

# sc_iperf_len -> REPLY «T O»: the seconds of a run counted (-t) and omitted before them (-O): the default 5 + 2 (13 rows
# in ≈ 3 min 40 s; 5 s at 8 streams hold steady after the 2 s of TCP ramp-up), the quick check 4 + 1
sc_iperf_len() { if [ "$SC_MODE" = quick ]; then REPLY="4 1"; else REPLY="5 2"; fi; }

# sc_iperf HOST PORT down|up -> REPLY: what sc_parse_iperf says of one run (download = -R: the point sends); a hard
# time limit over iperf3's own (a point that hangs ends as «error»)
sc_iperf() {
    local secs omit a=() st f="$SC_WORK/iperf.json"
    sc_iperf_len; secs=${REPLY% *}; omit=${REPLY#* }
    [ "$3" = down ] && a+=(-R)
    [ "$SC_IPERF_CT" = 1 ] && a+=(--connect-timeout 3000)
    [ "${SC_NET_RCAP:-0}" -gt 0 ] && a+=(-b "$((SC_NET_RCAP / SC_STREAMS))M")       # the cap of the budget, each stream
    a+=("$SC_IPFAM")
    sc_to $((secs + omit + 8)) "${SC_RUNAS[@]}" "${TOOL[iperf3]}" -c "$1" -p "$2" -P "$SC_STREAMS" -t "$secs" \
        -O "$omit" -J "${a[@]}" >"$f" 2>/dev/null
    st=$(sc_parse_iperf <"$f")
    sc_net_count "$(sc_parse_iperf_bytes <"$f")"
    rm -f -- "$f"
    REPLY=${st:-error}
}

# sc_past END -> true when the clock is past END (µs)
sc_past() { sc_clock; [ "$SC_NOW" -ge "$1" ]; }

# sc_iperf_row I — the row I of the table: its point, and when the download there gives nothing (down, busy, an
# error) — its fallback points in order (SP_ALT), each from the start: the row then shows the point that answered
# (SP_FB — how many were passed). A fallback is not tried after the time of the speed part.
sc_iperf_row() {
    local i=$1 rest alt cen cru oen oru host ports cap j used
    rest=${SP_ALT[$i]:-}
    sc_iperf_point "$i" "${rest:+1}"
    while [ "${SP_DST[$i]}" != ok ] && [ "${SP_DST[$i]}" != time ] && [ -n "$rest" ] && ! sc_past "$SC_SPEED_END"; do
        alt=${rest%%;*}; if [ "$alt" = "$rest" ]; then rest=""; else rest=${rest#*;}; fi
        IFS=, read -r cen cru oen oru host ports cap <<<"$alt"
        used=0; for ((j = 0; j < SPN; j++)); do [ "$j" != "$i" ] && [ "${SP_HOST[$j]}" = "$host" ] && used=1; done
        [ "$used" = 1 ] && continue          # another row shows that point (its own or its fallback): not twice
        SP_CEN[i]=$cen; SP_CRU[i]=$cru; SP_OEN[i]=$oen; SP_ORU[i]=$oru; SP_HOST[i]=$host; SP_PORTS[i]=$ports; SP_CAP[i]=${cap:-0}
        if [ "$SC_LANG" = ru ]; then SP_CITY[i]=$cru; else SP_CITY[i]=$cen; fi
        SP_FB[i]=$((${SP_FB[$i]:-0} + 1))
        sc_row_clear "$i"
        sc_ping_points "$i"
        sc_iperf_point "$i" "${rest:+1}"
    done
}

# sc_row_clear I — the measured values of the row I emptied (a new point in it)
sc_row_clear() {
    local i=$1
    SP_DOWN[i]=""; SP_UP[i]=""; SP_PING[i]=""; SP_DST[i]="time"; SP_UST[i]="time"; SP_DLB[i]=0; SP_ULB[i]=0
    SP_PORT[i]=""; SP_MINRTT[i]=""; SP_RDOWN[i]=""; SP_RUP[i]=""
    SP_PAVG[i]=""; SP_PMED[i]=""; SP_PJIT[i]=""; SP_PLOSS[i]=""; SP_PSAMP[i]=""; SP_PN[i]=0
}

# sc_iperf_now I DIR LAST TRY (the number of the try about to run) — the progress line of an iperf3 run («Москва · МТС · ↓ скачивание · 5 с · 8 потоков»)
sc_iperf_now() {
    local t txt
    sc_iperf_len; t=${REPLY% *}
    if [ "$SC_LANG" = ru ]; then txt=$'\x01b'"${SP_CRU[$1]:-} · ${SP_ORU[$1]:-}"$'\x01/\x01d · '; else txt=$'\x01b'"${SP_CEN[$1]:-} · ${SP_OEN[$1]:-}"$'\x01/\x01d · '; fi
    if [ "$3" = busy ]; then txt+="$(sc_t "busy, another test — try $4 of 4" "занят чужим тестом, попытка $4 из 4")"
    elif [ "$2" = down ]; then txt+="↓ $(sc_t "download · $t s · $SC_STREAMS streams" "скачивание · $t с · $SC_STREAMS потоков")"
    else txt+="↑ $(sc_t "upload · $t s · $SC_STREAMS streams" "отдача · $t с · $SC_STREAMS потоков")"; fi
    if [ "${SC_LIVE:-0}" = 1 ]; then sc_prog "$txt"$'\x01/'; sc_live_frame; else sc_vis "$txt"; sc_prog "$REPLY"; fi
}

# sc_iperf_point I [FB] — the current point of the row I: the download, then the upload on the same port (FB=1 — a
# fallback point waits: no download, no upload try here, the upload «skipped»). Busy — the next
# port of its range (a point of one port — the same one) after 1 s, at most 4 tries a direction; an error (the
# control channel closed) or a run cut by the time limit (a port of the point hung at the start: 12.11 st.samara —
# 2 of 4 tries, no interval, other ports fine) — one more try so, on the next port (a point of one port: none, a hang
# there repeats); no connection twice (or once on a point of one port) — the point is down, its upload is not tried.
# No try starts after the time of the point (2 runs + 20 s) or of the speed part. The traffic budget (sc_net_cap_check)
# caps a run (-b); a run that reaches 90 % of its cap is «not less than».
sc_iperf_point() {
    local i=$1 lo hi n k p="" st v rt rr tries dir end dead=0 secs capr cd=0 cu=0 c cap
    lo=${SP_PORTS[$i]%-*}; hi=${SP_PORTS[$i]#*-}; n=$((hi - lo + 1)); [ "$n" -ge 1 ] || n=1
    k=$((RANDOM % n))
    sc_iperf_len; secs=$(( ${REPLY% *} + ${REPLY#* } ))
    sc_clock; end=$((SC_NOW + (2 * secs + 20) * 1000000)); [ "$end" -gt "$SC_SPEED_END" ] && end=$SC_SPEED_END
    for dir in down up; do
        st="time"; tries=0
        while [ "$dead" = 0 ] && ! sc_past "$end"; do
            p=$((lo + k % n))
            SC_LROW=$i; SC_LTRY=$((tries + 1)); if [ "$st" = busy ]; then SC_LDIR=busy; else SC_LDIR=$dir; fi
            sc_iperf_now "$i" "$dir" "$st" "$SC_LTRY"
            sc_net_cap_check "$i" "$dir"; capr=${SC_NET_RCAP:-0}
            sc_iperf "${SP_HOST[$i]}" "$p" "$dir"
            st=${REPLY%% *}; tries=$((tries + 1))
            case "$st" in
                ok) break ;;
                no_answer) if [ "$n" -lt 2 ] || [ "$tries" -ge 2 ]; then dead=1; break; fi ;;
                busy) [ "$tries" -ge 4 ] && break ;;
                cut) st=error; if [ "$n" -lt 2 ] || [ "$tries" -ge 2 ]; then break; fi ;;
                *) [ "$tries" -ge 2 ] && break ;;
            esac
            k=$((k + 1)); sleep 1
        done
        [ "$dead" = 1 ] && st=no_answer
        [ -n "$p" ] && SP_PORT[i]=$p
        if [ "$st" = ok ]; then
            read -r v rt rr <<<"${REPLY#ok }"
            [ "$rr" = - ] && rr=""
            if [ "$dir" = down ]; then SP_DOWN[i]=$v; SP_RDOWN[i]=$rr; cd=$capr
            else SP_UP[i]=$v; SP_RUP[i]=$rr; cu=$capr; [ "$rt" != - ] && SP_MINRTT[i]=$rt; fi
        fi
        if [ "$dir" = down ]; then SP_DST[i]=$st; else SP_UST[i]=$st; fi
        if [ "$dir" = down ] && [ "$st" != ok ] && [ "$st" != time ] && [ "${2:-}" = 1 ]; then SP_UST[i]=skipped; break; fi
    done
    # «not less than»: the run gave 90 % or more of what held it — the port of the point (cap known; slower than the
    # server) or the cap of the traffic budget (the lower of the two)
    for dir in down up; do
        if [ "$dir" = down ]; then v=${SP_DOWN[$i]}; c=$cd; else v=${SP_UP[$i]}; c=$cu; fi
        cap=${SP_CAP[$i]:-0}
        if [ "$c" -gt 0 ] && { [ "$cap" -eq 0 ] || [ "$cap" -gt "$c" ]; }; then cap=$c; else c=0; fi
        if [ "$cap" -gt 0 ] && [ -n "$v" ] && awk -v x="$v" -v c="$cap" 'BEGIN { exit !(x >= 0.9 * c) }'; then
            if [ "$dir" = down ]; then SP_DLB[i]=1; else SP_ULB[i]=1; fi
            [ "$c" -gt 0 ] && SC_NET_NCAP=$((SC_NET_NCAP + 1))       # held by the budget, not by the port
        fi
    done
    # the ping of the row: the ICMP median of sc_ping_points, else the least min_rtt of our upload streams
    if [ -n "${SP_PMED[$i]:-}" ]; then SP_PING[i]=${SP_PMED[$i]}; elif [ -n "${SP_MINRTT[$i]}" ]; then SP_PING[i]=${SP_MINRTT[$i]}; fi
}

# sc_ping_points [I…] — the idle latency of every point of the table (or of the rows I…), all at once before the speed
# (20 ICMP echoes 0.2 s apart each): SP_PSAMP the replies (ms, in order — for a sparkline), SP_PN their number, SP_PAVG the
# mean, SP_PJIT the jitter (the mean |Δ| of neighbour replies, sc_jitter — gw/hostmon), SP_PLOSS the loss, %. No ping, no
# rights or ICMP closed — empty (the table then shows the min_rtt of iperf3).
sc_ping_points() {
    local i out r=() rows=("$@") pids=()
    sc_have ping || return 0
    if [ ${#rows[@]} -eq 0 ]; then for ((i = 0; i < SPN; i++)); do rows+=("$i"); done; fi
    sc_prog "$(sc_t "network · ping of the points" "сеть · пинг точек") (20 × $(sc_t "0.2 s" "0,2 с"))"
    for i in "${rows[@]}"; do
        ping "$SC_IPFAM" -n -c 20 -i 0.2 -W 2 -w 8 "${SP_HOST[$i]}" >"$SC_WORK/pp.$i" 2>&1 &
        pids+=($!)
    done
    [ ${#pids[@]} -eq 0 ] || wait "${pids[@]}"
    for i in "${rows[@]}"; do
        out=$(sc_parse_ping <"$SC_WORK/pp.$i")
        read -r -a r <<<"$out"
        [ ${#r[@]} -ge 2 ] && [ "${r[0]}" -gt 0 ] 2>/dev/null || continue
        SP_PLOSS[i]=$(awk -v s="${r[0]}" -v g="${r[1]}" 'BEGIN { printf "%.1f\n", 100 * (s - g) / s }')
        SP_PN[i]=$(( ${#r[@]} - 2 ))
        [ "${SP_PN[$i]}" -ge 1 ] || continue
        SP_PSAMP[i]="${r[*]:2}"
        SP_PAVG[i]=$(printf '%s\n' "${r[@]:2}" | awk '{ s += $1; n++ } END { if (n) printf "%.3f\n", s / n }')
        sc_median "${r[@]:2}"; SP_PMED[i]=$REPLY
        sc_jitter "${r[@]:2}"; [ -n "$REPLY" ] && SP_PJIT[i]=$(awk -v j="$REPLY" 'BEGIN { printf "%.3f\n", j }')
    done
}

# sc_row_done I — a row of the table is complete: SC_ON_ROW (a function of this script, empty by default; sc_main clears
# what the environment brought) is called with I, for a renderer that prints the rows one by one as the points finish
sc_row_done() {
    SC_LDONE=$(($1 + 1)); SC_LROW=-1
    if [ -n "${SC_ON_ROW:-}" ] && declare -F "$SC_ON_ROW" >/dev/null; then "$SC_ON_ROW" "$1"; fi
    return 0
}

# sc_speed_points -> the rows of the table (SP_*) from SC_IPERF_POINTS by the set of this check: q | b | l
sc_speed_points() {
    local set id sets reg cen cru oen oru host ports cap last=""
    case "$SC_POINTS" in quick) set=q ;; long) set=l ;; *) set=b ;; esac
    SPN=0
    while IFS='|' read -r id sets reg cen cru oen oru host ports cap; do
        [ -n "$id" ] || continue
        if [ "$sets" = "~" ]; then          # a fallback point of the row above (same id)
            [ "$id" = "$last" ] && SP_ALT[SPN - 1]+="${SP_ALT[$((SPN - 1))]:+;}$cen,$cru,$oen,$oru,$host,$ports,${cap:-0}"
            continue
        fi
        last=""
        [[ $sets == *"$set"* ]] || continue
        last=$id
        SP_ID[SPN]=$id; SP_M[SPN]=iperf3; SP_REG[SPN]=$reg; SP_HOST[SPN]=$host; SP_PORTS[SPN]=$ports; SP_CAP[SPN]=${cap:-0}
        SP_CEN[SPN]=$cen; SP_CRU[SPN]=$cru; SP_OEN[SPN]=$oen; SP_ORU[SPN]=$oru; SP_ALT[SPN]=""; SP_FB[SPN]=0
        if [ "$SC_LANG" = ru ]; then SP_CITY[SPN]=$cru; else SP_CITY[SPN]=$cen; fi
        sc_row_clear "$SPN"
        SPN=$((SPN + 1))
    done <<<"$SC_IPERF_POINTS"
}

# sc_net_iperf — the table: every point of the set in order within the time of the speed part (quick 60 s, --full 260 s,
# --all 600 s; a row the time did not reach stays «time»)
sc_net_iperf() {
    local i budget
    sc_speed_points
    SC_NET_BYTES=0; SC_NET_RCAP=0; SC_NET_NCAP=0
    case "$SC_POINTS" in quick) budget=60 ;; long) budget=600 ;; *) budget=260 ;; esac
    sc_clock; SC_SPEED_END=$((SC_NOW + budget * 1000000))
    # IPv4, unless only IPv6 reaches the Internet here (a route alone does not say it works: the GET of sc_sysinfo does)
    SC_IPFAM=-4
    if [ "${SYS[ipv4_ok]:-${SYS[ipv4]}}" != 1 ] && [ "${SYS[ipv6_ok]:-${SYS[ipv6]}}" = 1 ]; then SC_IPFAM=-6; fi
    sc_ping_points
    for ((i = 0; i < SPN; i++)); do
        if ! sc_past "$SC_SPEED_END"; then sc_iperf_row "$i"
        elif [ -n "${SP_PMED[$i]:-}" ]; then SP_PING[i]=${SP_PMED[$i]}; fi
        sc_row_done "$i"
    done
}

# sc_speed_best down|up -> REPLY the best value of the iperf3 rows ("" — none), REPLY2 1 when that row is «not less than»
sc_speed_best() {
    local i v best="" lb=0 f
    for ((i = 0; i < SPN; i++)); do
        [ "${SP_M[$i]}" = iperf3 ] || continue
        if [ "$1" = down ]; then v=${SP_DOWN[$i]}; f=${SP_DLB[$i]}; else v=${SP_UP[$i]}; f=${SP_ULB[$i]}; fi
        [ -n "$v" ] || continue
        if [ -z "$best" ] || awk -v a="$v" -v b="$best" 'BEGIN { exit !(a > b) }'; then best=$v; lb=$f; fi
    done
    REPLY=$best; REPLY2=$lb
}

# the «result» line of «speedtest -f json» (stdin; log lines are skipped) -> «down up ping jitter loss<TAB>id<TAB>name
# <TAB>location<TAB>country»: Mbit/s (bandwidth is bytes/s), ms, %. The addresses of the line are not read.
sc_parse_speedtest() {
    awk '
        function num(s, k,    t) { if (match(s, "\"" k "\"[ \t]*:[ \t]*[0-9.eE+-]+")) { t = substr(s, RSTART, RLENGTH); sub(/^[^:]*:[ \t]*/, "", t); return t } return "" }
        function str(s, k,    t) { if (match(s, "\"" k "\"[ \t]*:[ \t]*\"[^\"\\\\]*\"")) { t = substr(s, RSTART, RLENGTH); sub(/^[^:]*:[ \t]*"/, "", t); sub(/"$/, "", t); gsub(/[[:cntrl:]]/, "", t); gsub(/\302[\200-\237]/, "", t); return t } return "" }
        function obj(s, k) { if (match(s, "\"" k "\"[ \t]*:[ \t]*\\{[^{}]*")) return substr(s, RSTART, RLENGTH); return "" }
        /"type"[ \t]*:[ \t]*"result"/ {
            d = num(obj($0, "download"), "bandwidth"); u = num(obj($0, "upload"), "bandwidth"); p = obj($0, "ping")
            if (d == "" || u == "") next
            sv = obj($0, "server"); l = num($0, "packetLoss")
            bd = num(obj($0, "download"), "bytes"); bu = num(obj($0, "upload"), "bytes")
            printf "%.3f %.3f %s %s %s %s\t%s\t%s\t%s\t%s\n", d * 8 / 1e6, u * 8 / 1e6, num(p, "latency"), num(p, "jitter"), (l == "" ? "-" : l),
                (bd == "" || bu == "" ? "-" : sprintf("%.0f", bd + bu)), num(sv, "id"), str(sv, "name"), str(sv, "location"), str(sv, "country")
            exit
        }'
}

# Speedtest by Ookla, the nearest server by its own choice (the second block, no score): the pinned CLI with
# --accept-license --accept-gdpr (said in the report and README), HOME in the temporary directory (the CLI keeps its
# acceptance there), as nobody under root. ST[status] ok | failed; nothing on the screen when it fails.
sc_net_speedtest() {
    local out line nums
    ST=([status]=skipped [why]="${TWHY[speedtest]:-}")
    [ -n "${TOOL[speedtest]}" ] || return 0
    sc_prog "$(sc_t "network · Speedtest by Ookla" "сеть · Speedtest by Ookla") (≈ 30 $(sc_t "s" "с"))"
    out=$(sc_to 90 env HOME="$SC_HOME" XDG_CONFIG_HOME="$SC_HOME/.config" "${SC_RUNAS[@]}" "${TOOL[speedtest]}" \
        --accept-license --accept-gdpr -f json -p no 2>/dev/null | sc_parse_speedtest)
    ST[status]=failed; ST[why]=failed
    [ -n "$out" ] || return 0
    local id name city country down up ping jitter loss bytes
    IFS=$'\t' read -r nums id name city country <<<"$out"
    read -r down up ping jitter loss bytes <<<"$nums"
    [[ $bytes =~ ^[0-9]+$ ]] && ST[bytes]=$bytes
    ST[id]=$id; ST[name]=$name; ST[city]=$city; ST[country]=$country
    ST[down]=$down; ST[up]=$up; ST[ping]=$ping; ST[jitter]=$jitter; ST[loss]=$loss
    [ "${ST[loss]}" = - ] && ST[loss]=""
    sc_isnum "${ST[id]}" || ST[id]=""
    for line in name city country; do sc_clean "${ST[$line]}"; ST[$line]=$REPLY; done
    sc_isnum "${ST[down]}" && sc_isnum "${ST[up]}" && { ST[status]=ok; ST[why]=""; }
}

# sc_http_rows — the HTTP download (sc_net_down) as rows of the table: a region a row, download only
sc_http_rows() {
    local want
    SPN=0
    for want in any eu ru us; do
        [ -n "${DLR[$want]+x}" ] || continue
        SP_ID[SPN]=${DLR[$want]:-$want}; SP_M[SPN]=http; SP_HOST[SPN]=""; SP_PORTS[SPN]=""; SP_CAP[SPN]=0; SP_PORT[SPN]=""
        sc_region "$want"; SP_CITY[SPN]=$REPLY
        case "$want" in
            any) SP_CEN[SPN]=Nearest; SP_CRU[SPN]=Ближайшая ;; eu) SP_CEN[SPN]=Europe; SP_CRU[SPN]=Европа ;;
            ru) SP_CEN[SPN]=Russia; SP_CRU[SPN]=Россия ;; *) SP_CEN[SPN]=USA; SP_CRU[SPN]=США ;;
        esac
        SP_OEN[SPN]=${DLN[$want]:-}; SP_ORU[SPN]=${DLN[$want]:-}
        SP_DOWN[SPN]=${DLV[$want]:-}; SP_UP[SPN]=""; SP_PING[SPN]=""; SP_DLB[SPN]=${DLB[$want]:-0}; SP_ULB[SPN]=0
        if [ -n "${DLV[$want]:-}" ]; then SP_DST[SPN]=ok; else SP_DST[SPN]=no_answer; fi
        SP_UST[SPN]=skipped; SP_MINRTT[SPN]=""; SP_RDOWN[SPN]=""; SP_RUP[SPN]=""
        SP_PAVG[SPN]=""; SP_PMED[SPN]=""; SP_PJIT[SPN]=""; SP_PLOSS[SPN]=""; SP_PSAMP[SPN]=""; SP_PN[SPN]=0
        SP_REG[SPN]=$want; SP_ALT[SPN]=""; SP_FB[SPN]=0
        SPN=$((SPN + 1))
        sc_row_done $((SPN - 1))
    done
}

# sc_fetch SECS URL… -> stdout: the bodies one after another on one connection (HTTPS only, no redirects)
sc_fetch() {
    local s=$1; shift
    if [ "$SC_FETCH" = curl ]; then
        curl -sgf --proto '=https' --max-time "$s" --connect-timeout 4 "$@" 2>/dev/null
    else
        sc_to "$s" wget -q -O - --https-only --timeout=4 --tries=1 --max-redirect=0 "$@" 2>/dev/null
    fi
}

# sc_dl_sum FILE… -> «Mbit/s lower_bound»: 4 streams, every stream — dd statistics at 2 s (SIGUSR1) and at the end.
# By the clock as bench/channel.py test 8: the bytes after the warm-up of 2 s ÷ their time, summed over the streams;
# a stream with no window after the warm-up (the cap came first, or the file ended) counts all its bytes ÷ all its time,
# and when it was the cap the value is «not less than» (extra.budget). Nothing came, or fewer than SC_DL_MIN bytes in
# all (an error page, a redirect body) — empty.
sc_dl_sum() {
    local cap=$1 f i=0 out=""; shift
    for f in "$@"; do i=$((i + 1)); out+="$(sc_parse_dd <"$f" | awk -v i="$i" '{ print i, $0 }')"$'\n'; done
    awk -v cap="$cap" -v min="${SC_DL_MIN:-0}" '
        NF == 3 { k[$1]++; if (k[$1] == 1) { b1[$1] = $2; t1[$1] = $3 } bE[$1] = $2; tE[$1] = $3 }
        END {
            for (f in k) {
                tot += bE[f]
                if (k[f] >= 2 && tE[f] - t1[f] >= 1.0) rate += (bE[f] - b1[f]) / (tE[f] - t1[f])
                else if (tE[f] > 0) { rate += bE[f] / tE[f]; if (bE[f] >= cap) lb = 1 }
            }
            if (tot <= 0 || tot < min) exit
            printf "%.3f %d\n", rate * 8 / 1e6, lb
        }' <<<"$out"
}

# sc_dl_early FILE… -> true when, at the end of the warm-up, all the streams together have fewer than SC_DL_MIN bytes
# (dead, or held at a few KB by a filter on the way): the point is dropped at once instead of waiting out its time
sc_dl_early() {
    local f
    for f in "$@"; do sc_parse_dd <"$f" | head -n 1; done | awk -v min="${SC_DL_MIN:-0}" '{ s += $1 } END { exit !(s < min) }'
}

# sc_dl ID URL REQ SIZE CAP SECS -> REPLY «Mbit/s lower_bound» ("" — nothing came), SC_DL_BYTES — the bytes moved
sc_dl() {
    local id=$1 url=$2 req=$3 size=$4 cap=$5 secs=$6 per count reps u b i s pids=() files=() args=()
    per=$((cap / 4)); count=$((per / 1000000))
    if [ "$req" -gt 0 ]; then
        b=$((per < req ? per : req)); u=${url//\{bytes\}/$b}; reps=$((per / b + 1))
    else
        u=$url; reps=1; [ "$size" -gt 0 ] && reps=$((per / size + 1))
    fi
    [ "$reps" -gt 40 ] && reps=40
    for ((i = 0; i < reps; i++)); do args+=("$u"); done
    for s in 1 2 3 4; do
        files+=("$SC_WORK/dl.$id.$s")
        sc_fetch "$secs" "${args[@]}" | dd of=/dev/null bs=1000000 count="$count" iflag=fullblock 2>"$SC_WORK/dl.$id.$s" &
        pids+=("$!")
    done
    SC_DL_PIDS="${pids[*]}"
    sleep 2
    sc_kill_own USR1 "${pids[@]}"
    sleep 0.2 2>/dev/null || sleep 1
    if sc_dl_early "${files[@]}"; then
        # the dd end; curl, its pipe closed, ends at its next write or by its own --max-time
        sc_kill_own TERM "${pids[@]}"
        [ ${#pids[@]} -eq 0 ] || wait "${pids[@]}" 2>/dev/null
        SC_DL_PIDS=""; REPLY=""
    else
        [ ${#pids[@]} -eq 0 ] || wait "${pids[@]}" 2>/dev/null
        SC_DL_PIDS=""
        REPLY=$(sc_dl_sum "$((count * 1000000))" "${files[@]}")
    fi
    SC_DL_BYTES=$(for s in "${files[@]}"; do sc_parse_dd <"$s" | tail -n 1; done | awk '{ b += $1 } END { printf "%d\n", b }')
}

# sc_dl_enough V LB REF — a try that ends its region: it reached the cap (a lower bound), or gave at least a quarter of
# REF (the best so far in this run — the nearest point is measured first); no REF yet — any answer
sc_dl_enough() {
    [ "${2:-0}" = 1 ] && return 0
    awk -v v="$1" -v r="${3:-}" 'BEGIN { exit !(r == "" || v * 4 >= r) }'
}

# the download (score_abs «net_down»): 4 streams per point. A region tries its points in order and keeps its best; it
# goes on to the next point only when the last one gave nothing, or so little that the point itself is the limit (a
# speed-test file throttled on its side: 7–28 Mbit/s on a 2.8 Gbit/s server, 12.11); at most the points of the list.
# The value of the test is the best of all (bench/channel.py: «the value = the best of the tries»). GET only: no upload.
sc_net_down() {
    local regions want id reg en ru url req size cap secs r best="" bestlb=0 v lb ref
    if [ "$SC_MODE" = quick ]; then regions="any ru"; else regions="any eu ru us"; fi
    for want in $regions; do
        DLR[$want]=""; DLV[$want]=""; DLB[$want]=0; DLN[$want]=""; DLT[$want]=""
        # Kaskad caps a direction at 250 MB (quick) and 2 GB (full); here: quick 125 MB a region, full — 1 GB for the
        # nearest point (10 s) and 500 MB for each region (6 s a try); the tries of a region share its cap
        if [ "$SC_MODE" = quick ]; then cap=125000000; secs=6
        elif [ "$want" = any ]; then cap=1000000000; secs=10
        else cap=500000000; secs=6; fi
        while IFS='|' read -r id reg en ru url req size; do
            [ "$reg" = "$want" ] || continue
            [ "$cap" -ge 50000000 ] || break
            if [ "$SC_LANG" = ru ]; then r=$ru; else r=$en; fi
            sc_prog "$(sc_t "network · download" "сеть · скачивание") · $r · ${secs} s"
            SC_DL_BYTES=0
            sc_dl "$id" "$url" "$req" "$size" "$cap" "$secs"
            cap=$((cap - ${SC_DL_BYTES:-0}))
            v=""; lb=0; [ -n "$REPLY" ] && { v=${REPLY% *}; lb=${REPLY#* }; }
            DLT[$want]+="${DLT[$want]:+ }$id=${v:--}"
            [ -n "$v" ] || continue
            if [ -z "${DLV[$want]}" ] || awk -v a="$v" -v b="${DLV[$want]}" 'BEGIN { exit !(a > b) }'; then
                DLR[$want]=$id; DLV[$want]=$v; DLB[$want]=$lb; DLN[$want]=$r
            fi
            ref=$best
            if [ -z "$ref" ] || awk -v a="${DLV[$want]}" -v b="$ref" 'BEGIN { exit !(a > b) }'; then ref=${DLV[$want]}; fi
            sc_dl_enough "$v" "$lb" "$ref" && break
        done <<<"$SC_DL_POINTS"
        v=${DLV[$want]}
        if [ -n "$v" ] && { [ -z "$best" ] || awk -v a="$v" -v b="$best" 'BEGIN { exit !(a > b) }'; }; then
            best=$v; bestlb=${DLB[$want]}
        fi
    done
    if [ -n "$best" ]; then sc_set net_down "$best" http "" 0; X[net_down_lb]=$bestlb
    else sc_set net_down "" "" no_answer; X[net_down_lb]=0; fi
}

# the network: the latency and DNS («Отклик и DNS»), the speed table (iperf3; the HTTP download when iperf3 is missing or
# no point answered — download only), Speedtest by Ookla. «Канал» (net_bw) = the scale of Kaskad over the best download
# and the best upload of the iperf3 rows (Kaskad: «the value = the best of the tries»)
sc_net() {
    local v q
    ST=([status]=skipped [why]="${TWHY[speedtest]:-}")
    [ -n "${TOOL[iperf3]:-}" ] && [ "$SC_NONET" != 1 ] || SPN=0
    SC_PHASE=net; [ "${SC_LIVE:-0}" = 1 ] && sc_live_est net
    if [ "$SC_NONET" = 1 ]; then
        for v in rtt jitter loss dns net_down net_up; do sc_set "$v" "" "" skipped; done
    else
        sc_net_rtt
        sc_net_dns
        [ -n "${TOOL[iperf3]}" ] && sc_net_iperf
        sc_speed_best down
        if [ -n "$REPLY" ]; then
            q=0; [ "$SC_MODE" = quick ] && q=1          # quick: the score at most 700 (quick_cap) — ≈
            sc_set net_down "$REPLY" iperf3 "" "$q"; X[net_down_lb]=$REPLY2
            sc_speed_best up
            if [ -n "$REPLY" ]; then sc_set net_up "$REPLY" iperf3 "" "$q"; X[net_up_lb]=$REPLY2
            else sc_set net_up "" "" no_answer; X[net_up_lb]=0; fi
        elif [ -n "$SC_FETCH" ]; then
            SC_HTTP=1
            sc_net_down
            sc_http_rows
            sc_set net_up "" "" "$([ -n "${TOOL[iperf3]}" ] && echo iperf_no_answer || echo no_iperf)"; X[net_up_lb]=0
        else
            sc_set net_down "" "" no_fetch; sc_set net_up "" "" no_fetch
        fi
        sc_net_speedtest
    fi
    sc_group_score net_bw
    sc_group_score net_q
}

# ── the report ──────────────────────────────────────────────────────────────────────────────────────────────────────
# Two renderers of the same data — the text (sc_render_text) and the JSON (sc_report_json); they read only what the
# measurements left: SYS, V/SS/WHY/AP of the tests, GS/GA of the groups, X, the rows of the speed table SP_*, ST of
# Speedtest, TOOL/TSRC/TWHY of the tools. Nothing below measures anything.

SC_W=80        # the text report: at most 80 characters a line (the rules — 78)

sc_why_text() {
    case "$1" in
        no_sysbench) sc_t "no sysbench" "нет sysbench" ;;
        no_fio) sc_t "no fio" "нет fio" ;;
        no_openssl) sc_t "no openssl" "нет openssl" ;;
        few_samples) sc_t "too few samples to score" "мало замеров для балла" ;;
        no_answer) sc_t "no answer" "нет ответа" ;;
        no_icmp) sc_t "ping unavailable" "ping недоступен" ;;
        disk_space) sc_t "not enough free space" "мало свободного места" ;;
        disk_fs) sc_t "no suitable file system (tmpfs, overlay)" "нет подходящей файловой системы (tmpfs, overlay)" ;;
        skipped) sc_t "skipped (--no-net)" "пропущено (--no-net)" ;;
        no_iperf) sc_t "no iperf3: HTTP measures the download only" "нет iperf3: HTTP меряет только скачивание" ;;
        iperf_no_answer) sc_t "no iperf3 point answered" "ни одна точка iperf3 не ответила" ;;
        no_fetch) sc_t "no curl or wget" "нет curl и wget" ;;
        mem_implausible|mem_unstable) sc_t "indicative, no score" "ориентир, без балла" ;;
        no_shm) sc_t "no tmpfs /dev/shm" "нет tmpfs /dev/shm" ;;
        mem_room) sc_t "no room in RAM" "мало свободной памяти" ;;
        *) sc_t "not measured" "не измерено" ;;
    esac
}

# sc_tool_why WHY -> REPLY: why a tool is not used
sc_tool_why() {
    case "$1" in
        builtin) REPLY="SC_BUILTIN=1" ;;
        no_fetch) REPLY="SC_NO_FETCH=1" ;;
        off) REPLY=$(sc_t "turned off" "выключено") ;;
        no_apt) REPLY=$(sc_t "no apt (not Debian / Ubuntu)" "нет apt (не Debian / Ubuntu)") ;;
        no_package) REPLY=$(sc_t "not in the package lists of apt" "нет в списках пакетов apt") ;;
        too_big) REPLY=$(sc_t "the packages are over 50 MB" "пакеты больше 50 МБ") ;;
        download) REPLY=$(sc_t "the download failed" "не скачалось") ;;
        broken) REPLY=$(sc_t "does not start here" "не запускается здесь") ;;
        arch) REPLY="$(sc_t "no build for" "нет сборки для") ${SYS[arch]:-?}" ;;
        no_sha) REPLY=$(sc_t "nothing here checks SHA-256" "нечем проверить SHA-256") ;;
        checksum) REPLY=$(sc_t "the checksum did not match" "контрольная сумма не совпала") ;;
        extract) REPLY=$(sc_t "the archive did not unpack" "архив не распаковался") ;;
        no_fetch_tool) REPLY=$(sc_t "no curl or wget" "нет curl и wget") ;;
        *) REPLY=$(sc_t "unavailable" "недоступен") ;;
    esac
}

sc_unit() {
    case "$1" in
        evs) sc_t "ev/s" "соб./с" ;; mbs) sc_t "MB/s" "МБ/с" ;; mibs) sc_t "MiB/s" "МиБ/с" ;; gb) sc_t "GB" "ГБ" ;;
        iops) printf 'IOPS' ;; ms) sc_t "ms" "мс" ;; pct) printf '%%' ;; mbit) sc_t "Mbit/s" "Мбит/с" ;; *) printf '%s' "$1" ;;
    esac
}

sc_region() {
    case "$1" in
        any) REPLY=$(sc_t "Nearest" "Ближайшая") ;; eu) REPLY=$(sc_t "Europe" "Европа") ;;
        ru) REPLY=$(sc_t "Russia" "Россия") ;; us) REPLY=$(sc_t "USA" "США") ;; *) REPLY=$1 ;;
    esac
}

# sc_cpu_short MODEL -> REPLY: the model without (R), (TM), «CPU», «Processor», «NN-Core», the clock
sc_cpu_short() {
    local s=$1
    s=${s//(R)/}; s=${s//(r)/}; s=${s//(TM)/}; s=${s//(tm)/}; s=${s%% @ *}
    s=${s// CPU / }; s=${s% CPU}; s=${s// Processor/}
    [[ $s =~ ^(.*)\ [0-9]+-Core(.*)$ ]] && s="${BASH_REMATCH[1]}${BASH_REMATCH[2]}"
    sc_clean "$s"; REPLY=${REPLY:-?}
}

sc_virt_name() {
    case "${SYS[virt]}" in
        kvm) REPLY=KVM ;; qemu) REPLY=QEMU ;; vmware) REPLY=VMware ;; microsoft) REPLY=Hyper-V ;; xen) REPLY=Xen ;;
        lxc|lxc-libvirt) REPLY=LXC ;; openvz) REPLY=OpenVZ ;; docker) REPLY=Docker ;; podman) REPLY=Podman ;;
        none) REPLY=$(sc_t "bare metal" "без виртуализации") ;; vm) REPLY=$(sc_t "a VM" "ВМ") ;;
        unknown|"") REPLY=$(sc_t "virtualization unknown" "виртуализация не определена") ;; *) REPLY=${SYS[virt]} ;;
    esac
}

# ── the screen (Claude Design r2, owner 12.11) ──────────────────────────────────────────────────────────────────────
# Passport (3 lines) → «Scores, 0–1000» on one axis (7 rows: the score and the raw value) → the network table by
# regions (↓ / ↑ bars and numbers, ping, jitter sparkline and number, TCP retransmits; words for busy / no link) →
# Ookla → a 2-line legend → «✓ Done». The data is collected first (sc_collect), the screen only reads it.
# Three ways to show it: live (a TTY with colour, the frame fits: one screen filled in place by a ticker), a line log
# (a pipe, a file, NO_COLOR, a small terminal: no redraw, each finished row printed once) and the final screen.
#
# Markup of a screen line: \x01 + a role letter starts a colour, \x01/ ends it; sc_paint turns it into ANSI codes of
# the colour depth of the terminal or removes it. Roles: m mint, u upload, d grey, b bold, y busy, r no link, g done.
# \x02 + s|t|b — the spinner, the elapsed time, the progress bar: the ticker puts them in. Text from outside (Ookla
# names) has its control characters removed (sc_parse_speedtest), so it can carry neither.

# sc_vis S -> REPLY: S without the markup;  sc_vlen S -> REPLY: its width in characters
sc_vis() { REPLY=${1//$'\x01'?/}; }
sc_vlen() { sc_vis "$1"; sc_len "$REPLY"; }
# sc_rp S N / sc_lp S N -> REPLY: S padded with spaces on the right / the left to N characters (never cut)
sc_rp() { local s=$1 k; sc_vlen "$s"; k=$(($2 - REPLY)); [ "$k" -gt 0 ] && printf -v s '%s%*s' "$s" "$k" ''; REPLY=$s; }
sc_lp() { local s=$1 k; sc_vlen "$s"; k=$(($2 - REPLY)); [ "$k" -gt 0 ] && printf -v s '%*s%s' "$k" '' "$s"; REPLY=$s; }
sc_rep() { local s; printf -v s '%*s' "$2" ''; REPLY=${s// /$1}; }        # CHAR N -> REPLY: CHAR N times
# sc_cut TEXT N -> REPLY: TEXT (markup kept) at most N characters wide — longer: cut, «…» at the end, the role closed
sc_cut() {
    local s=$1 n=$2 out="" c i=0 k=0 e="…" w=1 cw
    sc_vlen "$s"; if [ "$REPLY" -le "$n" ]; then REPLY=$s; return 0; fi
    [ "${SC_ASCII:-0}" = 1 ] && { e="..."; w=3; }
    while [ "$i" -lt ${#s} ]; do
        c=${s:i:1}
        if [ "$c" = $'\x01' ]; then out+=${s:i:2}; i=$((i + 2)); continue; fi
        case "$c" in [$'\x80'-$'\xbf']) out+=$c; i=$((i + 1)); continue ;; esac   # the rest of a UTF-8 character
        cw=1                                                   # a wide character takes two columns (as sc_len)
        case "$c" in
            [$'\xe3'-$'\xed'$'\xf0'-$'\xf4']) cw=2 ;;
            $'\xef') case "${s:i+1:1}" in [$'\xa4'-$'\xab'$'\xbc']) cw=2 ;; esac ;;
        esac
        [ $((k + cw)) -gt $((n - w)) ] && break
        out+=$c; k=$((k + cw)); i=$((i + 1))
    done
    REPLY="$out$e"$'\x01/'
}

# sc_colors COLOR -> SC_CM: 24 | 256 | 16 | "" (no colour: NO_COLOR, --no-color, TERM=dumb, not a terminal, JSON)
sc_colors() {
    SC_CM=""
    if [ "$1" = 1 ] && [ -t 1 ] && [ "${TERM:-dumb}" != dumb ] && [ "$SC_JSON" != 1 ]; then
        case "${COLORTERM:-}" in truecolor|24bit) SC_CM=24 ;; *) case "${TERM:-}" in *256color*) SC_CM=256 ;; *) SC_CM=16 ;; esac ;; esac
    fi
    sc_color_codes "$SC_CM"
}
# sc_color_codes DEPTH — the codes of the roles for a colour depth: 24 (truecolor) | 256 | 16
sc_color_codes() {
    case "$1" in
        24) SC_RM='38;2;69;208;192'; SC_RU='38;2;135;175;175'; SC_RD='38;2;118;129;143'; SC_RY='38;2;229;192;123'
            SC_RR='38;2;236;138;120'; SC_RG='38;2;127;212;154' ;;
        256) SC_RM='38;5;79'; SC_RU='38;5;109'; SC_RD='38;5;244'; SC_RY='38;5;179'; SC_RR='38;5;174'; SC_RG='38;5;114' ;;
        *) SC_RM='36'; SC_RU='2;36'; SC_RD='90'; SC_RY='33'; SC_RR='31'; SC_RG='32' ;;
    esac
}

# sc_glyphs — the characters: UTF-8, or ASCII (--ascii, SC_ASCII=1, a locale set without UTF-8)
sc_glyphs() {
    if [ "$SC_ASCII" = 1 ]; then
        G_FULL='#'; G_TRACK='.'; G_PARTS=('' '' '' '' '' '' '' ''); G_SPARK=('_' '.' '-' '=' '+' '*' '#' '@')
        G_DOT='.'; G_PT='o'; G_TICK=':'; G_RL='|'; G_RX='+'; G_RR='|'; G_RH='-'; G_RUN='='; G_RUNT='-'
        G_SPIN=('|' '/' '-' "\\"); G_OK='ok'
    else
        G_FULL='█'; G_TRACK='░'; G_PARTS=('' '▏' '▎' '▍' '▌' '▋' '▊' '▉'); G_SPARK=('▁' '▂' '▃' '▄' '▅' '▆' '▇' '█')
        G_DOT='·'; G_PT='●'; G_TICK='┊'; G_RL='├'; G_RX='┼'; G_RR='┤'; G_RH='─'; G_RUN='━'; G_RUNT='─'
        G_SPIN=('⠋' '⠙' '⠹' '⠸' '⠼' '⠴' '⠦' '⠧' '⠇' '⠏'); G_OK='✓'
    fi
}

# sc_paint S -> REPLY: the markup as ANSI codes (or removed); in ASCII the other signs become ASCII too
sc_paint() {
    local s=$1
    if [ "$SC_ASCII" = 1 ]; then
        s=${s//·/.}; s=${s//—/-}; s=${s//–/-}; s=${s//…/...}; s=${s//↓/v}; s=${s//↑/^}; s=${s//≥/>=}; s=${s//≈/~}
        s=${s//×/x}; s=${s//─/-}
    fi
    if [ -z "${SC_CM:-}" ]; then REPLY=${s//$'\x01'?/}; return 0; fi
    s=${s//$'\x01'm/$'\033['"${SC_RM}m"}; s=${s//$'\x01'u/$'\033['"${SC_RU}m"}; s=${s//$'\x01'd/$'\033['"${SC_RD}m"}
    s=${s//$'\x01'y/$'\033['"${SC_RY}m"}; s=${s//$'\x01'r/$'\033['"${SC_RR}m"}; s=${s//$'\x01'g/$'\033['"${SC_RG}m"}
    s=${s//$'\x01'b/$'\033[1m'}; s=${s//$'\x01'\//$'\033[0m'}
    REPLY=${s//$'\x01'?/}
}

# ── numbers ──

# sc_gnum V [DECIMALS] -> REPLY: thousands by a plain space (U+0020: some terminals draw U+202F wider), «,» in RU
sc_gnum() {
    local s int frac="" out="" neg=""
    printf -v s "%.${2:-0}f" "$1"
    [ "${s:0:1}" = - ] && { neg=-; s=${s:1}; }
    int=${s%%.*}; [ "$s" != "$int" ] && frac=${s#*.}
    while [ ${#int} -gt 3 ]; do out=" ${int: -3}$out"; int=${int:0:${#int}-3}; done
    out="$int$out"
    if [ -n "$frac" ]; then if [ "$SC_LANG" = ru ]; then out+=",$frac"; else out+=".$frac"; fi; fi
    REPLY="$neg$out"
}
# Mbit/s: whole; from 10 000 — «10,2Г» (5 characters, the width of the column)
sc_f_mbit() {
    if awk -v v="$1" 'BEGIN { exit !(v >= 9999.5) }'; then sc_gnum "$(awk -v v="$1" 'BEGIN { print v / 1000 }')" 1; REPLY+=$(sc_t G Г)
    else sc_gnum "$1" 0; fi
}
# ms: below 10 — one decimal (1,6), from 10 — whole (38)
sc_f_ms() { if awk -v v="$1" 'BEGIN { exit !(v < 9.95) }'; then sc_gnum "$1" 1; else sc_gnum "$1" 0; fi; }
# retransmits: below 1000 — whole; to 9999 — «1,9к»; more — «12к»
sc_f_retr() {
    if [ "$1" -lt 1000 ]; then REPLY=$1
    elif [ "$1" -lt 9950 ]; then sc_gnum "$(awk -v v="$1" 'BEGIN { print v / 1000 }')" 1; REPLY+=$(sc_t k к)
    else sc_gnum "$(awk -v v="$1" 'BEGIN { print v / 1000 }')" 0; REPLY+=$(sc_t k к); fi
}
sc_f_time() { printf -v REPLY '%d:%02d' $(($1 / 60)) $(($1 % 60)); }   # SECONDS -> m:ss

# ── bars, the sparkline, the axis ──

# sc_hbar V MAX W ROLE -> REPLY: V of MAX on W cells: █ and a left eighth (rounded), the rest of the track ░ grey
sc_hbar() {
    local u full r used s t
    u=$(awk -v v="$1" -v m="$2" -v w="$3" 'BEGIN { u = (m > 0 ? int(v / m * w * 8 + 0.5) : 0); if (u > w * 8) u = w * 8; if (u < 0) u = 0; print u }')
    if [ "$SC_ASCII" = 1 ]; then u=$(( (u + 4) / 8 * 8 )); fi
    full=$((u / 8)); r=$((u % 8)); used=$((full + (r > 0 ? 1 : 0)))
    sc_rep "$G_FULL" "$full"; s=$REPLY; [ "$r" -gt 0 ] && s+=${G_PARTS[$r]}
    sc_rep "$G_TRACK" $(($3 - used)); t=$REPLY
    REPLY=$'\x01'"$4$s"$'\x01/\x01d'"$t"$'\x01/'
}

# sc_spark MS… -> REPLY: the replies in order as at most 8 columns (the largest of each of 8 equal parts), the height
# ⌊(x − the least of the point) / 2 ms⌋, 0…7: a steady ping stays flat ▁▁▁▁ however long the line
sc_spark() {
    local h out=""
    [ $# -ge 1 ] || { REPLY=""; return 0; }
    for h in $(printf '%s\n' "$@" | awk 'NF { v[n++] = $1 + 0 } END {
            if (!n) exit; mn = v[0]; for (i = 1; i < n; i++) if (v[i] < mn) mn = v[i]
            c = (n < 8 ? n : 8)
            for (j = 0; j < c; j++) { a = int(j * n / c); b = int((j + 1) * n / c); if (b <= a) b = a + 1
                mx = v[a]; for (i = a + 1; i < b; i++) if (v[i] > mx) mx = v[i]
                k = int((mx - mn) / 2); if (k > 7) k = 7; printf "%d ", k } }'); do
        out+=${G_SPARK[$h]}
    done
    REPLY=$out
}

# the axis of the scores: A cells (40; wider terminals up to 60); a score at round(s / 1000 × (A − 1))
sc_axis_pos() { REPLY=$(awk -v s="$1" -v a="$SC_AX" 'BEGIN { printf "%d", s / 1000 * (a - 1) + 0.5 }'); }
sc_axis_ticks() {
    local t; SC_TICKS=" "
    for t in 0 250 500 750 1000; do sc_axis_pos "$t"; SC_TICKS+="$REPLY "; done
}
# sc_plot SCORE -> REPLY: · up to the score, ● on it, the ticks ┊ after it
sc_plot() {
    local p=$1 i s=$'\x01d'
    for ((i = 0; i < SC_AX; i++)); do
        if [ "$i" -lt "$p" ]; then s+=$G_DOT
        elif [ "$i" = "$p" ]; then s+=$'\x01/\x01m'"$G_PT"$'\x01/\x01d'
        elif [[ $SC_TICKS == *" $i "* ]]; then s+=$G_TICK
        else s+=" "; fi
    done
    REPLY="$s"$'\x01/'
}

# ── the scores (7 rows) ──

# sc_score_rows -> SR_K / SR_L / SR_S / SR_A / SR_V: key, label, score ("" — none), ≈, the raw value (or why not)
sc_score_rows() {
    local k t v u ap
    SR_K=(cpu1 cpun mem disk4k diskf net lat); SR_L=(); SR_S=(); SR_A=(); SR_V=()
    for k in "${SR_K[@]}"; do
        t=""; v=""; u=""; ap=0
        case "$k" in
            cpu1) SR_L+=("$(sc_t "CPU, 1 core" "Процессор, 1 ядро")")
                  if [ -n "${SS[cpu_1t]:-}" ]; then t=cpu_1t; u=$(sc_t "ev/s" "соб/с"); else t=chacha_1t; u=$(sc_t "MB/s" "МБ/с"); fi ;;
            cpun) SR_L+=("$(sc_t "CPU, all cores" "Процессор, все ядра")"); t=cpu_mt; u=$(sc_t "ev/s" "соб/с") ;;
            mem) SR_L+=("$(sc_t "Memory" "Память")")
                 if [ -n "${SS[mem_bw]:-}" ]; then t=mem_bw; u=$(sc_t "MiB/s" "МиБ/с"); else t=mem_gb; u=$(sc_t "GB" "ГБ"); fi ;;
            disk4k) SR_L+=("$(sc_t "Disk, 4K random" "Диск, 4K случайный")"); t=disk_rr; u=IOPS ;;
            diskf) SR_L+=("$(sc_t "Disk, 4K + flush" "Диск, 4K + сброс")"); t=disk_fsync; u=$(sc_t "ms" "мс") ;;
            net) SR_L+=("$(sc_t "Network channel" "Сетевой канал")") ;;
            lat) SR_L+=("$(sc_t "Latency" "Задержка")"); t=rtt; u=$(sc_t "ms" "мс") ;;
        esac
        if [ "$k" = net ]; then
            SR_S+=("${GS[net_bw]:-}"); SR_A+=("${GA[net_bw]:-0}")
            if sc_isnum "${V[net_down]:-}"; then
                # «≥» close to the number, as in the table: «≥ 9 999 Мбит/с» would make the row 81 columns of 80
                sc_f_mbit "${V[net_down]}"; v="$REPLY $(sc_t "Mbit/s" "Мбит/с")"; [ "${X[net_down_lb]:-0}" = 1 ] && v="≥$v"
            else v=$(sc_t "not measured" "не измерен"); fi
            SR_V+=("$v"); continue
        fi
        SR_S+=("${SS[$t]:-}"); ap=${AP[$t]:-0}; SR_A+=("$ap")
        if sc_isnum "${V[$t]:-}"; then
            case "$t" in
                mem_gb) sc_gnum "${V[$t]}" 1 ;; disk_fsync|rtt) sc_f_ms "${V[$t]}" ;; *) sc_gnum "${V[$t]}" 0 ;;
            esac
            v="$REPLY $u"
        elif [ -n "${WHY[$t]:-}" ]; then v=$(sc_why_text "${WHY[$t]}")
        else v=""; fi
        SR_V+=("$v")
    done
}

# sc_scale_lines MODE -> lines into SCR: the labels, the ruler, the 7 rows. MODE: final | live (a row not measured yet —
# grey; the first open one of the system part — the spinner and «measuring»; the network ones — «after the network»)
sc_scale_lines() {
    local mode=$1 i a s lbl lab ru first_open=1 c
    sc_score_rows
    printf -v a '%*s' "$SC_AX" ''; lab=$a
    sc_axis_pos 250; c=$REPLY; lab="0${lab:1:$((c - 1))}250${lab:$((c + 3))}"
    sc_axis_pos 500; c=$((REPLY - 1)); lab="${lab:0:$c}500${lab:$((c + 3))}"
    sc_axis_pos 750; c=$((REPLY - 1)); lab="${lab:0:$c}750${lab:$((c + 3))}"
    lab="${lab:0:$((SC_AX - 4))}1000"
    sc_rp "$(sc_t "Scores, 0–1000" "Оценки, 0–1000")" 20; SCR+=($'\x01d'"$REPLY$lab"$'\x01/')
    ru=""
    for ((i = 0; i < SC_AX; i++)); do
        if [ "$i" = 0 ]; then ru+=$G_RL; elif [ "$i" = $((SC_AX - 1)) ]; then ru+=$G_RR
        elif [[ $SC_TICKS == *" $i "* ]]; then ru+="$G_RX"; else ru+="$G_RH"; fi
    done
    printf -v a '%20s' ''; SCR+=($'\x01d'"$a$ru"$'\x01/')
    for ((i = 0; i < ${#SR_K[@]}; i++)); do
        sc_rp "${SR_L[$i]}" 20; lbl=$REPLY
        if [ -n "${SR_S[$i]}" ]; then
            sc_axis_pos "${SR_S[$i]}"; sc_plot "$REPLY"; s=$REPLY
            if [ "${SR_A[$i]}" = 1 ]; then sc_lp "≈ ${SR_S[$i]}" 5; else sc_lp "${SR_S[$i]}" 5; fi
            SCR+=("$lbl$s"$'\x01b'"$REPLY"$'\x01/  \x01d'"${SR_V[$i]}"$'\x01/')
        elif [ "$mode" = live ] && [ "$SC_PHASE" != "done" ]; then
            case "${SR_K[$i]}" in
                net|lat) SCR+=($'\x01d'"$lbl$(sc_t "after the network" "после замера сети")"$'\x01/') ;;
                *) if [ "$first_open" = 1 ] && [ "$SC_PHASE" = sys ] && [ -z "${SR_V[$i]}" ]; then
                       first_open=0; SCR+=("$lbl"$'\x01m\x02s\x01/\x01d '"$(sc_t "measuring" "идёт замер")"$'\x01/')
                   else SCR+=($'\x01d'"$lbl"$'\x01/'); fi ;;
            esac
        else
            sc_rp "${SR_V[$i]:-—}" "$SC_AX"; a=$REPLY; sc_lp "—" 5
            SCR+=($'\x01d'"$lbl$a$REPLY"$'\x01/')
        fi
    done
}

# ── the network table ──

# sc_net_scale -> SC_NMAX: the full bar, Mbit/s: max(1000, the best measured rounded up to 1 / 2,5 / 10 / 25 / 100
# Gbit/s); in the line log it is fixed by the first row (SC_NMAX_FIX)
sc_net_scale() {
    local i best=0 v
    if [ -n "${SC_NMAX_FIX:-}" ]; then SC_NMAX=$SC_NMAX_FIX; return 0; fi
    for ((i = 0; i < SPN; i++)); do
        for v in "${SP_DOWN[$i]}" "${SP_UP[$i]}"; do sc_isnum "$v" && best=$(awk -v a="$best" -v b="$v" 'BEGIN { print (b > a ? b : a) }'); done
    done
    if [ "${ST[status]:-}" = ok ]; then
        for v in "${ST[down]:-}" "${ST[up]:-}"; do sc_isnum "$v" && best=$(awk -v a="$best" -v b="$v" 'BEGIN { print (b > a ? b : a) }'); done
    fi
    SC_NMAX=$(awk -v b="$best" 'BEGIN { n = split("1000 2500 10000 25000 100000", s, " "); for (i = 1; i <= n; i++) if (b <= s[i]) { print s[i]; exit } print s[n] }')
}
sc_f_scale() {   # MBIT -> REPLY «10 Гбит/с»
    if [ "$1" -ge 1000 ]; then sc_gnum "$(awk -v v="$1" 'BEGIN { print v / 1000 }')" "$([ $(($1 % 1000)) = 0 ] && echo 0 || echo 1)"
        REPLY+=" $(sc_t "Gbit/s" "Гбит/с")"
    else REPLY="$1 $(sc_t "Mbit/s" "Мбит/с")"; fi
}

# sc_net_names -> the name column N (the longest «city · operator» of the rows and their fallbacks + 1, at least 24),
# the bars B = ⌊(41 − N) / 2⌋ (at least 5; a name that would make it less gets a line of its own and N stays 24), the
# gap; wider terminals give the bars ⌊(COLUMNS − 80) / 2⌋ more cells, at most 20
sc_net_names() {
    local i n max=0 rest alt cen cru oen oru x
    for ((i = 0; i < SPN; i++)); do
        if [ "$SC_LANG" = ru ]; then sc_len "${SP_CRU[$i]} · ${SP_ORU[$i]}"; else sc_len "${SP_CEN[$i]} · ${SP_OEN[$i]}"; fi
        [ "$REPLY" -gt "$max" ] && max=$REPLY
        rest=${SP_ALT[$i]:-}
        while [ -n "$rest" ]; do
            alt=${rest%%;*}; if [ "$alt" = "$rest" ]; then rest=""; else rest=${rest#*;}; fi
            IFS=, read -r cen cru oen oru x <<<"$alt"
            if [ "$SC_LANG" = ru ]; then sc_len "$cru · $oru"; else sc_len "$cen · $oen"; fi
            [ "$REPLY" -gt "$max" ] && max=$REPLY
        done
    done
    n=$((max + 1)); [ "$n" -lt 24 ] && n=24
    if [ $(((41 - n) / 2)) -lt 5 ]; then n=24; fi
    SC_NW=$n; SC_BW=$(((41 - n) / 2)); [ "$SC_BW" -lt 5 ] && SC_BW=5
    SC_GAP=$((41 - n - 2 * SC_BW + 2))
    if [ "${SC_COLS:-80}" -ge 100 ]; then
        x=$(((SC_COLS - 80) / 2)); SC_BW=$((SC_BW + x)); [ "$SC_BW" -gt 20 ] && SC_BW=20
    fi
    SC_MIDW=$((2 * SC_BW + SC_GAP + 11))
}

# sc_name CITY OP -> REPLY: «city · operator» (the operator grey), padded to N; the long list — dots to the bars
sc_name() {
    local s=$1 k
    [ -n "$2" ] && s+=$'\x01d · '"$2"$'\x01/'
    if [ "$SC_POINTS" = long ]; then
        sc_vlen "$s"; k=$((SC_NW - REPLY - 1))
        if [ "$k" -ge 3 ]; then sc_rep . $((k - 1)); s+=$' \x01d'"$REPLY"$'\x01/'; fi
    fi
    sc_rp "$s" "$SC_NW"
}

# sc_tail PING SPARK JITTER RETR -> REPLY: the right part of a row (ping 4, the sparkline 8 after a space, 5, 5)
sc_tail() {
    local s
    sc_lp "$1" 4; s="$REPLY "; sc_rp "$2" 8; s+=$REPLY; sc_lp "$3" 5; s+=$REPLY; sc_lp "$4" 5; s+=$REPLY
    REPLY=$s
}
# sc_cellnum V LB BOLD -> REPLY: the number of a bar, 6 wide — a space after the bar even for «6 975» («≥ 931» when it
# is a lower bound). The columns of a row: 2 | name N | bar B, number 6 | gap − 1 | bar B, number 6 | ping 4 | space |
# sparkline 8 | jitter 5 | retransmits 5 = 79 for every N (2B + gap = 43 − N)
sc_cellnum() {
    local s
    sc_f_mbit "$1"; s=$REPLY
    if [ "$2" = 1 ]; then sc_vlen "≥ $s"; if [ "$REPLY" -le 5 ]; then s="≥ $s"; else s="≥$s"; fi; fi
    [ "$3" = 1 ] && s=$'\x01b'"$s"$'\x01/'
    sc_lp "$s" 6
}
# sc_word STATUS -> REPLY: a direction that gave no number, in words
sc_word() {
    case "$1" in
        busy) REPLY=$'\x01y'"$(sc_t "busy" "занят")"$'\x01/' ;; error) REPLY=$'\x01r'"$(sc_t "failed" "сбой")"$'\x01/' ;;
        no_answer) REPLY=$'\x01r'"$(sc_t "no answer" "нет ответа")"$'\x01/' ;; *) REPLY=$'\x01d—\x01/' ;;
    esac
}

# sc_row_ping I -> P (ping), K (sparkline), J (jitter), R (retransmits) of the row, as text
sc_row_ping() {
    local i=$1 rd=${SP_RDOWN[$1]:-} ru=${SP_RUP[$1]:-}
    P=$'\x01d—\x01/'; K=""; J=$'\x01d—\x01/'; R=$'\x01d—\x01/'
    sc_isnum "${SP_PING[$i]:-}" && { sc_f_ms "${SP_PING[$i]}"; P=$REPLY; }
    # shellcheck disable=SC2086
    [ -n "${SP_PSAMP[$i]:-}" ] && { sc_spark ${SP_PSAMP[$i]}; K=$REPLY; }
    sc_isnum "${SP_PJIT[$i]:-}" && { sc_f_ms "${SP_PJIT[$i]}"; J=$REPLY; }
    if [ -n "$rd$ru" ]; then sc_f_retr $(( ${rd:-0} + ${ru:-0} )); R=$REPLY; fi
}

# sc_row_names I -> CITY, OP of the row I in the language of the report
sc_row_names() {
    if [ "${SP_M[$1]}" = http ]; then CITY=${SP_CITY[$1]}; OP=${SP_OEN[$1]}
    elif [ "$SC_LANG" = ru ]; then CITY=${SP_CRU[$1]}; OP=${SP_ORU[$1]}; else CITY=${SP_CEN[$1]}; OP=${SP_OEN[$1]}; fi
}
# sc_net_row I [MARK] [NONAME] -> REPLY: the finished row I (NONAME=1: blank in the place of the name — it has a line
# of its own above)
sc_net_row() {
    local i=$1 mark=${2:-  } nm d u g mid st P K J R CITY OP
    sc_row_names "$i"
    if [ "${3:-0}" = 1 ]; then printf -v nm '%*s' "$SC_NW" ''; else sc_name "$CITY" "$OP"; nm=$REPLY; fi
    sc_row_ping "$i"
    printf -v g '%*s' $((SC_GAP - 1)) ''
    st="${SP_DST[$i]}/${SP_UST[$i]}"
    if [ "${SP_DST[$i]}" != ok ] && [ "${SP_UST[$i]}" != ok ]; then
        case "$st" in
            no_answer/*) sc_rp $'\x01r'"$(sc_t "no link" "нет связи")"$'\x01/\x01d · '"$(sc_t "no answer" "нет ответа")"$'\x01/' "$SC_MIDW"
                         mid=$REPLY; sc_tail $'\x01d—\x01/' "" $'\x01d—\x01/' $'\x01d—\x01/'; REPLY="$mark$nm$mid$REPLY"; return 0 ;;
            busy/*) sc_rp $'\x01y'"$(sc_t "busy" "занят")"$'\x01/\x01d · '"$(sc_t "another test" "идёт чужой тест")"$'\x01/' "$SC_MIDW" ;;
            error/*) sc_rp $'\x01r'"$(sc_t "failed" "сбой")"$'\x01/\x01d · '"$(sc_t "test broke off" "тест оборвался")"$'\x01/' "$SC_MIDW" ;;
            *) sc_rp $'\x01d'"$(sc_t "out of time" "нет времени")"$'\x01/' "$SC_MIDW" ;;
        esac
        mid=$REPLY; sc_tail "$P" "$K" "$J" "$R"; REPLY="$mark$nm$mid$REPLY"; return 0
    fi
    if [ "${SP_DST[$i]}" = ok ]; then
        sc_hbar "${SP_DOWN[$i]}" "$SC_NMAX" "$SC_BW" m; d=$REPLY; sc_cellnum "${SP_DOWN[$i]}" "${SP_DLB[$i]:-0}" 1; d+=$REPLY
    else sc_word "${SP_DST[$i]}"; sc_rp " $REPLY" $((SC_BW + 6)); d=$REPLY; fi
    if [ "${SP_UST[$i]}" = ok ]; then
        sc_hbar "${SP_UP[$i]}" "$SC_NMAX" "$SC_BW" u; u=$REPLY; sc_cellnum "${SP_UP[$i]}" "${SP_ULB[$i]:-0}" 0; u+=$REPLY
    else sc_word "${SP_UST[$i]}"; sc_rp " $REPLY" $((SC_BW + 6)); u=$REPLY; fi
    sc_tail "$P" "$K" "$J" "$R"
    REPLY="$mark$nm$d$g$u$REPLY"
}

# sc_live_row I -> REPLY: the row being measured (the spinner, what goes on), or a waiting one (grey)
sc_live_row() {
    local i=$1 nm d g P K J R txt CITY OP
    sc_row_names "$i"
    if [ "$i" != "${SC_LROW:--1}" ]; then REPLY=$'  \x01d'"$CITY · $OP"$'\x01/'; return 0; fi
    sc_name "$CITY" "$OP"; nm=$REPLY; sc_row_ping "$i"; printf -v g '%*s' $((SC_GAP - 1)) ''
    case "${SC_LDIR:-}" in
        busy) txt=$'\x01y'"$(sc_t "busy" "занят")"$'\x01/\x01d · '"$(sc_t "try" "попытка") ${SC_LTRY:-2} $(sc_t "of" "из") 4"$'\x01/'
              sc_rp "$txt" "$SC_MIDW"; d=$REPLY ;;
        up) sc_hbar "${SP_DOWN[$i]:-0}" "$SC_NMAX" "$SC_BW" m; d=$REPLY
            if [ "${SP_DST[$i]}" = ok ]; then sc_cellnum "${SP_DOWN[$i]}" "${SP_DLB[$i]:-0}" 1; d+=$REPLY
            else sc_word "${SP_DST[$i]}"; sc_rp " $REPLY" $((SC_BW + 6)); d=$REPLY; fi
            sc_rp $'\x01d'"↑ $(sc_t "measuring" "идёт замер")"$'\x01/' $((SC_BW + 6)); d+="$g$REPLY" ;;
        *) sc_rp $'\x01d'"↓ $(sc_t "measuring" "идёт замер")"$'\x01/' $((SC_BW + 5 + SC_GAP)); d=$REPLY
           sc_rp $'\x01d'"$(sc_t "waits" "ждёт")"$'\x01/' $((SC_BW + 6)); d+=$REPLY ;;
    esac
    sc_tail "$P" "$K" "$J" $'\x01d…\x01/'
    REPLY=$'\x01m\x02s\x01/ '"$nm$d$REPLY"
}

sc_region_name() {
    case "$1" in
        ru) sc_t "Russia" "Россия" ;; eu) sc_t "Europe" "Европа" ;; us) sc_t "USA" "США" ;; asia) sc_t "Asia" "Азия" ;;
        any) sc_t "Nearest" "Ближайшая" ;; *) printf '%s' "$1" ;;
    esac
}

# sc_net_head -> lines into SCR: «Network · iperf3 …» and the head of the columns
sc_net_head() {
    local t o s
    sc_iperf_len; t=${REPLY% *}; o=${REPLY#* }
    sc_f_scale "$SC_NMAX"; s=$REPLY
    if [ "$SC_HTTP" = 1 ]; then
        SCR+=($'\x01m'"$(sc_t "Network" "Сеть")"$'\x01/\x01d · '"$(sc_t "HTTP, download only" "HTTP, только скачивание") · $(sc_unit mbit) · $(sc_t "scale" "шкала") — $s"$'\x01/')
    else
        SCR+=($'\x01m'"$(sc_t "Network" "Сеть")"$'\x01/\x01d · iperf3 ↓↑ '"$(sc_t "for $t s, $SC_STREAMS streams" "по $t с, $SC_STREAMS потоков") · $(sc_t "Mbit/s and ms" "Мбит/с и мс") · $(sc_t "scale" "шкала") — $s"$'\x01/')
    fi
    printf -v o '%*s' $((2 + SC_NW)) ''
    sc_rp "↓ $(sc_t "download" "скачивание")" $((SC_BW + 5 + SC_GAP)); o+=$REPLY
    sc_rp "↑ $(sc_t "upload" "отдача")" $((SC_BW + 6)); o+=$REPLY
    sc_lp "$(sc_t "ping" "пинг")" 4; o+="$REPLY "; sc_rp jitter 13; o+=$REPLY; sc_lp "$(sc_t "retr." "повт.")" 5; o+=$REPLY
    SCR+=($'\x01d'"$o"$'\x01/')
}

# sc_group_line REG -> REPLY: the name of a region; the long list — the count and the medians of the columns
sc_group_line() {
    local reg=$1 i n=0 dn=() up=() pg=() jt=() s m
    s=$'\x01b'"$(sc_region_name "$reg")"$'\x01/'
    if [ "$SC_POINTS" != long ]; then REPLY=$s; return 0; fi
    for ((i = 0; i < SPN; i++)); do
        [ "${SP_REG[$i]}" = "$reg" ] || continue
        n=$((n + 1))
        [ "${SP_DST[$i]}" = ok ] && dn+=("${SP_DOWN[$i]}"); [ "${SP_UST[$i]}" = ok ] && up+=("${SP_UP[$i]}")
        sc_isnum "${SP_PING[$i]:-}" && pg+=("${SP_PING[$i]}"); sc_isnum "${SP_PJIT[$i]:-}" && jt+=("${SP_PJIT[$i]}")
    done
    s+=$'\x01d · '"$n"$'\x01/'; sc_rp "$s" $((2 + SC_NW)); s=$REPLY$'\x01d'
    m=""; [ ${#dn[@]} -gt 0 ] && { sc_median "${dn[@]}"; sc_f_mbit "$REPLY"; m="$(sc_t "median" "медиана") $REPLY"; }
    sc_rp "$m" $((SC_BW + 5 + SC_GAP)); s+=$REPLY
    m=""; [ ${#up[@]} -gt 0 ] && { sc_median "${up[@]}"; sc_f_mbit "$REPLY"; m="$(sc_t "median" "медиана") $REPLY"; }
    sc_rp "$m" $((SC_BW + 6)); s+=$REPLY
    m=""; [ ${#pg[@]} -gt 0 ] && { sc_median "${pg[@]}"; sc_f_ms "$REPLY"; m=$REPLY; }
    sc_lp "$m" 4; s+="$REPLY          "
    m=""; [ ${#jt[@]} -gt 0 ] && { sc_median "${jt[@]}"; sc_f_ms "$REPLY"; m=$REPLY; }
    sc_lp "$m" 5; REPLY="$s$REPLY"$'\x01/'
}

# sc_ookla_lines [LIVE] -> lines into SCR: the rule and the row of Speedtest by Ookla (only when it ran); LIVE=1 — while
# it may still run, the rule and a waiting row hold its place in the frame
sc_ookla_lines() {
    local r nm d u g P J name
    r="$G_RH Ookla Speedtest · $(sc_t "the nearest server · its terms accepted by a flag" "ближайший сервер · условия Ookla приняты флагом") "
    if [ "${ST[status]:-}" != ok ]; then
        [ "${1:-0}" = 1 ] && [ "$SC_MODE" != quick ] && [ "$SC_NOSPEEDTEST" != 1 ] && [ -z "${ST[status]:-}" ] || return 0
        sc_vlen "$r"; sc_rep "$G_RH" $((SC_W - REPLY)); SCR+=($'\x01d'"$r$REPLY"$'\x01/')
        SCR+=($'  \x01d'"$(sc_t "after the table" "после таблицы")"$'\x01/'); return 0
    fi
    sc_vlen "$r"; sc_rep "$G_RH" $((SC_W - REPLY)); SCR+=($'\x01d'"$r$REPLY"$'\x01/')
    name="${ST[city]:-?}"; [ -n "${ST[name]:-}" ] && name+=" · ${ST[name]}"
    sc_len "$name"
    if [ "$REPLY" -ge "$SC_NW" ]; then          # a line of its own, cut to the screen (a name of Ookla, not ours)
        sc_cut "  ${ST[city]:-?}"$'\x01d · '"${ST[name]:-}"$'\x01/' "$SC_W"; SCR+=("$REPLY"); printf -v nm '%*s' "$SC_NW" ''
    else sc_name "${ST[city]:-?}" "${ST[name]:-}"; nm=$REPLY; fi
    printf -v g '%*s' $((SC_GAP - 1)) ''
    sc_hbar "${ST[down]}" "$SC_NMAX" "$SC_BW" m; d=$REPLY; sc_cellnum "${ST[down]}" 0 1; d+=$REPLY
    sc_hbar "${ST[up]}" "$SC_NMAX" "$SC_BW" u; u=$REPLY; sc_cellnum "${ST[up]}" 0 0; u+=$REPLY
    P=$'\x01d—\x01/'; J=$'\x01d—\x01/'
    sc_isnum "${ST[ping]:-}" && { sc_f_ms "${ST[ping]}"; P=$REPLY; }
    sc_isnum "${ST[jitter]:-}" && { sc_f_ms "${ST[jitter]}"; J=$REPLY; }
    sc_tail "$P" "" "$J" $'\x01d—\x01/'
    SCR+=("  $nm$d$g$u$REPLY")
}

# sc_net_rows MODE -> lines into SCR: the regions and their rows (live: waiting rows grey, the current one moving);
# a name too long for its column — a line of its own above its numbers
sc_net_rows() {
    local mode=$1 i reg last=""
    for ((i = 0; i < SPN; i++)); do
        reg=${SP_REG[$i]:-any}
        if [ "$reg" != "$last" ] && [ "${SP_M[$i]}" != http ]; then last=$reg; sc_group_line "$reg"; SCR+=("$REPLY"); fi
        if [ "$mode" = live ] && [ "$SC_PHASE" != "done" ] && { [ "$i" -ge "${SC_LDONE:-0}" ] || [ "$i" = "${SC_LROW:--1}" ]; }; then
            sc_live_row "$i"; SCR+=("$REPLY"); continue
        fi
        sc_net_rows_one "$i"
    done
}

# ── the passport, the legend, the footer ──

sc_passport() {   # MODE -> lines into SCR
    local items=() a s r v4 v6
    s=$'\x01m'"$(sc_t "Server check" "Проверка сервера")"$'\x01/\x01d · bench.anten-ka.com\x01/'
    if [ "$1" = live ]; then r=$'\x01d'"$(sc_t "Ctrl+C — stop" "Ctrl+C — прервать")"$'\x01/'
    else                                     # the start of the check, the server's clock (the live frame and the log agree)
        if [ "$SC_LANG" = ru ]; then printf -v r '%(%d.%m.%Y %H:%M)T' "${SC_START:--1}"
        else printf -v r '%(%Y-%m-%d %H:%M)T' "${SC_START:--1}"; fi
        r=$'\x01d'"$r"$'\x01/'
    fi
    sc_vlen "$r"; sc_rp "$s" $((SC_W - REPLY)); SCR+=("$REPLY$r")
    items=("${SYS[os_short]:-${SYS[os]:-?}}"); sc_virt_name; items+=("$REPLY")
    sc_cpu_short "${SYS[cpu]:-?}"; items+=("$REPLY ×${SYS[nproc]:-?}")
    sc_gnum "$(sc_gb "${SYS[mem_kb]:-0}" 1)" 1; items+=("$(sc_t "RAM" "ОЗУ") $REPLY $(sc_unit gb)")
    if sc_isnum "${SYS[swap_kb]:-}" && [ "${SYS[swap_kb]}" -gt 0 ]; then sc_gnum "$(sc_gb "${SYS[swap_kb]}" 1)" 1; items+=("swap $REPLY $(sc_unit gb)"); fi
    if sc_isnum "${SYS[root_kb]:-}"; then sc_gnum "$(sc_gb "${SYS[root_kb]}" 0)" 0; items+=("$(sc_t "disk" "диск") $REPLY $(sc_unit gb)"); fi
    sc_join_wrap "" "${items[@]}"
    items=()
    if [[ " ${SYS[flags]:-} " == *" aes "* ]]; then items+=(AES-NI); else items+=("$(sc_t "no AES-NI" "без AES-NI")"); fi
    [ -n "${SYS[tcp_cc]:-}" ] && items+=("${SYS[tcp_cc]^^}")
    case "${SYS[wg]:-}" in loaded|builtin|module) items+=(WireGuard) ;; *) items+=("$(sc_t "no WireGuard" "без WireGuard")") ;; esac
    v4=${SYS[ipv4_ok]:-${SYS[ipv4]:-0}}; v6=${SYS[ipv6_ok]:-${SYS[ipv6]:-0}}
    if [ "$v4" = 1 ] && [ "$v6" = 1 ]; then items+=("IPv4 + IPv6"); elif [ "$v4" = 1 ]; then items+=("$(sc_t "IPv4 only" "только IPv4")")
    elif [ "$v6" = 1 ]; then items+=("$(sc_t "IPv6 only" "только IPv6")"); else items+=("$(sc_t "no Internet" "нет интернета")"); fi
    sc_join_wrap d "${items[@]}"
}
# sc_join_wrap ROLE ITEM… -> lines into SCR: the items joined by « · » (grey), wrapped at the width
sc_join_wrap() {
    local role=$1 line="" it pre="" post=""; shift
    [ -n "$role" ] && { pre=$'\x01'"$role"; post=$'\x01/'; }
    for it in "$@"; do
        if [ -z "$line" ]; then line=$it; continue; fi
        sc_vlen "$line · $it"
        if [ "$REPLY" -gt "$SC_W" ]; then SCR+=("$pre$line$post"); line=$it; continue; fi
        if [ -n "$role" ]; then line+=" · $it"; else line+=$'\x01d · \x01/'"$it"; fi
    done
    [ -n "$line" ] && SCR+=("$pre$line$post")
    return 0
}

# sc_legend -> lines into SCR: jitter and retransmits (2 lines), ≥ / ≈ when they are on the screen, what was not
# measured and why
sc_legend() {
    local i lb=0 ap=0 items=() t why what lbl g
    if [ "$SC_NONET" != 1 ] && [ "$SPN" -gt 0 ] && [ "$SC_HTTP" != 1 ]; then
        SCR+=($'\x01b'Jitter$'\x01/\x01d — '"$(sc_t "ping swings:" "как скачет пинг:")"$'\x01/ '"${G_SPARK[0]}${G_SPARK[0]}${G_SPARK[0]}${G_SPARK[0]}"$'\x01d '"$(sc_t "steady," "ровно,")"$'\x01/ '"${G_SPARK[0]}${G_SPARK[5]}${G_SPARK[1]}${G_SPARK[7]}"$'\x01d '"$(sc_t "jumpy. Calls won't notice under 5 ms." "рвано. До 5 мс звонки не заметят.")"$'\x01/')
        SCR+=($'\x01d'"$(sc_t "From 20 ms — stutters." "От 20 мс — рывки.")"$'\x01/ \x01b'"$(sc_t "Retr." "Повт.")"$'\x01/\x01d — '"$(sc_t "TCP packets sent again; 0 — no loss." "сколько пакетов TCP ушло заново; 0 — потерь нет.")"$'\x01/')
    fi
    for ((i = 0; i < SPN; i++)); do [ "${SP_DLB[$i]:-0}${SP_ULB[$i]:-0}" != 00 ] && [ "${SP_CAP[$i]:-0}" -gt 0 ] && lb=1; done
    for i in "${SR_A[@]}"; do [ "$i" = 1 ] && ap=1; done
    [ "$lb" = 1 ] && SCR+=($'\x01d'"$(sc_t "≥ — not less than: the port of the point is slower than this server" "≥ — не меньше: порт точки медленнее этого сервера")"$'\x01/')
    if [ "${SC_NET_NCAP:-0}" -gt 0 ]; then
        sc_gnum "$(awk -v b="$SC_NET_BYTES" 'BEGIN { printf "%.1f", b / 1e9 }')" 1; g=$REPLY
        SCR+=($'\x01d'"$(sc_t "≥ — held by the traffic budget: iperf3 $g GB of $((SC_NET_BUDGET / 1000000000)) (SC_NET_BUDGET_GB)" \
            "≥ — скорость держал бюджет трафика: iperf3 $g ГБ из $((SC_NET_BUDGET / 1000000000)) (SC_NET_BUDGET_GB)")"$'\x01/')
    fi
    [ "$ap" = 1 ] && SCR+=($'\x01d'"$(sc_t "≈ — approximate: a built-in method or a shorter run, the units of Kaskad" "≈ — приближённо: встроенный способ или укороченный замер, единицы Kaskad")"$'\x01/')
    for t in sysbench fio; do
        [ -z "${TOOL[$t]:-}" ] && [ -n "${TWHY[$t]:-}" ] || continue
        sc_tool_why "${TWHY[$t]}"; why=$REPLY
        if [ "$t" = sysbench ]; then what=$(sc_t "CPU and memory by built-in methods" "процессор и память — встроенными способами")
        else what=$(sc_t "the disk by dd, no random I/O" "диск — через dd, без случайного ввода-вывода"); fi
        items+=("$t: $why — $what")
    done
    for t in disk_fsync rtt dns net_up; do
        sc_isnum "${V[$t]:-}" && continue
        case "${WHY[$t]:-}" in no_sysbench|no_fio|skipped|"") continue ;; esac
        case "$t" in
            disk_fsync) lbl=$(sc_t "Disk, write + flush" "Диск, запись + сброс") ;; rtt) lbl=$(sc_t "Latency" "Задержка") ;;
            dns) lbl=DNS ;; net_up) lbl=$(sc_t "Upload" "Отдача") ;;
        esac
        items+=("$lbl: $(sc_why_text "${WHY[$t]}")")
    done
    for t in "${items[@]}"; do sc_wrap_into d "$(sc_t "Not measured" "Не измерено") · $t"; done
}
# sc_wrap_into ROLE TEXT -> lines into SCR: TEXT wrapped at the width by words, indented 2 after the first line
sc_wrap_into() {
    local role=$1 w line="" first=1
    for w in $2; do
        if [ -z "$line" ]; then line=$w; continue; fi
        sc_len "$line $w"
        if [ "$REPLY" -gt $((SC_W - (first ? 0 : 2))) ]; then
            if [ "$first" = 1 ]; then SCR+=($'\x01'"$role$line"$'\x01/'); else SCR+=($'  \x01'"$role$line"$'\x01/'); fi
            first=0; line=$w
        else line+=" $w"; fi
    done
    if [ -n "$line" ]; then if [ "$first" = 1 ]; then SCR+=($'\x01'"$role$line"$'\x01/'); else SCR+=($'  \x01'"$role$line"$'\x01/'); fi; fi
}

sc_footer() {   # -> a line into SCR: «✓ Done, m:ss · network N of M · nothing installed · bench.anten-ka.com»
    local i n=0 el net h c
    if [ "$SC_MODE" = quick ]; then          # the quick check (the default) says how to get the full one
        c="wget -qO- https://bench.anten-ka.com$([ "$SC_LANG" = en ] && echo /en)|bash -s -- --full"
        h=$(sc_t "Quick check. In full:" "Быстрая проверка. Подробно:")
        sc_len "$h $c"
        if [ "$REPLY" -le "$SC_W" ]; then SCR+=($'\x01d'"$h"$'\x01/ '"$c")
        else SCR+=($'\x01d'"$h"$'\x01/' "  $c"); fi
    fi
    sc_clock; el=$(( (SC_NOW - SC_T0) / 1000000 )); sc_f_time "$el"
    for ((i = 0; i < SPN; i++)); do { [ "${SP_DST[$i]}" = ok ] || [ "${SP_UST[$i]}" = ok ]; } && n=$((n + 1)); done
    if [ "$SC_NONET" = 1 ]; then net=$(sc_t "network not checked" "сеть не проверялась")
    else net="$(sc_t "network" "сеть") $n $(sc_t "of" "из") $SPN"; fi
    SCR+=($'\x01g'"$G_OK"$'\x01/ '"$(sc_t "Done" "Готово"), $REPLY"$'\x01d · '"$net · $(sc_t "nothing installed" "ничего не установлено") ·"$'\x01/ \x01m'"bench.anten-ka.com"$'\x01/')
}

# sc_screen MODE -> SCR: every line of the screen. MODE: final | live (the moving parts as \x02 marks for the ticker)
sc_screen() {
    local mode=$1 live=0
    [ "$mode" = live ] && [ "$SC_PHASE" != "done" ] && live=1
    SCR=()
    sc_passport "$mode"; SCR+=("")
    [ "$live" = 1 ] && sc_net_notice          # what the network will cost (kept to the end: the frame keeps its height)
    sc_scale_lines "$mode"
    if [ "$SC_NONET" != 1 ] && [ "$SPN" -gt 0 ]; then
        sc_net_scale; sc_net_names
        SCR+=(""); sc_net_head; sc_net_rows "$mode"; sc_ookla_lines "$live"
    fi
    SCR+=("")
    # the legend (its lines depend on what was measured) — on the final screen only: the live frame keeps its height
    [ "$live" = 1 ] || sc_legend
    if [ "$live" = 1 ]; then
        sc_rep "$G_RUNT" 62
        SCR+=($'\x02b \x01b\x02t\x01/\x01d '"$(sc_t "of" "из") ~${SC_EST_TXT:-?}"$'\x01/')
        SCR+=($'\x01d'"$(sc_t "Now" "Сейчас")"$'\x01/  \x02n')
    else
        sc_footer
    fi
}

# sc_screen_init COLOR — the colours, the characters, the size of the terminal (stdout a terminal: stty of /dev/tty;
# else 80 columns), the width of the screen and of the axis, the live state
sc_screen_init() {
    local tr="" tc=""
    sc_colors "$1"; sc_glyphs
    SC_COLS=80; SC_ROWS=24
    if [ -t 1 ]; then
        read -r tr tc < <(stty size </dev/tty 2>/dev/null) || :
        sc_isnum "$tc" && SC_COLS=$tc; sc_isnum "$tr" && SC_ROWS=$tr
    fi
    [ -n "${SC_TEST_COLS:-}" ] && SC_COLS=$SC_TEST_COLS
    SC_W=80; [ "$SC_COLS" -ge 100 ] && { SC_W=$SC_COLS; [ "$SC_W" -gt 120 ] && SC_W=120; }
    SC_AX=40; [ "$SC_COLS" -ge 100 ] && { SC_AX=$((40 + SC_COLS - 80)); [ "$SC_AX" -gt 60 ] && SC_AX=60; }
    sc_axis_ticks
    SC_LIVE=0; SC_WAS_LIVE=0; SC_PHASE=start; SC_LDONE=0; SC_LROW=-1; SC_EST=0; SC_NMAX_FIX=""; SC_LOG_NET=0
    SC_LOG_PEND=(); SC_LIVE_H=0
    SC_TICK_PID=""; printf -v SC_START '%(%s)T' -1
}

# sc_print_lines LINE… — painted, one a line (no redraw)
sc_print_lines() {
    local l
    sc_prog_clear          # the progress line (stderr, a terminal) would merge into the first line
    for l in "$@"; do sc_paint "$l"; printf '%s\n' "$REPLY"; done
}

# ── the line log (a pipe, a file, no colour, a small terminal): every line printed once, when it is final ──
# The passport first; the network rows as their points finish (the table head with the first row: the scale of the
# bars is fixed by it); then the scores (two of the seven come from the network), the legend and the footer.

sc_log_head() {
    if [ "$SC_JSON" = 1 ] || [ "${SC_LIVE:-0}" = 1 ]; then return 0; fi
    SCR=(); sc_passport final; SCR+=(""); sc_net_notice; sc_print_lines "${SCR[@]}"
}
# sc_net_notice -> lines into SCR (and a blank one): what the network part will cost in traffic and how to cut it —
# shown before the network (the log head, the live frame); nothing with --no-net
sc_net_notice() {
    local t gb
    [ "$SC_NONET" = 1 ] && return 0
    if [ "$SC_NOIPERF" = 1 ]; then t=$(sc_t "Network: no iperf3 (SC_NO_IPERF=1)." "Сеть: без iperf3 (SC_NO_IPERF=1).")
    elif [ "${SC_NET_BUDGET:-0}" -gt 0 ]; then
        gb=$((SC_NET_BUDGET / 1000000000))
        t=$(sc_t "Network: iperf3 up to $gb GB of traffic (SC_NET_BUDGET_GB): faster rows are held, marked «≥»." \
            "Сеть: iperf3 до $gb ГБ трафика (SC_NET_BUDGET_GB): быстрее — с пределом и пометкой «≥».")
    else t=$(sc_t "Network: iperf3 with no limit of traffic (SC_NET_BUDGET_GB=0)." "Сеть: iperf3 без предела трафика (SC_NET_BUDGET_GB=0).")
    fi
    if [ "$SC_MODE" != quick ] && [ "$SC_NOSPEEDTEST" != 1 ]; then
        t+=" $(sc_t "Speedtest by Ookla — up to 10 GB more, its terms accepted by a flag." "Speedtest by Ookla — ещё до 10 ГБ, его условия приняты флагом.")"
    fi
    if [ "$SC_MODE" != quick ]; then t+=" $(sc_t "Less: without --full / --all." "Меньше: без --full / --all.")"; fi
    t+=" $(sc_t "No iperf3: SC_NO_IPERF=1; no network: --no-net." "Без iperf3: SC_NO_IPERF=1; без сети: --no-net.")"
    sc_wrap_into d "$t"; SCR+=("")
}
sc_log_row() {   # I — SC_ON_ROW of the line log (nothing in JSON or on the live screen: stdout is theirs)
    local i=$1 j best
    if [ "$SC_JSON" = 1 ] || [ "${SC_LIVE:-0}" = 1 ]; then return 0; fi
    if [ "${SC_LOG_NET:-0}" = 0 ]; then
        # the head and the scale of the bars come with the first row that has a value; rows before it wait (the last
        # row does not: the head then has the scale of 1 Gbit/s)
        if [ -z "${SP_DOWN[$i]:-}${SP_UP[$i]:-}" ] && [ "$i" -lt $((SPN - 1)) ]; then SC_LOG_PEND+=("$i"); return 0; fi
        SC_LOG_NET=1; SC_LOG_REG=""
        best=$(awk -v a="${SP_DOWN[$i]:-0}" -v b="${SP_UP[$i]:-0}" 'BEGIN { print (a + 0 > b + 0 ? a + 0 : b + 0) }')
        SC_NMAX_FIX=$(awk -v b="$best" 'BEGIN { n = split("1000 2500 10000 25000 100000", s, " "); for (i = 1; i <= n; i++) if (b <= s[i]) { print s[i]; exit } print s[n] }')
        sc_net_scale; sc_net_names
        SCR=(); sc_net_head; sc_print_lines "${SCR[@]}"
        for j in "${SC_LOG_PEND[@]}"; do sc_log_row_lines "$j"; done
        SC_LOG_PEND=()
    fi
    sc_log_row_lines "$i"
}
sc_log_row_lines() {   # I — the region line (when it changes) and the row I, printed
    local i=$1
    SCR=()
    if [ "${SP_REG[$i]:-any}" != "$SC_LOG_REG" ] && [ "${SP_M[$i]}" != http ]; then SC_LOG_REG=${SP_REG[$i]:-any}; SCR+=($'\x01b'"$(sc_region_name "$SC_LOG_REG")"$'\x01/'); fi
    sc_net_rows_one "$i"
    sc_print_lines "${SCR[@]}"
}
sc_net_rows_one() {   # I -> the lines of one finished row into SCR (a name too long for its column: a line of its own)
    local CITY OP
    sc_row_names "$1"
    sc_len "$CITY · $OP"
    if [ "$REPLY" -ge "$SC_NW" ]; then SCR+=("  $CITY"$'\x01d · '"$OP"$'\x01/'); sc_net_row "$1" "  " 1
    else sc_net_row "$1"; fi
    SCR+=("$REPLY")
}
sc_log_tail() {
    local j
    if [ "${SC_LOG_NET:-0}" = 0 ] && [ ${#SC_LOG_PEND[@]} -gt 0 ]; then
        SC_LOG_NET=1; SC_LOG_REG=""; sc_net_scale; sc_net_names
        SCR=(); sc_net_head; sc_print_lines "${SCR[@]}"
        for j in "${SC_LOG_PEND[@]}"; do sc_log_row_lines "$j"; done
        SC_LOG_PEND=()
    fi
    SCR=()
    if [ "$SC_NONET" != 1 ] && [ "$SPN" -gt 0 ]; then
        [ "${SC_LOG_NET:-0}" = 1 ] || { sc_net_scale; sc_net_names; }
        sc_ookla_lines
    fi
    SCR+=("")
    sc_scale_lines final
    SCR+=(""); sc_legend; sc_footer
    sc_print_lines "${SCR[@]}"
}

# ── the live screen: the main process writes the frame (sc_live_frame), a ticker draws it every 0.1 s ──────────────

sc_live_ok() {   # -> true when the live screen can be used: a terminal with colour, at least 80 × (the frame + 2)
    [ "$SC_JSON" != 1 ] && [ -n "${SC_CM:-}" ] && [ -t 1 ] || return 1
    [ "${SC_COLS:-0}" -ge 80 ] && [ "${SC_ROWS:-0}" -ge 42 ] || return 1
    sc_screen live; [ $(( ${#SCR[@]} + 2 )) -le "$SC_ROWS" ]
}
sc_live_frame() {   # the current frame into $SC_WORK/live.f (whole: written aside, then moved)
    [ "${SC_LIVE:-0}" = 1 ] || return 0
    local l f="$SC_WORK/live.f" n
    sc_screen live
    n=${#SCR[@]}
    if [ "${SC_LIVE_H:-0}" -gt 2 ]; then
        while [ "$n" -lt "$SC_LIVE_H" ]; do SCR=("${SCR[@]:0:n-2}" "" "${SCR[@]:n-2}"); n=$((n + 1)); done
        [ "$n" -gt "$SC_LIVE_H" ] && SCR=("${SCR[@]:0:SC_LIVE_H-2}" "${SCR[@]:n-2}")
    fi
    {
        printf '%s\n' "$SC_EST"
        for l in "${SCR[@]}"; do sc_paint "$l"; printf '%s\n' "$REPLY"; done
    } >"$f.n" 2>/dev/null && mv -f "$f.n" "$f" 2>/dev/null
    return 0
}
sc_live_est() {   # -> SC_EST (s): what the check should take, from what is left
    local t o row el
    sc_iperf_len; t=${REPLY% *}; o=${REPLY#* }; row=$((2 * (t + o + 2)))
    sc_clock; el=$(( (SC_NOW - SC_T0) / 1000000 ))
    case "$1" in
        sys) if [ "$SC_MODE" = quick ]; then SC_EST=$((el + 40 + 6 + SPN * row)); else SC_EST=$((el + 75 + 6 + SPN * row + 35)); fi ;;
        *) if [ "$SC_MODE" = quick ]; then SC_EST=$((el + 6 + SPN * row)); else SC_EST=$((el + 6 + SPN * row + 35)); fi ;;
    esac
    sc_f_time "$SC_EST"; SC_EST_TXT=$REPLY
}
sc_live_ticker() {   # (a background process) redraws the frame: whole when it changed, else its moving lines only
    local f="$SC_WORK/live.f" nf="$SC_WORK/live.n" cur="" last="" tick=0 lines=() dyn=() i l n=0 est=0 el bar sp now
    local nowt="" s out
    trap 'exit 0' TERM INT HUP
    while kill -0 "$SC_MAIN_PID" 2>/dev/null && [ -e "$SC_WORK/live.on" ]; do
        cur=""; [ -r "$f" ] && { IFS= read -r -d '' cur <"$f" || :; }
        nowt=""; [ -r "$nf" ] && { IFS= read -r -d '' nowt <"$nf" || :; }
        if [ -n "$cur" ]; then
            if [ "$cur" != "$last" ]; then          # a new frame: split once, note the lines that move
                lines=(); dyn=(); s=$cur
                while [ -n "$s" ]; do lines+=("${s%%$'\n'*}"); if [[ $s == *$'\n'* ]]; then s=${s#*$'\n'}; else s=""; fi; done
                est=${lines[0]}; n=${#lines[@]}
                for ((i = 1; i < n; i++)); do [[ ${lines[$i]} == *$'\x02'* ]] && dyn+=("$i"); done
            fi
            sc_clock; el=$(( (SC_NOW - SC_T0) / 1000000 ))
            sp=${G_SPIN[$((tick % ${#G_SPIN[@]}))]}
            sc_f_time "$el"; printf -v now '%5s' "$REPLY"
            i=0; [ "$est" -gt 0 ] && i=$((el * 62 / est)); [ "$i" -gt 61 ] && i=61
            sc_rep "$G_RUN" "$i"; bar=$'\033['"${SC_RM}m$REPLY"$'\033[0m\033['"${SC_RD}m"; sc_rep "$G_RUNT" $((62 - i)); bar+="$REPLY"$'\033[0m'
            out=""
            if [ "$cur" != "$last" ]; then
                out=$'\0338'
                for ((i = 1; i < n; i++)); do
                    l=${lines[$i]}; l=${l//$'\x02s'/$sp}; l=${l//$'\x02t'/$now}; l=${l//$'\x02b'/$bar}; l=${l//$'\x02n'/$nowt}
                    out+=$'\r'"$l"$'\033[K'; [ "$i" -lt $((n - 1)) ] && out+=$'\n'
                done
                out+=$'\033[J'
            else
                for i in "${dyn[@]}"; do
                    l=${lines[$i]}; l=${l//$'\x02s'/$sp}; l=${l//$'\x02t'/$now}; l=${l//$'\x02b'/$bar}; l=${l//$'\x02n'/$nowt}
                    out+=$'\0338'; [ "$i" -gt 1 ] && out+=$'\033['"$((i - 1))B"
                    out+=$'\r'"$l"$'\033[K'
                done
            fi
            printf '%s' "$out"
            last=$cur
        fi
        tick=$((tick + 1))
        sleep 0.1 2>/dev/null || sleep 1
    done
}
sc_live_start() {
    local i
    SC_LIVE=1; SC_PROGRESS=0; SC_MAIN_PID=$$
    sc_screen live
    printf '\033[?25l'
    for ((i = 0; i < ${#SCR[@]}; i++)); do printf '\n'; done
    printf '\033[%dA\0337' "${#SCR[@]}"; SC_LIVE_H=${#SCR[@]}; SC_WAS_LIVE=1
    : >"$SC_WORK/live.on"
    sc_live_frame
    sc_live_ticker &
    SC_TICK_PID=$!
}
sc_live_stop() {   # the ticker stopped, the final screen drawn over the frame, the cursor back
    [ "${SC_LIVE:-0}" = 1 ] || return 0
    rm -f "$SC_WORK/live.on" 2>/dev/null
    if [ -n "${SC_TICK_PID:-}" ]; then kill "$SC_TICK_PID" 2>/dev/null; wait "$SC_TICK_PID" 2>/dev/null; SC_TICK_PID=""; fi
    SC_LIVE=0
    printf '\0338\033[J'
    return 0
}

# sc_render_text — the final screen: on a live terminal drawn over the frame; in the line log only what is left
sc_render_text() {
    if [ "${SC_WAS_LIVE:-0}" = 1 ]; then
        sc_live_stop
        SC_PHASE="done"; sc_screen final; sc_print_lines "${SCR[@]}"
        printf '\033[?25h'
    else
        SC_PHASE="done"; sc_log_tail
    fi
}

# sc_row_status I -> REPLY: the status of a row of the speed table — ok (both ways), partial (one), else the one of the
# download (busy, no_answer, error, time); an HTTP row has no upload: its download says it
sc_row_status() {
    local d=${SP_DST[$1]} u=${SP_UST[$1]}
    if [ "${SP_M[$1]}" = http ]; then REPLY=$d
    elif [ "$d" = ok ] && [ "$u" = ok ]; then REPLY=ok
    elif [ "$d" = ok ] || [ "$u" = ok ]; then REPLY=partial
    else REPLY=$d; fi
}

sc_json_bool() { if [ "${1:-0}" = 1 ]; then REPLY=true; else REPLY=false; fi; }

sc_report_json() {
    local t g s first=1 el want tr t1 i k
    sc_clock; el=$(( (SC_NOW - SC_T0) / 1000000 ))
    s="{\"tool\":\"server-check\",\"version\":\"$SC_VERSION\",\"scale\":\"kaskad $SC_SCALE_VERSION\",\"mode\":\"$SC_MODE\""
    s+=",\"points\":\"$SC_POINTS\",\"lang\":\"$SC_LANG\",\"elapsed_s\":$el,\"system\":{"
    for t in os kernel arch virt cpu root_fs tcp_cc qdisc wg headers tun; do
        sc_json_str "${SYS[$t]:-}"; s+="$([ $first = 1 ] || echo ,)\"$t\":$REPLY"; first=0
    done
    sc_json_num "${SYS[nproc]}"; s+=",\"nproc\":$REPLY"
    sc_json_num "${SYS[tpc]}"; s+=",\"threads_per_core\":$REPLY"
    s+=",\"cpu_flags\":["; first=1
    for t in ${SYS[flags]}; do s+="$([ $first = 1 ] || echo ,)\"$t\""; first=0; done
    s+="]"
    for t in mem_kb avail_kb swap_kb root_kb root_free_kb load; do sc_json_num "${SYS[$t]:-}"; s+=",\"$t\":$REPLY"; done
    for t in ipv4 ipv6 container; do sc_json_bool "${SYS[$t]}"; s+=",\"$t\":$REPLY"; done
    for t in ipv4_ok ipv6_ok; do
        case "${SYS[$t]}" in 1) s+=",\"$t\":true" ;; 0) s+=",\"$t\":false" ;; *) s+=",\"$t\":null" ;; esac
    done
    s+="},\"groups\":{"; first=1
    for g in cpu mem disk net_bw net_q; do sc_json_num "${GS[$g]:-}"; s+="$([ $first = 1 ] || echo ,)\"$g\":$REPLY"; first=0; done
    s+="},\"groups_approx\":{"; first=1
    for g in cpu mem disk net_bw net_q; do sc_json_bool "${GA[$g]:-0}"; s+="$([ $first = 1 ] || echo ,)\"$g\":$REPLY"; first=0; done
    s+="},\"tests\":{"; first=1
    for t in cpu_1t cpu_mt chacha_1t aes_1t chacha_mt mem_gb mem_bw disk_rr disk_rw disk_fsync net_down net_up rtt jitter loss dns; do
        s+="$([ $first = 1 ] || echo ,)\"$t\":{"; first=0
        sc_json_num "${V[$t]:-}"; s+="\"value\":$REPLY"
        sc_json_num "${SS[$t]:-}"; s+=",\"score\":$REPLY"
        sc_json_str "${M[$t]:-}"; s+=",\"method\":$REPLY"
        if [ -n "${WHY[$t]:-}" ]; then s+=",\"why\":\"${WHY[$t]}\""; else s+=",\"why\":null"; fi
        sc_json_bool "${AP[$t]:-0}"; s+=",\"approx\":$REPLY"
        sc_json_num "${NN[$t]:-}"; s+=",\"n\":$REPLY}"
    done
    s+="},\"extra\":{"; first=1
    for t in chacha_scale steal swap_gb disk_seq_r disk_seq_w disk_file_mb; do
        sc_json_num "${X[$t]:-}"; s+="$([ $first = 1 ] || echo ,)\"$t\":$REPLY"; first=0
    done
    sc_json_bool "${X[net_down_lb]:-0}"; s+=",\"net_down_at_least\":$REPLY"
    sc_json_bool "${X[net_up_lb]:-0}"; s+=",\"net_up_at_least\":$REPLY"
    s+="},\"tools\":{"; first=1
    for t in sysbench fio iperf3 speedtest; do
        sc_json_str "${TSRC[$t]:-}"; [ -n "${TSRC[$t]:-}" ] || REPLY=null
        s+="$([ $first = 1 ] || echo ,)\"$t\":{\"source\":$REPLY"; first=0
        if [ -n "${TWHY[$t]:-}" ]; then s+=",\"why\":\"${TWHY[$t]}\"}"; else s+=",\"why\":null}"; fi
    done
    s+="},\"traffic\":{"
    sc_json_num "${SC_NET_BYTES:-}"; s+="\"iperf3_bytes\":$REPLY"
    sc_json_num "${ST[bytes]:-}"; s+=",\"speedtest_bytes\":$REPLY"
    if [ "${SC_NET_BUDGET:-0}" -gt 0 ]; then s+=",\"budget_bytes\":$SC_NET_BUDGET"; else s+=",\"budget_bytes\":null"; fi
    s+=",\"floor_cap_mbit\":$SC_NET_CAPMBIT,\"held_runs\":${SC_NET_NCAP:-0}"
    s+="},\"speed\":["
    for ((i = 0; i < SPN; i++)); do
        [ "$i" = 0 ] || s+=","
        s+="{\"id\":\"${SP_ID[$i]}\",\"method\":\"${SP_M[$i]}\""
        sc_json_str "${SP_REG[$i]:-}"; s+=",\"region\":$REPLY"; sc_json_bool "$([ "${SP_FB[$i]:-0}" -gt 0 ] && echo 1)"
        s+=",\"fallback\":$REPLY"
        sc_json_str "${SP_CEN[$i]}"; s+=",\"city\":$REPLY"; sc_json_str "${SP_CRU[$i]}"; s+=",\"city_ru\":$REPLY"
        sc_json_str "${SP_OEN[$i]}"; s+=",\"operator\":$REPLY"; sc_json_str "${SP_ORU[$i]}"; s+=",\"operator_ru\":$REPLY"
        sc_json_str "${SP_HOST[$i]}"; s+=",\"host\":$REPLY"; sc_json_num "${SP_PORT[$i]}"; s+=",\"port\":$REPLY"
        sc_json_num "${SP_DOWN[$i]}"; s+=",\"down\":$REPLY"; sc_json_num "${SP_UP[$i]}"; s+=",\"up\":$REPLY"
        sc_json_num "${SP_PING[$i]}"; s+=",\"ping\":$REPLY"
        sc_json_num "${SP_PMED[$i]:-}"; s+=",\"ping_median\":$REPLY"; sc_json_num "${SP_PAVG[$i]:-}"; s+=",\"ping_avg\":$REPLY"; sc_json_num "${SP_PJIT[$i]:-}"; s+=",\"ping_jitter\":$REPLY"
        sc_json_num "${SP_PLOSS[$i]:-}"; s+=",\"ping_loss\":$REPLY"; sc_json_num "${SP_PN[$i]:-}"; s+=",\"ping_n\":$REPLY"
        s+=",\"ping_samples\":["; k=""
        for t in ${SP_PSAMP[$i]:-}; do sc_json_num "$t"; s+="$k$REPLY"; k=","; done
        s+="]"
        sc_json_num "${SP_MINRTT[$i]:-}"; s+=",\"min_rtt\":$REPLY"
        sc_json_num "${SP_RDOWN[$i]:-}"; s+=",\"retr_down\":$REPLY"; sc_json_num "${SP_RUP[$i]:-}"; s+=",\"retr_up\":$REPLY"
        sc_row_status "$i"; s+=",\"status\":\"$REPLY\",\"down_status\":\"${SP_DST[$i]}\",\"up_status\":\"${SP_UST[$i]}\""
        sc_json_bool "${SP_DLB[$i]}"; s+=",\"down_at_least\":$REPLY"; sc_json_bool "${SP_ULB[$i]}"; s+=",\"up_at_least\":$REPLY}"
    done
    s+="],\"speedtest\":{\"status\":\"${ST[status]:-skipped}\""
    if [ -n "${ST[why]:-}" ]; then s+=",\"why\":\"${ST[why]}\""; else s+=",\"why\":null"; fi
    for k in down up ping jitter loss; do sc_json_num "${ST[$k]:-}"; s+=",\"$k\":$REPLY"; done
    sc_json_num "${ST[id]:-}"; s+=",\"server_id\":$REPLY"
    for k in name city country; do sc_json_str "${ST[$k]:-}"; [ -n "${ST[$k]:-}" ] || REPLY=null; s+=",\"$k\":$REPLY"; done
    s+="},\"download\":["; first=1
    for want in any eu ru us; do
        [ "${SC_HTTP:-0}" = 1 ] && [ -n "${DLR[$want]+x}" ] || continue
        sc_json_num "${DLV[$want]:-}"
        s+="$([ $first = 1 ] || echo ,){\"region\":\"$want\",\"point\":\"${DLR[$want]:-}\",\"mbps\":$REPLY"
        sc_json_bool "${DLB[$want]:-0}"; s+=",\"at_least\":$REPLY"
        s+=",\"tries\":["; t1=1
        for tr in ${DLT[$want]:-}; do          # «id=Mbit/s» or «id=-»; ids are ours ([a-z0-9-])
            sc_json_num "${tr#*=}"; s+="$([ $t1 = 1 ] || echo ,){\"point\":\"${tr%%=*}\",\"mbps\":$REPLY}"; t1=0
        done
        s+="]}"
        first=0
    done
    s+="]}"
    printf '%s\n' "$s"
}

# ── the run ─────────────────────────────────────────────────────────────────────────────────────────────────────────

sc_usage() {
    if [ "$SC_LANG" = ru ]; then cat <<'EOT'
Проверка сервера Kaskad: процессор, память, диск, сеть — баллы 0–1000 по шкале Kaskad и скорость iperf3.
  wget -qO- https://bench.anten-ka.com|bash      English: wget -qO- https://bench.anten-ka.com/en|bash
  curl -fsSL https://bench.anten-ka.com|bash -s -- --full
  без ключей  быстрая проверка, ≈ 1,5 мин: iperf3 до Москвы, Франкфурта и Нью-Йорка, без Speedtest (--quick — то же)
  --full      подробно, ≈ 6 мин: 13 точек iperf3 (Россия, Европа, США), Speedtest by Ookla, тесты системы целиком
  --all       все 29 точек iperf3: ещё ЭР-Телеком (Дом.ру), Билайн, Европа, США, Азия (≈ 11 мин)
  --json      ответ в JSON для программ
  --no-net    без замеров сети
  --ascii     только ASCII (полосы # и .)
  --no-color  без цвета (или NO_COLOR=1)
  --lang en   по-английски
  Трафик сети: iperf3 до 5 ГБ (--full — до 30, --all — до 60): быстрые строки — с пределом и «≥»; предел —
  SC_NET_BUDGET_GB (0 — без предела). Speedtest (--full, --all) — ещё до 10 ГБ. Без iperf3 — SC_NO_IPERF=1, без сети —
  --no-net.
  Переменные: SC_LANG=ru|en, SC_FULL=1, SC_ALL=1, SC_JSON=1, SC_NO_NET=1, SC_NO_FETCH=1 (ничего не скачивать),
  SC_NO_IPERF=1, SC_NO_SPEEDTEST=1, SC_NET_BUDGET_GB=N, SC_ASCII=1, NO_COLOR=1.
Ничего не устанавливает: недостающие iperf3, Speedtest, sysbench и fio — только во временную папку, удаляется в конце.
Speedtest by Ookla запускается с --accept-license --accept-gdpr (условия Ookla).
EOT
    else cat <<'EOT'
Kaskad server check: CPU, memory, disk, network — scores 0–1000 on the Kaskad scale and the iperf3 speed.
  wget -qO- https://bench.anten-ka.com/en|bash
  curl -fsSL https://bench.anten-ka.com/en|bash -s -- --full
  no options  the quick check, ≈ 1.5 min: iperf3 to Moscow, Frankfurt and New York, no Speedtest (--quick is the same)
  --full      in full, ≈ 6 min: 13 iperf3 points (Russia, Europe, USA), Speedtest by Ookla, the system tests at length
  --all       all 29 iperf3 points: ER-Telecom (Dom.ru) cities, Beeline, Europe, USA, Asia (≈ 11 min)
  --json      machine-readable JSON
  --no-net    no network tests
  --ascii     ASCII only (bars of # and .)
  --no-color  no colour (or NO_COLOR=1)
  --lang ru   in Russian
  Network traffic: iperf3 up to 5 GB (--full up to 30, --all up to 60): fast rows are held, marked «≥»; the limit —
  SC_NET_BUDGET_GB (0 — none). Speedtest (--full, --all) — up to 10 GB more. No iperf3 — SC_NO_IPERF=1, no network —
  --no-net.
  Environment: SC_LANG=ru|en, SC_FULL=1, SC_ALL=1, SC_JSON=1, SC_NO_NET=1, SC_NO_FETCH=1 (fetch nothing),
  SC_NO_IPERF=1, SC_NO_SPEEDTEST=1, SC_NET_BUDGET_GB=N, SC_ASCII=1, NO_COLOR=1.
Installs nothing: missing iperf3, Speedtest, sysbench and fio go into its temporary directory only, removed at the end.
Speedtest by Ookla runs with --accept-license --accept-gdpr (the terms of Ookla).
EOT
    fi
}

# sc_kill_own SIGNAL PID… — a signal to the processes that are still children of this shell (a PID of ours that has
# ended and been taken by another process is left alone)
sc_kill_own() {
    local sig=$1 p pp; shift
    for p in "$@"; do
        [[ $p =~ ^[0-9]+$ ]] || continue
        pp=$(awk '{ sub(/^.*\) /, ""); print $2 }' "/proc/$p/stat" 2>/dev/null)
        [ "$pp" = "$$" ] && kill "-$sig" "$p" 2>/dev/null
    done
    return 0
}

sc_cleanup() {
    local p
    if [ -n "${SC_TICK_PID:-}" ]; then kill "$SC_TICK_PID" 2>/dev/null; wait "$SC_TICK_PID" 2>/dev/null; SC_TICK_PID=""; fi
    if [ "${SC_LIVE:-0}" = 1 ]; then SC_LIVE=0; printf '\0338\033[%dB\n\033[?25h' "${SC_LIVE_H:-0}"; fi
    # shellcheck disable=SC2046,SC2086
    sc_kill_own TERM ${SC_DL_PIDS:-} $(jobs -p 2>/dev/null)
    [ "${SC_PROGRESS:-0}" = 1 ] && printf '\r\033[K' >&2
    for p in "${SC_WORK:-}" "${SC_WORK2:-}" "${SC_WORK3:-}"; do
        case "$p" in */server-check.?*|*/.server-check.?*) [ -d "$p" ] && rm -rf -- "$p" ;; esac
    done
    SC_WORK=""; SC_WORK2=""; SC_WORK3=""
}

# sc_collect — every measurement, in order; the progress line on stderr says where it is (nothing is printed on stdout)
sc_collect() {
    local live=0
    if [ "$SC_JSON" != 1 ] && sc_live_ok; then live=1; SC_PROGRESS=0; fi    # the frame fits: live, no progress line
    sc_sysinfo
    SC_PHASE=sys
    if [ "$live" = 1 ]; then sc_live_est sys; sc_live_start; else sc_log_head; fi
    sc_tools
    [ -n "${TOOL[iperf3]:-}" ] || SPN=0
    sc_live_frame
    sc_cpu
    sc_mem
    sc_disk
    sc_net
    sc_prog_clear
}

sc_main() {
    sc_init
    SC_LANG=${SC_LANG:-$SC_DEFAULT_LANG}
    # the quick check by default (owner 12.11); SC_QUICK=1 / --quick — the same (kept for old commands)
    SC_MODE=quick; SC_POINTS=quick; SC_ON_ROW=""
    [ "${SC_FULL:-0}" = 1 ] && { SC_MODE=full; SC_POINTS=base; }
    [ "${SC_ALL:-0}" = 1 ] && { SC_MODE=full; SC_POINTS=long; }
    SC_JSON=${SC_JSON:-0}; SC_NONET=${SC_NO_NET:-0}; SC_ASCII=${SC_ASCII:-0}
    # a locale set without UTF-8 — ASCII (LC_ALL=C below is for the parsers only)
    case "${LC_ALL:-${LC_CTYPE:-${LANG:-}}}" in ""|*UTF-8*|*utf-8*|*UTF8*|*utf8*) ;; *) SC_ASCII=1 ;; esac
    SC_NOFETCH=${SC_NO_FETCH:-0}; SC_NOIPERF=${SC_NO_IPERF:-0}; SC_NOSPEEDTEST=${SC_NO_SPEEDTEST:-0}
    local color=1; [ -n "${NO_COLOR:-}" ] && color=0
    while [ $# -gt 0 ]; do
        case "$1" in
            --lang) SC_LANG=${2:-}; shift ;;
            --lang=*) SC_LANG=${1#*=} ;;
            --quick) SC_MODE=quick; SC_POINTS=quick ;;
            --full) SC_MODE=full; SC_POINTS=base ;;
            --all) SC_MODE=full; SC_POINTS=long ;;
            --json) SC_JSON=1 ;;
            --no-net) SC_NONET=1 ;;
            --ascii) SC_ASCII=1 ;;
            --no-color) color=0 ;;
            -h|--help) case "$SC_LANG" in ru|en) ;; *) SC_LANG=$SC_DEFAULT_LANG ;; esac; sc_usage; return 0 ;;
            --version) echo "server-check $SC_VERSION (Kaskad scale $SC_SCALE_VERSION)"; return 0 ;;
            *) echo "server-check: unknown option «$1» (--help)" >&2; return 2 ;;
        esac
        shift
    done
    case "$SC_LANG" in ru|en) ;; *) echo "server-check: --lang ru|en" >&2; return 2 ;; esac
    sc_net_budget_init
    if [ "$(uname -s 2>/dev/null)" != Linux ]; then
        echo "server-check: $(sc_t "Linux servers only" "только для серверов Linux")" >&2
        return 1
    fi
    if [ "${BASH_VERSINFO[0]}" -lt 4 ] || { [ "${BASH_VERSINFO[0]}" -eq 4 ] && [ "${BASH_VERSINFO[1]}" -lt 2 ]; }; then
        echo "server-check: bash ≥ 4.2" >&2
        return 1
    fi
    export LC_ALL=C
    declare -gA SYS=() V=() SF=() SS=() M=() WHY=() AP=() NN=() GS=() GSF=() GA=() X=() DLR=() DLV=() DLB=() DLN=() DLT=() \
        TOOL=() TSRC=() TWHY=() ST=()
    declare -ga SP_ID=() SP_M=() SP_HOST=() SP_PORTS=() SP_PORT=() SP_CAP=() SP_CEN=() SP_CRU=() SP_OEN=() SP_ORU=() \
        SP_CITY=() SP_DOWN=() SP_UP=() SP_PING=() SP_DST=() SP_UST=() SP_DLB=() SP_ULB=() SP_MINRTT=() SP_RDOWN=() \
        SP_RUP=() SP_PAVG=() SP_PMED=() SP_PJIT=() SP_PLOSS=() SP_PSAMP=() SP_PN=() SP_REG=() SP_ALT=() SP_FB=() SC_RUNAS=() \
        SC_SYSBENCH_CMD=() SC_FIO_CMD=()
    SPN=0; SC_HTTP=0
    SC_PROGRESS=0; [ -t 2 ] && SC_PROGRESS=1
    sc_screen_init "$color"
    [ "$SC_JSON" = 1 ] || SC_ON_ROW=sc_log_row          # the text report prints rows as they finish; JSON prints once
    [ "$SC_NONET" = 1 ] || sc_speed_points
    SC_TIMEOUT=0; sc_have timeout && SC_TIMEOUT=1
    SC_FETCH=""; if sc_have curl; then SC_FETCH=curl; elif sc_have wget; then SC_FETCH=wget; fi
    SC_WORK=""; SC_WORK2=""; SC_WORK3=""; SC_DL_PIDS=""
    trap sc_cleanup EXIT
    trap 'sc_cleanup; exit 130' INT
    trap 'sc_cleanup; exit 143' TERM
    trap 'sc_cleanup; exit 129' HUP
    if [ -d /var/tmp ] && [ -w /var/tmp ]; then SC_WORK=$(mktemp -d /var/tmp/server-check.XXXXXX 2>/dev/null); fi
    [ -n "$SC_WORK" ] || SC_WORK=$(mktemp -d "${TMPDIR:-/tmp}/server-check.XXXXXX" 2>/dev/null)
    if [ -z "$SC_WORK" ] || [ ! -d "$SC_WORK" ]; then
        echo "server-check: $(sc_t "cannot create a temporary directory" "не удалось создать временную папку")" >&2
        return 1
    fi
    SC_BIN="$SC_WORK/bin"; SC_HOME="$SC_WORK/home"
    mkdir -m 755 "$SC_BIN" && mkdir -m 700 "$SC_HOME" || return 1
    sc_runas_init
    sc_clock; SC_T0=$SC_NOW
    sc_collect
    if [ "$SC_JSON" = 1 ]; then sc_report_json; else sc_render_text; fi
    return 0
}

if [ "${SC_SOURCE_ONLY:-0}" != 1 ]; then sc_main "$@" </dev/null; exit $?; fi
