# dotproto

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

1. Clone this repo first — it must happen while `~/.proto` doesn't exist yet, since
   git won't clone into the non-empty directory the installer creates:

   ```sh
   git clone https://github.com/LRNZ09/dotproto.git ~/.proto
   ```

2. Install proto into it:

   ```sh
   curl -fsSL https://moonrepo.dev/install/proto.sh | bash
   ```

3. Wire it into fish (`~/.config/fish/config.fish`):

   ```fish
   set -gx PROTO_HOME "$HOME/.proto"
   fish_add_path "$PROTO_HOME/shims" "$PROTO_HOME/bin"
   ```

   `fish_add_path` skips directories that don't exist yet and re-runs on every
   shell start, so `shims/` (created by step 4) is picked up by the next shell.

4. Install everything pinned in `.prototools`:

   ```sh
   proto install
   ```

5. Verify with `proto status`.

### Already have proto installed?

`~/.proto` exists and isn't empty, so a clone won't work — adopt the config in-place
instead. Inside `~/.proto` (this overwrites the machine's local `.prototools`):

```sh
git init && git remote add origin https://github.com/LRNZ09/dotproto.git && git fetch && git switch -f main
```

Then continue from step 4.

## Command cheatsheet

Inspect:

| Command | Purpose |
| --- | --- |
| `proto status` | Every configured tool: pinned vs resolved vs installed |
| `proto versions <tool>` | Available versions (`--installed` for local ones) |
| `proto outdated` | Check pins against latest releases (`--update` rewrites the pins) |
| `proto bin <tool>` | Absolute path of the executable the shim would run |
| `proto debug config` | Every loaded `.prototools` + the merged result — use when a version resolves unexpectedly |

Install & pin:

| Command | Purpose |
| --- | --- |
| `proto install` | Install everything configured in `.prototools` |
| `proto install <tool> [version]` | Install one tool (rarely needed — auto-install is on) |
| `proto pin <tool> <version>` | Pin for the current project (`./.prototools`) |
| `proto pin <tool> <version> --to global` | Pin machine-wide |
| `proto unpin <tool> [--from <scope>]` | Remove a pin |
| `proto uninstall <tool> [version]` | Remove installed versions |

Plugins (tools proto doesn't know natively):

| Command | Purpose |
| --- | --- |
| `proto plugin search <query>` | Find community plugins |
| `proto plugin add <id> <locator> --to global` | Register a plugin source |
| `proto plugin list` / `proto plugin info <id>` | What's registered / details + inventory |

Maintenance:

| Command | Purpose |
| --- | --- |
| `proto upgrade` | Upgrade proto itself |
| `proto clean` | Purge stale tool versions (auto-clean is on) |
| `proto regen` | Rebuild the shims from scratch |
| `proto diagnose` | Health-check the proto installation |

### Pin scopes

`--to` picks which `.prototools` a pin lands in. Resolution walks up from the current
directory (nearest file wins), with the global file always loaded last as the fallback:

| Scope | File | Wins when |
| --- | --- | --- |
| `local` (default) | `./.prototools` | inside that project |
| `user` | `~/.prototools` | anywhere in your home tree, unless a project overrides |
| `global` | `~/.proto/.prototools` | nothing else pins the tool |

## Worked example: node & openjdk

A language proto knows natively — no plugin needed:

```sh
proto versions node             # what's out there
proto install node lts          # install a version, touches no config
proto pin node lts --to global  # machine-wide default

cd ~/dev/legacy-app
proto pin node 20               # project override -> ./.prototools
node --version                  # v20.x in here...
cd ~ && node --version          # ...the global lts everywhere else
```

A tool proto doesn't know — third-party plugin first, then the same lifecycle:

```sh
proto plugin search jdk         # find a community plugin
proto plugin add openjdk "github://eplightning/openjdk-adoptium-proto-plugin" --to global
proto pin openjdk 26 --to global  # the plugin alone is NOT enough: no pin, no shims
proto install openjdk           # installs + registers the java/javac/javadoc shims
java -version                   # Temurin via ~/.proto/shims/java
```

Then check the state of the world:

```sh
proto status                    # pinned vs resolved vs installed, per tool
proto outdated                  # anything newer? --update rewrites the pins
```

## Backup

Everything in this directory regenerates **except `.prototools` and this README**.
This directory is a git repo tracking exactly those two files, pushed to
[LRNZ09/dotproto](https://github.com/LRNZ09/dotproto) — a machine loss costs nothing.
