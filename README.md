# Solana On AKASH
## Motivation
This repository is meant to serve as an example for how to run a load balanced solana RPC nodes local to your DApp. It does not give specifics on the 
architecture of Solana, and should not be used as a substitute for Solana's documentation. It is highly recommended to 
read [Solana's Documentation](https://docs.solana.com/running-validator) on running a RPC service node. This repository 
should be used in conjunction with Solana's guide. It provides a preliminary answer to Akash RFP (https://forum.akash.network/t/build-a-scalable-solana-rpc-cluster-on-akash/2716).

This repository gives three examples of potential Network setups (Main-beta net, Test-net and Dev-net). of a cluster of two solana RPC nodes local to your app that are load balanced by an NGINX server that can be used 
as an entry point for querying on-chain Solana data.

The end goal of this guide is to have a cluster of solana RPC nodes running in a decentralized cloud environment like Akash.
 
## Running a cluster of load-balanced RPC Solana Nodes
#### Choosing an instance type
Solana's documentation recommends certain Hardware Requirments with the highest number of cores available and a CUDA enabled GPU 
([see here](https://docs.solana.com/running-validator/validator-reqs)). Solana uses GPUs to increase throughput and 
the documentation recommends using Nvidia Turing or Volta family GPUs which are available through most cloud providers. 

The Solana ledger is stored on disk. As of this writing, specifying the minimum required ledger length uses roughly 
200GiB of disk space. When provisioning your instance, you should choose a disk size of at least 200GiB to ensure 
sufficient space for the ledger.
## Implemented features

<p>
  We implemented node_deploy.yml file and lb_deploy file, which are located in solana-omnibus/Production-Ready/ directory. This files are used for deployment on akash network. In the node_deploy.yml is configured solana RPC node. In the lb_deploy.yml is nginx load balancer for balancing the traffic between dapp and the validator. To configure ngnix env variables, nodes need to be deployed to provider first, and check which ports are opened.
</p>

![Screenshot from 2021-09-30 05-37-04](https://user-images.githubusercontent.com/82784007/135428104-01eebb1d-04b2-4f09-8efe-b7aa9c9167a6.png)
