# Contributing to dexy

dexy is a single Bash script. Keep it simple, portable, and POSIX-friendly
where reasonable.

## Local development

```sh
git clone https://github.com/jonobri/dexy
cd dexy
./dexy status        # safe, never needs sudo
shellcheck dexy      # if you have it: brew install shellcheck
```

`on` / `off` / `toggle` call `sudo pmset`, so test those by hand.

## Style

- Bash with `set -euo pipefail`.
- No external dependencies beyond macOS built-ins (`pmset`, `awk`, `sudo`).
- Keep output greppable and honour `NO_COLOR`.
- Run `shellcheck` clean before opening a PR.

## Cutting a release

The Homebrew tap ([jonobri/homebrew-tap](https://github.com/jonobri/homebrew-tap))
is updated **automatically** — you only tag a release here.

1. Bump `DEXY_VERSION` in `dexy` and the `.TH` line in `dexy.1`.
2. Commit on `main`.
3. Tag and push:
   ```sh
   git tag vX.Y.Z
   git push origin main vX.Y.Z
   gh release create vX.Y.Z --title "dexy X.Y.Z" --notes "…"
   ```
4. Publishing the release fires the **Bump Homebrew formula** workflow
   (`.github/workflows/bump-formula.yml`), which:
   - downloads the new release tarball,
   - computes its `sha256`,
   - rewrites `url` + `sha256` in the tap's `Formula/dexy.rb`,
   - commits and pushes to the tap.

   You can also run it manually from the Actions tab (`workflow_dispatch`)
   with a tag input if you ever need to re-bump.

> **Note:** the workflow only updates `url` and `sha256`. Any *other* formula
> change (e.g. a new `install` step) must be edited in the tap repo by hand.

### How the auto-bump authenticates

The workflow pushes to a *different* repo, so it can't use the default
`GITHUB_TOKEN`. Instead it uses a **deploy key** scoped to the tap repo only:

- a write-enabled deploy key lives on `jonobri/homebrew-tap`, and
- its private half is stored as the `TAP_DEPLOY_KEY` secret on this repo.

To rotate it:

```sh
ssh-keygen -t ed25519 -f /tmp/tap_deploy -N "" -C "dexy-formula-bump"
gh repo deploy-key add /tmp/tap_deploy.pub -R jonobri/homebrew-tap \
  --title "dexy-formula-bump" --allow-write
gh secret set TAP_DEPLOY_KEY -R jonobri/dexy < /tmp/tap_deploy
rm -f /tmp/tap_deploy /tmp/tap_deploy.pub
```
