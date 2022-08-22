---
title:  "How To Reproduce Results"
layout: default
permalink: /redo-howto
---

### How-To Reproduce the Results

This page explains how to reproduce the experiments of the [Diablo-v2 paper](https://infoscience.epfl.ch/record/294268?ln=en) with 
limited resources as depicted in [this screencast](https://nextcloud.in.tum.de/index.php/s/beDCpoE4cq9KdH4).
For the sake of simplicity, we illustrate how to run the experiments on a single local machine with multiple docker containers.
As we show this is sufficient to run small workloads of native transfers and smart contracts for Algorand, Avalanche, Diem, Ethereum, 
Quorum. However, [Solana requires at least 128GB of memory per validator node](https://docs.solana.com/running-validator/validator-reqs) 
and cannot be tested in this simplified setting.

##### Requirements (human time: 2 minutes, machine time: 1h30min depending on the network bandwidth)

 * Memory: 8 GB
 * CPUs: 4 cores
 * Storage space: 8 GB
 * x64 CPU (not ARM for compatibility with VirtualBox)
 * Install VirtualBox
 * Download the [OVA image](https://nextcloud.in.tum.de/index.php/s/RDy4Df3x9JTsLGG) (with ubuntu, [diablo](https://github.com/NatoliChris/diablo-benchmark/), perl, [minion](https://github.com/gauthier-voron/minion), all 6 blockchains)

##### Run (human time: 7 minutes / machine time: 19 minutes)

Start the image with VirtualBox with login/password:

In the home directory, you can find 
 * ```minion```, the deployment scripts to run the blockchain network, diablo and collect the results
 * ```install```, contains the pre-build binaries of the blockchain protocols and the diablo benchmark
 * ```scripts```, the scripts used to examine the collected results

Move to the minion directory:
```bash
cd minion
```

We use the ```eurosys``` binary with the ```--skip-install``` argument since we have all the binaries in the provided image.
(To do a fresh installation, please refer to the last section at the bottom of this page.)
The ```eurosys``` binary runs minion locally and on a previously allocated static set of machines ```setup.txt``` with a specified workload.

The ```setup.txt``` file indicates how to ssh into the machines, the number of ```primary``` and ```secondary``` nodes on 
which diablo executes and the number of ```chain``` nodes on which to deploy the blockchain.
```
vagrant@127.0.0.1 = primary,secondary,chain
```

###### Native transfers (human time: 4 minutes / machine time: 9 minutes)
We first execute a ```native``` transfer workload on each blockchain. This workload is specified in the ```workload-native-10.yaml``` file, 
which sends 10 transactions per second during 30 seconds:
```bash
./bin/eurosys --skip-install workload-native-10.yaml setup.txt
```

###### Smart contracts  (human time: 3 minutes / machine time: 10 minutes)
We then execute a smart ```contract``` workload on each blockchaun. This workload is specified in the ```workload-contract-10.yaml``` file, which 
lasts for 30 seconds and sends 10 invocations of the ```buy``` function of a smart contract (representing the NASDAQ Microsoft shares) per second:
```bash
./bin/eurosys --skip-install workload-contract-10.yaml setup.txt
```

The workoad file ```workload-100.yaml``` simply sends 100 transactions per second during 2 minutes.
```bash
./bin/eurosys workload-100.yaml setup.txt
```
It effectively runs indirectly the benchmarks (at least 30 seconds is needed for Ethereum to produce enough blocks):
```bash
diablo primary -vvv --port=5000 \
--env="accounts=accounts.yaml" \
--env="contracts=dapps-directory" \
--output=results.json --compress --stat \
1 setup.yaml workload.yaml
```
```bash
diablo secondary -v --tag="local" \
--port=5000 127.0.0.1
```
as decribed in [Diablo-v2](https://infoscience.epfl.ch/record/294268?ln=en)

All the workloads are located in the minion directory of the image.
Some involve smart contract: workload-amazon.yaml

##### Collect results (human time: 3 minutes / machine time: 0 minute)

The execution creates one archive file per pair of blockchain and workload in the minion folder of the name:
```
[blockchain_name]-[region_number]-[#secondaries]-[#blockchain_nodes]-[workload_name]-[timestamp].results.tar.gz
```

You can inspect the output, by unarchiving the file:
```bash
tar xf0 algorand-1-1-1-native-10_2022-08-21-22-48-58.results.tar.gz algorand-1-1-1-native-10_2022-08-21-22-48-58.results/
```
This extracts several files indicating the ```topology```, the ```setup```, the ```workload```, the blockchain ```name```, as well as the standard
and error outputs of each ```primary```, ```secondary``` and ```chain``` nodes. 
In the diablo ```primary``` standard output, we can see the following aggregate information:
  * submit number: the number of transactions that were requested
  * commit number: the number of transactions that were committed successfully
  * abort number: the number of failed transactions
  * average load: the average rate at which the secondaries send transactions to blockchain nodes
  * average throughput: the average throughput performance at which the blockchain nodes treat transactions
  * average latency: the average time it takes for blockchain nodes to commit a transaction
  * median latency: the median time it takes for blockchain nodes to commit a transaction

You can then move the result files to the folder ```scripts/results/{native,contract}``` for analysis.
```bash
mv *native*.tar.gz ~/scripts/results/native/
mv *contract*.tar.gz ~/scripts/results/contracts/
```
This way we can convert the ```results``` with submit, commit, abort times from a JSON format to a CSV formatted file ```results.csv```:
```bash
cd ~/scripts
./csv-results results results.csv
```
On each line, we can see the results of an archive containing performance results for each blockchain. 

### Install yourself (human time: ? / machine time: ?)

First, we need to install the following dependencies:
 * download minion 
 * download perl 
 * download diablo 
 * build diablo 
 * download all blockchains
 * build all blockchains

Note that the diablo installation script ```script/remote/linux/apt/install-diablo```
points to a specific version of diablo:
```bash
diablo_url = https://github.com/NatoliChris/diablo-benchmark.git
diablo_checkout = 'aec'
```


