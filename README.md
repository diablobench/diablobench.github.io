#### Home

Diablo is a benchmark suite to evaluate blockchain systems on the same ground.
It was developed in a partnership between [University of Sydney](https://www.sydney.edu.au/) [CSRG](https://gramoli.github.io/csrg/) and the 
[Swiss Federal Institute of Technology Lausanne (EPFL)](https://www.epfl.ch/en/) [DCL](https://dcl.epfl.ch/site/)
to evaluate blockchain and distributed ledger technologies when running realistic applications.
The name Diablo stems from DIstributed Analytical BLOckchain benchmark.
If you use Diablo, please cite our scientific article:

*[Diablo: A Benchmark Suite for Blockchains](https://gramoli.github.io/pubs/Eurosys23-Diablo.pdf)*.
V. Gramoli, R. Guerraoui, A. Lebedev, C. Natoli and G. Voron.
*Proceedings of the 18th ACM European Conference on Computer Systems (EuroSys)*, 2023.

#### Blockchains
Diablo was used to evaluate the following blockchains:
 * [Algorand](https://github.com/algorand)
 * [Avalanche](https://github.com/ava-labs/avalanchego)
 * [Diem](https://github.com/diem/diem)
 * [Ethereum](https://github.com/ethereum/go-ethereum)
 * [Hyperledger Fabric](https://github.com/hyperledger/fabric) (Compatible only with [Diablo v1](https://infoscience.epfl.ch/record/285731?&ln=en))
 * [Proof-of-Collaboration](https://arxiv.org/pdf/2302.02325.pdf)
 * [Quorum](https://github.com/ConsenSys/quorum)
 * [Redbelly](https://gramoli.github.io/pubs/IPDPS23-SmartRedbelly.pdf)
 * [Solana](https://github.com/solana-labs/solana)
 * [Zcash](https://github.com/brandonzstride/blockchains_crypto_go_zcash/)


#### DApps
Diablo features several decentralized applications (DApps), including:
 * Dota 2: one of the most popular multiplayer game of Steam, 
 * FIFA: a web service experiencing the FIFA requests during the soccer worldcup, 
 * NASDAQ: an exchange with the NASDAQ workload of the GAFAM stock trades,
 * Twitter: a microblogging DApp with the trace of tweets at the release of *The Castle in the Sky* anime,
 * Uber: a mobility service DApp with a Uber workload, 
 * YouTube: a video sharing service with a YouTube workload.

#### Using Diablo

- *Simple demo*: In order to play with Diablo, download our virtualBox image and run some tests by following [these tests](redo-howto).
- *Reproducibility*: To reproduce our results, you will need to setup a network configuration and follow a [fresh installation](fresh-install).
- *Artifact*: our [artifact](artifact) comprises the documentation to run scripts and software to reproduce the results of [our paper](https://gramoli.github.io/pubs/Eurosys23-Diablo.pdf).

#### Going further

- *Add your blockchain*: Feel free to add your own blockchain to Diablo by following the [blockchain instructions](blockchain-howto).
- *Add your DApp*: Feel free to add your own DApp/workload to Diablo by following the [DApp instructions](dapp-howto).

Please [let us know](mailto:csrg.sydney@gmail.com?subject=[Diablo]) if you added a new DApp or evaluated a new blockchain with Diablo.

<!-- 
[source](https://github.com/NatoliChris/diablo-benchmark/) 
[test](
http://194.182.162.199/benchmark) -->
