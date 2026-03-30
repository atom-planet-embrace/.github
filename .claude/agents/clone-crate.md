# Clone Crate Agent

Clone a GitHub repository and set it up as a local fork of a crates.io crate.

## Input

A GitHub repository URL (e.g., `https://github.com/Roughsketch/imagesize`).

## Steps

1. **Parse the URL** to extract the repo name (the last path segment, e.g., `imagesize`).

2. **Clone the repo** into the current working directory as `ai-<name>`:
   ```
   git clone <url> ai-<name>
   ```

3. **Read `Cargo.toml`** to determine if this is a single crate or a cargo workspace:
   - If the root `Cargo.toml` contains a `[workspace]` section with `members`, this is a **workspace**. Enumerate all member crate paths and read each member's `Cargo.toml` to collect every `[package] name`.
   - If it is a single crate (no `[workspace]`), read the crate name from the `[package] name` field and proceed as before.

   > **Workspace handling:** If this is a workspace, apply steps 10–18 as described below. Steps that are marked *per member* must be performed for **each** workspace member crate (using that member's crate name, `Cargo.toml`, and crate root file). Steps that are *repo-wide* are performed once, covering all members.

4. **Look up the latest version on crates.io** *(per member for workspaces)*:
   - For each crate name, fetch `https://crates.io/api/v1/crates/<crate-name>`.
   - Extract the `crate.newest_version` field from the JSON response.
   - For a workspace, look up the version of the first member crate.

5. **Fetch the published commit hash from docs.rs**:
   - Fetch `https://docs.rs/crate/<crate-name>/<version>/source/.cargo_vcs_info.json`.
   - Extract the `git.sha1` field from the JSON response — this is the exact commit that was published.
   - Note: the `path_in_vcs` field may be non-empty for workspace members in a monorepo.

6. **Reset the branch to the published commit**:
   ```
   git reset --hard <sha1>
   ```
   This ensures the fork starts from the exact code that was published to crates.io, not unreleased commits on HEAD.

7. **Rename the default branch to `main`** if it is not already `main`:
   ```
   git branch -m <current-branch> main
   ```

8. **Rename the `origin` remote to `upstream`**:
   ```
   git remote rename origin upstream
   ```

9. **Get the HEAD commit hash** (short and full):
    ```
    git rev-parse --short HEAD   # e.g., ce9eb0f
    git rev-parse HEAD            # e.g., ce9eb0fb1626c7c4a4772a3f1b166ac5a753a4fb
    ```

10. **Create the fork marker commit** *(repo-wide, once)* — an empty commit:
    ```
    git commit --allow-empty -m "Fork <crate-name> at <short-hash>" -m "Fork crates.io/crates/<crate-name> at <full-hash>"
    ```
    For a workspace, list all member crate names in the commit message (e.g., `"Fork crate-a, crate-b at <short-hash>"`).

11. **Convert the crate to `no_std`** *(per member for workspaces)*:
    - Add `#![cfg_attr(not(test), no_std)]` to the crate root (`lib.rs` or `main.rs`). Use `cfg_attr(not(test), ...)` so that test builds retain full `std` access.
    - Add a `std` feature to `Cargo.toml` `[features]` if one does not already exist. `std` must **not** be a default feature.
    - Replace `std::error::Error` with `core::error::Error` (stabilized in Rust 1.81). Do not feature-gate it.
    - Replace uses of `std::io` with `no_std_io` where needed — but first check if the `std::io` usage is just reading from a byte slice (e.g., `Cursor::new(slice)` + `read_exact`). If so, replace with `copy_from_slice` — no dependency needed. Only add `no_std_io` when true streaming I/O is required:
      ```toml
      no_std_io = { version = "0.6", default-features = false }
      ```
    - When the `std` feature is enabled, also enable `no_std_io/std` (if `no_std_io` was added):
      ```toml
      [features]
      std = ["no_std_io/std"]
      ```
    - For `std::time::SystemTime::now()` usage, apply the `Now` trait pattern: define `pub trait Now { fn now() -> Duration; }` and provide a `StdNow` implementation behind the `std` feature. Reference `https://github.com/atom-planet-embrace/ai-chrono/blob/main/src/offset/utc.rs` for the canonical pattern. Make `Default` impls that call `now()` conditional on `std` feature or use a no_std fallback.
    - For runtime CPU feature detection macros (`std::arch::is_x86_feature_detected!`, `std::arch::is_aarch64_feature_detected!`), create crate-local wrapper macros that use runtime detection when `std` is available and compile-time `cfg!(target_feature = ...)` fallback when it is not. Gate macros by target architecture (`#[cfg(any(target_arch = "x86", target_arch = "x86_64"))]` etc.). The `extern crate std` declaration should only be gated on `feature = "std"`, never on arch features.
    - Tests, doc tests, benchmarks, and fuzz tests can all assume the `std` feature is enabled (e.g., `#[cfg(feature = "std")]` or adding `std` to dev-dependencies features).
    - **Do not modify existing dependencies initially.** Only change dependencies in response to build failures (see below).
    - **Verify the changes before committing:**
      1. Run `cargo test --features std` — ensure tests still pass. If 0 tests run, investigate: tests may have been accidentally disabled.
      2. Run `cargo build --target thumbv7m-none-eabi --no-default-features` — verify the crate compiles under `no_std` without default features.
      3. Run `cargo check` (default features, no `std`) — verify that default features (e.g., arch/SIMD features) do not pull in `std`.
      4. If the `no_std` build fails due to a **dependency**, resolve it using this cascade:
         1. **Check for an existing fork:** Run `gh repo view atom-planet-embrace/ai-<dep-name>` (try both `ai-<dep-name>` and `ai_<dep-name>`). If a fork exists, replace the dependency with a crates.io dependency:
            ```toml
            <dep-name> = { package = "ai-<dep-name>", version = "<version>" }
            ```
         2. **Try `default-features = false`:** If no fork exists, add `default-features = false` to the failing dependency in `Cargo.toml` and rebuild.
         3. **Report the error:** If neither approach resolves the failure, report the error and stop.
      5. Re-verify until all three commands succeed.
    - Stage and commit:
      ```
      git add -A && git commit -m "Convert <crate-name> to no_std"
      ```

12. **Rename the crate in `Cargo.toml`** *(per member for workspaces, plus root)*:
    - Change `[package] name` from `<crate-name>` to `ai-<crate-name>`.
    - Change `[package] description` to `"A no_std fork of <crate-name>"`.
    - For workspaces: also update the root `Cargo.toml` — rename member paths if they reference the old crate name, and update any `[workspace.dependencies]` entries that reference the old crate names.

13. **Update the README.md** *(per member if README exists, plus root)*:
    - Add a new paragraph at the very beginning of `README.md` (before existing content).
    - The paragraph has two sentences:
      1. `This is a fork of the [<crate-name>](https://crates.io/crates/<crate-name>) crate.`
      2. `The git repository is located at <upstream-git-url>.` (the original GitHub URL provided as input)
    - For workspaces: update the root `README.md`, and also update each member's `README.md` if one exists (using that member's crate name in the fork sentence).

