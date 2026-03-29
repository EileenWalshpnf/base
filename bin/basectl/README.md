# `basectl`

Base infrastructure control CLI: a terminal UI for watching Base chain health—block production,
sync, flashblocks throughput, DA backlog, HA conductor layout, and related metrics. Use it when you
want an operator-focused dashboard against live RPC/WebSocket endpoints instead of ad-hoc scripts.

## Relationship to the library crate

The interactive logic lives in the workspace crate [`basectl-cli`](../../crates/infra/basectl/)
(`crates/infra/basectl`). This binary is a thin entry point that parses CLI arguments, loads
[`ChainConfig`](../../crates/infra/basectl/src/config.rs), and dispatches into `basectl-cli` (for
example `run_app`, `run_app_with_view`, `run_flashblocks_json`). See that crate’s README for
embedding or testing the same views programmatically.

## Configuration (`-c` / `--config`)

The global option is **`-c`** or **`--config`** (default: `mainnet`). Resolution is implemented in
`ChainConfig::load` in `basectl-cli`:

1. **Built-in names** — `mainnet`, `sepolia`, or `devnet` supply default RPC/WebSocket URLs and
   registry-derived addresses where applicable.
2. **User overrides** — if `~/.config/base/networks/<name>.yaml` exists, it is merged on top of a
   matching built-in base, or loaded alone when the name is not built-in.
3. **File path** — any other value that exists as a path is read as a standalone YAML config.

For **`devnet`**, `system_config` and `batcher_address` are filled by calling the op-node method
`optimism_rollupConfig` (the devnet must be running).

## Typical usage

From the repository root, after building:

```sh
cargo run -p basectl --release -- --config sepolia
cargo run -p basectl --release -- -c mainnet conductor
```

### Just recipe

The repo `Justfile` defines:

```just
just basectl               # same as -c mainnet (default)
just basectl sepolia       # -c sepolia
just basectl devnet        # -c devnet
```

That recipe runs `cargo run -p basectl --release -- -c <config>`. It does not pass subcommands; add
them after `--` when invoking `cargo run` directly, for example:

```sh
cargo run -p basectl --release -- -c devnet conductor
```

For the Docker-based devnet, the `devnet` Just submodule defines **`just devnet conductor`**, which
runs `basectl` with `--config devnet conductor` (conductor monitor; requires the devnet stack to be
up). See [`etc/docker/README.md`](../../etc/docker/README.md) for devnet commands.

## Subcommands

All subcommands inherit the global `--config` / `-c`. Short flags below are **visible aliases**
from the CLI definition in `bin/basectl/src/cli.rs`:

| Command | Alias | Purpose |
|--------|-------|---------|
| *(none)* | — | Full default TUI (`run_app`). |
| `config` | `c` | Chain configuration view. |
| `flashblocks` | `f` | Flashblocks view; add `--json` for newline-delimited JSON (`run_flashblocks_json`) instead of the TUI. |
| `da` | `d` | Data availability backlog monitor. |
| `command-center` | `cc` | Combined command-center view. |
| `conductor` | `co` | HA conductor cluster monitor. |

## Release binaries

Release builds publish this binary alongside the node; you can also install it with
[`baseup`](../../baseup/README.md) (`baseup --bin basectl`).

## License

Licensed under the [MIT License](https://github.com/base/base/blob/main/LICENSE).
