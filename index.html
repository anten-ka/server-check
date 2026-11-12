#!/usr/bin/env bash
# server-check.sh — how good is this server: CPU, memory, disk, network and the system, in one file (Kaskad PRO).
#
#   bash server-check.sh [--lang ru|en] [--quick] [--json] [--no-net] [--ascii] [--no-color]
#   curl -fsSL <url>/server-check.sh | bash             options: … | bash -s -- --quick   or   … | SC_QUICK=1 bash
#   wget -qO- <url>/server-check.sh | bash
#
# Read-only for the system: installs nothing, writes only into its own «mktemp -d» (removed on exit, Ctrl+C, kill; the
# memory test without sysbench — one more in /dev/shm, removed right after it),
# never touches services or configs; no telemetry — only GETs to public speed-test points. The IP of the server is never
# printed. Scores 0–1000 are the scale of Kaskad (lib/kaskad/score_abs.py, abs-0.5-k1): the same curves, anchors,
# weights and group means; no total and no verdicts (owner 23.10 / 27.10). sysbench / fio are used when present (the
# exact methods of Kaskad, lib/kaskad/bench/suite.py), otherwise built-in ones (openssl, dd, /dev/shm, getent, bash /dev/tcp) —
# marked «≈». Environment: SC_LANG=ru|en, SC_QUICK=1, SC_JSON=1, SC_NO_NET=1, SC_ASCII=1, NO_COLOR=1; SC_BUILTIN=1 —
# ignore sysbench and fio (to compare the built-in methods with them on one server).
# Everything below is functions; the last line calls them: a download cut short runs nothing.
# Tests: tools/server-check/test.sh (parsers and scores on fixtures; no benchmark).

if [ -z "${BASH_VERSION:-}" ]; then
    echo "server-check: run it with bash:  curl -fsSL URL | bash" >&2
    exit 1
fi

