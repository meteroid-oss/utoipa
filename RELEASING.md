# Releasing (meteroid-oss fork)

This fork publishes three crates to crates.io, in dependency order:

| crates.io name           | path             | import as (unchanged) | version |
| ------------------------ | ---------------- | --------------------- | ------- |
| `meteroid-utoipa-config` | `utoipa-config/` | `utoipa_config`       | 0.1.2   |
| `meteroid-utoipa-gen`    | `utoipa-gen/`    | `utoipa_gen`          | 5.6.0   |
| `meteroid-utoipa`        | `utoipa/`        | `utoipa`              | 5.6.0   |

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
2. On GitHub, go to **Releases → Draft a new release**, choose a new tag, write the
   notes, and **Publish**.

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

> `release`-triggered workflows always run from the copy of the workflow file on
> the **default branch**, so it must be on `master` to fire on a release.

## Which tag do I pick?

The three crates have different versions, but you only pick **one** tag, and it's
just a trigger — it does not decide what gets published (that comes from each
crate's `Cargo.toml`). Use the **`meteroid-utoipa` version**, e.g. `v5.6.0`. That
is the crate you and downstream depend on, so the release *is* "meteroid-utoipa
5.6.0". The other two ride along automatically:

- `meteroid-utoipa-gen` moves **in lockstep** with `meteroid-utoipa` (same number —
  the derive-macro crate and its runtime are a matched pair), so it is covered.
- `meteroid-utoipa-config` is independent and rarely changes; whenever you bump its
  own `Cargo.toml`, it publishes on the next release regardless of the tag.

The workflow emits a warning if the release tag's version doesn't match the
`meteroid-utoipa` version, to catch a forgotten bump.

## Versioning

These crates publish under their own names (`meteroid-utoipa*`), so their versions
are independent of upstream `utoipa` — there is no collision on crates.io. Bump
only the crates that carry a change of the fork's own (versus upstream
juhaku/utoipa):

- `meteroid-utoipa-gen` and `meteroid-utoipa` currently diverge from upstream 5.5.0
  by two opt-in, backward-compatible features (`tagged_discriminator` and flatten
  support in `IntoParams`), hence the minor bump to **5.6.0**.
- `meteroid-utoipa-config` is unchanged from upstream, so it stays at **0.1.2**.

Keep `meteroid-utoipa` and `meteroid-utoipa-gen` on the same version. When you
rebase onto a newer upstream release, jump to match it (e.g. upstream 5.7.0 →
publish `5.7.0`). Versions must be valid semver (`MAJOR.MINOR.PATCH`);
four-component versions like `5.5.0.1` are not accepted.