14. **Update code references** *(per member for workspaces, single repo-wide pass)*:
    - Compute the Rust identifier forms: the original name with hyphens replaced by underscores (`old_ident`), and the new `ai-<name>` with hyphens replaced by underscores (`new_ident`). For example, `image-size` → `image_size` and `ai-image-size` → `ai_image_size`.
    - Search all files for references to the original crate name (both hyphenated and underscore forms).
    - Replace them with the new name, **except** any line containing `"This is a fork of"`.
    - **Never modify `CHANGELOG.md` or `CHANGELOG` files.**
    - **Only replace existing references** — do not insert new text into doc comments or other files.
    - This covers: `use` statements, `extern crate`, doc comments, README mentions, test references, etc.

15. **Rewrite GitHub owner/org URLs** *(repo-wide, once)*:
    - Search all files for GitHub URLs pointing to the original owner/organization (extracted from the input URL, e.g., `github.com/Roughsketch/...`).
    - Replace the owner/org portion with `atom-planet-embrace` (e.g., `github.com/atom-planet-embrace/...`).
    - **Never modify `CHANGELOG.md` or `CHANGELOG` files.**

16. **Commit the rename changes** *(repo-wide, once)*:
    ```
    git add -A && git commit -m "Rename <old-crate-name> to ai-<old-crate-name>"
    ```
    For a workspace, include all renamed crates in the commit message.

17. **Update GitHub workflows** *(repo-wide, once)*:
    - Read the GitHub workflow files in `.github/workflows/`.
    - Update them to use the fork's conventions (e.g., updated crate name, URLs).
    - If no existing workflow step tests `no_std` compatibility, add steps that run:
      ```
      cargo build --target thumbv7m-none-eabi --no-default-features
      cargo check
      ```
      The first command verifies pure `no_std` compilation. The second verifies that default features (which may enable arch/SIMD features) do not pull in `std`.
    - Stage and commit:
      ```
      git add -A && git commit -m "Update GitHub workflows for ai-<old-crate-name>"
      ```

18. **Print a summary** *(repo-wide, once)* of what was done, including the clone path, upstream remote, no_std conversion, rename, updated URLs, and workflow changes. For workspaces, list all member crates that were processed.
