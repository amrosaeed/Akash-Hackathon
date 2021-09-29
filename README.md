# Akash-Hackathon

## Implemented features

<p>
  We implemented solana.yml file, which is located in /akash/devnet directory. This file is used for deployment on akash network. In the file are configured 3 solana RPC nodes and nginx load balancer. We are not sure how to configure env for nginx (HOST and HOSTS variables), because we don't know the domain of the deployed containers.
</p>
<p>
  We also created docker-compose for running the local cluster. In this case we are using "build" key inside yml file to create new nginx image from Dockerfile. We configured nginx to load balance between the nodes.
</p>

## Solana-omnibus branch
At this branch we created folder for production ready solana nodes.
