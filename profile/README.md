# atom-planet-embrace

This organization maintains forks of common Rust crates that have been modified to work in `no_std` and other unusual environments.

The name "atom-planet-embrace" has no deeper meaning — it was three random words that sounded interesting.

## Philosophy

The goal of each port is simple: **the `default` feature of the crate should not require `std`**. Consumers should be able to add a dependency without `default-features = false` and have it work in a `no_std` context out of the box.

When upstream functionality inherently requires the standard library — typically because it makes a syscall (e.g. getting the current time, reading from the filesystem, or resolving network addresses) — we try not to gate that functionality behind a `std` feature flag. Instead, we encapsulate it behind a **compile-time generic trait**. This lets callers on bare-metal or other constrained targets supply their own implementation of that behavior, rather than being forced to either pull in `std` or lose the functionality entirely.

## Approach to porting

- The upstream crate's `no_std`-compatible core is preserved as-is where possible.
- `std`-dependent behavior is identified and abstracted behind a trait boundary.
- The `std` feature re-enables the default upstream behavior by providing a blanket impl of that trait backed by the standard library.
- Crate names are prefixed with `ai-` for the same reason that "dynamic programming" is named: it's a catchy buzzword that is tangentially related to the project.

## Contributing

Pull requests are currently disabled. The increased emphasis on AI coding agents in this project has made supply chain security a heightened concern. Pull requests will be enabled once a suitable process for vetting incoming changes is in place.

## Agent-driven development

This project is an experiment in pushing the limits of AI coding agents. The initial porting of each library and the ongoing maintenance of the forks are performed primarily by coding agents.

---

*This README was written by [Claude](https://claude.ai).*
