---
title:  "Fresh install"
layout: default
permalink: /fresh-install
---

In order to replicate the performance graphs of [Diablo-v2: A Benchmark for Blockchain Systems](https://infoscience.epfl.ch/record/294268?ln=en),  
one needs to replicate the different configuration settings as detailed in the [AWS experiments](aws-experiments) below and to install and run Diablo and the blockchains on the corresponding machines of each configuration before [plotting the figures](#plotting-the-figures-human-time-2-minutes--machine-time-5-minutes) as explained below.

### Install yourself (human time: couple of hours / machine time: couple of hours)

We need to proceed as follows:
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

More precisely, the steps are:

 * Install gcc 
```bash
sudo apt update
sudo apt install build-essential
```
 * Install [PerlBrew](https://perlbrew.pl/) with perl v5.34.0 and dependencies:
```bash
curl -L https://install.perlbrew.pl | bash
```
   - Follow the instructions in the command output and restart the shell
```bash
perlbrew install perl-5.34.0
perlbrew switch perl-5.34.0
cpan YAML
cpan JSON
```
 * Install pyenv and python with the dependencies
```bash
curl https://pyenv.run | bash
```
 * Follow the instructions in the command output and restart the shell
```bash
pyenv install 3.10.6
pyenv global 3.10.6
pip install PyYAML
``` 
 * Generate an ssh key and add it to authorized keys for minion to establish connections to localhost
```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519 -N ''
cat ~/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys
```
 * Clone the Artifact Evalation Committee (aec) branch of the minion repo
```bash
git clone -b aec https://github.com/lebdron/minion
```
 * Run the example experiment, which will install the blockchain binaries and diablo
```bash
./bin/eurosys workload-native-10.yaml setup.txt
```


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
