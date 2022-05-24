---
title:  "How to add a DApp"
layout: default
permalink: /dapp-howto
---

This page explains how to add a DApp to Diablo. 
The first step is to upload a DApp as a smart contract on 
the blockchain. The second step is to configure the workload 
configuration file as described below.

Below is an example of a gaming DApp configuration file with 
4 variables: ```acc``` is a set of 2,000 user accounts, 
```dapp``` is a set containing one instance of the dota DApp, 
```loc```  is the set of Secondaries tagged with the string, 
```us-east-2``` is an AWS availability zone and ```end``` is the 
set of all endpoints.
These two variables are used in the definition of the ```M``` function
which defines 3 clients invoking the DApp from accounts in ```acc```
with the parameters parsed from ```update(1, 1)``` at the rate 
specified in the ```load``` section: each client sends 4,432 TPS for 
the first 50 seconds then 4,438 TPS for the next 70 seconds, 
which is the end of the test.


```
let:
  - &loc { sample: !location [ "us-east-2" ] }|\label{line:workload-loc}|
  - &end { sample: !endpoint [ ".*" ] }|\label{line:workload-end}|
  - &acc { sample: !account { number: 2000 } }|\label{line:workload-acc}|
  - &dapp { sample: !contract { name: "dota" } }|\label{line:workload-dapp}|
workloads:
  - number: 3|\label{line:workload-client-start}|
    client:
      location: *loc
      view: *end|\label{line:workload-client-end}|
      behavior:
        - interaction: !invoke
            from: *acc
            contract: *dapp
            function: "update(1, 1)"
          load:|\label{line:workload-load-start}|
            0: 4432
            50: 4438
            120: 0|\label{line:workload-load-end}|
```
