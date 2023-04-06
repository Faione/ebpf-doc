# landscape

# Rust ebpf ecosystem


[aya-rs](https://github.com/aya-rs/aya)
[libbpf-rs](https://github.com/libbpf/libbpf-rs)

## Libpf-rs

**feature**

- rust-style libbpf

**components**

kern ebpf code(C)
interface(automatical genaration)
usr code(rust)

1. write ebpf code by C
2. add build scrpit to project
3. libbpf-cargo automatically generates interface code
   - compiling ebpf program
   - loading or attaching bytecode
   - method to access bytecode segement
4. wrting usr code by both rust and interfaces


the ebpf byte code will be compiled together with rust 

## Aya

**feature**

- completely rust


**components**

- kern ebpf code(rust)
- interface(template)
- usr code(rust)
