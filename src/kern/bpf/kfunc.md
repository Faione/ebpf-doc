# KFuncs

与 bpf helper 类似，但 kfuncs 不提供接口的稳定性，即意味着会跟随着 kernel 版本变化而变化，开发者可以使用已有的 kfuncs 如 core kfuncs， 也可以根据需要自定义 kfuncs 来暴露给 bpf 程序使用

[^1]: [kfuncs](https://docs.kernel.org/bpf/kfuncs.html)
[^2]: [core_kfuncs](https://docs.kernel.org/bpf/kfuncs.html#core-kfuncs)