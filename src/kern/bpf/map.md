# Maps

Bpf Maps 提供了用户与内核之间通用格式的数据分享存储，同时 bpf 程序之间也能够通过 maps 共享数据。用户空间中对 maps 的访问依赖 bpf 相关的 syscall

[ebpf_maps](https://blog.51cto.com/u_15703183/5464440)

## Perf Event && Ring Buffer

Perf Event 与 Ring Buffer 虽同属 BPF_MAP，但实质上并不具备 Map 的功能，其均类似队列，对外输出信息，不同于其他Map，依赖用户程序的轮询来取出数据，Perf Event 与 Ring Buffer 均支持 poll，即当队列中没有数据时，会阻塞当前进程以释放cpu资源, 而当有数据写入时再唤醒进程执行

Ring Buffer 与 Perf Event 在实现上的不同在于
- Perf Event是 Per CPU的， 而 Ring Buffer 能够在不同CPU之间能够共享， 从而更高效地利用内存资源
- 即使在多核场景中，Ring Buffer 也能够提供时序性保证

[ring_buf](https://docs.kernel.org/bpf/ringbuf.html)
[ring_buf_and_perf_event](http://arthurchiao.art/blog/bpf-ringbuf-zh/)

## Hash

对应于 Hash Map，允许使用结构体作为 key， kernel负责 k/v pair 内存空间的创建，回收，默认情况下，Hash Map 会使用一个预先申请的 hash table，使用 `BPF_F_NO_PREALLOC ` 可以关闭

[hash_map](https://docs.kernel.org/bpf/map_hash.html)
[advanced_hash_map](http://arthurchiao.art/blog/bpf-advanced-notes-3-zh/)

## Array

对应于数组，map array 使用 u32 作为下标，通过创建时 `max_entries ` 的设置来决定 array 的大小，array中的所有元素都会提前创建并进行0初始化

[array](https://docs.kernel.org/bpf/map_array.html)


## Pinning

获取 ebpf 程序 或者 map 的文件描述符后，可以通过 `BPF_OBJ_PIN` 将其持久化到文件系统中， 路径由用户指定，默认情况使用 `LIBPF_PIN_BY_NAME` 保存到 `/sys/fs/bpf` 中的同名文件，此文件保留了对 ebpf 程序的引用，因而ebpf程序不会被释放掉(即使close bpf_fd), 使用 unlink 或将 pin文件删除，则会清除pin的引用

pin 住的 ebpf 程序可以通过 `BPF_OBJ_GET` 系统调用打开，并获取其 bpf_fd

在libbpf中，对于一个声明了 `LIBPF_PIN_BY_NAME` 属性的 map， loads时不会对其进行创建，而是从文件系统中获取其描述符，随后可以基于对obj的封装与此描述符进行操作

而在ebpf程序中，同样可以访问 `LIBPF_PIN_BY_NAME` 属性的 map， 从而在 ebpf 程序之间共享数据，因为此时的ebpf程序中指向了相同的map