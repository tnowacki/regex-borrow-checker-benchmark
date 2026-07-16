# Borrow-checker performance benchmark

Reproduces the wall-time performance of the new regex-based reference-safety checker vs the deployed (graph-based) approach.

The tool times both reference-safety analyses per Move function, in isolation, over the corpus of published Sui packages, and reports the ratio of the means plus richer statistics (median, p90/p95/p99, min, max, std dev, total).

## What it measures

For each non-native function it times:

- **old**: `move_bytecode_verifier::reference_safety::verify` (deployed, borrow-graph based), and
- **new**: `move_bytecode_verifier::regex_reference_safety::verify` (regex based).

Both run back-to-back on the same thread per function. Timing uses nanosecond resolution internally and is reported in microseconds.

## Requirements

- A recent stable Rust toolchain (edition 2024, i.e. Rust >= 1.85). Install via https://rustup.rs. The script does not install anything for you.
- Network access on first build (to fetch the pinned Sui crates) and to download the dataset.
  - NOTE: the `sui-packages` dataset is large. It takes roughly 13-16 GB (a shallow clone, which the script does, needs ~13 GB; a full clone ~16 GB), as of 15 July 2026.

## Run

```sh
./run.sh
```

The script will:

1. check for `cargo`/`rustc`
2. locate the dataset. It uses `$SUI_PACKAGES_DIR` if set, otherwise prompts for a directory (default `$HOME/sui-packages`) and offers to shallow-clone `MystenLabs/sui-packages`
3. ask which corpus to run: **mainnet most used** (`packages/mainnet_most_used`, fast) or **all mainnet** (`packages/mainnet`, full/slow)
4. ask for the number of parallel jobs (default 1)
5. build (in **release**) and run.

You can also run the tool directly (it has exactly two settings):

```sh
cargo build --release --manifest-path reference-safety-bench/Cargo.toml
./reference-safety-bench/target/release/reference-safety-bench <TARGET_DIR> --jobs <N>
```

## Dataset notes

`packages/mainnet_most_used/` entries are symlinks into `packages/mainnet/`.

The corpus contains only packages the deployed bytecode verifier accepts, which includes the deployed (graph-based) approach. The regex-based approach is strictly more expressive. As such, both analyses should accept every function.