sc_init() {
    SC_VERSION="1.0"
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
    # The latency points: id|region|English|Russian|host (TCP 443: the connection only, nothing sent)
    SC_LAT_POINTS='fra|eu|Frankfurt|Франкфурт|fra-de-ping.vultr.com
ams|eu|Amsterdam|Амстердам|ams-nl-ping.vultr.com
hel|eu|Helsinki|Хельсинки|hel1-speed.hetzner.com
sto|eu|Stockholm|Стокгольм|sto-se-ping.vultr.com
waw|eu|Warsaw|Варшава|waw-pl-ping.vultr.com
yandex|ru|Yandex|Яндекс|mirror.yandex.ru
selectel|ru|Selectel|Selectel|speedtest.selectel.ru
nj|us|New Jersey|Нью-Джерси|nj-us-ping.vultr.com
lax|us|Los Angeles|Лос-Анджелес|lax-ca-us-ping.vultr.com'
    # The download points: id|region|English|Russian|url|bytes of one request ({bytes}; 0 — a file)|size of the file.
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
    local s=$1 c
    c=${s//[^$'\x80'-$'\xbf']/}
    REPLY=$(( ${#s} - ${#c} ))
}

# sc_pad TEXT WIDTH -> REPLY: TEXT padded with spaces on the right to WIDTH characters (Cyrillic counts as one)
sc_pad() {
    local s=$1 n
    sc_len "$s"; n=$(( $2 - REPLY ))
    while [ "$n" -gt 0 ]; do s+=" "; n=$((n - 1)); done
    REPLY=$s
}

# sc_rpad TEXT WIDTH -> REPLY: TEXT aligned to the right
sc_rpad() {
    local s=$1 n p=""
    sc_len "$s"; n=$(( $2 - REPLY ))
    while [ "$n" -gt 0 ]; do p+=" "; n=$((n - 1)); done
    REPLY=$p$s
}

# sc_fnum VALUE [DIGITS] -> REPLY: as Kaskad i18n.fmt_num — ru «33 467,5» (U+202F), en «33,467.5»; no DIGITS — by size
sc_fnum() {
    local v=$1 d=${2:-} s int frac="" sep out="" neg=""
    sc_isnum "$v" || { REPLY="—"; return; }
    if [ -z "$d" ]; then
        d=$(awk -v x="$v" 'BEGIN { print (x >= 100 ? 0 : (x >= 10 ? 1 : 2)) }')
    fi
    printf -v s "%.${d}f" "$v"
    [ "${s:0:1}" = "-" ] && { neg="-"; s=${s:1}; }
    int=${s%%.*}; [ "$s" != "$int" ] && frac=${s#*.}
    if [ "$SC_LANG" = ru ]; then sep=$'\xe2\x80\xaf'; else sep=","; fi      # U+202F, the bytes: LC_ALL=C here
    while [ ${#int} -gt 3 ]; do out="$sep${int: -3}$out"; int=${int:0:${#int}-3}; done
    out="$int$out"
    if [ -n "$frac" ]; then
        if [ "$SC_LANG" = ru ]; then out="$out,$frac"; else out="$out.$frac"; fi
    fi
    REPLY="$neg$out"
}

# sc_plural N ONE FEW MANY -> REPLY (Russian: 1 ядро, 2 ядра, 5 ядер, 21 ядро)
sc_plural() {
    local n=$(( $1 % 100 )) m=$(( $1 % 10 ))
    if [ "$n" -ge 11 ] && [ "$n" -le 14 ]; then REPLY=$4
    elif [ "$m" -eq 1 ]; then REPLY=$2
    elif [ "$m" -ge 2 ] && [ "$m" -le 4 ]; then REPLY=$3
    else REPLY=$4; fi
}

# sc_cores N -> REPLY: «1 core / 2 cores», «1 ядро / 2 ядра / 5 ядер»
sc_cores() {
    if [ "$SC_LANG" = ru ]; then sc_plural "$1" "ядро" "ядра" "ядер"; REPLY="$1 $REPLY"
    elif [ "$1" = 1 ]; then REPLY="1 core"; else REPLY="$1 cores"; fi
}

# sc_bar SCORE WIDTH -> REPLY: a bar of WIDTH cells for 0–1000, eighths of a cell (ASCII: # and .)
sc_bar() {
    local s=${1:-} w=$2 full part i b=""
    if ! sc_isnum "$s"; then
        for ((i = 0; i < w; i++)); do if [ "$SC_ASCII" = 1 ]; then b+=" "; else b+="·"; fi; done
        REPLY=$b; return
    fi
    local e=$(( s * w * 8 / 1000 ))
    full=$(( e / 8 )); part=$(( e % 8 ))
    local parts=("" "▏" "▎" "▍" "▌" "▋" "▊" "▉")
    for ((i = 0; i < full; i++)); do if [ "$SC_ASCII" = 1 ]; then b+="#"; else b+="█"; fi; done
    if [ "$full" -lt "$w" ]; then
        if [ "$part" -gt 0 ] && [ "$SC_ASCII" != 1 ]; then b+=${parts[$part]}; i=$((i + 1)); fi
        for (( ; i < w; i++)); do if [ "$SC_ASCII" = 1 ]; then b+="."; else b+="░"; fi; done
    fi
    REPLY=$b
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
    [ "$SC_PROGRESS" = 1 ] || return 0
    local el
    sc_clock; el=$(( (SC_NOW - SC_T0) / 1000000 ))
    printf '\r\033[K %s %d:%02d  %s' "›" $((el / 60)) $((el % 60)) "$*" >&2
}

sc_prog_clear() { [ "$SC_PROGRESS" = 1 ] && printf '\r\033[K' >&2; return 0; }

sc_say() { [ "$SC_JSON" = 1 ] && return 0; sc_prog_clear; printf '%s\n' "$*"; }

# ── parsers (stdin -> stdout; test.sh runs them on fixtures) ────────────────────────────────────────────────────────

# PRETTY_NAME of /etc/os-release
sc_parse_osrel() {
    awk -F= '$1 == "PRETTY_NAME" { v = substr($0, index($0, "=") + 1); gsub(/^["'\'']|["'\'']$/, "", v); print v; exit }'
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

sc_report_system() {
    [ "$SC_JSON" = 1 ] && return 0
    local line flags="" wg hd tun lbl w=11 v4 v6
    line="${SC_C_DIM}$(printf '%.0s─' $(seq 1 78))${SC_C_0}"
    sc_say ""
    sc_say " ${SC_C_B}$(sc_t "Kaskad server check" "Проверка сервера Kaskad") ${SC_VERSION}${SC_C_0} · $(
        if [ "$SC_MODE" = quick ]; then sc_t "quick" "быстрая"; else sc_t "full" "полная"; fi) · $(
        sc_t "scale" "шкала") Kaskad ${SC_SCALE_VERSION}"
    sc_say " ${SC_C_DIM}$(sc_t "Nothing is installed, temporary files are removed at the end; no telemetry." \
        "Ничего не устанавливается, временные файлы удаляются в конце; без телеметрии.")${SC_C_0}"
    sc_say "$line"
    sc_pad "$(sc_t "System" "Система")" $w; lbl=$REPLY
    local virt=${SYS[virt]}
    case "$virt" in
        none) virt=$(sc_t "no virtualization" "без виртуализации") ;;
        vm) virt=$(sc_t "a virtual machine" "виртуальная машина") ;;
        unknown) virt=$(sc_t "virtualization unknown" "виртуализация не определена") ;;
    esac
    sc_say "  $lbl ${SYS[os]} · $(sc_t "kernel" "ядро") ${SYS[kernel]} · ${SYS[arch]} · $virt"
    case " ${SYS[flags]} " in *" aes "*) flags+=", AES" ;; esac
    case " ${SYS[flags]} " in *" avx2 "*) flags+=", AVX2" ;; esac
    case " ${SYS[flags]} " in *" vaes "*) flags+=", VAES" ;; esac
    sc_cores "${SYS[nproc]}"; local cores=$REPLY
    if [ -n "${SYS[tpc]}" ] && [ "${SYS[tpc]}" -gt 1 ]; then cores+=" (SMT ×${SYS[tpc]})"; fi
    sc_pad "$(sc_t "CPU" "Процессор")" $w; lbl=$REPLY
    sc_say "  $lbl ${SYS[cpu]} · $cores${flags:+ · ${flags#, }}"
    sc_pad "$(sc_t "Memory" "Память")" $w; lbl=$REPLY
    sc_fnum "$(sc_gb "${SYS[mem_kb]}" 2)" 2; local mt=$REPLY
    sc_fnum "$(sc_gb "${SYS[avail_kb]}" 2)" 2; local ma=$REPLY
    sc_fnum "$(sc_gb "${SYS[swap_kb]}" 2)" 2; local sw=$REPLY
    sc_say "  $lbl $mt $(sc_t "GB" "ГБ"), $(sc_t "available" "доступно") $ma $(sc_t "GB" "ГБ") · $(sc_t "swap" "подкачка") $sw $(sc_t "GB" "ГБ")"
    sc_pad "$(sc_t "Disk" "Диск")" $w; lbl=$REPLY
    if [ -n "${SYS[root_kb]}" ]; then
        sc_fnum "$(sc_gb "${SYS[root_kb]}" 1)" 1; local dt=$REPLY
        sc_fnum "$(sc_gb "${SYS[root_free_kb]}" 1)" 1; local df=$REPLY
        sc_say "  $lbl / ${SYS[root_fs]:-?} · $dt $(sc_t "GB" "ГБ"), $(sc_t "free" "свободно") $df $(sc_t "GB" "ГБ")"
    else
        sc_say "  $lbl —"
    fi
    sc_pad "$(sc_t "Network" "Сеть")" $w; lbl=$REPLY
    v4=$(sc_yes "${SYS[ipv4]}"); v6=$(sc_yes "${SYS[ipv6]}")
    [ -n "${SYS[ipv4_ok]}" ] && v4+=" ($(sc_works "${SYS[ipv4_ok]}"))"
    [ -n "${SYS[ipv6_ok]}" ] && v6+=" ($(sc_works "${SYS[ipv6_ok]}"))"
    local load=""; [ -n "${SYS[load]}" ] && { sc_fnum "${SYS[load]}" 2; load=" · $(sc_t "load" "загрузка") $REPLY"; }
    sc_say "  $lbl IPv4 $v4 · IPv6 $v6 · TCP ${SYS[tcp_cc]:-?}/${SYS[qdisc]:-?}$load"
    case "${SYS[wg]}" in
        loaded) wg=$(sc_t "loaded" "загружен") ;; builtin) wg=$(sc_t "in kernel" "в ядре") ;;
        module) wg=$(sc_t "kernel module" "модуль ядра") ;; *) wg=$(sc_t "no" "нет") ;;
    esac
    case "${SYS[headers]}" in
        yes) hd=$(sc_t "installed" "установлены") ;; apt) hd=$(sc_t "in apt" "есть в apt") ;;
        *) hd=$(sc_t "not found" "не найдены") ;;
    esac
    tun=$(sc_yes "$([ "${SYS[tun]}" = yes ] && echo 1 || echo 0)")
    sc_pad "VPN" $w; lbl=$REPLY
    sc_say "  $lbl WireGuard: $wg · $(sc_t "kernel headers" "заголовки ядра"): $hd · /dev/net/tun: $tun"
    if [ "${SYS[container]}" = 1 ]; then
        sc_pad "" $w
        sc_say "  $REPLY $(sc_t "a container: no kernel modules of its own, amneziawg-go only" \
            "контейнер: свой модуль ядра не загрузить, только amneziawg-go")"
    fi
    sc_say "$line"
}

