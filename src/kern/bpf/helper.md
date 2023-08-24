# helpers

helper函数[^1]形式类似于系统调用，是内核暴漏给bpf程序的特殊函数，考虑到bpf程序运行在内核态，因而不会有上下文切换的开销，实际与kernel module中调用kernel函数类似，但由于调用局限在给定的范围内，因而能够在提供灵活性的同时，保证ebpf程序执行的安全性

[^1]: [bpf_helpers](https://elixir.bootlin.com/linux/latest/source/kernel/bpf/helpers.c)
[^2]: [bpf_helper_doc](https://man7.org/linux/man-pages/man7/bpf-helpers.7.html)


## bpf_get_current_cgroup_id

获取当前task所属cgroup的cgroup_id
- cgroup_id 出现在 cgroup v2 中，实际为cgroup对应 kfs node 的id

## bpf_get_current_pid_tgid

获取当前cpu上执行任务的 pid, tgid

``` c
/*
 * bpf_get_current_pid_tgid
 *
 * 	Get the current pid and tgid.
 *
 * Returns
 * 	A 64-bit integer containing the current tgid and pid, and
 * 	created as such:
 * 	*current_task*\ **->tgid << 32 \|**
 * 	*current_task*\ **->pid**.
 */
static __u64 (*bpf_get_current_pid_tgid)(void) = (void *) 14;
```

在 Linux 系统中，进程 ID（PID）和线程组 ID（TGID）是两个不同的概念
- 进程 ID（PID）：每个正在运行的进程都有一个唯一的 ID。虽然在某些情况下 PID 可以重复使用（例如，当一个进程终止时，其 PID 可能会立即被新进程重用），但通常情况下每个进程都具有唯一的 PID。PID 用于在系统内部标识、管理和跟踪进程。
- 线程组 ID（TGID）：线程组 ID 是所有线程的唯一标识符。与进程不同，Linux 系统将线程视为进程和线程组之间的一种关系，因此 TGID 表示当前线程所属的进程或线程组的 ID。也就是说，在单线程程序中，PID 和 TGID 是相同的。但在多线程程序中，每个线程都有自己的 ID，但它们共享相同的 TGID

总结来说：
- 每个进程都有一个唯一的 PID。
- 所有线程共享相同的 TGID。
- 在单线程程序中，PID 和 TGID 相同。
- 在多线程程序中，每个线程都有自己的 PID，但它们共享相同的 TGID

通常意义上的 pid 为 tgid

```C
__u64 pid_tgid = bpf_get_current_pid_tgid();
__u32 tgid = pid_tgid >> 32;
__u32 pid = pid_tgid;
```

## bpf_get_current_uid_gid

获取当前用户的 uid 和 gid

```C
/*
 * bpf_get_current_uid_gid
 *
 * 	Get the current uid and gid.
 *
 * Returns
 * 	A 64-bit integer containing the current GID and UID, and
 * 	created as such: *current_gid* **<< 32 \|** *current_uid*.
 */
static __u64 (*bpf_get_current_uid_gid)(void) = (void *) 15;
```