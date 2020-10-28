# Develop Log
A log.

Ordered Goal:
1. Easy to use.
2. Simple and elegant, less code (For write and understand).
3. Open for further modification or integration.

Tech:
- python? go?

## TODOs
- [x] Read tiup
- [x] Use python or go?
  - Go.
- Starter.
  - [x] Write a starter based on `tiup`.
- Restart.
  - [x] Find a way to restart.
  - [x] whether tidb control can restart a tikv process?
- Partition.
  - [ ] Write small program to validate `etcd/pkg/proxy`

## 201027
## A minor cli program with cobra
How to use cobra: https://towardsdatascience.com/how-to-create-a-cli-in-golang-with-cobra-d729641c7177

## a starter based on `tiup`
Just a simple wrap to `tiup`. This function is built within tiup. So whether we can make it a component of `tiup` in the future?

## Restart a tikv instance
`tiup cluster restart`

Can use command `tiup cluster stop test --node 10.0.2.15:20160`. Now the last problem is to build a proxy.

## tiup playground don't have a cluster name.
https://docs.pingcap.com/tidb/dev/tiup-cluster
> Similar to the TiUP playground component used for local deployment, the TiUP cluster component quickly deploys TiDB for production environment. Compared with playground, the cluster component provides more powerful cluster management features, including upgrading, scaling, and even operation and auditing.

## Proxy building
Refer to test of `proxy`.

- [x] `proxy.Server` always need a `From` address. How to discard packages from all destination to `advertise-addr`?

0.0.0.0 Represents any address.

Understand the inside to use it properly, read `listenAndServe()` in `server.go` first.

## 201026
## How to restart a tikv server?
Might be on the dashboard.

Maybe tidb control can do. https://docs.pingcap.com/zh/tidb/dev/tikv-control

Restart by hand:

Kill the process of tikv. Then restart by hand. It works.

Restart command is like below. The path is `/tidb-deploy/tikv-20161`, 20161 is port number.

```bash
bin/tikv-server --addr 0.0.0.0:20161 --advertise-addr 10.0.2.15:20161 --status-addr 0.0.0.0:20181 --pd 10.0.2.15:2379 --data-dir /tidb-data/tikv-20161 --config conf/tikv.toml --log-file /tidb-deploy/tikv-20161/log/tikv.log
```

Ok. Build a 'bare hand' version with system commands.

## Proxy.
Here comes proxy. 

All communication to a tikv instance is through `--advertise-addr`. So if we could interact package delivered to this port, the work are done.

## 201025
## Restart instance & What is `instances`?
> use "tiup list" to fetch the latest components manifest

`Tiup` is made up of components. Maybe we can develop this as a component?

```
cs18@games101vm:~/Code/PingCAP_Internship/MinorChallenge$ tiup list
Available components:
Name               Owner    Description
----               -----    -----------
alertmanager       pingcap  Prometheus alertmanager
bench              pingcap  Benchmark database with different workloads
blackbox_exporter  pingcap  Blackbox prober exporter
br                 pingcap  TiDB/TiKV cluster backup restore tool
cdc                pingcap  CDC is a change data capture tool for TiDB
client             pingcap  A simple mysql client to connect TiDB
cluster            pingcap  Deploy a TiDB cluster for production
ctl                pingcap  TiDB controller suite
dm                 pingcap  Data Migration Platform manager
dm-master          pingcap  dm-master component of Data Migration Platform
dm-worker          pingcap  dm-worker component of Data Migration Platform
dmctl              pingcap  dmctl component of Data Migration Platform
drainer            pingcap  The drainer componet of TiDB binlog service
dumpling           pingcap  Dumpling is a CLI tool that helps you dump MySQL/TiDB data
grafana            pingcap  Grafana is the open source analytics & monitoring solution for every database
insight            pingcap  TiDB-Insight collector
node_exporter      pingcap  Exporter for machine metrics
package            pingcap  A toolbox to package tiup component
pd                 pingcap  PD is the abbreviation for Placement Driver. It is used to manage and schedule the TiKV cluster
pd-recover         pingcap  PD Recover is a disaster recovery tool of PD, used to recover the PD cluster which cannot start or provide services normally
playground         pingcap  Bootstrap a local TiDB cluster
prometheus         pingcap  The Prometheus monitoring system and time series database
pump               pingcap  The pump componet of TiDB binlog service
pushgateway        pingcap  Push acceptor for ephemeral and batch jobs
tidb               pingcap  TiDB is an open source distributed HTAP database compatible with the MySQL protocol
tidb-lightning     pingcap  TiDB Lightning is a tool used for fast full import of large amounts of data into a TiDB cluster
tiflash            pingcap  The TiFlash Columnar Storage Engine
tikv               pingcap  Distributed transactional key-value database, originally created to complement TiDB
tiup               pingcap  TiUP is a command-line component management tool that can help to download and install TiDB platform components to the local system
```