sc_yes() { if [ "${1:-0}" = 1 ]; then sc_t "yes" "да"; else sc_t "no" "нет"; fi; }
sc_works() { if [ "${1:-0}" = 1 ]; then sc_t "works" "работает"; else sc_t "no access" "нет доступа"; fi; }

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
            v=$(sc_to 22 sysbench cpu --cpu-max-prime=10000 --threads=1 --time=2 run 2>/dev/null | sc_parse_sysbench_cpu)
            sc_isnum "$v" && vals+=("$v")
        done
        REPLY=""; [ ${#vals[@]} -gt 0 ] && sc_median "${vals[@]}"
        sc_set cpu_1t "$REPLY" sysbench
        sc_prog "$(sc_t "CPU · sysbench, all cores" "процессор · sysbench, все ядра") ($n) ${secs} s"
        v=$(sc_to $((secs + 20)) sysbench cpu --cpu-max-prime=10000 --threads="$n" --time="$secs" run 2>/dev/null \
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
        v=$(sc_to $((t + 20)) sysbench memory --memory-block-size=1M --memory-total-size=1T --threads=1 --time="$t" run \
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
    REPLY=$(sc_to "$to" fio --name=kbench --filename="$f" --size="$size" --direct=1 --ioengine=libaio \
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
    local n a i out rtts meds=() jits=() jn=0 losses=() sent=0 via=icmp
    if [ "$SC_MODE" = quick ]; then n=10; else n=20; fi
    sc_prog "$(sc_t "network · latency to 1.1.1.1, 8.8.8.8, 77.88.8.8" "сеть · отклик до 1.1.1.1, 8.8.8.8, 77.88.8.8")"
    i=0
    if sc_have ping; then
        for a in $SC_ANCHORS; do
            ping -n -c "$n" -i 0.2 -W 2 -w $((n / 5 + 4)) "$a" >"$SC_WORK/ping.$i" 2>&1 &
            i=$((i + 1))
        done
        wait
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
        for a in $SC_ANCHORS; do sc_tcp_rtt "$a" 53 "$((n / 2))" >"$SC_WORK/tcp53.$i" & i=$((i + 1)); done
        wait
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

# the latency by regions: TCP handshakes to port 443 of public points, the median (informational, no score)
sc_net_regions() {
    local samples id reg en ru host n i=0
    if [ "$SC_MODE" = quick ]; then n=3; else n=5; fi
    sc_prog "$(sc_t "network · latency by region" "сеть · отклик по регионам")"
    while IFS='|' read -r id reg en ru host; do
        [ -n "$id" ] || continue
        sc_tcp_rtt "$host" 443 "$n" >"$SC_WORK/lat.$id" &
    done <<<"$SC_LAT_POINTS"
    wait
    while IFS='|' read -r id reg en ru host; do
        [ -n "$id" ] || continue
        mapfile -t samples <"$SC_WORK/lat.$id"
        LAT[$id]=""
        if [ ${#samples[@]} -gt 0 ]; then sc_median "${samples[@]}"; LAT[$id]=$REPLY; fi
    done <<<"$SC_LAT_POINTS"
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
        wait "${pids[@]}" 2>/dev/null
        SC_DL_PIDS=""; REPLY=""
    else
        wait "${pids[@]}" 2>/dev/null
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
    sc_set net_up "" "" get_only
}

sc_net() {
    if [ "$SC_NONET" = 1 ]; then
        for v in rtt jitter loss dns net_down net_up; do sc_set "$v" "" "" skipped; done
    else
        sc_net_rtt
        sc_net_dns
        sc_net_regions
        if [ -n "$SC_FETCH" ]; then sc_net_down
        else sc_set net_down "" "" no_fetch; sc_set net_up "" "" no_fetch; fi
    fi
    sc_group_score net_bw
    sc_group_score net_q
}

# ── the report ──────────────────────────────────────────────────────────────────────────────────────────────────────

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
        get_only) sc_t "the check only downloads" "проверка только скачивает" ;;
        no_fetch) sc_t "no curl or wget" "нет curl и wget" ;;
        mem_implausible|mem_unstable) sc_t "indicative, no score" "ориентир, без балла" ;;
        no_shm) sc_t "no tmpfs /dev/shm" "нет tmpfs /dev/shm" ;;
        mem_room) sc_t "no room in RAM" "мало свободной памяти" ;;
        *) sc_t "not measured" "не измерено" ;;
    esac
}

