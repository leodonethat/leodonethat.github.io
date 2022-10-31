---
title:  "Running our own Ethereum node"
date:   2022-10-15 13:00:00 +0200
toc: true
toc_sticky: true
categories: blockchain
tags: blockchain development infrastructure
---

It can feel overwhelming to of running our own node, where do we even start?

I started with the [official documentation](https://ethereum.org/en/run-a-node/) and it took me quite a bit of reading to get a good grasp of the situation. I decided to take notes in case they are helpful for other folks wanted to break into the field.

The first question I had to ask myself is do I really want to install a full node? would a different solution be enough.

The main objective for me installing a node is to learn more about the technology so doing everything from scratch seems like a good way to do so. In addition to learning about it, nother objective I have is to be able to do data analysis with all the information publicly available on the blockchain. It seems ideal to be able to download a full history of transactions and play with them.

# Options
There are other ways to get access to the blockchain besides running your own node. If the amount of data you want to process is small or you have a very well defined use case you want to try I probably wouldn't recommend running your own node. To make development faster I would use one of the companies specializing in blockchain infrastructure or I would run a managed node on the cloud. I don't have much experience with them so I won't live a list pros and cons. I will only mention a few: alchemy, infura, quicknode and blockdaemon. Note that at the time of the writing, alchemy is valued at 10B and aims to become the aws of blockchain. These aren't really small companies anymore.

Coming back to our humble objective: run an etheruem node and do some interesting data analysis.

I learned there are [three different types of nodes](https://ethereum.org/en/developers/docs/nodes-and-clients/) you can run:
* **Light**: this node downloads only block headers with a summary of the contents in a block. It can't participate in consensus and still needs to connect to a full node to get its information. This option doesn't look very interesting.
* **Full**: this is closer to what we were looking for. It participates in block validation, provides data on request and stores blockchain data. The caveat with this type of node is that it doesn't store a full record of transactions, it only stores the most recent 128 blocks. We could get started with this one but the data analysis would be limited to only the last 128 blocks plus if those blocks keep changing our results won't be easily reproducible.
* **Archive**: a full node with the complete history of the blockchain. This is exactly what we were looking for! The complete set of transaction in ethereum since the very beginning.

# Blockchain clients

Now we know we want our own Ethereum Archive. I don't know yet how we will extract or process the data but we will cross that bridge when we get there. For the moment, let's download our archive!

Here we get to our next decision, which client do we use? it turns out there are [several of them](https://ethereum.org/en/developers/docs/nodes-and-clients/#execution-clients).

[Erigon](https://github.com/ledgerwatch/erigon) seems to do a really good use of space and can store the whole archive in +2TB while [GETH](https://github.com/ethereum/go-ethereum) needs +10TB.

I decided to go with Erigon and will install that one.

A small complication here is that my laptop doesn't have enough space to store the extra terabytes needed. I did some reseach and found a really nice portable drive on amazon: [sandisk extreme portable ssd 4TB](https://www.amazon.com/SanDisk-4TB-Extreme-Portable-SDSSDE61-4T00-G25/dp/B08RX4QKXS). It's the biggest external hard drive I have found so far (4TB) and I am extreamly impressed by its physcial size. It's tiny! I didn't pay attention when I ordered it and was so surprised when I received it. It's about 10cm long and 1cm thick.

---

## Execution Layer (Erigon)

Now that we have chosen a client and have a place to put all the data let's give it a try!

A good place to start is the [readme of the project Erigon](https://github.com/ledgerwatch/erigon).

Walkthroughs found online:
* [Getting Started with Erigon on Ubuntu](https://chasewright.com/getting-started-with-turbo-geth-on-ubuntu/)
* [Setting up a Full Erigon Ethereum Node](https://surfthedream.com.au/setting-up-a-full-erigon-ethereum-node/)
* [Running an Erigon Archive Node](https://magnushansson.xyz/blog_posts/crypto_defi/2021-12-27-Run-Erigon-Archive-Node)

### Installing Erigon

Things you need:
* git
* Xcode Command Line Tools installed
* Go

[Install Xcode Command Line Tools with Homebrew](https://mac.install.guide/commandlinetools/3.html)

``` bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

```


``` bash
git clone --recurse-submodules -j8 https://github.com/ledgerwatch/erigon.git
cd erigon
make erigon
./build/bin/erigon --datadir mainnet
```

``` bash
INFO[09-21|16:49:03.690] Build info                               git_branch=HEAD git_tag=v2022.09.01-dirty git_commit=4067b7c4da6c5d741d3027d95ae2afdf6b7a943a
INFO[09-21|16:49:03.691] Starting Erigon on Ethereum mainnet...
INFO[09-21|16:49:03.694] Maximum peer count                       ETH=100 total=100
INFO[09-21|16:49:03.694] starting HTTP APIs                       APIs=eth,erigon,engine
INFO[09-21|16:49:03.694] torrent verbosity                        level=WRN
INFO[09-21|16:49:05.796] Set global gas cap                       cap=50000000
INFO[09-21|16:49:05.884] Opening Database                        
```

Here we buckle up! it took quite a while to download 220GB. Unfortunately, the process wasn't completely smooth and there were some errors

``` bash
EROR[09-21|06:04:08.060] Staged Sync                              err="[1/15 Headers] BuildMissedIndices: TransactionsIdx: at=7000000-7500000, runtime error: slice bounds out of range [:-1], [block_snapshots.go:1537 panic.go:890 panic.go:139 decompress.go:523 block_snapshots.go:1518 block_snapshots.go:1541 block_snapshots.go:760 block_snapshots.go:801 asm_arm64.s:1165]"
INFO[09-21|06:04:08.561] [Snapshots] Fetching torrent files metadata
INFO[09-21|06:04:08.971] [Snapshots] Stat                         blocks=6500k indices=6500k alloc=5.9GB sys=9.0GB
EROR[09-21|06:04:09.010] Staged Sync                              err="[1/15 Headers] BuildMissedIndices: TransactionsIdx: at=7000000-7500000, runtime error: slice bounds out of range [:-1], [block_snapshots.go:1537 panic.go:890 panic.go:139 decompress.go:523 block_snapshots.go:1518 block_snapshots.go:1541 block_snapshots.go:760 block_snapshots.go:801 asm_arm64.s:1165]"
```

I asked in the Erigon discord and learned a couple of new commands:

``` bash
make downloader
./build/bin/downloader torrent_hashes --verify --datadir=mainnet
```

Which verifies the download and gave me more details of the errors and another command to run the download with it in place

``` bash
./build/bin/erigon --downloader.verify --datadir mainnet
```

Weirdly, that one ran for a few hours and gave me another strange error but I ran it again and it managed to finish this time 🥳

## Consensus Layer (Teku)

 I thought that would be enough to get the client running but I also learned things have changed with the Merge and I now needed a Consensus Layer. I didn't know what that was so I asked for pointers and Alex from Erigon shared the following links:

https://ethereum.org/en/upgrades/get-involved/#clients
https://ethereum.org/en/developers/docs/nodes-and-clients/#consensus-clients
https://ethereum.org/en/developers/docs/nodes-and-clients/client-diversity/#consensus-clients

The gist of it is that a "full node requires running a pair of these clients: an execution layer client and a consensus layer client".

I found this article very helpful: [Execution Layer (EL) and Consensus Layer (CL) Node Clients (2022)](https://www.alchemy.com/overviews/execution-layer-and-consensus-layer-node-clients)

The short story here is that the old self-contained Erigon client has now become the execution layer and in addition to it we now need to install a second client to run the consensus layer.

---

### Installing Teku

In the previous section we installed Erigon, which runs the execution layer. Now we will complement this with a second client running the consensus layer.

There are several options at the moment.

* [Lighthouse](https://github.com/sigp/lighthouse)
* [Nimbus](https://github.com/status-im/nimbus-eth2)
* [Teku](https://consensys.net/knowledge-base/ethereum-2/teku/)
* [Prysm](https://github.com/prysmaticlabs/prysm)
* [Lodestar](https://lodestar.chainsafe.io/)

I decided to give Teku a try because is written in Java (a language I speak), it's open source and it's developed by ConsenSys.

[Installing Teku from binaries](https://docs.teku.consensys.net/en/latest/HowTo/Get-Started/Installation-Options/Install-Binaries/)


``` bash
brew tap ConsenSys/teku
brew install ConsenSys/teku/teku
```

Command to start Start teku:

``` bash
teku \
    --ee-endpoint=http://localhost:8551 \
    --ee-jwt-secret-file=/Users/leodonethat/eth/erigon/mainnet/jwt.hex \
    --metrics-enabled=true \
    --rest-api-enabled=true \
    --data-base-path=/Users/leodonethat/eth/teku/ \
    --data-storage-mode=archive \
    --network=mainnet
```

We realize we don't have a file `jwtsecret.hex` yet.

It turns out Erigon automatically creates one with name `jwt.hex` in the datadir directory so we will try using that one. Let's try launching Erigon and then Teku


``` bash
/Users/leodonethat/eth/build/bin/erigon --datadir mainnet --http.api=eth,erigon,web3,net,debug,trace,txpool
```

**Note:** Erigon has a number of APIs available, we have to make sure to include the one we need when launching it or we will get an error "the method does not exist/is not available" when making calls from different APIs.

``` bash
teku \
    --ee-endpoint=http://localhost:8551 \
    --ee-jwt-secret-file=/Users/leodonethat/eth/erigon/mainnet/jwt.hex \
    --metrics-enabled=true \
    --rest-api-enabled=true \
    --data-base-path=/Users/leodonethat/eth/teku/ \
    --data-storage-mode=archive \
    --network=mainnet
```

At this point we have our Ethereum node up and running 🥳. The next step is waiting for it to sync. Be ready for it to take a veeeeeeeery long time. We are talking about days if you have a solid internet connection.

We can use the `eth_syncing` command to see how we are doing. Here we note that we are currently at node `0x0`. This is definetly take a while.


``` bash
% curl -H 'Content-Type: application/json' -X POST http://localhost:8545 -d  '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":51}'

{"jsonrpc":"2.0","id":51,"result":{"currentBlock":"0x0","highestBlock":"0xe5ab0f","stages":[{"stage_name":"Headers","block_number":"0xe5ab0f"},{"stage_name":"BlockHashes","block_number":"0xe4e1bf"},{"stage_name":"Bodies","block_number":"0xe4e1bf"},{"stage_name":"Senders","block_number":"0xe4e1bf"},{"stage_name":"Execution","block_number":"0x0"},{"stage_name":"Translation","block_number":"0x0"},{"stage_name":"HashState","block_number":"0x0"},{"stage_name":"IntermediateHashes","block_number":"0x0"},{"stage_name":"AccountHistoryIndex","block_number":"0x0"},{"stage_name":"StorageHistoryIndex","block_number":"0x0"},{"stage_name":"LogIndex","block_number":"0x0"},{"stage_name":"CallTraces","block_number":"0x0"},{"stage_name":"TxLookup","block_number":"0x0"},{"stage_name":"Finish","block_number":"0x0"}]}}
```

Once the node finishes syncing we will be able to interact with the Ethereum blockchain. I suggest reading more about the [JSON-RPC interface](https://ethereum.org/en/developers/docs/apis/json-rpc/) to see what commands are available.
