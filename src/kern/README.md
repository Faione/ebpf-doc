# kern program

## libbpf

开发 ebpf C 程序所需要的包，包括 helper 函数，相关宏定义等

```shell
# arch
sudo pacman -S libbpf
# ubuntu
sudo apt-get install libbpf-dev
```

## vmlinux.h

[vmlinux](https://www.grant.pizza/blog/vmlinux-header/)

[vmlinux.h](https://www.ebpf.top/post/intro_vmlinux_h/) 中包含了linux kernel 内部定义的结构体及变量，如果需要对内核数据进行读取/写入，使用该头文件非常方便

```shell
bpftool btf dump file /sys/kernel/btf/vmlinux format c > vmlinux.h
```
