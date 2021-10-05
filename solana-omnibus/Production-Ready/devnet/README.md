# node_deply.yml

## env: (Key)

```
env:
      - SOLANA_RUN_SH_CLUSTER_TYPE=devnet
      - |
        SOLANA_RUN_SH_VALIDATOR_ARGS=
        --trusted-validator dv1ZAGvdsz5hHLwWXsVnM94hWf1pjbKVau1QVkaMJ92
        --trusted-validator dv2eQHeP4RFrJZ6UeiZWoc3XTtmtZCUKxxCApCDcRNV
        --trusted-validator dv4ACNkpYPcE3aKmYDqZm9G5EB3J4MRoeE7WNDRBVJB
        --trusted-validator dv3qDFk1DTF36Z62bNvrCXe9sKATA6xvVy6A798xxAS
        --no-untrusted-rpc
        --private-rpc
        --rpc-port 8899
        --entrypoint entrypoint.devnet.solana.com:8001
        --entrypoint entrypoint2.devnet.solana.com:8001
        --entrypoint entrypoint3.devnet.solana.com:8001
        --entrypoint entrypoint4.devnet.solana.com:8001
        --entrypoint entrypoint5.devnet.solana.com:8001
        --expected-genesis-hash EtWTRABZaYq6iMfeYKouRu166VU2xqa1wcaWoxPkrZBG
        --wal-recovery-mode skip_any_corrupted_record
        --limit-ledger-size
        --dynamic-port-range 8010-8020
``` 

* The `--trusted-validator` are operated by Solana Labs.

* The `--private-rpc` prevents your RPC port from being published for use by other nodes.

* The `--rpc-port 8899` to the port you want to expose.  

* The `--entrypoint` and The `--expected-genesis-hash` parameters are all specific to the cluster you are joining.

* The `--limit-ledger-size` parameter allows you to specify how many ledger shreds your node retains on disk. If you do not include this parameter, the validator will keep the entire ledger until it runs out of disk space. The default value attempts to keep the ledger disk usage under 200GB.

* The `--dynamic-port-range 8010-8020` 

### Minimizing Validator Port Exposure

The validator requires that various UDP and TCP ports be open for inbound traffic from all other Solana validators. While this is the most efficient mode of operation, and is strongly recommended, it is possible to restrict the validator to only require inbound traffic from one other Solana validator.

First add the `--restricted-repair-only-mode` argument. This will cause the validator to operate in a restricted mode where it will not receive pushes from the rest of the validators, and instead will need to continually poll other validators for blocks. The validator will only transmit UDP packets to other validators using the Gossip and ServeR ("serve repair") ports, and only receive UDP packets on its Gossip and Repair ports.

The Gossip port is bi-directional and allows your validator to remain in contact with the rest of the cluster. Your validator transmits on the ServeR to make repair requests to obtaining new blocks from the rest of the network, since Turbine is now disabled. Your validator will then receive repair responses on the Repair port from other validators.

To further restrict the validator to only requesting blocks from one or more validators, first determine the identity pubkey for that validator and add the `--gossip-pull-validator PUBKEY --repair-validator PUBKEY` arguments for each PUBKEY. This will cause your validator to be a resource drain on each validator that you add, so please do this sparingly and only after consulting with the target validator.

Your validator should now only be communicating with the explicitly listed validators and only on the Gossip, Repair and ServeR ports.

#### Only Caveat regard to Akash providers

The only caveat is that it is not possible to export non-http/https (80/443) TCP ports, so-called nodePort ports such as `8899/tcp`, `8010:8020/udp` directly as-is.
The Akashnet providers use Kubernetes under the hood for actually running your containers. The Kubernetes control plane allocates a port from a range specified by `--service-node-port-range` flag `(default: 30000-32767)`(nodePort)
This means the operator will have to first deploy the Solana node and then check what ports the provider's Kubernetes assigned to your Akash deployment.
And then use some load balancer / reverse proxy (i.e. nginx / haprorxy / traefik) to forward to these ports.
You would also need to tell Solana validator to use load-balancer's hostname by specifying it via the `--public-rpc-address <HOST:PORT>` argument to Solana validator node.