sc_unit() {
    case "$1" in
        evs) sc_t "ev/s" "соб./с" ;; mbs) sc_t "MB/s" "МБ/с" ;; mibs) sc_t "MiB/s" "МиБ/с" ;; gb) sc_t "GB" "ГБ" ;;
        iops) printf 'IOPS' ;; ms) sc_t "ms" "мс" ;; pct) printf '%%' ;; mbit) sc_t "Mbit/s" "Мбит/с" ;; *) printf '%s' "$1" ;;
    esac
}

# sc_row LABEL VALUE UNIT SCORE NOTE [APPROX] [AT_LEAST] — a line of numbers: the label, the value, its unit, the score;
# the note right after what the line has (no value — after «—», no score — after the unit)
sc_row() {
    local label=$1 v=$2 unit=$3 s=$4 note=$5 ap=${6:-0} lb=${7:-0} val out
    if sc_isnum "$v"; then sc_fnum "$v"; val=$REPLY; [ "$lb" = 1 ] && val="≥ $val"; [ "$ap" = 1 ] && val="≈ $val"
    else val="—"; unit=""; s=""; fi
    sc_pad "$label" 36; label=$REPLY
    sc_rpad "$val" 11; out="   $label$REPLY"
    if [ -n "$s" ]; then
        sc_pad "$unit" 7; out+=" $REPLY"
        sc_rpad "$s" 4; out+=$REPLY
    elif [ -n "$unit" ]; then
        out+=" $unit"
    fi
    [ -n "$note" ] && out+="  ${SC_C_DIM}$note${SC_C_0}"
    sc_say "$out"
}

# sc_test_row TEST LABEL UNIT
sc_test_row() {
    local t=$1 note=""
    [ -n "${WHY[$t]:-}" ] && note=$(sc_why_text "${WHY[$t]}")
    local lb=0; [ "$t" = net_down ] && lb=${X[net_down_lb]:-0}
    sc_row "$2" "${V[$t]:-}" "$3" "${SS[$t]:-}" "$note" "${AP[$t]:-0}" "$lb"
}

# sc_barline LABEL GROUP — «label  bar  score», the graphics before the number
sc_barline() {
    local lbl
    sc_pad "$1" 22; lbl=$REPLY
    sc_bar "${GS[$2]:-}" 30
    local mark="  "; [ "${GA[$2]:-0}" = 1 ] && [ -n "${GS[$2]:-}" ] && mark="≈ "
    sc_say " $lbl${SC_C_BAR}$REPLY${SC_C_0}${mark}${SC_C_B}${GS[$2]:-—}${SC_C_0}"
}

# sc_head NAME GROUP… — the head of a section: one group — its bar on the same line; more — a bar a line
sc_head() {
    local name=$1 g; shift
    sc_say ""
    if [ $# -eq 1 ]; then
        sc_barline "${SC_C_B}$name${SC_C_0}" "$1"
        return 0
    fi
    sc_say " ${SC_C_B}$name${SC_C_0}"
    for g in "$@"; do sc_group_label "$g"; sc_barline "  $REPLY" "$g"; done
}

sc_group_label() {
    case "$1" in
        cpu) REPLY=$(sc_t "CPU" "Процессор") ;; mem) REPLY=$(sc_t "Memory" "Память") ;; disk) REPLY=$(sc_t "Disk" "Диск") ;;
        net_bw) REPLY=$(sc_t "Channel" "Канал") ;; net_q) REPLY=$(sc_t "Latency and DNS" "Отклик и DNS") ;;
    esac
}

# sc_wrap INDENT ITEM… — items joined with « · », wrapped at 78 characters, every line after the first indented
sc_wrap() {
    local pre=$1 line="" it width n; shift
    sc_len "$pre"; width=$REPLY
    for it in "$@"; do
        sc_len "$line · $it"; n=$REPLY
        if [ -n "$line" ] && [ $((width + n)) -gt 78 ]; then
            sc_say "$pre$line"; line=$it
            printf -v pre '%*s' "$width" ""
        else
            line+="${line:+ · }$it"
        fi
    done
    [ -n "$line" ] && sc_say "$pre$line"
    return 0
}

