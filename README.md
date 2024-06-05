# RubbleDB: CPU-Efficient Replication with NVMe-oF
## Abstract
Due to the need to perform expensive background compaction operations, the CPU is often a performance bottleneck of persistent key-value stores. In the case of replicated storage systems, which contain multiple identical copies of the data, we make the observation that CPU can be traded off for spare network bandwidth. Compactions can be executed only once, on one of the nodes, and the already-compacted data can be shipped to the other nodes’ disks, saving them significant CPU time. In order to further drive down total CPU consumption, the file replication protocol can leverage NVMe-oF, a networked storage protocol that can offload the network and storage datapaths entirely to the NIC, requiring zero involvement from the target node’s CPU. However, since NVMe-oF is a one-sided protocol, if used naively, it can easily cause data corruption or data loss at the target nodes.

We design RubbleDB, the first key-value store that takes advantage of NVMe-oF for efficient replication. RubbleDB introduces several novel design mechanisms that address the challenges of using NVMe-oF for replicated data, including pre-allocation of static files, a novel file metadata mapping mechanism, and a new method that enforces the order of applying version edits across replicas. These ideas can be applied to other settings beyond key-value stores, such as distributed file and backup systems. We implement RubbleDB on top of RocksDB and show it provides consistent CPU savings and increases throughput by up to 1.9× and reduces tail latency by up to 93.4% for write-heavy workloads, compared to replicated key-value stores, such as ZippyDB, which conduct compactions on all replica nodes.

## Setup
### 0. Overview
We have prepared a script `rocksdb/rubble/setup.sh` to automatically set up everything on a CloudLab experiment. To use this script, please create an experiment with `r650` or `r6525` nodes as workers. The number of worker nodes equals to the replication factor (RF). For example, if you would like to run experiments under RF=3, three `r650` or `r6525` nodes are needed. Besides, we need to add a client node to the experiment of any type. For simplicity, we use the the same type `r650` for both worker and client nodes to show the case of RF=3 below.

### 1. Create a CloudLab experiment
Initiate a CloudLab experiment from [here](https://www.cloudlab.us/instantiate.php). The specific exepriment configuration is:
- Profile: `small-lan`
- Number of Nodes: `4`
- Select OS image: `UBUNTU 20.04`
- Optional physical node type: `r650`

Once the experiment is ready, you will see a status page as below showing the ID and hostname for each node. For example, `node0`'s hostname is `clnode257.clemson.cloudlab.us`.

![Experiment Example](./assets/experiment.jpg)

### 2. Set up the environment remotely
Run the `setup.sh` script **_from your laptop_** to handle all the burden like NVMe-oF configuration, dependency installation, and code compilation:

```bash
bash setup.sh [your_cloudlab_username] [is_mlnx] [shard_num] [hostnames]
```

- is_mlnx: 0 or 1 to indicate wheather the NIC is Mellanox or not. Both `r650` and `r6525` uses Mellanox NICs, so use 1 here.
- shard_num: number of data partition in RubbleDB. Since we would like to evenly spread the workload on all nodes, use a multiple of RF here, e.g., 3, 6, or 9.
- hostnames: the hostname for each node we see in the experiment status, e.g., `clnode257.clemson.cloudlab.us`.
  
As an example, the complete command for a three-shard experiment is:
```bash
bash setup.sh haoyuli 1 3 clnode257.clemson.cloudlab.us clnode258.clemson.cloudlab.us clnode271.clemson.cloudlab.us clnode256.clemson.cloudlab.us
```

This scripts take ~30 minutes to finish. If everything goes smoothly, you will see a message `Done!` printed in the end.

## Run
If you follow the above steps, `node0` will be the client, where we can initiate the evaluation. Now we need to ssh into `node0`:

```bash
ssh haoyuli@clnode257.clemson.cloudlab.us
sudo su
cd /root/YCSB/
bash eval.sh [phase] [workload] [qps] [log_suffix] [client_num] [mode] [shard_num] [replication_factor]
```
- phase: `load` or `run`, corresponding to the load or run phase in YCSB
- workload: a-g, corresponding to YCSB workload a-g
- qps: request per second
- log_suffix: the log will be saved as ycsb-[log_suffix].log
- client_num: the number of parallel YCSB client threads
- mode: `rubble` or `baseline`
- shard_num: same as the previous definition
- replication_factor: same as above

To run a loading phase followed by running workload A:

```bash
bash eval.sh load a 100000 rubble-offload-load 4 rubble 3 3
bash eval.sh run a 80000 rubble-offload-a 4 rubble 3 3
```

## Citation
```
@inproceedings{li2023rubbledb,
  title={$\{$RubbleDB$\}$:$\{$CPU-Efficient$\}$ Replication with $\{$NVMe-oF$\}$},
  author={Li, Haoyu and Jiang, Sheng and Chen, Chen and Raina, Ashwini and Zhu, Xingyu and Luo, Changxu and Cidon, Asaf},
  booktitle={2023 USENIX Annual Technical Conference (USENIX ATC 23)},
  pages={689--703},
  year={2023}
}
```
