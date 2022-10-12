---
title:  "Artifact"
layout: default
permalink: /artifact
---

### Abstract 

The publication of [Diablo](https://gramoli.github.io/pubs/Eurosys23-Diablo.pdf) comes with an artifact, a digital object we provided to allow 
to reproduce our results, that was judged available and functional by an independent committee as part of 
[this process]([https://sysartifacts.github.io/eurosys2023/](https://sysartifacts.github.io/eurosys2023/call#process)).
Note that, after discussion with the committee, we concluded that the scale of some of our experiments (on 200 machines spread across 5 continents) 
is beyond the scope of what an artifact evaluation committee can reasonably reproduce.
Our artifact lists the steps to:
 * either run a [Diablo dummy test in a VirtualBox image](redo-howto)
 * or to launch distributed experiments by [intalling Diablo on a distributed set of machines](fresh-install).

### Description & Requirements

To run Diablo easily, we provide a VirtualBox image with all dependencies, running the workloads on the blockchains as indicated in a screencast.

### How to access

All the code necessary to reproduce our experiments is available publicly online. 
In particular, it contains: 
 * The Diablo benchmarking framework soure code, including every decentralized application benchmark and their associated realistic workloads.
 * A set of scripts called minion that helps deploying the code to the availability zones of Amazon Web Services where our paper deployed 
 the evaluated blockchains.
 * The links to the source code of each of the blockchain we evaluated:
   - Algorand (hash ```116c06e``` from [https://github.com/algorand](https://github.com/algorand))
   - Avalanche (hash ```7840200``` from [https://github.com/ava-labs/avalanchego](https://github.com/ava-labs/avalanchego))
   - Diem (hash ```4b3bd1e``` of [https://github.com/diem/diem](https://github.com/diem/diem))
   - Ethereum (hash ```72c2c0a``` of [https://github.com/ethereum/go-ethereum](https://github.com/ethereum/go-ethereum))
   - Quorum (hash ```919800f``` of [https://github.com/ConsenSys/quorum](https://github.com/ConsenSys/quorum))
   - Solana (hash ```0d36961``` of [https://github.com/solana-labs/solana](https://github.com/solana-labs/solana))

In order to simplify the tests, we made a VirtualBox image, a step-by-step markdown
documentation and a screencast available online:
 * VirtualBox Image (7.1 GB). 
The login and password are both ```vagrant```.
 * Screencast (56 MB)
 * Markdown documentation: [https://diablobench.github.io/redo-howto](https://diablobench.github.io/redo-howto)

#### Hardware dependencies

It is recommended to use a x86-64 architecture to simplify the evaluation. In particular, 
VirtualBox, which is used in our screencast, does not support ARM processors, we thus recommend one machine with at least: 8GB memory, 
4 vCPUs/cores and 10 GB storage space.
For a fresh setup, one can use as many as 200 machines, each with up to 36 vCPUs and 72 GiB of memory, spread across 5 continents.

#### Software dependencies
There is no additional software dependency required, besides VirtualBox, if you download the VirtualBox image to run Diablo.
Otherwise, it is recommended to use Ubuntu OS and the following software as indicated in 
[https://diablobench.github.io/fresh-install](https://diablobench.github.io/fresh-install):
```make```, ```gcc```, ```perl```, ```perlbrew-5.34.0```, ```pyenv 3.10.6```, ```python```, ```ssh```, ```git```, ```minion```, ```diablo```.

#### Benchmarks

In addition to executing native transfers on the 6 blockchains, we also used realistic workloads:
Dota 2, Fifa, Nasdaq, Uber, YouTube.
They are separated from the source code and can be downloaded at the [fresh install page](fresh-install).

## Set-up (1h30min)

We present a simple setup that builds upon a VirtualBox image that allows to deploy the blockchain protocols, 
shows how to run two simple workloads and collect the results. We defer the instructions to run experiments on up 
to 200 virtual machines in 5 continents to the [online fresh install page](fresh-install).

The setup consists of downloading a VirtualBox 
image and to follow the step-by-step documentation at [https://diablobench.github.io/redo-howto](redo-howto) or the screencast at the same page to generate results in a simplified setting. 

 * Install VirtualBox.
 * Download the [image](https://nextcloud.in.tum.de/index.php/s/RDy4Df3x9JTsLGG).
 * Start the image with VirtualBox with login and password as ```vagrant```.
 * Run the experiments on the blockchains with a simple native transfer workload, ```workload-native-10.yaml```:

```bash
cd minion
./bin/eurosys --skip-install workload-native-10.yaml setup.txt
```
Run now the experiments with a DApp workload, ```workload-contract-10.yaml```:
```bash
./bin/eurosys --skip-install workload-contract-10.yaml setup.txt
```
Note that the workloads corresponding to each DApp of the paper are available as well.
These experiments aggregate the performance results of all blockchains located in files of the name:
```
[blockchain_name]-[region_number]-[#secondaries]-[#blockchain_nodes]-[workload_name]-[timestamp].results.tar.gz
```

For example, let us look at Algorand's performance results:

```bash
tar xf0 algorand-1-1-1-native-10_2022-08-21-22-48-58.tar.gz algorand-1-1-1-native-10_2022-08-21-22-48-58/
```

This outputs the performance obtained by Algorand by inspecting the standard output log of the Diablo primary node that aggregated the results.
For example, during the tests recorded in the screencast, we could see that 299 transactions were sent to Algorand, out of which 187 were successfully committed. As none were aborted, the rest of the transactions were pending. The average load was 10 transactions sent per second, and the average throughput was 6.3 transactions per second with an average latency of 12.2 seconds and a median latency of 11.4 seconds. 

One can also inspect more detailed results by moving the results to the scripts folder to convert them from the JSON format to the CSV format:
```bash
mv *native*.tar.gz ~/scripts/results/native/
mv *contract*.tar.gz ~/scripts/results/contracts/
cd ~/scripts
./csv-results results results.csv
```

On each line of ```results.csv```, we can now see the performance results of an archive for each given blockchain. 
The latencies are expressed in seconds and follow the transaction submission times. In the screencast example, 
the first submitted transaction for Algorand at time 0.10 second took 0.53 seconds to commit (first line).


### Evaluation workflow
 
#### Major Claims

We enumerate here the major claims (Cx) of the paper.
 * *(C1)*: We demonstrate that the performance of 6 state-of-the-art blockchains, including Algorand, Avalanche, Ethereum, Diem, Quorum and Solana is 
 heavily dependent on the underlying  experimental settings in which they are evaluated.
 * *(C2)*: Real DApps may not even execute successfully as some of their functions would consume more than the maximum allowed gas per transaction.
 * *(C3)*: The Algorand, Avalanche, Ethereum, Diem, Quorum and Solana blockchains are not capable of handling the demand of the selected centralized 
 applications when deployed on modern commodity computers from individuals across the world.

#### Experiments
We now describe how to verify the claims C1-C3 using the following experiments E1-E3:

*Experiment (E1):* The first experiment shows that the type of workload impacts the performance of a blockchain.

*[Preparation]*
Describe in this block the steps required to prepare and configure the environment for this experiment.
Follow up the instructions in [Section Setup](Set-up (1h30min)).

*[Execution]*
Execute both workloads, with 10 transactions sent per second and 100 transactions sent per second:

```bash
./bin/eurosys --skip-install workload-native-10.yaml setup.txt
./bin/eurosys --skip-install workload-100.yaml setup.txt
```

*[Results]*
You can see that both workloads led to different set of results, for example in Algorand:

```bash
tar xfO algorand-1-1-1-native-10_2022-08-21-22-48-58.results.tar.gz algorand-1-1-1-native-10_2022-08-21-22-48-58.results/diablo-primary-127.0.0.1-out.log
tar xfO algorand-1-1-1-native-100_2022-08-21-23-08-04.results.tar.gz algorand-1-1-1-native-100_2022-08-21-23-08-04.results/diablo-primary-127.0.0.1-out.log
```
This confirms that different experimental settings lead to different results.

*Experiment (E2):* The second experiment shows that certain realistic DApps cannot be executed because of the gas per transaction cap.

*[Preparation]*
Follow up the instructions in Section~\ref{sec:setup}.

*[Execution]*
Execute Uber workload:

```bash
./bin/eurosys --skip-install workload-uber.yaml setup.txt
```

*[Results]*
You can see for Solana that transactions are submitted but not committed:

```bash
tar xfO solana-1-1-1-smart-uber_2022-08-21-22-48-58.results.tar.gz solana-1-1-1-smart-uber_2022-08-21-22-48-58.results/diablo-primary-127.0.0.1-out.log
```
This confirms that some blockchains cannot execute some realistic DApps due to transaction gas limit.

*Experiment (E3):*
This experiment consists of observing that none of the tested blockchains can commit all transaction of any DApp workloads on a fresh install on 
200 AWS VMs of type c5.2xlarge (8vCPUs, 16GB memory) spread equally across the following availability zones: Cape Town, Tokyo, Mumbai, Sydney, 
Stockholm, Milan, Bahrain, Sao Paulo, Ohio and Oregon.
For more details, follow the fresh install described on the [fresh install page](https://diablobench.github.io/fresh-install). 
