# node_deply.yml

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

## Minimizing Validator Port Exposure

The validator requires that various UDP and TCP ports be open for inbound traffic from all other Solana validators. While this is the most efficient mode of operation, and is strongly recommended, it is possible to restrict the validator to only require inbound traffic from one other Solana validator.

First add the `--restricted-repair-only-mode` argument. This will cause the validator to operate in a restricted mode where it will not receive pushes from the rest of the validators, and instead will need to continually poll other validators for blocks. The validator will only transmit UDP packets to other validators using the Gossip and ServeR ("serve repair") ports, and only receive UDP packets on its Gossip and Repair ports.

The Gossip port is bi-directional and allows your validator to remain in contact with the rest of the cluster. Your validator transmits on the ServeR to make repair requests to obtaining new blocks from the rest of the network, since Turbine is now disabled. Your validator will then receive repair responses on the Repair port from other validators.

To further restrict the validator to only requesting blocks from one or more validators, first determine the identity pubkey for that validator and add the `--gossip-pull-validator PUBKEY --repair-validator PUBKEY` arguments for each PUBKEY. This will cause your validator to be a resource drain on each validator that you add, so please do this sparingly and only after consulting with the target validator.

Your validator should now only be communicating with the explicitly listed validators and only on the Gossip, Repair and ServeR ports.

### Only Caveat regard to Akash providers

The only caveat is that it is not possible to export non-http/https (80/443) TCP ports, so-called nodePort ports such as `8899/tcp`, `8010:8020/udp` directly as-is.
The Akashnet providers use Kubernetes under the hood for actually running your containers. The Kubernetes control plane allocates a port from a range specified by `--service-node-port-range` flag `(default: 30000-32767)`(nodePort)
This means the operator will have to first deploy the Solana node and then check what ports the provider's Kubernetes assigned to your Akash deployment.
And then use some load balancer / reverse proxy (i.e. nginx / haprorxy / traefik) to forward to these ports.
You would also need to tell Solana validator to use load-balancer's hostname by specifying it via the `--public-rpc-address <HOST:PORT>` argument to Solana validator node.
