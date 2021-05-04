# Intro

This repository is our submission to **Scaling Ethereum 2021**.

Our team consists of developers of the state channel library *go-perun*. Our goal was to experiment with rollup technology and to see if we can make our library work with them and how much gas we can save. We focus on the EVM-compatible rollups Arbitrum and Optimism.

Our submission consists of the following components:

* **Contracts:** We modified the go-perun contracts and prepared deployment scripts that allow us to deploy to either a standard Ethereum blockchain, an Arbitrum Rollup, or an Optimism Rollup by just changing a single command line argument. 
* **Demo**: We modified the go-perun demo client to work with Rollups. This involved implementing ERC20 functionality and incorporating a new command line switch for selecting the network that we want to run on. In addition, we had to take care of some specialties of Rollups like setting the correct gas price and making sure that we do not use ETH functionality.
* **Library**: We needed to adapt the go-perun library in a few ways to make it work with Rollups. For example, we could not rely on estimating the gas correctly and instead needed to change our configuration accordingly.

# Walkthrough

## Download source code

Clone the repository and initialize the submodules repository.

```sh
git clone https://github.com/perun-network/channels-on-rollups
cd channels-on-rollups
git submodule update --init --recursive
```

## Start your local blockchain

### Ganache

https://github.com/trufflesuite/ganache-cli#using-ganache-cli

Make sure to set the gas price to 0.
```
ganache-cli --gasPrice 0
```

We assume that the node is listening at `127.0.0.1:8545` for http and websocket.

### Arbitrum

https://developer.offchainlabs.com/docs/local_blockchain

We assume that the node is listening at `127.0.0.1:8547` for http and at `127.0.0.1:8548` for websocket.

### Optimism
https://community.optimism.io/docs/developers/integration.html#step-2-testing-contracts

We assume that the node is listening at `127.0.0.1:8545` for http and at `127.0.0.1:8546` for websocket.

## Deploy the contracts
```sh
cd perun-eth-contracts
yarn
yarn run hardhat run scripts/deploy.js --network <network>
```
Substitute `<network>` with `ganache`, `arbitrum_local`, or `optimism_local`.

## Configure the clients
Put the printed contracts into `perun-eth-demo/network.yaml`.

## Start the clients

Start two clients in `perun-eth-demo` with:
```sh
go run . demo --config alice.yaml --chain <network>
go run . demo --config bob.yaml --chain <network>
```
Substitute `<network>` with `arbitrum_local` or `optimism_local`.

## Have fun 😄  
   
Once both CLIs are running, e.g. in Alice's terminal, propose a payment channel
to Bob with 10 *PerunToken* deposit from both sides via the following command.
```
> open bob peruntoken 10 10
```
In Alice's terminal, accept the appearing channel proposal.
```
🔁 Incoming channel proposal from bob with peruntoken funding [My: 10, Peer: 10].
Accept (y/n)? > y
```
The terminal will print the hashes of two transaction: *IncreaseAllowance* and *Deposit*.

Now you can execute off-chain payments, e.g. in Bob's terminal with
```
> send alice 1
```
The updated balance will immediately be printed in both terminals, but no
transaction will be visible on-chain.

You may always check the current status with command `info`.

You can also run a performance benchmark with command
```
> benchmark alice 5 100
```
which will send 5 *PerunToken* in 100 micro-transactions from Bob to Alice. Transaction performance will be printed in a table.

Finally, you can settle the channel on either side with
```
> close alice
```
which will send one `concludeFinal` and two withdrawal transactions to the chain.

Now you can exit the CLI with command `exit` or `Ctrl+D`.
