# `depscan`

Finds `*_grpc.pb.go` client methods, lists direct and transitive consumers using `gopls`, and shows which functions reference them.

## Install

- Requires `python3`, `go`, and `gopls` in PATH.

## Usage

```
python3 depscan [PROJECT_ROOT]
```

Options:
- `--segments, -n`  Number of leading path segments to show for transitive consumers. Default: 1
- `--format table|tsv`  Output format for the summary. Default: table
- `--expand-top-level-pkg DIR [DIR ...]`  Replace functions located under these top-level dirs with their callers in the output. Default: pkg
- `--include-tests`  Include Go test files (`*_test.go`) in callers and function listings. Default: excluded
- `--no-progress`  Disable the TTY-only Unicode progress bar during scanning.
- `--consumer-source functions|imports`  How to derive the Consumer column. `functions` uses the first segment of function FQFNs (default). `imports` uses the import graph.
- `--gopls-remote [ADDR]`  Pass `-remote=auto` or `-remote=ADDR` to `gopls` to reuse a shared daemon/cache (e.g., `tcp://127.0.0.1:3737`).
- `--debug-expand`  Print debug info while replacing top-level functions with callers (what gets dropped/added and what’s skipped).

By default, callers from test files are excluded. Use `--include-tests` to opt in.

## Output

The summary shows one row per method usage with four columns:
- Package: service/package name (e.g., `vault`, `proxy_manager`)
- Method: method name (e.g., `GetSecret`)
- Consumer: derived from `--consumer-source` (default: first path segment of referencing functions)
- Functions: fully-qualified function names referencing the method

In table format, rows are grouped for readability:
- Package values are shown once per group; subsequent rows omit the Package cell.
- Method values are shown once per method within the package; subsequent rows for the same method omit the Method cell.
- Consumer separators: within a method, a thin horizontal separator is drawn across Consumer→Functions when the consumer changes. Between methods, a separator spans Method→Functions. Between packages, a full-width separator is used.

In `tsv` format, the same four columns are output without grouping visuals.

## Notes

- Progress bar: a Unicode progress bar is shown on TTYs with granular sub-steps; suppressed with `--no-progress` or on non-TTY.
- Test files: `*_test.go` are excluded by default from callers/expansion; use `--include-tests` to include them.
- Top-level pkg expansion: functions originating under dirs specified by `--expand-top-level-pkg` (default `pkg`) are replaced by their callers.
- Gopls cache reuse: use `--gopls-remote` to point `gopls` to a remote daemon (`auto` or explicit address) for improved performance.
