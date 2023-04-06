# try with bpftools

# Programming

一个ebpf项目大体可分为三部分
- 内核态程序: 使用C/C++/Rust等语言编写，符合ebpf规范的源代码，将被交叉编译为bpf程序
- 接口层: 提供与内核态bpf程序交互的方法，如读写map
- 用户态程序: 负责在用户态对bpf的输出进一步进行处理，也能够加载/卸载bpf程序，支持多种语言

只有LLVM支持的语言才可以被直接编译为bpf程序，如C/C++/Rust

简单而言，需要首先选择用户态的开发语言，然后找到相应的bpf开发框架(能够生成接口层与内核态程序骨架), 再进行开发

## 内核态程序

一个简易的ebpf程序，包含两个主要部分
- license: 字符数组，使用 `SEC("license")` 属性标记，值为 `Dual BSD/GPL`, 允许ebpf程序在运行时进行检验
- attach function: 函数，使用 `SEC("tp/syscalls/sys_enter_write")` 属性标记， 其中 `tp/syscalls/sys_enter_write` 指定了函数将要挂载的位置，函数名称会作为ebpf程序的名称，而每当系统调用`write()`触发时，就会触发函数的执行

```c
// minimal.bpf.o
#include <linux/bpf.h>
#include <bpf/bpf_helpers.h>

char LICENSE[] SEC("license") = "Dual BSD/GPL";

int my_pid = 0;

SEC("tp/syscalls/sys_enter_write")
int handle_tp(void *ctx)
{
    // 通过helper函数获得pid
	int pid = bpf_get_current_pid_tgid() >> 32;

	if (pid != my_pid)
		return 0;

	bpf_printk("BPF triggered from PID %d.\n", pid);

	return 0;
}
```

编译

```
clang -O2 -target bpf -c minimal.bpf.c -o minimal.bpf.o
```

## bpftool

将bpf程序加载到内核, `bpftool prog help` 查看完整命令
- bpf prog: 编译好的ebpf程序
- path: 用来挂载bpffs, 并将bpf prog保存到bpffs中，以下命令会将bpffs挂载到`$PWD`, 然后将bpf prog保存到`tp_demo`，一般会将bpf prog挂载到`sys/fs/bpf`中
- type: bpf 程序的类型

```shell
# 加载bpf prog
sudo bpftool prog load minimal.bpf.o $PWD/tp_demo type tracepoint

# 查看bpf progs
sudo bpftool prog list
```

从内核卸载bpf程序，按照 bpf object 的lifetime, 只需要将 `tp_demo` 文件删除，就会从内核中卸载bpf程序