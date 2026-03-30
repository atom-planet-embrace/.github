# atom-planet-embrace

This organization maintains forks of common Rust crates that have been modified to work in `no_std` and other unusual environments.

The name "atom-planet-embrace" has no deeper meaning — it was three random words that sounded interesting.

## Philosophy

The goal of each port is simple: **the `default` feature of the crate should not require `std`**. Consumers should be able to add a dependency without `default-features = false` and have it work in a `no_std` context out of the box.

When upstream functionality inherently requires the standard library — typically because it makes a syscall (e.g. getting the current time, reading from the filesystem, or resolving network addresses) — we try not to gate that functionality behind a `std` feature flag. Instead, we encapsulate it behind a **compile-time generic trait**. This lets callers on bare-metal or other constrained targets supply their own implementation of that behavior, rather than being forced to either pull in `std` or lose the functionality entirely.

## Crates

| Name | Version | Status | Description |
|------|---------|--------|-------------|
| [`ai_byteorder_lite`](https://github.com/atom-planet-embrace/ai-byteorder-lite) | 0.1.0 | [![Build Status](https://github.com/atom-planet-embrace/ai-byteorder-lite/actions/workflows/ci.yml/badge.svg)](https://github.com/atom-planet-embrace/ai-byteorder-lite/actions) | Library for reading/writing numbers in big-endian and little-endian. |
| [`ai-chrono`](https://github.com/atom-planet-embrace/ai-chrono) | 0.4.44 | [![Build Status](https://github.com/atom-planet-embrace/ai-chrono/actions/workflows/test.yml/badge.svg)](https://github.com/atom-planet-embrace/ai-chrono/actions) | Date and time library for Rust |
| [`ai_color_quant`](https://github.com/atom-planet-embrace/ai_color_quant) | 1.1.0 | [![Build Status](https://github.com/atom-planet-embrace/ai_color_quant/actions/workflows/rust.yml/badge.svg)](https://github.com/atom-planet-embrace/ai_color_quant/actions) | Color quantization library to reduce n colors to 256 colors. |
| [`ai-exr`](https://github.com/atom-planet-embrace/ai-exrs) | 1.74.0 | [![Build Status](https://github.com/atom-planet-embrace/ai-exrs/actions/workflows/rust.yml/badge.svg)](https://github.com/atom-planet-embrace/ai-exrs/actions) | Read and write OpenEXR files without any unsafe code |
| [`ai-fax`](https://github.com/atom-planet-embrace/ai-fax) | | [![Build Status](https://github.com/atom-planet-embrace/ai-fax/actions/workflows/ci.yml/badge.svg)](https://github.com/atom-planet-embrace/ai-fax/actions) | Decoder and Encoder for CCITT Group 3 and 4 bi-level image encodings used by fax machines TIFF and PDF. |
| [`ai-fdeflate`](https://github.com/atom-planet-embrace/ai-fdeflate) | 0.3.8 | [![Build Status](https://github.com/atom-planet-embrace/ai-fdeflate/actions/workflows/rust.yml/badge.svg)](https://github.com/atom-planet-embrace/ai-fdeflate/actions) | Fast specialized deflate implementation |
| [`ai-flate2`](https://github.com/atom-planet-embrace/ai-flate2) | 1.1.9 | [![Build Status](https://github.com/atom-planet-embrace/ai-flate2/actions/workflows/main.yml/badge.svg)](https://github.com/atom-planet-embrace/ai-flate2/actions) | DEFLATE compression and decompression exposed as Read/BufRead/Write streams. Supports miniz_oxide and multiple zlib implementations. Supports zlib, gzip, and raw deflate streams. |
| [`ai-gif`](https://github.com/atom-planet-embrace/ai-gif) | 0.14.1 | [![Build Status](https://github.com/atom-planet-embrace/ai-gif/actions/workflows/rust.yml/badge.svg)](https://github.com/atom-planet-embrace/ai-gif/actions) | GIF de- and encoder |
| [`ai-image-webp`](https://github.com/atom-planet-embrace/ai-image-webp) | 0.2.4 | [![Build Status](https://github.com/atom-planet-embrace/ai-image-webp/actions/workflows/ci.yml/badge.svg)](https://github.com/atom-planet-embrace/ai-image-webp/actions) | WebP encoding and decoding in pure Rust |
| [`ai-imagesize`](https://github.com/atom-planet-embrace/ai-imagesize) | 0.14.0 | [![Build Status](https://github.com/atom-planet-embrace/ai-imagesize/actions/workflows/ci.yml/badge.svg)](https://github.com/atom-planet-embrace/ai-imagesize/actions) | Quick probing of image dimensions without loading the entire file. |
| [`ai-lebe`](https://github.com/atom-planet-embrace/ai-lebe) | 0.5.3 | [![Build Status](https://github.com/atom-planet-embrace/ai-lebe/actions/workflows/rust.yml/badge.svg)](https://github.com/atom-planet-embrace/ai-lebe/actions) | Tiny, dead simple, high performance endianness conversions with a generic API |
| [`ai-moxcms`](https://github.com/atom-planet-embrace/ai-moxcms) | 0.8.1 | [![Build Status](https://github.com/atom-planet-embrace/ai-moxcms/actions/workflows/ci.yml/badge.svg)](https://github.com/atom-planet-embrace/ai-moxcms/actions) | Simple Color Management in Rust |
| [`ai-png`](https://github.com/atom-planet-embrace/ai-png) | 0.18.1 | [![Build Status](https://github.com/atom-planet-embrace/ai-png/actions/workflows/rust.yml/badge.svg)](https://github.com/atom-planet-embrace/ai-png/actions) | PNG decoding and encoding library in pure Rust |
| [`ai-pxfm`](https://github.com/atom-planet-embrace/ai-pxfm) | 0.1.28 | [![Build Status](https://github.com/atom-planet-embrace/ai-pxfm/actions/workflows/ci.yml/badge.svg)](https://github.com/atom-planet-embrace/ai-pxfm/actions) | Fast and accurate math |
| [`ai-quick-error`](https://github.com/atom-planet-embrace/ai-quick-error) | 2.0.1 | [![Build Status](https://github.com/atom-planet-embrace/ai-quick-error/actions/workflows/ci.yml/badge.svg)](https://github.com/atom-planet-embrace/ai-quick-error/actions) | A macro which makes error types pleasant to write. |
| [`ai-resvg`](https://github.com/atom-planet-embrace/ai-resvg) | 0.47.0 | [![Build Status](https://github.com/atom-planet-embrace/ai-resvg/actions/workflows/main.yml/badge.svg)](https://github.com/atom-planet-embrace/ai-resvg/actions) | An SVG rendering library. |
| [`ai-resvg-capi`](https://github.com/atom-planet-embrace/ai-resvg) | | [![Build Status](https://github.com/atom-planet-embrace/ai-resvg/actions/workflows/main.yml/badge.svg)](https://github.com/atom-planet-embrace/ai-resvg/actions) | A no_std fork of resvg-capi |
| [`ai-strict-num`](https://github.com/atom-planet-embrace/ai-strict-num) | 0.2.0 | [![Build Status](https://github.com/atom-planet-embrace/ai-strict-num/actions/workflows/main.yml/badge.svg)](https://github.com/atom-planet-embrace/ai-strict-num/actions) | A collection of bounded numeric types |
| [`ai-tiff`](https://github.com/atom-planet-embrace/ai-image-tiff) | | [![Build Status](https://github.com/atom-planet-embrace/ai-image-tiff/actions/workflows/rust.yml/badge.svg)](https://github.com/atom-planet-embrace/ai-image-tiff/actions) | TIFF decoding and encoding library in pure Rust |
| [`ai-usvg`](https://github.com/atom-planet-embrace/ai-resvg) | 0.47.0 | [![Build Status](https://github.com/atom-planet-embrace/ai-resvg/actions/workflows/main.yml/badge.svg)](https://github.com/atom-planet-embrace/ai-resvg/actions) | An SVG simplification library. |
| [`ai-weezl`](https://github.com/atom-planet-embrace/ai-weezl) | | [![Build Status](https://github.com/atom-planet-embrace/ai-weezl/actions/workflows/rust.yml/badge.svg)](https://github.com/atom-planet-embrace/ai-weezl/actions) | Fast LZW compression and decompression. |
| [`ai-xmlwriter`](https://github.com/atom-planet-embrace/ai-xmlwriter) | 0.1.0 | [![Build Status](https://github.com/atom-planet-embrace/ai-xmlwriter/actions/workflows/ci.yml/badge.svg)](https://github.com/atom-planet-embrace/ai-xmlwriter/actions) | A simple, streaming XML writer. |
| [`ai-zune-inflate`](https://github.com/atom-planet-embrace/ai-zune-inflate) | 0.2.54 | [![Build Status](https://github.com/atom-planet-embrace/ai-zune-inflate/actions/workflows/ci.yml/badge.svg)](https://github.com/atom-planet-embrace/ai-zune-inflate/actions) | A heavily optimized deflate decompressor in Pure Rust |

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
