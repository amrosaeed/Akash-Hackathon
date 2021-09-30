# Solana On AKASH
## Motivation
This repository is meant to serve as an example for how to run a load balanced solana RPC nodes local to your DApp. It does not give specifics on the 
architecture of Solana, and should not be used as a substitute for Solana's documentation. It is highly recommended to 
read [Solana's Documentation](https://docs.solana.com/running-validator) on running a validator. This repository 
should be used in conjunction with Solana's guide. It provides a preliminary answear to Akash RFP [Aash RFP](https://forum.akash.network/t/build-a-scalable-solana-rpc-cluster-on-akash/2716) real-world examples of a cluster setup, and 
should act as a starting point for participating in mainnet validation. 

This repository gives three examples of potential Validator-Network setups (Main-beta net, Test net and Dev net). for a cluster of two solana nodes that are load balanced by an NGINX server that can be used 
as an entry point for querying on-chain Solana data.

The end goal of this guide is to have a solana validator cluster running in a decentralized cloud environment like Akash.
 
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