## Profiles: (Key) ---> Hardware Requirements

```
profiles:
  compute:
    solana:
      resources:
        cpu:
          units: 16
        memory:
          size: 256Gi
        storage:
          size: 200Gi #512Mi
``` 
### Hardware Recommendations:

   * CPU
        12 cores / 24 threads, or more
        2.8GHz, or faster
        AVX2 instruction support (to use official release binaries, self-compile otherwise)
        Support for AVX512f and/or SHA-NI instructions is helpful
        The AMD Zen3 series is popular with the validator community
   * RAM
        128GB, or more
        Motherboard with 256GB capacity suggested
   * Disk
        PCIe Gen3 x4 NVME SSD, or better
        Accounts: 500GB, or larger. High TBW (Total Bytes Written)
        Ledger: 1TB or larger. High TBW suggested
        OS: (Optional) 500GB, or larger. SATA OK
        The OS may be installed on the ledger disk, though testing has shown better performance with the ledger on its own disk
        Accounts and ledger can be stored on the same disk, however due to high IOPS, this is not recommended
        The Samsung 970 and 980 Pro series SSDs are popular with the validator community
   * GPUs
        Not strictly necessary at this time
        Motherboard and power supply speced to add one or more high-end GPUs in the future suggested
        
### RPC Node Recommendations:

   * CPU
        16 cores / 32 threads, or more
   * RAM
        256 GB, or more
   * Disk
        Consider a larger ledger disk if longer transaction history is required
        Accounts and ledger should not be stored on the same disk
        
### Virtual machines on Cloud Platforms (AKASH)

While you can run a validator on a cloud computing platform, it may not be cost-efficient over the long term.

However, it may be convenient to run non-voting api **Our Case** nodes on VM instances for your own internal usage ** Local DApp **. This use case includes exchanges and services built on Solana.

In fact, the mainnet-beta validators operated by the team are currently (Mar. 2021) run on `GCE n2-standard-32` (32 vCPUs, 128 GB memory) instances with 2048 GB SSD for operational convenience.

For other cloud platforms, select instance types with similar specs.

Also note that `egress internet traffic` **Can be mitigated** usage may turn out to be high, especially for the case of running staked validators **Not Our Case**.

### Docker (Vital Aspect for AKASH deplyment)

Running validator for live clusters (including mainnet-beta) inside Docker is not recommended and generally not supported. This is due to concerns of general Docker's containerzation overhead and resultant performance degradation unless specially configured.

An excellent 2014 IBM research paper “An Updated Performance Comparison of Virtual Machines and Linux Containers” by Felter et al. provides a comparison between bare metal, KVM, and Docker containers. The general result is: Docker is nearly identical to native performance and faster than KVM in every category.

The exception to this is Docker’s NAT — if you use port mapping (e.g., docker run -p 8080:8080), then you can expect a minor hit in latency, as shown below. However, you can now use the host network stack (e.g., docker run --net=host) when launching a Docker container, which will perform identically to the Native column (as shown in the Redis latency results lower down).

![image](https://user-images.githubusercontent.com/82784007/135408671-f4c05ae6-eb3b-40c8-85b0-67f4c867290f.png)

They also ran latency tests on a few specific services, such as Redis. You can see that above 20 client threads, highest latency overhead goes Docker NAT, then KVM, then a rough tie between Docker host/native. 

![image](https://user-images.githubusercontent.com/82784007/135408979-70b13006-0b44-44c8-87fa-cdab71ff7c3e.png)

Just because it’s a really useful paper, here are some other figures. 

#### Taking a look at Disk I/O:

![image](https://user-images.githubusercontent.com/82784007/135409312-f5387e0d-6b8a-4eac-a6d4-5ffa1833ede3.png)

#### Now looking at CPU overhead:

![image](https://user-images.githubusercontent.com/82784007/135409394-6ba55142-e252-4b78-9c9f-42f1b028d14f.png)

#### Now some examples of memory (read the paper for details, memory can be extra tricky):

![image](https://user-images.githubusercontent.com/82784007/135409572-8e98579a-2c09-4684-a0e3-dc1f31ae1a85.png)






