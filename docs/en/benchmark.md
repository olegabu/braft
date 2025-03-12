

# Preface

Almost all frameworks, modules, and libraries use high performance as one of the most important labels (of course, Braft is no exception). However, developers often only understand performance based on throughput or QPS numbers, and performance testing becomes a game of trying to brush numbers without considering whether the scenario meets the actual application scenario. There are two common methods for "improving performance numbers":

* **Batch:** The system actively waits for the number of requests to reach a certain number or a timeout period, and then combines a request into a request and sends it to the backend system. Depending on the value of batch_size / request_size, it can increase "QPS" by dozens to hundreds of times. (This will be explained in detail later).
* **Client unlimited asynchronous sending**: This allows the system to be in a high-load state all the time, without any waiting, and avoids some system calls and thread synchronization overhead.

Although these designs can produce good benchmark data, they are completely out of touch with the actual application scenarios. Taking batch as an example, the problem with batch is that it does not essentially provide guidance on how to improve the system's QPS. In performance testing, concurrency is usually very high, and there will be a steady stream of requests coming in, so each request does not need to wait for a long time to meet the batch size conditions. However, in real scenarios, concurrency is not that high, which means that each request must "wait" for a set value, and latency cannot reach the optimal solution. At this time, engineers are often immersed in optimizing timeouts, batch sizes, and other parameter adjustments, thereby ignoring truly meaningful things such as analyzing system bottlenecks. In addition, with the gradual improvement of hardware performance, the delays of the network and disk themselves are getting shorter and shorter. This mechanism cannot take into account both the delay under low load and the throughput under high load.

In braft, we mainly use the following methods to improve performance:

- The data flow is fully concurrent. The leader writes to the local disk and copies data to the followers in parallel.
- Improve locality as much as possible and give full play to the role of cache at different levels
- Isolate access to different hardware as much as possible and improve throughput through pipelining
- Reduce the size of the critical lock section as much as possible, and use lock-free/wait-free algorithms on the critical path.

# Test scenario

The performance of different user state machines varies greatly. In this scenario, we first remove the influence of the user state machine. The client starts multiple threads to write a certain size of log to the server through synchronous RPC. The server calls the libraft interface to submit the log to the replication group. When the server receives the event that the raft log is successfully submitted, it replies to the client and collects the throughput and latency data on the client. The comparison group is fio writing to the local disk in sequence with the same number of threads. Through such tests, we hope to see how much performance data the application will sacrifice in exchange for higher data security after using raft compared to writing to the local disk.

If there is no special instruction, the default number of client threads is 100, and fsync is enabled by default when writing files.

## vs FIO (2016.3)

fio test command (where -bs=xxx will be replaced with the size corresponding to the test):

```
./fio -filename=./10G.data -iodepth 100 -thread -rw=write -ioengine=libaio --direct=1 -sync=1 -bs=512 -size=10G -numjobs=1 -runtime=120 -group_reporting -name=mytest
```

 

**Hardware Information**

CPU:  12核， Intel(R) Xeon(R) CPU E5-2620 v2 @ 2.10GHz  

Disk: LENOVO SAS 300G, random write IOPS~=800, sequential write throughput~=200M/s

Network card: 10G network card, multi-queue is not enabled

Number of copies: 3

![img](../images/benchmark0.png)

## vs Logcabin(2017.2)

Since there are few standalone RAFT implementations in the industry, we made some adjustments to Logcabin, added an rpc service, and replied to the client immediately after the log synchronization was completed (the patch part will be open source soon). We also counted the QPS on the client side. The specific data is shown in the figure below.

![img](../images/benchmark.png)