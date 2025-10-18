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
- `--langs go|python|csharp [...]`  Languages to scan for callers. Default: `go`. Add `python` and/or `csharp` to include those.
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

## Cross-language scanning (Python and C#)

You can include Python and C# callers via the `--langs` flag:

- `go` (default): Uses `gopls references` from the generated Go gRPC stubs.
- `python` (default): Heuristic scan that looks for calls like `.Method(` and `.Method.future(` in `*.py` files that reference the expected gRPC stub (e.g., `MyServiceStub`) or import `*_pb2_grpc`. Functions are reported as module-relative FQFNs like `path/to/module.Class.method` or `path/to/module.function`.
- `csharp` (default): Heuristic scan that looks for `.Method(` and `.MethodAsync(` in `*.cs` files that reference the expected client type (e.g., `MyService.MyServiceClient`). Functions are reported as `Namespace.Class.Method`.

Behavior and caveats:
- Tests are excluded by default; `--include-tests` opts in for all languages.
- The Consumer column still defaults to `functions` mode, derived from the first path segment of the function’s package/module path. The `imports` consumer mode currently applies to Go only.
- The Python/C# scans are dependency-free heuristics designed to be fast and reasonably accurate. If you need higher precision, consider integrating language servers in the future.

Examples:

```
# Scan Go only (default)
python3 depscan

# Include Python and C# callers
python3 depscan --langs go python csharp

# Include tests across languages
python3 depscan --langs python csharp --include-tests
```
