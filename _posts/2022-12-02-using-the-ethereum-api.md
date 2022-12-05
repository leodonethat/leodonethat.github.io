---
title:  "Using the Ethereum API"
date:   2022-11-30 13:00:00 +0200
toc: true
toc_sticky: true
categories: blockchain
tags: blockchain ethereum development python
published: true
---

# Making sure we have an entry point
There are different way to access a blockchain. For instance, we can
[run our own Ethereum node]({% post_url 2022-11-14-running_our_own_ethereum_node %})
or we can use services like [Infura](https://www.infura.io/), [Alchemy](https://www.alchemy.com/)
, and [QuickNode](https://www.quicknode.com/). Since my objective is to do
analytics, it's better to have the data locally for quick access. It's also going
to be a lot cheaper if I end up making millions of request.

Our starting point are the [Ethereum docs for the JSON-RPC API](https://ethereum.org/en/developers/docs/apis/json-rpc/).
{: .notice--info}

# First tests with curl
Once we have our end point ready, we can start making requests to it. In our case,
our local endpoint will be `127.0.0.1:8545`. Let's try the call `eth_syncing`:

```bash
$ curl -H 'Content-Type: application/json' -X POST --data '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}' 127.0.0.1:8545
```
```
{"jsonrpc":"2.0","id":1,"result":false}
```

This is great, we have the endpoint up and running. Let's see what is our latest
block with the aptly named `eth_blockNumber`

```bash
$ curl -H 'Content-Type: application/json' -X POST --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' 127.0.0.1:8545
```
```
{"jsonrpc":"2.0","id":1,"result":"0xf3cb4f"}
```

Which from hex to decimal is 15977295. Pretty cool!

What about something a little more interesting, maybe `eth_getBlockByNumber`
(with the number 1 as a parameter).

```bash
$ curl -H 'Content-Type: application/json' -X POST --data '{"jsonrpc":"2.0","method":"eth_getBlockByNumber","params":["0x1", true],"id":1}'  127.0.0.1:8545
```

![eth block one]({{ site.url }}{{ site.baseurl }}/assets/images/block1.png){:width="100%"}{:.align-left}

Now we are getting somewhere!

# Getting fancier with Python

Although we can start processing blocks in this way, reading json blobs in the
terminal sounds like a lot of work. It's a lot easier to use a library built for that purpose.
Since I am thinking of doing some data analysis down the road I would like to use python. Googling
a little brings us to [Web3.py A Python library for interacting with Ethereum](https://github.com/ethereum/web3.py),
excellent!

**Note:** I will use a [Jupyter](https://jupyter.org/) notebook instead of Python
IDE or terminal because it's very nice for experimentation and also because I would
like to create charts.

Make sure you have it installed in you machine. You need `python3` and `pip` for it.

```bash
$ pip install jupyterlab
$ jupyter-lab
```

You should now have it running in `http://localhost:8888/lab`.

Once there we can open a Python Notebook.


```python
%pip install Web3

from web3 import Web3
w3 = Web3(Web3.HTTPProvider('http://127.0.0.1:8545'))
w3.isConnected()
```
```bash
True
```

Excellent! now we have access to the blockchain through Python.

To confirm, let's get the Block 0:

```python
w3.eth.get_block(0)
```

```python
AttributeDict({'difficulty': 17179869184,
 'extraData': HexBytes('0x11bbe8db4e347b4e8c937c1c8370e4b5ed33adb3db69cbdb7a38e1e50b1b82fa'),
 'gasLimit': 5000,
 'gasUsed': 0,
 'hash': HexBytes('0xd4e56740f876aef8c010b86a40d5f56745a118d0906a34e69aec8c0db1cb8fa3'),
 'logsBloom': HexBytes('0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000'),
 'miner': '0x0000000000000000000000000000000000000000',
 'mixHash': HexBytes('0x0000000000000000000000000000000000000000000000000000000000000000'),
 'nonce': HexBytes('0x0000000000000042'),
 'number': 0,
 'parentHash': HexBytes('0x0000000000000000000000000000000000000000000000000000000000000000'),
 'receiptsRoot': HexBytes('0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421'),
 'sha3Uncles': HexBytes('0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347'),
 'size': 540,
 'stateRoot': HexBytes('0xd7f8974fb5ac78d9ac099b9ad5018bedc2ce0a72dad1827a1709da30580f0544'),
 'timestamp': 0,
 'totalDifficulty': 17179869184,
 'transactions': [],
 'transactionsRoot': HexBytes('0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421'),
 'uncles': []})
```

This is getting interesting!

We can use `w3.eth.blockNumber` to see what is the latest block available. In my case it's
`15977295` which is a little behind because I don't sync continuously. I am more interested
in doing analytics than interacting with the blockchain in real time.

Here is a snippet to get the most important values of each transaction in that block

```python
blck_content =  w3.eth.get_block(15977295, full_transactions = True)

blck_num = blck_content.get('number')
blck_timestamp = datetime.fromtimestamp(blck_content.get('timestamp'))
blck_date = blck_timestamp.date()
blck_time = blck_timestamp.time()
blck_txns = blck_content.get('transactions')

print(f'blck_num,blck_date,blck_time,txn_idx,txn_gas,txn_gas_price,txn_val,txn_type')

for txn in blck_txns:
  txn_gas = txn.get('gas')
  txn_gas_price = txn.get('gasPrice')
  txn_idx = txn.get('transactionIndex')
  txn_val = txn.get('value')
  txn_type = txn.get('type')

  print(f'{blck_num},{blck_date},{blck_time},{txn_idx},{txn_gas},{txn_gas_price},{txn_val},{txn_type}')
```
```
blck_num,blck_date,blck_time,txn_idx,txn_gas,txn_gas_price,txn_val,txn_type
15977295,2022-11-15,20:43:23,0,218301,1401620818145,15977295,0x2
15977295,2022-11-15,20:43:23,1,191297,377215935888,15977295,0x2
15977295,2022-11-15,20:43:23,2,158330,333812206330,15977295,0x2
15977295,2022-11-15,20:43:23,3,173539,250741356398,15977295,0x2
15977295,2022-11-15,20:43:23,4,333510,108763583475,15977295,0x2
```

From there, we would only need to iterate through all blocks in order to extract information for all transactions.

```python
for blck_idx in range(0, 100000):
  blck_content =  w3.eth.get_block(blck_idx)
```

We have to keep in mind that's a lot of data though. We either need to extract very little if we are going to keep it in memory or post-process it to leave it in an external file o database.


I tried running it for the first 100K blocks and it turns out there aren't that many transactions. Here are the first few:

```
46147,2015-08-07,04:30:33,0,21000,50000000000000,31337,0x0
46169,2015-08-07,04:36:53,0,21000,909808707606,19900000000000000000,0x0
46170,2015-08-07,04:37:10,0,21000,500000000000,599989500000000000000,0x0
46194,2015-08-07,04:43:03,0,21000,1000000000000,100000000000000000000,0x0
46205,2015-08-07,04:46:15,0,21000,500000000000,803989500000000000000,0x0
46214,2015-08-07,04:49:54,0,21750,50000000000000,31337,0x0
46217,2015-08-07,04:50:51,0,21000,65334370444,0,0x0
46219,2015-08-07,04:51:01,0,21800,50000000000000,31337,0x0
46220,2015-08-07,04:51:31,0,21000,64178193561,100000000000000000000,0x0
46230,2015-08-07,04:52:51,0,21000,71288549894,50000000000000000000,0x0
```

Now that we know how to extract data from the blockchain, let's do something fun with it. Instead of printing it to the terminal, we will add each row to a DataFrame.

**Note:** appending rows to a DataFrame one by one is very inefficient. It copies each row, each time. Turning what looks like a linear walk into a quadratic one. One way to approach this issue is to create a list of lists instead and pass them to the DataFrame constructor.

```python
from web3 import Web3
from datetime import datetime
import pandas as pd
import numpy as np

w3 = Web3(Web3.HTTPProvider('http://127.0.0.1:8545'))

txns_list =[]
for blck_idx in range(0, 100000):
  blck_content =  w3.eth.get_block(blck_idx, full_transactions = True)

  blck_num = np.int64(blck_content.get('number'))
  blck_timestamp = datetime.fromtimestamp(blck_content.get('timestamp'))
  blck_date = blck_timestamp.date()
  blck_time = blck_timestamp.time()
  blck_txns = blck_content.get('transactions')

  for txn in blck_txns:
    txn_gas = np.int64(txn.get('gas'))
    txn_gas_price = np.int64(txn.get('gasPrice'))
    txn_idx = np.int64(txn.get('transactionIndex'))
    txn_val = np.int64(Web3.fromWei(txn.get('value'), 'ether')) # in ether not wei!
    txn_type = txn.get('type')

    txn_list = [blck_num, blck_date, blck_time, txn_idx, txn_gas, txn_gas_price, txn_val, txn_type]
    txns_list.append(txn_list)

txns_df = pd.DataFrame(txns_list, columns = ['blck_num', 'blck_date', 'blck_time', 'txn_idx', 'txn_gas', 'txn_gas_price', 'txn_val', 'txn_type'])

txns_df_daily = txns_df.groupby('blck_date')['txn_val'].sum()
txns_df_daily.plot(x = 'blck_date', y = 'txn_val', kind = 'bar')
```

![eth_value_daily]({{ site.url }}{{ site.baseurl }}/assets/images/eth_value_daily.png){:width="100%"}{:.align-center}

It's far from being the most sophisticated graph ever but seeing how much _Ether_
has been transacted on a daily basis is an excellent starting point!
