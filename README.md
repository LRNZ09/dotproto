# ~/.proto

Home directory of [proto](https://moonrepo.dev/proto), the single version manager for
every toolchain on this machine (node, go, bun, deno, rust, ruby, …).

## How proto works

Every tool command is a **shim**: running `node` actually runs `~/.proto/shims/node`,
which resolves the version to use *at call time* — from a `.prototools` file in the
current directory, then its ancestors, then the global [`.prototools`](.prototools) in
this directory — and executes that version from `tools/`. Tools proto doesn't support
natively (biome, gh, jq, zig, openjdk, …) exist only because of third-party plugins
declared under `[plugins.tools]` in `.prototools` — lose that block and those tools
stop being installable. With `auto-install = true`, a missing version installs itself
on first use.

To see everything pinned globally, read [`.prototools`](.prototools); to see what's
installed vs. resolved, run `proto status`.

| Path | What it is |
| --- | --- |
| `.prototools` | Global version pins + plugin sources — **the source of truth** |
| `bin/` | The `proto` binary and version-suffixed tool binaries (`go-1.24`, …) |
| `shims/` | Version-resolving shims — first on `PATH` |
| `tools/` | The actual installs, one directory per tool per version |
| `plugins/` | Cached plugin definitions (TOML/WASM) |
| `builders/` | Helpers for tools built from source (e.g. `ruby-build`) |
| `cache/`, `temp/` | Disposable |

## Getting started on a new machine

1. Install proto (creates `~/.proto`):

   ```sh
   curl -fsSL https://moonrepo.dev/install/proto.sh | bash
   ```

2. Wire it into fish (`~/.config/fish/config.fish`):

   ```fish
   set -gx PROTO_HOME "$HOME/.proto"
   set -gx PATH "$PROTO_HOME/shims:$PROTO_HOME/bin" $PATH
   ```

3. Restore the config from this repo, inside `~/.proto`:

   ```sh
   git init && git remote add origin https://github.com/LRNZ09/dotproto.git && git fetch && git checkout -f main
   ```

4. Install everything pinned in `.prototools`:

   ```sh
   proto install
   ```

5. Verify with `proto status`.

## Daily commands

| Command | Purpose |
| --- | --- |
| `proto status` | List configured tools and their install status |
| `proto outdated` | Check pinned versions against latest releases |
| `proto pin <tool> <version> --to global` | Pin globally (default `--to local` writes `./.prototools`) |
| `proto install <tool> [version]` | Install one tool (rarely needed — auto-install is on) |
| `proto upgrade` | Upgrade proto itself |
| `proto clean` | Purge stale tool versions (auto-clean is on) |

To add a tool proto doesn't know: add its plugin under `[plugins.tools]` in
`.prototools`, pin a version, done.

## Backup

Everything in this directory regenerates **except `.prototools` and this README**.
This directory is a git repo tracking exactly those two files, pushed to
[LRNZ09/dotproto](https://github.com/LRNZ09/dotproto) — a machine loss costs nothing.