sc_report_cpu() {
    [ "$SC_JSON" = 1 ] && return 0
    local n=${SYS[nproc]} note
    sc_head "$(sc_t "CPU" "ПРОЦЕССОР")" cpu
    sc_test_row cpu_1t "$(sc_t "1 core · sysbench" "1 ядро · sysbench")" "$(sc_unit evs)"
    sc_test_row cpu_mt "$(sc_t "All cores ($n) · sysbench" "Все ядра ($n) · sysbench")" "$(sc_unit evs)"
    sc_test_row chacha_1t "$(sc_t "1 core · ChaCha20-Poly1305" "1 ядро · ChaCha20-Poly1305")" "$(sc_unit mbs)"
    sc_test_row aes_1t "$(sc_t "1 core · AES-128-GCM" "1 ядро · AES-128-GCM")" "$(sc_unit mbs)"
    note=""; [ -n "${X[chacha_scale]}" ] && { sc_fnum "${X[chacha_scale]}" 2; note="×${REPLY}"; }
    [ -n "${WHY[chacha_mt]}" ] && note=$(sc_why_text "${WHY[chacha_mt]}")
    [ -n "${V[cpu_mt]}" ] && note="${note:+$note · }$(sc_t "not scored: sysbench is here" "не в балле: есть sysbench")"
    sc_row "$(sc_t "All cores ($n) · ChaCha20-Poly1305" "Все ядра ($n) · ChaCha20-Poly1305")" "${V[chacha_mt]}" \
        "$(sc_unit mbs)" "${SS[chacha_mt]}" "$note" 1
    sc_row "$(sc_t "Steal during the tests" "Кража (steal) во время тестов")" "${X[steal]}" "$(sc_unit pct)" "" ""
}

sc_report_mem() {
    [ "$SC_JSON" = 1 ] && return 0
    sc_head "$(sc_t "MEMORY" "ПАМЯТЬ")" mem
    local why
    sc_test_row mem_gb "$(sc_t "RAM" "Объём")" "$(sc_unit gb)"
    if [ "${M[mem_bw]:-}" = sysbench ] || { [ -z "${M[mem_bw]:-}" ] && [ "$SC_SYSBENCH" = 1 ]; }; then
        sc_test_row mem_bw "$(sc_t "Write, 1 thread · sysbench" "Запись, 1 поток · sysbench")" "$(sc_unit mibs)"
    else
        sc_test_row mem_bw "$(sc_t "Read, 1 thread · dd from /dev/shm" "Чтение, 1 поток · dd из /dev/shm")" "$(sc_unit mibs)"
    fi
    sc_row "$(sc_t "Swap" "Подкачка")" "${X[swap_gb]}" "$(sc_unit gb)" "" ""
    if [ -z "${SS[mem_bw]:-}" ] && [ -n "${SS[mem_gb]:-}" ]; then
        case "${WHY[mem_bw]:-}" in
            mem_implausible) why=$(sc_t "over 60 GB/s per thread is cache, not RAM" "выше 60 ГБ/с на поток — это кэш, не память") ;;
            mem_unstable) why=$(sc_t "the passes differ more than 2×" "проходы расходятся больше чем вдвое") ;;
            no_shm|mem_room) why="$(sc_t "speed not measured" "скорость не измерена") ($(sc_why_text "${WHY[mem_bw]}"))" ;;
            *) why=$(sc_t "speed not measured" "скорость не измерена") ;;
        esac
        sc_say "   ${SC_C_DIM}$(sc_t "Group score — RAM size only:" "Балл группы — только объём:") $why${SC_C_0}"
    fi
}

sc_report_disk() {
    [ "$SC_JSON" = 1 ] && return 0
    local m=${M[disk_fsync]:-${M[disk_rr]:-}} f note=""
    sc_head "$(sc_t "DISK" "ДИСК")" disk
    if [ -n "${X[disk_file_mb]}" ]; then
        note="$(sc_t "test file" "тестовый файл") ${X[disk_file_mb]} $(sc_t "MiB" "МиБ")${X[disk_dir_fs]:+ · ${X[disk_dir_fs]}}"
        sc_say "   ${SC_C_DIM}$note${m:+ · $m}${SC_C_0}"
    fi
    sc_test_row disk_rr "$(sc_t "Random read 4K, queue 32" "Случайное чтение 4K, очередь 32")" IOPS
    sc_test_row disk_rw "$(sc_t "Random write 4K, queue 32" "Случайная запись 4K, очередь 32")" IOPS
    sc_test_row disk_fsync "$(sc_t "Write 4K + flush to disk" "Запись 4K + сброс на диск")" "$(sc_unit ms)"
    local fw="" fr=""
    if [ "${X[disk_direct]:-1}" = 0 ]; then
        fw=$(sc_t "no direct I/O" "без прямого ввода-вывода"); fr=$(sc_t "needs direct I/O" "нужен прямой ввод-вывод")
    elif [ "$SC_MODE" = quick ] && [ "${M[disk_rr]:-}" = fio ]; then
        fw=$(sc_t "in a full check" "в полной проверке"); fr=$fw
    fi
    [ -n "${X[disk_seq_w]}" ] && [ "${X[disk_direct]:-1}" != 0 ] && fw=""
    [ -n "${X[disk_seq_r]}" ] && fr=""
    sc_row "$(sc_t "Sequential write 1M" "Последовательная запись 1M")" "${X[disk_seq_w]}" "$(sc_unit mbs)" "" "$fw"
    sc_row "$(sc_t "Sequential read 1M" "Последовательное чтение 1M")" "${X[disk_seq_r]}" "$(sc_unit mbs)" "" "$fr"
}

