# Prog Types


## Auto Attach

Prog Types [^1] 决定了 bpf 程序在内核中的 hook point，可以在 bpf 程序的 `SEC()` 中定义类型以及额外信息，若额外信息被定义，则libbpf 会获取此信息，来自动决定 bpf 程序的挂载位置，用户也能够在 open 之后进行手动更改，或者是 attach 时进行手动指定

如下 `kprobe` 为bpf程序的类型，而 `__x64_sys_clone` 则是提供的额外信息，因而能够自动

```c
// kernel
SEC("kprobe/__x64_sys_clone")
int BPF_KPROBE(sys_clone){}

// user
// auto attach
skel.attach()? 

// manual attach
let mut progs = skel.progs_mut();
let sys_clone = progs.sys_clone();

// 注意保存 link 以便后续释放资源，否则 bpf 程序会在 attach 之后就立即被drop
let link = sys_clone.attach_kprobe(false, "__x64_sys_clone")?;
link.detach()?;
```

[^1]: [bpf_prog_types](https://docs.kernel.org/bpf/libbpf/program_types.html)