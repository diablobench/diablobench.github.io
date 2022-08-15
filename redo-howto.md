---
title:  "How To Reproduce Results"
layout: default
permalink: /redo-howto
---

### How-To Reproduce the Results

##### Requirements (human time: 10 minutes)

 * Install VirtualBox v?
 * Restore the provided image (with diablo, perl, minion, blockchains)
 * Storage space: 10 GB or less
 * Memory: 8 GB
 * CPUs: 4 cores

##### Run (human time: 1 minute / machine time: 3 minutes)

Use the ```eurosys``` binary to run minion locally and on a previously allocated static set of machines.

```setup.txt``` indicates how to ssh into the machine and how many secondaries (secondary) and blockchain nodes (chain) to use. 
```
vagrant@127.0.0.1 = primary,secondary,chain,chain,chain,chain
```

The workoad file ```workload-100.yaml``` simply sends 100 transactions per second during 2 minutes.
```
./bin/eurosys workload-100.yaml setup.txt
```
It effectively runs indirectly the benchmarks (at least 30 seconds is needed for Ethereum to produce enough blocks):
```
diablo primary -vvv --port=5000 \
--env="accounts=accounts.yaml" \
--env="contracts=dapps-directory" \
--output=results.json --compress --stat \
1 setup.yaml workload.yaml

diablo secondary -v --tag="local" \
--port=5000 127.0.0.1
```
as decribed in [Diablo-v2](https://infoscience.epfl.ch/record/294268?ln=en)

All the workloads are located in the minion directory of the image.
Some involve smart contract: workload-amazon.yaml

##### Collect results (human time: / machine time: )

The execution creates an archive file in the minion folder of the name:
```
[blockchain_name]-[region_number]-[#secondaries]-[#blockchain_nodes]-[workload_name]-[timestamp].results.tar.gz
```
In the archive, you can find the primary standard and error outputs.
In the std output, you can find, the aggregated results:
  * submit number: the number of transactions that were requested
  * commit number: the number of transactions that were committed successfully
  * abort number: the number of failed transactions
  * average load: the average rate at which the secondaries send transactions to blockchain nodes
  * average throughput: the average throughput performance at which the blockchain nodes treat transactions
  * average latency: the average time it takes for blockchain nodes to commit a transaction
  * median latency: the median time it takes for blockchain nodes to commit a transaction

### Install yourself (human time: / machine time: )

 * download minion 
 * download perl 
 * download diablo 
 * build diablo 
 * download all blockchains
 * build all blockchains
