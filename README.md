# What's in this fork

Manage dependencies by vcpkg by applying patches found in braft port.
The results have been committed into this fork, to build and run it skip to [Build](#build).

## Install 

Build tools. Tested with the version installed by default on Ubuntu 22.

```sh
sudo apt install cmake g++ ninja-build flex bison pkg-config
```

Package manager vcpkg.

```sh
git clone https://github.com/microsoft/vcpkg
```

Add location to where you cloned vcpkg to your profile.

```sh
export VCPKG_ROOT='~/workspace/vcpkg'
export PATH="$VCPKG_ROOT:$PATH"
```

## Patch

Apply patches found in vcpkg port of [braft](https://github.com/microsoft/vcpkg/tree/master/ports/braft) 
as listed in [portfile.cmake](https://github.com/microsoft/vcpkg/blob/master/ports/braft/portfile.cmake).

```sh
git reset --hard 8d0128e02a2959f9cc427d5f97ed730ee6a6b410
wget -O gcc_11.patch "https://github.com/baidu/braft/commit/361ef01185b88baf90b7926f992c8e71fc4aefc2.patch?full_index=1"
git apply $VCPKG_ROOT/ports/braft/fix-build.patch
git apply $VCPKG_ROOT/ports/braft/fix-dependency.patch
git apply $VCPKG_ROOT/ports/braft/export-target.patch
git apply gcc_11.patch
git apply $VCPKG_ROOT/ports/braft/fix-glog.patch
git apply $VCPKG_ROOT/ports/braft/protobuf.patch
```

## Build

You may need to change paths to where your tools are installed in CMakePresets.json.

Configure as default or release.

```sh
cmake --preset default .
```

Build.

```sh
cmake --build bld
```

---


# Overview
An industrial-grade C++ implementation of [RAFT consensus algorithm](https://raft.github.io/) and [replicated state machine](https://en.wikipedia.org/wiki/State_machine_replication) based on [brpc](https://github.com/brpc/brpc). braft is designed and implemented for scenarios demanding for high workload and low overhead of latency, with the consideration for easy-to-understand concepts so that engineers inside Baidu can build their own distributed systems individually and correctly.

It's widely used inside Baidu to build highly-available systems, such as:
* Storage systems: Key-Value, Block, Object, File ...
* SQL storages: HA MySQL cluster, distributed transactions, NewSQL systems ...
* Meta services: Various master modules, Lock services ...

# Getting Started

* Build [brpc](https://github.com/brpc/brpc/blob/master/docs/cn/getting_started.md) which is the main dependency of braft.

* Compile braft with cmake
  
  ```shell
  $ mkdir bld && cd bld && cmake .. && make
  ```

* Play braft with [examples](./example).

# Docs

* Read [overview](./docs/cn/overview.md) to know what you can do with braft.
* Read [benchmark](./docs/cn/benchmark.md) to have a quick view about performance of braft
* [Build Service based on braft](./docs/cn/server.md)
* [Access Service based on braft](./docs/cn/client.md)
* [Cli tools](./docs/cn/cli.md)
* [Replication Model](./docs/cn/replication.md)
* Consensus protocol:
  * [RAFT](./docs/cn/raft_protocol.md)
  * [Paxos](./docs/cn/paxos_protocol.md)
  * [ZAB](./docs/cn/zab_protocol.md)
  * [QJM](./docs/cn/qjm.md)

# Discussion

* Add Weixin id ***zhengpf__87*** or ***xiongk_2049*** with a verification message '**braft**', then you will be invited into the discussion group. 
