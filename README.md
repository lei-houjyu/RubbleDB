# RubbleDB
Eliminate compaction jobs in secondary nodes within a group of replicated RocksDB.

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
