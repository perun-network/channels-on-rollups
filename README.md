<h1 align="center"><br>
    <a href="https://perun.network/"><img src=".assets/proll.png" alt="Perun" width="196"></a>
<br></h1>

<h4 align="center">PRoll: Perun channels on Rollups 🎢<br>
<i>Towards fast, cheap, and versatile transactions on and across rollups.</i></h4>
</p>

# Intro

This repository is our submission to **Scaling Ethereum 2021**.  

Our team consists of developers of the state channel library [*go-perun*](https://github.com/hyperledger-labs/go-perun) from [*PolyCrypt*](https://polycry.pt). Our goal was to experiment with rollup technology and to see if we can make our library work with them and how much gas we can save. We focus on the EVM-compatible rollups Arbitrum and Optimism.

Our submission consists of the following components:

* **Contracts:** We modified the go-perun contracts and prepared deployment scripts that allow us to deploy to either a standard Ethereum blockchain, an Arbitrum Rollup, or an Optimism Rollup by just changing a single command line argument. 
* **Demo**: We modified the go-perun demo client to work with Rollups. This involved implementing ERC20 functionality and incorporating a new command line switch for selecting the network that we want to run on. In addition, we had to take care of some specialties of Rollups like setting the correct gas price and making sure that we do not use ETH functionality.
* **Library**: We needed to adapt the go-perun library in a few ways to make it work with Rollups. For example, we could not rely on estimating the gas correctly and instead needed to change our configuration accordingly.

## Security Disclaimer
The authors take no responsibility for any loss of digital assets or other damage caused by the use of this software.  
**Do not use this software with real funds**.

# Walkthrough

## Download source code

Clone the repository and initialize the submodules repository.

```sh
git clone https://github.com/perun-network/channels-on-rollups
cd channels-on-rollups
git submodule update --init --recursive
```

## Start your local blockchain
### Arbitrum

https://developer.offchainlabs.com/docs/local_blockchain

We assume that the node is listening at `127.0.0.1:8547` for http and at `127.0.0.1:8548` for websocket.  
For testing you can also use the *Kovan4* testnet, configuration files are included.

### Optimism
https://community.optimism.io/docs/developers/integration.html#step-2-testing-contracts

We assume that the node is listening at `127.0.0.1:8545` for http and at `127.0.0.1:8546` for websocket.  
For testing you can also use the *optimism-Kovan* testnet, configuration files are included.

## Deploy the contracts
```sh
cd perun-eth-contracts
yarn
yarn run hardhat proll-deploy --network <network> --filename <network>_contracts.json
```
Replace `<network>` with `arbitrum_local`, or `optimism_local`.  
This will write the deployed contract addresses to `<network>_contracts.json` for later use.

Account index 0 and 1 are pre-funded with `PerunToken`. You may need to change these indices if you use more than two clients.  
Pre-funding more than two addresses does not work in optimism due to gas limit restrictions.

## Configuration

You can either set the contract addresses manually in the `perun-eth-demo/network.yaml`
file or let the demo fetch them from the JSON file that has been created in the 
previous step using the `--contracts` flag when starting the clients.  

➡ One thing that you do need to do manually, is to copy the `PerunToken.assetholder` address of *optimism* into the `PerunToken.alternative` address of *arbitrum* and vice versa.  
Otherwise you will get an error about an "unknown asset".

## Start the Hubs

Start two hubs in `perun-eth-demo` with:
```sh
go run . demo --config hub1.yaml --contracts ../perun-eth-contracts/<network>_contracts.json --chain <network>
go run . demo --config hub2.yaml --contracts ../perun-eth-contracts/<network>_contracts.json --chain <network>
```
Replace `<network>` with `arbitrum_local` or `optimism_local`.

## Start the Clients

Start two clients in `perun-eth-demo` with:
```sh
go run . demo --config alice.yaml --contracts ../perun-eth-contracts/<network>_contracts.json --chain <network>
go run . demo --config bob.yaml --contracts ../perun-eth-contracts/<network>_contracts.json --chain <network>
```
Replace `<network>` with `arbitrum_local` or `optimism_local`.

You should now have 4 CLI windows running;  
- Hub1 (Optimism)
- Alice (Optimism)
- Hub2 (Arbitrum)
- Bob (Arbitrum)

## Open ledger channels
First open a ledger channel between the *optimism* *hub1* and *alice* by running this in *alice* CLI:
```
> open optimism peruntoken 10 10
```
In *hub1*'s terminal, accept the appearing channel proposal.
```
🔁 Incoming channel proposal from alice with funding [My: 10 PRN, Peer: 10 PRN].
Accept (y/n)? > y
```

Then open a ledger channel between the *arbitrum* *hub2* and *bob* by running this in *bob* CLI:
```
> open arbitrum peruntoken 10 10
```
In *hub2*'s terminal, accept the appearing channel proposal.
```
🔁 Incoming channel proposal from bob with funding [My: 10 PRN, Peer: 10 PRN].
Accept (y/n)? > y
```
Pay attention to the channel ID that is printed in *bob*s CLI and copy it:
```
✅ Opened channel 0x123456789…
```

## Open virtual channel

Run in *alice* terminal:
```sh
> openv bob optimism 0x123456789… peruntoken 7 7
```
Replace the id here with the one that you copied from above.  
➡  Using the `openv` command opens a *virtual* state channel, in contrast to *open* which just opens a *ledger* channel.

## Use the channel 😄

Now you can execute off-chain payments, e.g. in *bob*'s terminal with
```
> send alice 1
```
The updated balance will immediately be printed in both terminals, but no
transaction will be visible on-chain.

You may always check the current status with `info peruntoken`.

You can also run a performance benchmark with command
```
> benchmark alice 5 100
```
which will send 5 *PerunToken* in 100 micro-transactions from Bob to Alice. Transaction performance will be printed in a table.

To close the channel, *alice* does:
```
> close bob
```
and *bob*:
```
> close alice
```

Now you can exit the CLI with command `exit` or `Ctrl+D`.


## Contact

You can get in touch with the hackers directly:
- Matthias Geihs [mail](mailto:matthias@perun.network), [github](https://github.com/matthiasgeihs)
- Philipp-Florens Lehwalder [mail](mailto:philipp@perun.network), [github](https://github.com/cryptphil)
- Oliver Tale-Yazdi [mail](mailto:oliver@perun.network), [github](https://github.com/ggwpez)

or for business inquiries please contact us at [info@perun.network](mailto:info@perun.network) or on [perun.network](https://perun.network).
