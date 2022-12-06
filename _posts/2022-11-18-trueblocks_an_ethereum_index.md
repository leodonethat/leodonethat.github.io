---
title:  "TrueBlocks - an Ethereum index"
date:   2022-11-14 13:00:00 +0200
toc: true
toc_sticky: true
categories: blockchain
tags: blockchain development infrastructure
published: true
---

Something a little surprising is the fact that running our own Ethereum node
(like Erigon) is not enough for us to easily access the data.

The rpcdaemon is enough in the sense that we do get access to it but the issue
is that the [api](https://ethereum.org/en/developers/docs/apis/json-rpc/) isn't.
For instance, there is no way to get a list of all the transactions given an
account number. To do that, we would have to walk through block by block
and filtering the transactions for that particular account. Depending on our
use case that's totally fine of course. Assuming we do want an easy way to access
all transactions of a given account then there is a better way. TrueBlock creates
this index of accounts for us.

# Installing TrueBlocks

We will follow the [instruction on their main page](https://trueblocks.io/docs/install/install-trueblocks/)

``` bash
% go version
go version go1.19 darwin/arm64
```

Our version is >=1.16 ✔️

We check for the dependencies

``` bash
brew install cmake ninja
brew install git
brew install clang-format
brew install jq
```

We get the repo from GitHub and build it

``` bash
cd eth
git clone -b develop https://github.com/trueblocks/trueblocks-core
cd trueblocks-core
git checkout develop
mkdir build && cd build
cmake ../src
make
```

We add the bin directory to the path

``` bash
PATH="/Users/leodonethat/eth/trueblocks-core/bin:${PATH}"
export PATH
```

By defaul, the rpc provider is our local machine in port 8045. Which is where the
erigon rpcdaemon runs in my case and will be yours if you followed the same steps.
In case it's not, you have to edit the config file to add our rpc daemon. Please
note adding and external provider (like Infura) might get pricey as the indexer
will make many many calls.
* Location: `~/Library/Application\ Support/TrueBlocks/trueBlocks.toml`
* Param: `rpcProvider = "http://localhost:8545`

# Testing TrueBlocks

```bash
% chifra blocks 12345

Could not establish ts file: while calling contract: Post "http://localhost:8545": dial tcp 127.0.0.1:8545: connect: connection refused

  Warning: The RPC server (http://localhost:8545) was not found. Either start it, or edit the rpcProvider
  value in the file /Users/leodonethat/Library/Application Support/TrueBlocks/trueBlocks.toml. Quitting...
```

Well, I forgot to launch Erigon. Let's do that in a different window and
try again.

``` bash
% cd eth/erigon
erigon % ./build/bin/erigon --datadir mainnet --http.api=eth,erigon,web3,net,debug,trace,txpool
```

``` bash
% cd eth/trueblocks-core
trueblocks-core % chifra blocks 12345
{ "data": [
{
  "gasLimit": 5000,
  "gasUsed": 0,
  "hash": "0x767c2bfb3bdee3f78676c1285cd757bcd5d8c272cef2eb30d9733800a78c0b6d",
  "blockNumber": 12345,
  "parentHash": "0x4b3c1d7e65a507b62734feca1ee9f27a5379e318bd52ae62de7ba67dbeac66a3",
  "miner": "0xad5c1768e5974c231b2148169da064e61910f31a",
  "difficulty": 735512610763,
  "finalized": true,
  "timestamp": 1438367030,
  "baseFeePerGas": 0
}] }
```

Excellent!! 🥳

Once we now it's working we go ahead and run a full index.

```bash
trueblocks-core % chifra scrape
INFO[14-11|18:08:32.876] Writing block zero allocations for 8893 allocs, nAddresses: 8893
INFO[14-11|18:08:32.901] Wrote 8893 address and 8893 appearance records to $INDEX/000000000-000000000.bin (snapped to grid)
INFO[14-11|18:08:51.775] Block=2000 have 2535 appearances of 2000000 (0.1%). Need 1997465 more. Added 2535 records (1.27 apps/blk).
INFO[14-11|18:08:56.875] Block=4000 have 4917 appearances of 2000000 (0.2%). Need 1995083 more. Added 2382 records (1.19 apps/blk).
...
```

# Example with a known address

After that we can start playing playing with the data and get all transactions associated with an address:

```bash
% chifra export 0xab5801a7d398351b8be11c439e05c5b3259aec9b
```

That address has lots of transactions though so be prepared for quite a bit of data.

![vb_address_transactions]({{ site.url }}{{ site.baseurl }}/assets/images/vb_address_transactions.png){:width="100%"}{:.align-center}
