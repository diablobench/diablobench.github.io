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
Quorum. However, [Solana typically requires at least 128GB of memory per validator node](https://docs.solana.com/running-validator/validator-reqs).
Have a look at another [documentation](fresh-install) for a fresh install.

##### Requirements (human time: 2 minutes, machine time: 1h30min depending on the network bandwidth)

 * Install VirtualBox
 * x64 CPU (not ARM for compatibility with VirtualBox)
 * Memory allocated to VirtualBox: 8 GB
 * vCPUs allocated to VirtualBox: 4 cores
 * Storage space  allocated to VirtualBox: 10 GB
 * Download the [OVA image](https://nextcloud.in.tum.de/index.php/s/RDy4Df3x9JTsLGG) (with ubuntu, [diablo](https://github.com/NatoliChris/diablo-benchmark/), perl, [minion](https://github.com/gauthier-voron/minion), all 6 blockchains)

##### Run (human time: 7 minutes / machine time: 19 minutes)

Start the image with VirtualBox with login/password:

In the home directory, you can find 
 * ```minion```, the deployment and workload files to run the blockchain network, diablo and to collect the results
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
We then execute a smart ```contract``` workload on each blockchain. This workload is specified in the ```workload-contract-10.yaml``` file, which 
lasts for 30 seconds and sends 10 invocations of the ```buy``` function of a smart contract (representing the NASDAQ Microsoft shares) per second:
```bash
./bin/eurosys --skip-install workload-contract-10.yaml setup.txt
```

The ```eurosys``` binary effectively runs the benchmarks with the commands of [Diablo-v2](https://infoscience.epfl.ch/record/294268?ln=en):
```bash
diablo primary -vvv --port=5000 \
--env="accounts=accounts.yaml" \
--env="contracts=dapps-directory" \
--output=results.json --compress --stat \
1 setup.txt workload-contract-10.yaml
```
```bash
diablo secondary -v --tag="local" \
--port=5000 127.0.0.1
```

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
On each line of ```results.csv```, we can now see the performance results of an archive for each given blockchain. 
The latencies are expressed in seconds and follow the transaction submission times. So for example, the first submitted transaction for algorand at time 0.10 second took 0.53 seconds to commit (first line).

###### Results specification

In particular, the ```results.csv``` file organises the results of each experiment on a different line with data organized in the following comma-separated columns. The first line contains the colum names, each with the following meaning:
- uid: unique identifier
- blockchain: name of the blockchain 
- region: the AWS regions we used for the experiment
- secondary: number of Diablo Secondaries (acting as clients by sending requests to validators) that are deployed in the experiment
- miner: number of miners/validators in the blockchain network of the experiment
- machine: the AWS virtual machine specification as available at https://aws.amazon.com/ec2/instance-types/c5/
- interaction: the type of requests sent (it can be a transfer request or smart contract invocation request)
- lastsub: the duration in seconds between the benchmark start time and the time of the last transaction submission time
- submit: the number of submitted requests
- lastcom: the duration in seconds between the benchmark start time and the last transaction commit time
- commit: the number of committed requests
- latecommit: the number of requests that were committed after all requests were submitted
- avglat: the average latency of the committed transactions
- submits: a semi-colon separated list of duration (in seconds) between the benchmark start time and the times each request was submitted
- latencies: a semi-colon separated list of latencies for each committed request
