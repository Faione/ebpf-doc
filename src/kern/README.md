# kern program

## vmlinux.h

[vmlinux](https://www.grant.pizza/blog/vmlinux-header/)

[vmlinux.h](https://www.ebpf.top/post/intro_vmlinux_h/) 中包含了linux kernel 内部定义的结构体及变量，如果需要对内核数据进行读取/写入，使用该头文件非常方便

```shell
bpftool btf dump file /sys/kernel/btf/vmlinux format c > vmlinux.h
```

[kern_ebpf_doc](https://www.kernel.org/doc/html/latest/bpf)
[](https://prototype-kernel.readthedocs.io/en/latest/bpf/ebpf_maps.html)