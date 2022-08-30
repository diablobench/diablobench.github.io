---
title:  "Fresh install"
layout: default
permalink: /fresh-install
---


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
