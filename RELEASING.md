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

1. Bump the version in the relevant `Cargo.toml` file(s) and commit.
2. Tag and push:

   ```bash
   git tag meteroid-utoipa-v5.5.0
   git push origin meteroid-utoipa-v5.5.0
   ```

   The **Publish crates** workflow (`.github/workflows/publish-crates.yaml`) runs on
   any `meteroid-utoipa-v*` tag. You can also trigger it manually from the Actions
   tab, with an optional dry-run.

The workflow publishes each crate in order and **skips versions already on
crates.io**, so re-running after a partial failure is safe. Because the crates
depend on each other, a brand-new crate name must be published before the crate
that depends on it — the workflow handles this by publishing in order and letting
cargo wait for the index between steps.