sc_report_net() {
    [ "$SC_JSON" = 1 ] && return 0
    local id reg en ru host r v want items name
    sc_head "$(sc_t "NETWORK" "СЕТЬ")" net_bw net_q
    sc_test_row net_down "$(sc_t "Download, best point, 4 streams" "Скачивание, лучшая точка, 4 потока")" "$(sc_unit mbit)"
    sc_test_row net_up "$(sc_t "Upload" "Отдача")" "$(sc_unit mbit)"
    local via=""; [ "${M[rtt]:-}" = tcp ] && via=" · TCP"
    sc_test_row rtt "$(sc_t "Latency 1.1.1.1, 8.8.8.8, 77.88.8.8$via" "Отклик 1.1.1.1, 8.8.8.8, 77.88.8.8$via")" "$(sc_unit ms)"
    sc_test_row jitter "$(sc_t "Jitter" "Дрожание")" "$(sc_unit ms)"
    sc_test_row loss "$(sc_t "Packet loss" "Потери пакетов")" "$(sc_unit pct)"
    sc_test_row dns "$(sc_t "DNS, system resolver" "DNS, системный резолвер")" "$(sc_unit ms)"
    [ "$SC_NONET" = 1 ] && return 0
    sc_say "   ${SC_C_DIM}$(sc_t "Download by point, Mbit/s" "Скачивание по точкам, Мбит/с")${SC_C_0}"
    for want in any eu ru us; do
        [ -n "${DLR[$want]+x}" ] || continue
        sc_region "$want"; sc_pad "$REPLY" 11; r=$REPLY
        if [ -n "${DLV[$want]:-}" ]; then
            sc_fnum "${DLV[$want]}" 0; v=$REPLY; [ "${DLB[$want]:-0}" = 1 ] && v="≥ $v"
            sc_pad "${DLN[$want]:-}" 26; name=$REPLY
            sc_rpad "$v" 9
            sc_say "     $r$name$REPLY"
        else
            sc_pad "" 26; name=$REPLY
            sc_rpad "—" 9
            sc_say "     $r$name$REPLY  ${SC_C_DIM}$(sc_t "no answer" "нет ответа")${SC_C_0}"
        fi
    done
    sc_say "   ${SC_C_DIM}$(sc_t "Latency by region (TCP handshake), ms" "Отклик по регионам (TCP-рукопожатие), мс")${SC_C_0}"
    for want in eu ru us; do
        items=()
        while IFS='|' read -r id reg en ru host; do
            [ "$reg" = "$want" ] || continue
            if [ "$SC_LANG" = ru ]; then r=$ru; else r=$en; fi
            if [ -n "${LAT[$id]:-}" ]; then sc_fnum "${LAT[$id]}"; v=$REPLY; else v="—"; fi
            items+=("$r $v")
        done <<<"$SC_LAT_POINTS"
        sc_region "$want"; sc_pad "$REPLY" 11
        sc_wrap "     $REPLY" "${items[@]}"
    done
}

sc_region() {
    case "$1" in
        any) REPLY=$(sc_t "Nearest" "Ближайшая") ;; eu) REPLY=$(sc_t "Europe" "Европа") ;;
        ru) REPLY=$(sc_t "Russia" "Россия") ;; us) REPLY=$(sc_t "USA" "США") ;;
    esac
}

sc_report_summary() {
    [ "$SC_JSON" = 1 ] && return 0
    local g el line took
    line="${SC_C_DIM}$(printf '%.0s─' $(seq 1 78))${SC_C_0}"
    sc_say ""
    sc_say "$line"
    sc_say " ${SC_C_B}$(sc_t "SCORES BY GROUP" "БАЛЛЫ ПО ГРУППАМ")${SC_C_0}"
    for g in cpu mem disk net_bw net_q; do sc_group_label "$g"; sc_barline "  $REPLY" "$g"; done
    sc_say " ${SC_C_DIM}$(sc_t "Kaskad scale 0–1000." "Шкала Kaskad 0–1000.")${SC_C_0}"
    sc_say " ${SC_C_DIM}$(sc_t "≈ — a built-in method or a derived scale, units as in Kaskad (README)." \
        "≈ — встроенный способ или выведенная шкала, единицы как в Kaskad (README).")${SC_C_0}"
    sc_say " ${SC_C_DIM}$(sc_t "No total: in Kaskad it also counts the stability over days." \
        "Общего итога нет: в Kaskad он учитывает и стабильность за несколько суток.")${SC_C_0}"
    sc_clock; el=$(( (SC_NOW - SC_T0) / 1000000 ))
    if [ "$el" -ge 60 ]; then took="$((el / 60)) $(sc_t "min" "мин") $((el % 60)) $(sc_t "s" "с")"
    else took="$el $(sc_t "s" "с")"; fi
    sc_say " $(sc_t "Took" "Заняло") $took · $(sc_t "nothing installed, temporary files removed" \
        "ничего не установлено, временные файлы удалены")"
    sc_say "$line"
}

