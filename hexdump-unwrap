#!/usr/bin/env bash
# unwrap.sh — peel stacked gzip/bzip2/tar layers off a hexdump until plaintext.
# Usage:  ./unwrap.sh [-t NAME] [-c] <hexdump-file>
#   -t NAME   basename for the temp rolling file (default: current)
#   -c        clean up temp/unrolled files at the end (best-effort)
set -euo pipefail

work="current"     # -t overrides this
cleanup=0          # -c sets this to 1

while getopts ":t:c" opt; do
    case "$opt" in
        t)  work="$OPTARG" ;;
        c)  cleanup=1 ;;
        :)  echo "option -$OPTARG needs an argument" >&2; exit 2 ;;
        \?) echo "unknown option -$OPTARG" >&2; exit 2 ;;
    esac
done
shift $((OPTIND - 1))                            # drop parsed flags; $1 is now the file

src="${1:?usage: $0 [-t NAME] [-c] <hexdump-file>}"

# Step 0: reverse the hexdump into raw bytes.
xxd -r "$src" > "$work"

# Step 1..N: peel layers, driven by `file`, until we hit text.
while true; do
    t=$(file -b "$work")                         # -b = type string only
    case "$t" in
        *gzip*)         gzip  -dc "$work" > "$work.next" ;;  # -d decompress, -c stdout
        *bzip2*)        bzip2 -dc "$work" > "$work.next" ;;
        *tar*)          tar   xOf "$work" > "$work.next" ;;  # -O: member -> stdout
        *ASCII*|*text*) echo "=== plaintext reached ==="; cat "$work"; break ;;
        *) echo "stop — unhandled type: $t" >&2; exit 1 ;;
    esac
    mv "$work.next" "$work"                       # roll output back into $work
done

# Optional: remove the unrolled temp material. Best-effort — ignore if read-only.
if (( cleanup )); then
    rm -f "$work" "$work.next" 2>/dev/null || :
fi

