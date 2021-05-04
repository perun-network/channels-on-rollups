## Walkthrough

### Deploy the contracts
```sh
git submodule update --init --recursive

cd perun-eth-contracts
yarn
yarn run hardhat run scripts/deploy.js --network <network>
```
Substitute `<network>` with `arbitrum_local` or `optimism_local`.

### Configure the nodes  
Put the printed contracts into `perun-eth-demo/network.yaml`.

### Start the nodes  

Start two nodes in `perun-eth-demo` with:
```sh
go run . --log-level trace demo --config alice.yaml --chain <network>
go run . --log-level trace demo --config bob.yaml --chain <network>
```
Substitute `<network>` with `arbitrum_local` or `optimism_local`.

### Have fun 😄  
   
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