sc_report_json() {
    local t g s first=1 el id reg en ru host url req size want tr t1
    sc_clock; el=$(( (SC_NOW - SC_T0) / 1000000 ))
    s="{\"tool\":\"server-check\",\"version\":\"$SC_VERSION\",\"scale\":\"kaskad $SC_SCALE_VERSION\",\"mode\":\"$SC_MODE\""
    s+=",\"lang\":\"$SC_LANG\",\"elapsed_s\":$el,\"system\":{"
    first=1
    for t in os kernel arch virt cpu root_fs tcp_cc qdisc wg headers tun; do
        sc_json_str "${SYS[$t]:-}"; s+="$([ $first = 1 ] || echo ,)\"$t\":$REPLY"; first=0
    done
    sc_json_num "${SYS[nproc]}"; s+=",\"nproc\":$REPLY"
    sc_json_num "${SYS[tpc]}"; s+=",\"threads_per_core\":$REPLY"
    s+=",\"cpu_flags\":["; first=1
    for t in ${SYS[flags]}; do s+="$([ $first = 1 ] || echo ,)\"$t\""; first=0; done
    s+="]"
    for t in mem_kb avail_kb swap_kb root_kb root_free_kb load; do sc_json_num "${SYS[$t]:-}"; s+=",\"$t\":$REPLY"; done
    for t in ipv4 ipv6 container; do if [ "${SYS[$t]}" = 1 ]; then s+=",\"$t\":true"; else s+=",\"$t\":false"; fi; done
    for t in ipv4_ok ipv6_ok; do
        case "${SYS[$t]}" in 1) s+=",\"$t\":true" ;; 0) s+=",\"$t\":false" ;; *) s+=",\"$t\":null" ;; esac
    done
    s+="},\"groups\":{"; first=1
    for g in cpu mem disk net_bw net_q; do sc_json_num "${GS[$g]:-}"; s+="$([ $first = 1 ] || echo ,)\"$g\":$REPLY"; first=0; done
    s+="},\"groups_approx\":{"; first=1
    for g in cpu mem disk net_bw net_q; do
        s+="$([ $first = 1 ] || echo ,)\"$g\":$([ "${GA[$g]:-0}" = 1 ] && echo true || echo false)"; first=0
    done
    s+="},\"tests\":{"; first=1
    for t in cpu_1t cpu_mt chacha_1t aes_1t chacha_mt mem_gb mem_bw disk_rr disk_rw disk_fsync net_down net_up rtt jitter loss dns; do
        s+="$([ $first = 1 ] || echo ,)\"$t\":{"; first=0
        sc_json_num "${V[$t]:-}"; s+="\"value\":$REPLY"
        sc_json_num "${SS[$t]:-}"; s+=",\"score\":$REPLY"
        sc_json_str "${M[$t]:-}"; s+=",\"method\":$REPLY"
        if [ -n "${WHY[$t]:-}" ]; then s+=",\"why\":\"${WHY[$t]}\""; else s+=",\"why\":null"; fi
        if [ "${AP[$t]:-0}" = 1 ]; then s+=",\"approx\":true"; else s+=",\"approx\":false"; fi
        sc_json_num "${NN[$t]:-}"; s+=",\"n\":$REPLY}"
    done
    s+="},\"extra\":{"; first=1
    for t in chacha_scale steal swap_gb disk_seq_r disk_seq_w disk_file_mb; do
        sc_json_num "${X[$t]:-}"; s+="$([ $first = 1 ] || echo ,)\"$t\":$REPLY"; first=0
    done
    if [ "${X[net_down_lb]:-0}" = 1 ]; then s+=",\"net_down_at_least\":true"; else s+=",\"net_down_at_least\":false"; fi
    s+="},\"download\":["; first=1
    for want in any eu ru us; do
        [ -n "${DLR[$want]+x}" ] || continue
        sc_json_num "${DLV[$want]:-}"
        s+="$([ $first = 1 ] || echo ,){\"region\":\"$want\",\"point\":\"${DLR[$want]:-}\",\"mbps\":$REPLY"
        if [ "${DLB[$want]:-0}" = 1 ]; then s+=",\"at_least\":true"; else s+=",\"at_least\":false"; fi
        s+=",\"tries\":["; t1=1
        for tr in ${DLT[$want]:-}; do          # «id=Mbit/s» or «id=-»; ids are ours ([a-z0-9-])
            sc_json_num "${tr#*=}"; s+="$([ $t1 = 1 ] || echo ,){\"point\":\"${tr%%=*}\",\"mbps\":$REPLY}"; t1=0
        done
        s+="]}"
        first=0
    done
    s+="],\"latency\":["; first=1
    while IFS='|' read -r id reg en ru host; do
        [ -n "$id" ] && [ -n "${LAT[$id]+x}" ] || continue
        sc_json_num "${LAT[$id]:-}"
        s+="$([ $first = 1 ] || echo ,){\"region\":\"$reg\",\"point\":\"$id\",\"ms\":$REPLY}"; first=0
    done <<<"$SC_LAT_POINTS"
    s+="]}"
    printf '%s\n' "$s"
}

# ── the run ─────────────────────────────────────────────────────────────────────────────────────────────────────────

