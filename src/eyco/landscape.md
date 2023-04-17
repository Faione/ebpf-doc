# landscape

[bpf_tutorial](https://github.com/eunomia-bpf/bpf-developer-tutorial)郑昱笙老姐的教程，主要讲的是libbpf的使用方法，非常不错

[bolipi](https://www.bolipi.com/ebpf/index)国内活跃的ebpf社区，提供一些入门教程，关键是还有一些企业项目的路标，适合有一定学习基础后上手项目

[ebpf_slide](https://github.com/gojue/ebpf-slide)中包含有大量ebpf相关的slide, 且有分类，初学者可以参考`eBPF基础知识`板块进行学习

[kern_ebpf_doc](https://www.kernel.org/doc/html/latest/bpf)是内核中的ebpf介绍, 包括help functions，prog type, map type 等介绍，可以协助进行ebpf的开发

[ebpf_top](https://www.ebpf.top/)中收集了大量ebpf相关的博文翻译，其中大部分博文都来自于[facebook](https://facebookmicrosites.github.io/bpf/)

## bcc

[bcc](https://github.com/iovisor/bcc)是当前最流行的ebpf开发框架，全称是bpf compiler collection, 提供高度封装的api来辅助ebpf程序的开发，并在运行时编译，由于封装了大量 ebpf程序开发，编译，加载的内容，极大降低了ebpf程序的开发流程，但是相应的灵活性也有所降低，同时由于需要在运行时进行编译，不可避免的目标机器上的编译器环境支持

[bcc-tools](https://github.com/iovisor/bcc/blob/master/docs/tutorial.md)中介绍了使用bcc实现的一套工具，用于对linux进行性能分析，同时对应的源码也是学习bcc开发模式的很好教程

[bcc-libbpf-tools](https://github.com/iovisor/bcc/tree/master/libbpf-tools)是基于libbpf实现的一套bcc工具，相比于libbpf官方仓库，提供了更丰富的example, 学习libbpf时可以参考此仓库中的内容

## libbpf

[libbpf_bootstrap](https://github.com/libbpf/libbpf-bootstrap)是基于libbpf的ebpf开发框架，主要理念是CO-RE，相比于bcc, 无需在目标机器上安装编译器，但考虑版本之间内核的区别，往往只能CO-RO, 依赖较新版本的内核特性

框架核心在于提供ebpf kern和user程序之间沟通的接口，如装载ebpf程序，设置ebpf程序中的变量，获取ebpf kern程序的输出，编译ebpf程序并编码到user程序中等

开发者需要完成 ebpf kern 程序和 user 程序中的主要逻辑的开发，其中 ebpf kern 程序的开发需要遵守对应版本内核的规范，可以参考内核代码中的 `tools/testing/selftests/bpf`, `samples/bpf`中的代码， 以及bcc代码仓库中的libpf-tools

开发ebpf kern程序要求对kernel有一定了解， 而开发 ebpf user 程序需要了解 libbpf 中的接口，主要包括如何加载ebpf代码，如何获取ebpf kern代码中的maps，如何从event map中读取数据进行处理等

libbpf-rs提供了rust风格的libbpf接口，从而能够使用rust进行ebpf user程序的开发，开发流程与c一致。[libpf-example](https://github.com/libbpf/libbpf-bootstrap/tree/master/examples)中提供了rust/c的示例程序，[libbpf-rs-example](https://github.com/libbpf/libbpf-rs/tree/master/examples)中还有额外的rust示例程序

## aya

[aya](https://aya-rs.dev/)与libbpf-rs类似，都以CO-RE为目标，但提供原生rust的ebpf kern程序开发，并通过LLVM编译为ebpf程序。aya实质是一种rust原教旨主义的开发框架，考虑到当前rust在kernel仍然处于起步阶段，因此实际开发并不方便，表现为极少example(aya提供), prog类型支持不完全等, 只能说前景非常巨大