# Mirror-Crates v1.1.0

**Release Date:** 2026-04-11

Mirror-Crates v1.1.0 is a substantial cleanup and workflow-correction release.

This version focuses less on adding surface-area features and more on making the tool behave the way operators expect it to behave. The result is a more honest, more usable, and more storage-efficient mirroring pipeline.

## Highlights

- **Fast update runs now behave like update runs** - Existing crate files are trusted by default, with opt-in `-verify-existing` when a full checksum sweep is desired.
- **Bundle mode is now storage-correct** - `-bundle-mode only` is the default, so rolling bundles no longer silently duplicate the mirror as loose files.
- **Bundle workflows are now first-class** - Bundles preserve shard layout internally, emit per-bundle manifests, and produce a top-level `bundles.index.jsonl`.
- **Sidecars now match the storage mode** - Loose-file mirrors use adjacent `.crate.json` files; bundled mirrors use aggregated JSONL sidecars.
- **Prometheus is now on by default** - Metrics and pprof start on `:9090` automatically unless disabled.
- **Documentation now matches reality** - README, help, architecture, and Prometheus docs were rewritten around the actual workflow and current behavior.

## What Changed

### Downloader

- Standardized default concurrency to `128`
- Added fast incremental update behavior
- Added opt-in `-verify-existing`
- Added explicit bundle storage modes: `only|add`
- Made bundle-first mode default to `only`
- Improved startup logging and final run summaries
- Clarified manifest semantics as a JSONL audit log for the run

### Bundles

- Bundles now preserve the crates.io shard layout inside the archive
- Each bundle writes its own manifest JSONL
- Bundled runs now write a top-level `bundles.index.jsonl`
- Bundle-first runs no longer keep loose crate files unless explicitly requested

### Sidecars

- Added `files|jsonl` output modes
- Loose mirrors continue to support adjacent `.crate.json` files
- Bundled mirrors can now generate a single JSONL metadata stream
- JSONL sidecars can be enriched from downloader manifest bundle metadata

### Tooling

- Added `extract-bundles` CLI to restore rolling bundles into a normal shard tree
- Expanded the Python wrapper so it forwards the real day-to-day downloader flags
- Improved wrapper startup/final summaries and index update reporting

### Observability

- Prometheus and pprof now start automatically on `:9090`
- `-listen` can still override the address or disable the listener with an empty string
- Status output and metric labels now reflect the real result categories:
  - `downloaded`
  - `existing`
  - `verified_existing`
  - `error`

## Removed from Scope

- The in-repo Archive-Hasher component is no longer part of the project direction for release work. Archive hashing and detached signing are handled by a separate project and are not considered core Mirror-Crates responsibilities.

## Upgrade Notes

Operators upgrading from v1.0.0 should be aware of these behavior changes:

- `-concurrency` now defaults to `128`
- Prometheus now starts automatically on `:9090`
- Bundle mode now defaults to `only`, which removes loose crate files after bundling
- Routine updates are faster because existing files are trusted unless `-verify-existing` is set
- JSONL sidecars are the recommended metadata format for bundled mirrors

## Recommended Workflows

### Loose-file mirror

```powershell
python Clone-Index.py --index-dir "S:\Rust-Crates\crates.io-index" --output-dir "S:\Rust-Crates\crates.io" --threads 128 --progress-interval 5s --non-interactive
```

### Fast update run

```powershell
.\bin\download-crates.exe -index-dir "S:\Rust-Crates\crates.io-index" -out "S:\Rust-Crates\crates.io" -concurrency 128 -progress-interval 5s
```

### Bundle-first mirror

```powershell
.\bin\download-crates.exe -index-dir "S:\Rust-Crates\crates.io-index" -out "S:\Rust-Crates\staging" -bundle -bundle-mode only -bundle-size-gb 8 -bundles-out "S:\Rust-Crates\bundles" -manifest "S:\Rust-Crates\manifest.jsonl" -concurrency 128
```

### Bundle-oriented sidecars

```powershell
.\bin\generate-sidecars.exe -index-dir "S:\Rust-Crates\crates.io-index" -output-mode jsonl -jsonl-out "S:\Rust-Crates\bundle-sidecars.jsonl" -manifest "S:\Rust-Crates\manifest.jsonl" -concurrency 128 -include-yanked
```

### Restore bundles to loose files

```powershell
.\bin\extract-bundles.exe -bundles-dir "S:\Rust-Crates\bundles" -out "S:\Rust-Crates\restored"
```

## Validation

The release state was validated with:

- `go test ./...`
- `python Clone-Index.py --help`
- `go run ./cmd/download-crates -h`
- `go run ./cmd/generate-sidecars -h`
- `go run ./cmd/extract-bundles -h`

## What's Next

The next planned follow-up release will focus on:

- Bundle inspection utilities
- End-to-end integration coverage for bundle create/extract flows
- Additional bundle/manifest utilities for operator workflows

## Documentation

- [README](README.md)
- [Architecture](Docs/Architecture.md)
- [Prometheus Guide](Docs/Prometheus.md)
- [Airgap Guide](Docs/Airgap-Guide.md)
- [Quickstart (Windows)](Docs/Quickstart-Windows.md)

## License

MIT License. See [LICENSE](LICENSE) for details.
