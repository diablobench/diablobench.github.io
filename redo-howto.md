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

 * Memory: 8 GB
 * CPUs: 4 cores
 * Storage space: 8 GB
 * x64 CPU (not ARM for compatibility with VirtualBox)
 * Install VirtualBox
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

#### AWS experiments

Install Perl 5.34.0 as described in [fresh install guide](fresh-install).

Install [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) and [set up](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-quickstart.html) your credentials.

[Create a key pair](https://docs.aws.amazon.com/cli/latest/userguide/cli-services-ec2-keypairs.html) with the name ```aec```.

Prepare the image with all the required software, including the blockchain binaries.
```bash
./bin/minion prepare --base='*ubuntu*20.04*amd64*server*' --key=aec --name=artifact --region=ap-south-1 --region=af-south-1 --region=eu-south-1 --region=eu-north-1 --region=us-east-2 --region=ap-northeast-1 --region=sa-east-1 --region=ap-southeast-2 --region=me-south-1 --region=us-west-2 --resize=72 --install-region=eu-south-1 --size=1 --security-group=default --type=c5.9xlarge --verbose
```

Download the [workloads](https://nextcloud.in.tum.de/index.php/s/DzBg4dzNHwfjeRd).

##### Testnet topology

Boot 20 c5.xlarge machines in Ohio, 10 of which will be used for the blockchain, and 10 will be used for Diablo.
```bash
./bin/minion boot --image=artifact --key=aec --region=us-east-2 --size=20 --security-group=default --type=c5.xlarge -vv
```
Leave this running in a terminal.

In another terminal, for all the blockchains and simple transfer workloads, run the experiments. Pass ```sfr-id@region``` from the output of the previous ```boot``` command.
blockchain-name: ```algorand```, ```diem```, ```quorum-ibft```, ```poa```, ```solana```, ```avalanche```.
blockchain-type: ```algorand```, ```diem```, ```other```.
n: 100, 1000, 10000.
```bash
./bin/minion run -vv blockchain-name blockchain-type-testnet-workload-n.yaml sfr-id@region
```

Use ```Ctrl-C``` to terminate the ```boot``` command and shutdown the machines.

##### Datacenter topology

Boot 14 c5.9xlarge machines in Ohio, 10 of which will be used for the blockchain, and 4 will be used for Diablo.
```bash
./bin/minion boot --image=artifact --key=aec --region=us-east-2 --size=14 --security-group=default --type=c5.9xlarge -vv
```
Leave this running in a terminal.

In another terminal, for all the blockchains and simple transfer workloads, run the experiments. Pass ```sfr-id@region``` from the output of the previous ```boot``` command.
blockchain-name: ```algorand```, ```diem```, ```quorum-ibft```, ```poa```, ```solana```, ```avalanche```.
blockchain-type: ```algorand```, ```diem```, ```other```.
n: 100, 1000, 10000.
```bash
./bin/minion run -vv blockchain-name blockchain-type-datacenter-workload-n.yaml sfr-id@region
```

Use ```Ctrl-C``` to terminate the ```boot``` command and shutdown the machines.

##### Devnet topology

Boot 2 c5.xlarge machines in all the regions, 1 of which will be used for the blockchain, and 1 will be used for Diablo.
```bash
./bin/minion boot --image=artifact --key=aec --region=ap-south-1 --region=af-south-1 --region=eu-south-1 --region=eu-north-1 --region=us-east-2 --region=ap-northeast-1 --region=sa-east-1 --region=ap-southeast-2 --region=me-south-1 --region=us-west-2 --size=2 --security-group=default --type=c5.xlarge -vv
```
Leave this running in a terminal.

In another terminal, for all the blockchains and simple transfer workloads, run the experiments. Pass ```sfr-id@region``` for all the regions from the output of the previous ```boot``` command.
blockchain-name: ```algorand```, ```diem```, ```quorum-ibft```, ```poa```, ```solana```, ```avalanche```.
blockchain-type: ```algorand```, ```diem```, ```other```.
n: 100, 1000, 10000.
```bash
./bin/minion run -vv blockchain-name blockchain-type-devnet-workload-n.yaml sfr-id@region
```

Use ```Ctrl-C``` to terminate the ```boot``` command and shutdown the machines.

##### Community topology

Boot 21 c5.xlarge machines in all the regions, 20 of which will be used for the blockchain, and 1 will be used for Diablo.
```bash
./bin/minion boot --image=artifact --key=aec --region=ap-south-1 --region=af-south-1 --region=eu-south-1 --region=eu-north-1 --region=us-east-2 --region=ap-northeast-1 --region=sa-east-1 --region=ap-southeast-2 --region=me-south-1 --region=us-west-2 --size=21 --security-group=default --type=c5.xlarge -vv
```
Leave this running in a terminal.

In another terminal, for all the blockchains and simple transfer workloads, run the experiments. Pass ```sfr-id@region``` for all the regions from the output of the previous ```boot``` command.
blockchain-name: ```algorand```, ```diem```, ```quorum-ibft```, ```poa```, ```solana```, ```avalanche```.
blockchain-type: ```algorand```, ```diem```, ```other```.
n: 100, 1000, 10000.
```bash
./bin/minion run -vv blockchain-name blockchain-type-community-workload-n.yaml sfr-id@region
```

Use ```Ctrl-C``` to terminate the ```boot``` command and shutdown the machines.

##### Consortium topology

Boot 21 c5.2xlarge machines in all the regions, 20 of which will be used for the blockchain, and 1 will be used for Diablo.
```bash
./bin/minion boot --image=artifact --key=aec --region=ap-south-1 --region=af-south-1 --region=eu-south-1 --region=eu-north-1 --region=us-east-2 --region=ap-northeast-1 --region=sa-east-1 --region=ap-southeast-2 --region=me-south-1 --region=us-west-2 --size=21 --security-group=default --type=c5.2xlarge -vv
```
Leave this running in a terminal.

In another terminal, for all the blockchains and simple transfer workloads, run the experiments. Pass ```sfr-id@region``` for all the regions from the output of the previous ```boot``` command.
blockchain-name: ```algorand```, ```diem```, ```quorum-ibft```, ```poa```, ```solana```, ```avalanche```.
blockchain-type: ```algorand```, ```diem```, ```other```.
contract-type: ```amazon```, ```apple```, ```dota```, ```facebook```, ```football```, ```gafam```, ```google```, ```microsoft```, ```uber```, ```youtube```. Note that ```youtube``` contract is not available for Algorand.
```bash
./bin/minion run -vv blockchain-name blockchain-type-workload-contract-type.yaml sfr-id@region
```

Use ```Ctrl-C``` to terminate the ```boot``` command and shutdown the machines.

You can now produce ```results.csv``` from all the archives produced by minion.

#### Plotting the figures (human time: 2 minutes / machine time: 5 minutes)

Prepare Python 3.10 environment as described in [fresh install guide](fresh-install) and install matplotlib.
```
pip install matplotlib
```

Download the [scripts.tar.gz](https://nextcloud.in.tum.de/index.php/s/FjWQiygDA7D6Y4m) and unarchive it.

Download the [results.csv.gz file](https://nextcloud.in.tum.de/index.php/s/M3MwgpggogcjNB5) and unarchive it inside the scripts directory.

Create the ```configs.csv``` file.
```
./csv-configs ./results.csv ./configs.csv
```

Create the figure pdf files.
```
make figures
```

It should produce the following:

 - deployment.pdf => Fig. 3
 - *-workload.pdf => Table 2
 - execution.pdf => Fig. 5
 - overview.pdf => Fig. 2
 - peak.pdf => Fig. 6
 - robustness.pdf => Fig. 4
 - setup.pdf => Table 3
