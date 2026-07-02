# Releasing (meteroid-oss fork)

This fork publishes three crates to crates.io, in dependency order:

| crates.io name           | path             | import as (unchanged) |
| ------------------------ | ---------------- | --------------------- |
| `meteroid-utoipa-config` | `utoipa-config/` | `utoipa_config`       |
| `meteroid-utoipa-gen`    | `utoipa-gen/`    | `utoipa_gen`          |
| `meteroid-utoipa`        | `utoipa/`        | `utoipa`              |

The crates.io **package** names are prefixed with `meteroid-`, but the **library**
names are kept (`utoipa`, `utoipa_gen`, `utoipa_config`). Downstream code therefore
stays a drop-in with upstream:

```toml
[dependencies]
meteroid-utoipa = "5"
```

```rust
use utoipa::ToSchema; // still `utoipa::...`
```

The other workspace crates (`utoipa-swagger-ui`, `utoipa-axum`, ...) are **not**
published from this fork; they only build locally against the renamed core.

## One-time setup

Add a repository secret `CARGO_REGISTRY_TOKEN` (Settings → Secrets and variables →
Actions) — a crates.io API token with publish scope from
crates.io → Account Settings → API Tokens. The three crate names must be owned by
the token's account (the first publish claims them).

## Cutting a release

Releases are managed through **GitHub Releases**:

1. Bump the version of whatever changed in the relevant `Cargo.toml` file(s) and
   merge to the default branch. The versions published come from `Cargo.toml`, not
   from the tag name. In practice `meteroid-utoipa` and `meteroid-utoipa-gen` move
   together; `meteroid-utoipa-config` rarely changes.
2. On GitHub, go to **Releases → Draft a new release**, choose a new tag (e.g.
   `v5.5.1`, created when you publish), write the notes, and **Publish**.

Publishing the release fires the **Publish crates** workflow
(`.github/workflows/publish-crates.yaml`). You can also run it by hand from the
Actions tab (**Run workflow**) with an optional dry-run.

The workflow publishes each crate in dependency order and **skips versions already
on crates.io**, so:

- only the crates whose version you actually bumped get published;
- re-running after a partial failure is safe;
- a brand-new crate must land before the crate that depends on it — handled by
  publishing in order while cargo waits for the index between steps.

If a release publishes nothing, every crate version was already on crates.io —
you probably forgot to bump the `Cargo.toml` version(s).

> `release`-triggered workflows always run from the copy of this file on the
> **default branch**, so this workflow must be on `master` to fire on a release.

### Versioning

Because these crates are published under their own names (`meteroid-utoipa*`),
their versions are independent of upstream `utoipa` — there is no collision. The
initial `5.5.0` mirrors the upstream base the fork tracks; bump the patch
(`5.5.1`, `5.5.2`, …) for fork changes, and jump to match upstream when you rebase
onto a newer release (e.g. `5.6.0`). Versions must be valid semver
(`MAJOR.MINOR.PATCH`); four-component versions like `5.5.0.1` are not accepted.
