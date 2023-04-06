# rust

```shell
# libbpf crate
cargo add libbpf-rs

# libbpf build tool
cargo add --build libbpf-cargo

# commandline crate
cargo add clap --features derive

# syscall crate
cargo add nix
```

build.rs

```rust
use std::{env, path::PathBuf};

use libbpf_cargo::SkeletonBuilder;

const SRC: &str = "src/bpf/netstat.bpf.c";

fn main() {
    let mut out =
        PathBuf::from(env::var_os("OUT_DIR").expect("OUT_DIR must be set in build script"));
    out.push("netstat.skel.rs");
    SkeletonBuilder::new()
        .source(SRC)
        .build_and_generate(&out)
        .unwrap();
    println!("cargo:rerun-if-changed={SRC}")
}
```