sc_usage() {
    if [ "$SC_LANG" = ru ]; then cat <<'EOF'
Проверка сервера Kaskad: процессор, память, диск, сеть и система — баллы 0–1000 по шкале Kaskad.
  bash server-check.sh [--lang ru|en] [--quick] [--json] [--no-net] [--ascii] [--no-color]
  curl -fsSL URL | bash        с параметрами: curl -fsSL URL | bash -s -- --quick
  --quick   быстрая проверка (≈ 1 мин; полная ≈ 2–3 мин)
  --json    ответ в JSON для программ
  --no-net  без замеров сети
  Переменные: SC_LANG=ru|en, SC_QUICK=1, SC_JSON=1, SC_NO_NET=1, SC_ASCII=1, NO_COLOR=1.
Ничего не устанавливает; пишет только во временную папку и удаляет её в конце. Без телеметрии.
EOF
    else cat <<'EOF'
Kaskad server check: CPU, memory, disk, network and the system — scores 0–1000 on the Kaskad scale.
  bash server-check.sh [--lang ru|en] [--quick] [--json] [--no-net] [--ascii] [--no-color]
  curl -fsSL URL | bash        with options: curl -fsSL URL | bash -s -- --quick
  --quick   a quick check (≈ 1 min; full ≈ 2–3 min)
  --json    machine-readable JSON
  --no-net  no network tests
  Environment: SC_LANG=ru|en, SC_QUICK=1, SC_JSON=1, SC_NO_NET=1, SC_ASCII=1, NO_COLOR=1.
Installs nothing; writes only into its own temporary directory and removes it at the end. No telemetry.
EOF
    fi
}

sc_lang_auto() {
    local v
    for v in "${LC_ALL:-}" "${LC_MESSAGES:-}" "${LANG:-}"; do
        [ -n "$v" ] || continue
        case "$v" in ru*|*_RU*) REPLY=ru ;; *) REPLY=en ;; esac
        return 0
    done
    REPLY=en
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
    # shellcheck disable=SC2046,SC2086
    sc_kill_own TERM ${SC_DL_PIDS:-} $(jobs -p 2>/dev/null)
    [ "${SC_PROGRESS:-0}" = 1 ] && printf '\r\033[K' >&2
    for p in "${SC_WORK:-}" "${SC_WORK2:-}" "${SC_WORK3:-}"; do
        case "$p" in */server-check.?*|*/.server-check.?*) [ -d "$p" ] && rm -rf -- "$p" ;; esac
    done
    SC_WORK=""; SC_WORK2=""; SC_WORK3=""
}

sc_main() {
    sc_init
    sc_lang_auto; SC_LANG=${SC_LANG:-$REPLY}
    SC_MODE=full; [ "${SC_QUICK:-0}" = 1 ] && SC_MODE=quick
    SC_JSON=${SC_JSON:-0}; SC_NONET=${SC_NO_NET:-0}; SC_ASCII=${SC_ASCII:-0}
    local color=1; [ -n "${NO_COLOR:-}" ] && color=0
    while [ $# -gt 0 ]; do
        case "$1" in
            --lang) SC_LANG=${2:-}; shift ;;
            --lang=*) SC_LANG=${1#*=} ;;
            --quick) SC_MODE=quick ;;
            --full) SC_MODE=full ;;
            --json) SC_JSON=1 ;;
            --no-net) SC_NONET=1 ;;
            --ascii) SC_ASCII=1 ;;
            --no-color) color=0 ;;
            -h|--help) case "$SC_LANG" in ru|en) ;; *) SC_LANG=en ;; esac; sc_usage; return 0 ;;
            --version) echo "server-check $SC_VERSION (Kaskad scale $SC_SCALE_VERSION)"; return 0 ;;
            *) echo "server-check: unknown option «$1» (--help)" >&2; return 2 ;;
        esac
        shift
    done
    case "$SC_LANG" in ru|en) ;; *) echo "server-check: --lang ru|en" >&2; return 2 ;; esac
    if [ "$(uname -s 2>/dev/null)" != Linux ]; then
        echo "server-check: $(sc_t "Linux servers only" "только для серверов Linux")" >&2
        return 1
    fi
    if [ "${BASH_VERSINFO[0]}" -lt 4 ] || { [ "${BASH_VERSINFO[0]}" -eq 4 ] && [ "${BASH_VERSINFO[1]}" -lt 2 ]; }; then
        echo "server-check: bash ≥ 4.2" >&2
        return 1
    fi
    export LC_ALL=C
    declare -gA SYS=() V=() SF=() SS=() M=() WHY=() AP=() NN=() GS=() GSF=() GA=() X=() LAT=() DLR=() DLV=() DLB=() DLN=() DLT=()
    SC_PROGRESS=0; [ -t 2 ] && SC_PROGRESS=1
    SC_C_B=""; SC_C_DIM=""; SC_C_BAR=""; SC_C_0=""
    if [ "$color" = 1 ] && [ -t 1 ] && [ "${TERM:-dumb}" != dumb ] && [ "$SC_JSON" != 1 ]; then
        SC_C_B=$'\033[1m'; SC_C_DIM=$'\033[2m'; SC_C_BAR=$'\033[36m'; SC_C_0=$'\033[0m'
    fi
    SC_TIMEOUT=0; sc_have timeout && SC_TIMEOUT=1
    SC_FETCH=""; if sc_have curl; then SC_FETCH=curl; elif sc_have wget; then SC_FETCH=wget; fi
    SC_SYSBENCH=0; SC_FIO=0
    if [ "${SC_BUILTIN:-0}" != 1 ]; then
        sc_have sysbench && SC_SYSBENCH=1
        sc_have fio && SC_FIO=1
    fi
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
    sc_clock; SC_T0=$SC_NOW
    sc_sysinfo
    sc_report_system
    sc_cpu;  sc_report_cpu
    sc_mem;  sc_report_mem
    sc_disk; sc_report_disk
    sc_net;  sc_report_net
    sc_prog_clear
    if [ "$SC_JSON" = 1 ]; then sc_report_json; else sc_report_summary; fi
    return 0
}

if [ "${SC_SOURCE_ONLY:-0}" != 1 ]; then sc_main "$@" </dev/null; exit $?; fi
