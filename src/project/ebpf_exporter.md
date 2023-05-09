# ebpf_exporter

ebpf_exporter 能够很方便地将 ebpf 程序接入到 prometheus 生态中，并基于容器进行部署。但是这种接入是有限制地，并非任意ebpf程序都可以被转化为 prometheus 指标进行暴露。实质上 ebpf exporter 作用在于将内核空间中的 metric 转化为用户空间中的 prometheus metric, 即 ebpf 程序中应当采用 `HASH` 类型的map, 其中 map-key 是label value, 而 map-value 则是 value, 并实现类似于 prometheus hist\guage\counter 的功能

## Arch

ebpf_exporter总体上由三部分构成
- exporter: 运行在用户态，负责主要的运行逻辑，如ebpf装载、ebpf数据读取、指标转化，指标服务等
- ebpf: 运行在内核态，附加在给定的装载点上，收集内核数据
- config: 配置描述了如何从内核中提取metrics。每个配置都有一个相应的eBPF代码，该代码在内核中运行以生成这些metric

## Config

config 中需要做的就是将 map-value 转化为对应的 metric value, 将 map-key 转换为对应的 metric label。其中， value通常类型单一，key则会比较复杂, 因此在key的解析中，引入了 decoder 来进行处理

### Metrics

在这部分字段中需要指定ebpf中的map, 并设置其转化到用户空间中的prometheus metric的配置

### Labels

此字段中需要定义 label name, 并设置 ebpf map-key 转化为 label value 的方法

**key2lables**

Lables能够将内核 map keys 转化为prometheus lable

通常来自kernel的map value 大小通常为u64, 而key的可能是u64, 也可能是一个复杂的struct

label可以通过在配置文件中指定 decoder 的方式来进行转换

对于 struct 类型的 map keys 按照以下规则进行对齐(对应类型的起始地址必须是boundary的倍数)
- u64 must be aligned at 8 byte boundary
- u32 must be aligned at 4 byte boundary
- u16 must be aligned at 2 byte boundary

**Decoders**

Decoders读入指定大小的byte slice并将其转化为字符串

`cgroup`

将 `bpf_get_current_cgroup_id` 输出的 u64 类型转换为可读的cgroup path
- `/sys/fs/cgroup/system.slice/ssh.service`

`dname`

dname 解码器能够将 string 解码为 dns 名称，如下对于label qname, 会将 key 首先转换为 string， 然后再衔接 dname 将 string 转化为 dns 名称

```yaml
- name: qname
  decoders:
    - name: string
    - name: dname
```

`inet_ip`

将ipv4(u32)地址或ipv6(u128)地址转化为字符串

`ksym`

将内核地函数址转化为函数名称

`majorminor`

> 设备可以通过 `mknod` 命令创建

将32位的 major and minor 设备号转化为与 `/dev` 中类似的设备名称
- 主设备号通常由设备的开发者指定，并且大多数情况下是静态的。主设备号的范围是0-255之间，其中0表示传统的字符设备，1表示内存设备，其他数字都可以用来表示其他类型的设备
- 次设备号则是由设备驱动程序或设备节点的创建者指定的。它用于区分同一种类的设备中的不同实例。例如，在同一台计算机上有多个磁盘驱动器，每个驱动器都属于磁盘设备类别，但它们具有不同的次设备号以便区分

`regexp`

允许用户传入正则表达式的列表，其中任何一个匹配， 则就会返回

`static_map`

静态映射, 按照设置好的映射关系进行映射，如设置 `allow_unknown`， 则不在映射关系中的 key 会转化为 `key:key`, 否则则转化为 `unknown:key`

```yaml
- name: operation
  decoders:
    - name:static_map
      allow_unknown: true
      static_map:
        1: read
        2: write
```

`string`

`uint`