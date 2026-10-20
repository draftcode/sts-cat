---
name: releng
description: Release engineering for sts-cat. Use whenever the user asks to cut, prepare, tag, or ship an sts-cat release, bump the version, write the CHANGELOG for a release, commit "releng vX.Y.Z", or redo a failed release workflow (re-tag), even if they only say "release it" or "ship a patch".
---

# sts-cat release engineering

Run steps in order and stop at each gate marked (user). Ask before every commit, push, or tag; the user publishes the GitHub release. Never use `git -C`; `cd` into the repo instead. Put temp files in `tmp/` under the repo root.

## 1. Decide the version

(user) The user decides patch or minor. Releng is committed directly on `main` with no PR, matching history (`releng v0.4.0`, `releng v0.4.1`).

## 2. Prepare

1. On up-to-date `main`, bump `version` in `Cargo.toml` and run `cargo build` to update `Cargo.lock`.
2. In `CHANGELOG.md`, rename `## [unreleased]` to `## X.Y.Z (YYYY-MM-DD)` using today's date. Check the previous release also got its dated heading.
3. Sections, only those that apply: `### Enhancements`, `### Bug fixes`. End entries with PR links, e.g. `[#1](https://github.com/sorah/sts-cat/pull/1)`.

## 3. Commit and push

The pre-commit hook is hk 2.x (`hk.pkl`, `hk = "2"` pinned in `mise.toml`); commit through mise so the pinned hk runs:

```
mise exec -- git commit -m 'releng vX.Y.Z' Cargo.toml Cargo.lock CHANGELOG.md
git push origin main
```

## 4. Tag

```
git tag --sign vX.Y.Z -m vX.Y.Z
git tag -v vX.Y.Z
git push origin vX.Y.Z
```

If signing fails with `Inappropriate ioctl for device`, ask the user to unlock their key; never skip signing.

## 5. Release workflow

The tag triggers `.github/workflows/release.yml`: test -> create-release (DRAFT, checks the tag matches `Cargo.toml`) -> build-binaries (musl x86_64/aarch64), build-lambda (x86_64 and arm64 via cargo-lambda/zig), terraform-module -> update-release-description -> publish-crate.

Watch it (`gh run watch`). (user) The user publishes the draft release.

## 6. Redo a failed release

Only with user approval, when the workflow must rerun on a fixed commit:

1. Land the fix on `main` and push.
2. Drop the draft first: `gh release delete vX.Y.Z --yes`.
3. Re-tag the fix commit and force-push the tag:

```
git tag -f --sign vX.Y.Z -m vX.Y.Z
git push --force origin vX.Y.Z
```

Known pitfall: cargo-lambda must be >= 1.9.2 (pinned in `mise.toml`). Older cargo-lambda bundles cargo-zigbuild 0.20.1, which fails arm64 linking with rustc >= 1.98 (`unsupported linker arg: --fix-cortex-a53-843419`, rust-lang/rust#161486). mise-action caches tools, so an unpinned `latest` can silently keep an old version.