So much components!

## Deploy a tidb cluster.
```bash
tiup cluster deploy test v4.0.7 ./topo.yaml --user root -p
tiup cluster list
```

## 201024
### tiup
What is `tiup`?
- [doc](https://github.com/pingcap/tiup/blob/master/doc/user/README.md)
- [Quick Start Guide for the TiDB Database Platform](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb)

It's written by go.

tryout:
```bash
curl --proto '=https' --tlsv1.2 -sSf https://tiup-mirrors.pingcap.com/install.sh | sh
# In my zshrc:
# export PATH=/Users/ofey/.tiup/bin:$PATH
```

Error: requires cgo or $USER
```bash
(base) λ ~/Code/PingCAP_Internship/MinorChallenge/ master* tiup playground
panic: cannot get current user information: user: Current requires cgo or $USER set in environment

goroutine 1 [running]:
github.com/pingcap/tiup/pkg/localdata.InitProfile(0x10)
        github.com/pingcap/tiup@/pkg/localdata/profile.go:60 +0x290
github.com/pingcap/tiup/pkg/environment.Mirror(0x1bde380, 0xc00007f900)
        github.com/pingcap/tiup@/pkg/environment/env.go:45 +0x34
github.com/pingcap/tiup/cmd.newMirrorPublishCmd(0xc00043c840)
        github.com/pingcap/tiup@/cmd/mirror.go:297 +0x6b
github.com/pingcap/tiup/cmd.newMirrorCmd(0xc0000f0840)
        github.com/pingcap/tiup@/cmd/mirror.go:81 +0x1ab
github.com/pingcap/tiup/cmd.init.0()
        github.com/pingcap/tiup@/cmd/root.go:133 +0x4d0
```

ohhhh... OSX might not be okay.

Switch to Linux.

## How to start clusters?
> - 一键启动集群可以参考或基于 tiup playground https://github.com/pingcap/tiup/tree/master/components/playground

Refer to the implementation of tiup playground. `tiup` use `cobra` as cli library.

doc: https://github.com/spf13/cobra

Can we implement the required functions based on current softwares?

> 启动集群：cli start cluster --tikv.path bin/tikv-server --tikv.count 3

## etcd
> etcd is a distributed reliable key-value store for the most critical data of a distributed system, with a focus on being
> 
> - 模拟网络分区可以参考或基于 https://github.com/etcd-io/etcd/blob/master/pkg/proxy

`etcd/pkg/proxy` is a independent package.
> pkg/ is a collection of utility packages used by etcd without being specific to etcd itself. A package belongs here only if it could possibly be moved out into its own repository in the future.

```go
// Server defines proxy server layer that simulates common network faults:
// latency spikes and packet drop or corruption. 
```
https://github.com/etcd-io/etcd/blob/8fc5ef4a039c657da90834b64230cc61caa68b7c/pkg/proxy/server.go#L50

We can reuse this to build a prototype. Set up a proxy to control traffic to certain instances.

