# Mango CLI downloads

Public Linux CLI binaries for Mango, for use in cloud agents and containers.
The application source and development history remain private. This repository
contains distribution instructions only; releases contain binaries and the
installer, not the application source.

## Install during dependency setup

Requires Linux x86-64 or ARM64, `curl`, `tar`, and `sha256sum` (or `shasum`).
No GitHub login, token, Go toolchain, C compiler, or GUI is required.

```sh
set -eu
MANGO_VERSION=v0.10.2
setup_dir=$(mktemp -d)
trap 'rm -rf "$setup_dir"' EXIT
curl --fail --silent --show-error --location --proto '=https' --proto-redir '=https' \
  "https://github.com/marcokoste/mango-releases/releases/download/$MANGO_VERSION/install.sh" \
  --output "$setup_dir/install.sh"
sh "$setup_dir/install.sh" --version "$MANGO_VERSION" --bin-dir "$HOME/.local/bin"
export PATH="$HOME/.local/bin:$PATH"
mango --version
```

Pin the version in the project's dependency setup. Omitting the installer's
`--version` selects the latest stable release. When HOME is unavailable, supply
an explicit writable `--bin-dir` and put it on PATH. The installer verifies the
archive SHA-256 and executable version before replacing a working binary.

## Register a checkout and include Mango changes in a PR

```sh
# Keep machine-local config and discovery staging outside the checkout.
export MANGO_CONFIG_DIR=/tmp/mango-state
mango init --key my-project --repo "$PWD" --track-issues-in-git=true
mango add-issue my-project --title "Fix checkout" --desc "Investigate regression" --type Bug --priority 3
mango add-doc my-project --title "Checkout investigation" --body "Findings"
git status --short
git add -- mango-issues/ docs/
git diff --cached
```

Use a persistent state directory if the agent's environment survives multiple
shells. Persist PATH and MANGO_CONFIG_DIR using the provider's setup environment.
Init is noninteractive and idempotent. For monorepos, add
`--component-root apps/tool` for an existing directory within the checkout.
A reused project key pointing at a new checkout needs deliberate `--update-repo`.

Issue JSON and doc Markdown live in the repository. Commit their intended changes
with the code changes so they appear in the agent's PR. Priority is 0 (whenever),
1 (soon), 2 (today), or 3 (right now, product-critical). To change it, use
`mango issue my-project ID --priority 2`. Run `mango help` for the complete CLI.

Discovery findings start in local SQLite. Use `mango discover my-project --all`,
`mango discoveries my-project`, `mango promote DISCOVERY_ID`, and
`mango discover-sync my-project` to persist approved references in issue/doc
files, then stage those files. Do not commit config.yaml or discoveries.db.

Existing Git ignore rules still apply. If a changed file is absent from status,
run `git check-ignore -v -- mango-issues/ABCDE.json`. Deliberately correct the
project's ignores or stage the intended file with `git add -f -- PATH`.

## Distribution contents

Each release includes `install.sh`, `checksums.txt`, and the amd64 and ARM64
archives. Each archive contains only `mango`, `LICENSE`, and `INSTALL.txt`.
GitHub's automatic source archives contain this distribution README only.
Linux binaries include SQLite support. Apple signing and the desktop GUI are
not provided by these Linux CLI downloads.
