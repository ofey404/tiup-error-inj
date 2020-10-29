# Minor task for Pingcap Internship Interview

基于 `tiup` 进行开发，代码存放在[ofey404/tiup-error-inj2](https://github.com/ofey404/tiup-error-inj2)

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
使用 Golang 来编写。可以利用现有的模块。

说到隔离某个实例，最先想到的功能肯定是 iptables。隔离某个实例，就创建对应的规则。可以使用 --owner 参数来指定对应的进程，参照 [iptable Tutorial](https://www.frozentux.net/iptables-tutorial/iptables-tutorial.html#OWNERMATCH)。

不过它是内核级功能，普通的 go 程序不能像它一样直接控制经过某个端口的包，也不能抢占端口。

> 让一个 tikv 实例连接不上其它 tikv 实例的 advertise-addr

Proxy server 做到的事情是端口转发+处理。

结合 proxy 的特性和隔离的要求，可以做出以下设计：

```plain text
+-----------------------+                                         +--------------------------+
|                       |                                         |                          |
|            +----------+                                         +---------+                |
|            |          <-----------------------------+           |         |                |
|            |Advertise |                             |           |Advertise|                |
|            |Addr      +-------------+               |           |Addr     |                |
|            |          |    +--------v-------+       |           |         |                |
|            +----------+    |                |       |           +---------+                |
|                       |    | Proxy Server 1 |       |           |                          |
|                       |    |                |       |           |                          |
|  tikv1     +----------+    +--------+-------+       |           +--------+      tikv2      |
|            |          |             |               |           |        |                 |
|            |  Listen  |             |               |           | Source |                 |
|            |  Port    +<------------+               +-------------Port   |                 |
|            |          |                                         |        |                 |
|            +----------+                                         +--------+                 |
|                       |                                         |                          |
|                       |                                         |                          |
|            +----------+                                         |                          |
|            |          |                                         +----------+               |
|            |  Source  |                                         |          |               |
|            |  Port    |                                         |  Listen  |               |
|            |          |                                         |  Port    |               |
|            +----------+                                         |          |               |
|                       |                                         +----------+               |
|                       |                                         |                          |
|                       |                                         |                          |
+-----------------------+                                         +--------------------------+
```

在启动每一个实例的时候，指定一个对应的 advertise-addr，同时启动一个 Proxy server 将 advertise-addr 的流量转发到真实的 Listen Port。

初始 proxy 可以设置成直接转发，执行`partition`等命令时，则修改 proxy server 的设置。

这个设计还可以更精细地控制错误的类型，因为设置的每一个 proxy 相当于完全控制了这条连接的延迟、丢包等信息。

proxy server 程序的性质：
- 需要在集群启动期间保持运行。
- 执行 `partition` 命令时，修改后台的 proxy server 的状态。
- 最好能在集群关闭时自动停止。

参照 playground 的实现：
- 不需要考虑持久化。
- 所有实例在同一个单机上运行，不需要考虑将 proxy 运行在多台机器上（来均衡负载）的情况。

start, restart 等功能和 playground 高度重合，可以考虑为 playground 增加一个子命令/flag 这种实现方式。

### partition 命令
- [ ] 如何更改 Proxy 设置？
  - 在执行 partition 命令的时候，用进程间通信方式告知对应的 Proxy Server。服务没有中断时间，但实现较为复杂。
  - 或者直接关闭 Proxy Server 然后用新的配置重启一次。实现简单，但是下线/上线之间服务会中断，不利于频繁/精细的调节网络状况。

## Milestone

- 10.24 - 10.25 日：全天满课……
- 10.25 - 10.27 日：阅读必要的文档，实验了一些功能，详见[开发日志](./develop_log.md)
- 10.27 - 10.29 日：阅读 `tiup` 的代码，在其基础上实现功能。代码存放在[tiup-error-inj2](https://github.com/ofey404/tiup-error-inj2)