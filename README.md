# Gnosis Protocol subgraph

<img align="right" width="350" src="./docs/subgraph.png">

Implements a subgraph for the [Gnosis Protocol](https://github.com/gnosis/dex-contracts)

- [Subgraph on Mainnet](https://thegraph.com/explorer/subgraph/gnosis/protocol)
- [Subgraph on Rinkeby](https://thegraph.com/explorer/subgraph/gnosis/protocol-rinkeby)
- [Subgraph on xDAI](https://thegraph.com/explorer/subgraph/gnosis/protocol-xdai)

For more information about:

- The Gnosis Protocol: https://docs.gnosis.io/protocol
- The graph: https://thegraph.com/docs

---

## Installing Dependencies

```bash
yarn
```

## Setup for Local Development

> First install dependencies (see above)

1. Run a local ganache-cli and migrate the dependencies
   > TODO: Initially this step was done in the same project, but there's a dependency issue to solve in dex-contracts. It's better for now cloning dex-contracts and migrating separatelly

```bash
# Clone dex-contracts project
#   It doesn't matter where you clone the project, this project is independent from dex-subgraph
git clone https://github.com/gnosis/dex-contracts
cd dex-contracts

# Install dependencies and build project
# Make sure your node version is `v12.*`, as `v14.*` is not yet supported by the dex-contracts project
yarn && yarn build

# Run a local ganache (in another tab)
yarn run-ganache

# Migrate dependencies
npx truffle migrate

# Setup some test data
yarn truffle-exec scripts/ganache/setup_thegraph_data.js
```

2. Run a local The Graph Node

```bash
# Clone The Graph node
git clone https://github.com/graphprotocol/graph-node/
cd graph-node/docker

# [only Linux] Inject host IP in docker-compose.yml
# You might need to run it as `root`
./setup.sh

# Start a local Graph Node
# If you ran `./setup.sh` as `root`, you'll need to do that here too
docker-compose up
```

3. In dex-subgraph. Create a local subgraph:

```bash
# Create a new subgraph
yarn create-ganache

# Deploy it
yarn deploy
```

The subgraph should be accessible in: <http://127.0.0.1:8000/subgraphs/name/gnosis/protocol/graphql>

<details><summary>Example of GraphQL subscription to try:</summary>

```graphql
subscription UserData {
  users {
    id

    orders {
      id
      orderId
      owner {
        id
      }
      buyToken {
        id
        address
        name
        symbol
      }
      sellToken {
        id
        address
        name
        symbol
      }
      txHash
      txLogIndex
    }

    deposits {
      id
      tokenAddress
      amount
      txHash
    }

    withdrawals {
      tokenAddress
      txHash
    }

    withdrawRequests {
      tokenAddress
      txHash
    }
  }
}
```

</details>

## Update to a new version of the contracts

> First setup for local development (see above)

1. Make sure you use the latest version of `dex-contracts`

```bash
# See all available versions in: https://www.npmjs.com/package/@gnosis.pm/dex-contracts
npm ls @gnosis.pm/dex-contracts
```

2. Update the ABI

```bash
# Install new version of dex-contracts
yarn add @gnosis.pm/dex-contracts@<new-version> --save

# Update ABI
yarn abi
```

3. Make sure the addresses are updated

```bash
# Show the addresses for all the networks
yarn addresses

# Update the addresses and the start block
vim config/ganache.json
vim config/rinkeby.json
vim config/xdai.json
vim config/mainnet.json
```

4. Review the differences between the two version in the contract, and adapt all the handlers.

5) Regenerate the model:

```bash
yarn gen
```

## Deploy to rinkeby or mainnet

> First update the version of dex-contracts if it's not up to date

Deploy to the different networks:

```bash
# Make sure you are authenticated
graph auth https://api.thegraph.com/deploy/ <your-access-token>

# Deploy to rinkeby
yarn deploy-rinkeby

# Deploy to xdai
yarn deploy-xdai

# Deploy to staging
yarn deploy-staging

# Deploy to mainnet
#   IMPORTANT: Make sure the graph is well tested in staging environment
yarn deploy-mainnet
```

## Troubleshooting

```bash
# Delete all containers and data
docker-compose down && rm -rf data

# Run the graph
docker-compose up
```
