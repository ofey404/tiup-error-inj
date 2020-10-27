# Minor task for Pingcap Internship Interview

## Task Description
> 小作业：实现一个工具，支持在本地一键启动一个 “支持动态错误注入的 TiDB 集群” 用于测试，同时希望工具比较易用。
> 
> 工具可以接受的输入参数：
> - pd/tidb/tikv 二进制文件路径
> - pd/tidb/tikv 实例数
> - 支持以下错误注入的功能
>   - 重启一个 tikv 实例
>   - 给一个 tikv 实例制造网络分区（比如：让一个 tikv 实例连接不上其它 tikv 实例的 advertise-addr）
> 
> 举个例子，你可以实现一个命令行工具
> ```
> 启动集群：cli start cluster --tikv.path bin/tikv-server --tikv.count 3
>
> 重启 tikv-0 实例：cli restart tikv-0
>
> 隔离 tikv-0 实例：cli partition tikv-0
> ```
> 
> 一些参考资料：
> - 一键启动集群可以参考或基于 tiup playground https://github.com/pingcap/tiup/tree/master/components/playground
> - 模拟网络分区可以参考或基于 https://github.com/etcd-io/etcd/blob/master/pkg/proxy
> - tikv advertise-addr 的概念可以考虑 https://docs.pingcap.com/zh/tidb/dev/command-line-flags-for-tikv-configuration#--advertise-addr

## Design


## Milestone
