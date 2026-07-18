---
title: Home
---

**`tq` — query TSV like a table, not a text file.** tq is the query language of the [tsvsheet](https://github.com/tsvsheet/tsvsheet) ecosystem: a `|`-separated pipeline of relational verbs over a TSV or tsvt table, with every embedded expression written in the tsvsheet formula language. The `tq` binary is a thin frontend over the [go-tq](https://tsvsheet.github.io/docs.go-tq/) engine — TSV in on stdin or a file, TSV out on stdout — sitting between `cut` (columns, no predicates) and `jq` (predicates, no tables).

```sh
tq 'where [stars] > 1000 | derive ratio = round([stars] / [forks], 2) | select name, stars, ratio | sort -stars | limit 10' repos.tsv
```

## Usage

```text
tq '<query>' [file]
```

The file may be omitted or `-` to read stdin, so `tq` composes in ordinary pipes: `curl … | tq 'select name, stars' | tq 'sort -stars'`.

| Flag | Effect |
| --- | --- |
| `--no-header` | Treat every row as data; column references become positional (`[N]`) only. |
| `--strict` | Abort on the first expression error value (default: error values are data; `where` drops such rows). |
| `--raw` | Skip the compute-first pass — query raw cell text even when the input holds `=formula` cells. |
| `--max-cells N` | Bound the admitted input (rows × columns); exceeded → exit 1. |
| `--at RFC3339` | Fix the clock volatile functions (`today()`, `now()`) read — reproducible runs. |
| `--log-level`, `--log-format` | Standard tsvsheet logging controls. |

Exit codes: **0** success · **2** query syntax error, with line and column · **1** any other error — invalid usage or a program failure against the data (unknown column, raw A1 reference, headerless `rename`, strict abort, input ceiling, unreadable file) — with a one-line diagnostic on stderr and nothing on stdout.

`tq completion bash|zsh|fish` emits shell completion.

## The language, in one paragraph

Verbs: `select` · `drop` · `where` · `derive` · `rename` · `sort` (stable, typed, `-` descending) · `distinct` · `limit` · `offset` · `group keys { name = aggregate, … }`. The first input row is the header; columns are `[name]` or `[N]` in expressions and bare identifiers in verb positions. Expressions are exactly tsvsheet's formula language — same functions, coercions, and error values — except `|` always means "next stage". When the input is a `.tsvt`, the sheet is computed first and the query sees values. The normative specification lives in the [tq language repo](https://github.com/tsvsheet/tq).

## Install

```sh
go install github.com/tsvsheet/tq.go/cmd/tq@latest
```

Homebrew/Scoop packaging follows the tsvsheet formulas once released.
