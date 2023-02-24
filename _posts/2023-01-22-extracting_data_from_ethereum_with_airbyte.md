---
title:  "Using Airbyte to extract data from Ethereum"
date:   2023-01-22 13:00:00 +0200
toc: true
toc_sticky: true
categories: blockchain
tags: blockchain ethereum development data stack
published: true

gallery:
  - url: /assets/images/airbyte_connector.png
    image_path: /assets/images/airbyte_connector.png
    alt: "airbyte connector"
    title: "airbyte connector"
  - url: /assets/images/airbyte_source.png
    image_path: /assets/images/airbyte_source.png
    alt: "airbyte source"
    title: "airbyte source"
---

We have seen how to [run our own Ethereum node](({% post_url 2022-11-14-running_our_own_ethereum_node %}))
and how to [access it through its API](({% post_url 2022-12-02-using-the-ethereum-api %})). In that scenario,
we used Python to make API calls and saw the output in the terminal. An alternative to writing custom python
scripts for data ingestion is to use a third-party tool.

We can imagine how convenient would be to delegate all data engineering work.
When focusing on data analytics, we want to get our hands on the data as quickly as possible. Fortunately for us,
a few of these tools have emerged as part of the _Modern Data Stack_.

The extraction tool I have used the most and that seems to be leading the pack in terms of integrations is [Fivetran](https://www.fivetran.com/) (valued at  $5.6B in their last round). With very little effort, we can cover the **EL** part of an **EL**T pipeline. We can Extract
data from a source and Load it into a data warehouse. Fivetran is extremely convenient if we want to use a popular source with an
[existing connector](https://www.fivetran.com/connectors). Unfortunately, there isn't much we can do if there isn't a connector
for the data source we want. Since Fivetran is fully managed and cloud-based, there is no way for us to make any changes or modifications to it.
An extra consideration is that it can get pretty pricey, although from personal experience the price is still much lower than
investing data engineering resources to do the same.

In addition to fully-managed platforms, there are open-source alternatives operating in the same space. [Meltano](https://github.com/meltano/meltano) and [Airbyte](https://github.com/airbytehq/airbyte) are the ones I am familiar with. Out of these two, Airbyte has a little more
traction and a more user-friendly interface. They also have a [cloud-based offer](https://airbyte.com/pricing) competing toe-to-toe with Fivetran (helped by a series B round that valued then at $1.5B).

# Airbyte as our data ingestion platform

After considering our options above, we decide to go with Airbyte:
* Open-source alternative
* Solid company financials (raised $150M in their series B)
* Energetic community (8.9K GitHub stars, 300+ connectors)
* Built-in scheduler
* User-friendly interface
* Connectors can be written in any language

Although there is no pre-existing connector to extract data from a blockchain, Airbyte has some great [documentation to write our own connector](https://docs.airbyte.com/connector-development/).

# Getting started

```bash
git clone https://github.com/airbytehq/airbyte.git
cd airbyte
docker-compose up
```

If everything went well you will see a message saying Airbyte is ready at http://localhost:8000/. When you head there, use `airbyte` for the username and `password` for the password. This is how this first interface looks to me:

![airbyte_start]({{ site.url }}{{ site.baseurl }}/assets/images/airbyte_start.png){:width="100%"}{:.align-center}

# Creating a new connector

We have our Airbyte instance ready to go but unfortunately no connectors for our Ethereum node. Here is where the fun starts!

* [Connector Development](https://docs.airbyte.com/connector-development/)
* [Connector Development Kit (Python)](https://docs.airbyte.com/connector-development/cdk-python/)
* [Airbyte Protocol](https://docs.airbyte.com/understanding-airbyte/airbyte-protocol/)
* [cdk tutorial python http](https://docs.airbyte.com/connector-development/tutorials/cdk-tutorial-python-http/getting-started/)

Start a new branch in the git repository (I like the shortcuts from [git town](https://www.git-town.com/))

```bash
airbyte $ git hack add/airbyte-source-ethereum
```

> [master] git fetch --prune --tags
[master] git rebase origin/master
[master] git branch add/airbyte-source-ethereum master
[master] git checkout add/airbyte-source-ethereum
Switched to branch 'add/airbyte-source-ethereum'

We then follow the instructions from Airbyte:

```bash
cd airbyte-integrations/connector-templates/generator
./generate.sh
```

![airbyte_generator]({{ site.url }}{{ site.baseurl }}/assets/images/airbyte_generator.png){:width="100%"}{:.align-center}


Install dependencies

```bash
cd ../../connectors/source-ethereum-api
python -m venv .venv # Create a virtual environment in the .venv directory
source .venv/bin/activate # enable the venv
pip install -r requirements.txt
```

```bash
python main.py spec
```
> {"type": "SPEC", "spec": {"documentationUrl": "https://docsurl.com", "connectionSpecification": {"$schema": "http://json-schema.org/draft-07/schema#", "title": "Ethereum Api Spec", "type": "object", "required": ["TODO"], "properties": {"TODO": {"type": "string", "description": "describe me"}}}}}

We have to remember all the boilerplate code was generated in `airbyte-integrations/connectors/source-ethereum-api`. The README file in that directory is a great start. We have to learn how to run the connector locally:

```bash
python main.py spec
python main.py check --config sample_files/config.json
python main.py discover --config sample_files/config.json
python main.py read --config sample_files/config.json --catalog integration_tests/configured_catalog.json
```

## First steps using the Ethereum API

Of course, we don't get very far if we run these commands using only the code skeleton. For it to do anything useful we have to add our own logic. In this case, the files we have to edit are:

```
configured_catalog.json
source_ethereum_api/schemas/ethereum_api.json
source_ethereum_api/spec.yaml
source_ethereum_api/source.py
```

Something tricky in this case is the fact the Ethereum API expects a post request with the methods and params in the body of the request (versus a get request). The method we have to overload in Airbyte is called `request_body_json()`

As a first test, I hardcoded the params of this request:

```python
@property
def http_method(self) -> str:
    return "POST"

def request_body_json(
    self,
    stream_state: Mapping[str, Any] = None,
    stream_slice: Mapping[str, Any] = None,
    next_page_token: Mapping[str, Any] = None,
) -> Optional[Union[Dict[str, Any], str]]:
    """
    curl -H 'Content-Type: application/json' -X POST
            --data '{"jsonrpc":"2.0","method":"eth_getBlockByNumber",
            "params":["-0x0", true],"id":1}'  127.0.0.1:8545
    """
    data = {
        "jsonrpc": "2.0",
        "method": "eth_getBlockByNumber",
        "params": ["0x0", True],
        "id":1,
    }
    return data
```

And then tried running this connector locally:

```bash
python main.py read --config sample_files/config.json --catalog configured_catalog.json
```

```
{"type": "LOG", "log": {"level": "INFO", "message": "Starting syncing SourceEthereumApi"}}
{"type": "LOG", "log": {"level": "INFO", "message": "Syncing stream: ethereum_api "}}
{"type": "RECORD", "record": {"stream": "ethereum_api", "data": {"jsonrpc": "2.0", "id": 1, "result": {"difficulty": "0x400000000", "extraData": "0x11bbe8db4e347b4e8c937c1c8370e4b5ed33adb3db69cbdb7a38e1e50b1b82fa", "gasLimit": "0x1388", "gasUsed": "0x0", "hash": "0xd4e56740f876aef8c010b86a40d5f56745a118d0906a34e69aec8c0db1cb8fa3", "logsBloom": "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000", "miner": "0x0000000000000000000000000000000000000000", "mixHash": "0x0000000000000000000000000000000000000000000000000000000000000000", "nonce": "0x0000000000000042", "number": "0x0", "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000", "receiptsRoot": "0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421", "sha3Uncles": "0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347", "size": "0x21c", "stateRoot": "0xd7f8974fb5ac78d9ac099b9ad5018bedc2ce0a72dad1827a1709da30580f0544", "timestamp": "0x0", "totalDifficulty": "0x400000000", "transactions": [], "transactionsRoot": "0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421", "uncles": []}}, "emitted_at": 1674022756600}}
{"type": "LOG", "log": {"level": "INFO", "message": "Read 1 records from ethereum_api stream"}}
{"type": "LOG", "log": {"level": "INFO", "message": "Finished syncing ethereum_api"}}
{"type": "LOG", "log": {"level": "INFO", "message": "SourceEthereumApi runtimes:\nSyncing stream ethereum_api 0:00:00.006586"}}
{"type": "LOG", "log": {"level": "INFO", "message": "Finished syncing SourceEthereumApi"}}
```

## Updating the logic

After a first try connecting to the Ethereum API from Airbyte, we take a step back to thik about how we are going to implement the full connector. A great place to look for inspiration are the Python tutorials:

* [Python CDK Speedrun: Creating a Source](https://docs.airbyte.com/connector-development/tutorials/cdk-speedrun/)
* [Python CDK: Creating a HTTP API Source](https://docs.airbyte.com/connector-development/tutorials/cdk-tutorial-python-http/getting-started/)

The important part here is the implementation of the incremental sync in [this example](https://docs.airbyte.com/connector-development/tutorials/cdk-tutorial-python-http/read-data).

Here is a link to the source.py with the logic. The gist of it is that we will download data one block at a time. Taking advantage
of the `eth_getBlockByNumber` API call and remembering which one was the last block downloaded so that we can
do incremental updates.

```bash
python main.py read --config sample_files/config.json --catalog configured_catalog.json
```

```json
{"type": "LOG", "log": {"level": "INFO", "message": "Starting syncing SourceEthereumApi"}}
{"type": "LOG", "log": {"level": "INFO", "message": "Syncing stream: ethereum_api "}}
{"type": "RECORD", "record": {"stream": "ethereum_api", "data": {"jsonrpc": "2.0", "id": 1, "result": {"difficulty": "0x400000000", "extraData": "0x11bbe8db4e347b4e8c937c1c8370e4b5ed33adb3db69cbdb7a38e1e50b1b82fa", "gasLimit": "0x1388", "gasUsed": "0x0", "hash": "0xd4e56740f876aef8c010b86a40d5f56745a118d0906a34e69aec8c0db1cb8fa3", "logsBloom": "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000", "miner": "0x0000000000000000000000000000000000000000", "mixHash": "0x0000000000000000000000000000000000000000000000000000000000000000", "nonce": "0x0000000000000042", "number": "0x0", "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000", "receiptsRoot": "0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421", "sha3Uncles": "0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347", "size": "0x21c", "stateRoot": "0xd7f8974fb5ac78d9ac099b9ad5018bedc2ce0a72dad1827a1709da30580f0544", "timestamp": "0x0", "totalDifficulty": "0x400000000", "transactions": [], "transactionsRoot": "0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421", "uncles": []}}, "emitted_at": 1674205296618}}
{"type": "STATE", "state": {"type": "STREAM", "stream": {"stream_descriptor": {"name": "ethereum_api"}, "stream_state": {"number": 0}}, "data": {"ethereum_api": {"number": 0}}}}
{"type": "RECORD", "record": {"stream": "ethereum_api", "data": {"jsonrpc": "2.0", "id": 1, "result": {"difficulty": "0x3ff800000", "extraData": "0x476574682f76312e302e302f6c696e75782f676f312e342e32", "gasLimit": "0x1388", "gasUsed": "0x0", "hash": "0x88e96d4537bea4d9c05d12549907b32561d3bf31f45aae734cdc119f13406cb6", "logsBloom": "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000", "miner": "0x05a56e2d52c817161883f50c441c3228cfe54d9f", "mixHash": "0x969b900de27b6ac6a67742365dd65f55a0526c41fd18e1b16f1a1215c2e66f59", "nonce": "0x539bd4979fef1ec4", "number": "0x1", "parentHash": "0xd4e56740f876aef8c010b86a40d5f56745a118d0906a34e69aec8c0db1cb8fa3", "receiptsRoot": "0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421", "sha3Uncles": "0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347", "size": "0x219", "stateRoot": "0xd67e4d450343046425ae4271474353857ab860dbc0a1dde64b41b5cd3a532bf3", "timestamp": "0x55ba4224", "totalDifficulty": "0x7ff800000", "transactions": [], "transactionsRoot": "0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421", "uncles": []}}, "emitted_at": 1674205296619}}
{"type": "STATE", "state": {"type": "STREAM", "stream": {"stream_descriptor": {"name": "ethereum_api"}, "stream_state": {"number": 1}}, "data": {"ethereum_api": {"number": 1}}}}
{"type": "RECORD", "record": {"stream": "ethereum_api", "data": {"jsonrpc": "2.0", "id": 1, "result": {"difficulty": "0x3ff001000", "extraData": "0x476574682f76312e302e302d30636463373634372f6c696e75782f676f312e34", "gasLimit": "0x1388", "gasUsed": "0x0", "hash": "0xb495a1d7e6663152ae92708da4843337b958146015a2802f4193a410044698c9", "logsBloom": "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000", "miner": "0xdd2f1e6e498202e86d8f5442af596580a4f03c2c", "mixHash": "0x2f0790c5aa31ab94195e1f6443d645af5b75c46c04fbf9911711198a0ce8fdda", "nonce": "0xb853fa261a86aa9e", "number": "0x2", "parentHash": "0x88e96d4537bea4d9c05d12549907b32561d3bf31f45aae734cdc119f13406cb6", "receiptsRoot": "0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421", "sha3Uncles": "0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347", "size": "0x220", "stateRoot": "0x4943d941637411107494da9ec8bc04359d731bfd08b72b4d0edcbd4cd2ecb341", "timestamp": "0x55ba4241", "totalDifficulty": "0xbfe801000", "transactions": [], "transactionsRoot": "0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421", "uncles": []}}, "emitted_at": 1674205296621}}
```

## Creating the docker image

Our source connector will be its own image so we have to take a moment to create it. Note this has to be executed in the connector main directory as it reads from its own Dockerfile.

```
docker build . -t airbyte/source-ethereum-api:dev
```

After the image is built, we test our connector through it.

```
docker run --rm airbyte/source-ethereum-api:dev spec
```

Note: a docker image is a like having a virtual computer running. The programs running from docker do not share the same network as the local machine. If we execute the commands above without any changes things won't work. There are two things we have to change:

1. When running `erigon` we need an extra param: `--http.vhosts="*"`
2. When running our source connector in the docker image, instead of `http//localhost` we use `http://host.docker.internal`

By this point we should be getting the same results we got above with `python main.py read`.

# Adding our connector to the Airbyte UI

We are almost there!

Now we follow the steps to [add the connector to the API/UI](https://docs.airbyte.com/connector-development/tutorials/building-a-python-source/#step-11-add-the-connector-to-the-apiui)

Note the fields we specified in the spec.yaml file are the fields that will appear in the ui fields.

{% include gallery layout="half" %}

# Adding a destination
Once we have our source connector, we have to tell Airbyte where to put the data it's extracting. We are going to start as simply as possible with a CSV file

## Local CSV file

![airbyte_local_csv]({{ site.url }}{{ site.baseurl }}/assets/images/airbyte_local_csv.png){:width="100%"}{:.align-center}
<br>
![airbyte_local_csv_success]({{ site.url }}{{ site.baseurl }}/assets/images/airbyte_local_csv_success.png){:width="100%"}{:.align-center}

Sync Succeeded!! 🥳

If we take a look a the details of the sync we will find the path to the file.

![airbyte_local_csv_example]({{ site.url }}{{ site.baseurl }}/assets/images/airbyte_local_csv_example.png){:width="100%"}{:.align-center}

Here we have a csv file with three fields:
* `_airbyte_ab_id`
* `_airbyte_emitted_at`
* `_airbyte_data`

Where `_airbyte_data` is a json blob for each Ethereum block!

## Postgres database

The Local CSV file is a great start but is not very powerful. We can take things to the next level by using a database like postgres as our destination.

To give it a try, start by running a local instance of the database (docker is great in this case). Once that is running we go back to the Airbyte UI and create a new connection. With `Ethereum API` as the source and `Postgres` as the destination.

![airbyte_connection_postgres]({{ site.url }}{{ site.baseurl }}/assets/images/airbyte_connection_postgres.png){:width="100%"}{:.align-center}

Note that in addition to the `Raw data (JSON)` we have selected [Normalized Tabular Data](https://docs.airbyte.com/understanding-airbyte/basic-normalization/). With this option, Airbyte will process the blob in `_airbyte_data` and turn it into its own table.

If everything goes well we should see another Sync Success!! 🥳

![airbyte_postgres_success]({{ site.url }}{{ site.baseurl }}/assets/images/airbyte_postgres_success.png){:width="100%"}{:.align-center}

Now we can use a client like [Postico](https://eggerapps.at/postico2/) and check the results of the Sync:

![airbyte_postico_results]({{ site.url }}{{ site.baseurl }}/assets/images/airbyte_postico_results.png){:width="100%"}{:.align-center}


# Processing Ethereum blocks

Once we have the blocks in a database table, we can use sql for any kind of analysis. Imagine a much (much!) younger brother of [Dune Analytics](https://dune.com/home)!

![airbyte_ethereum_sql]({{ site.url }}{{ site.baseurl }}/assets/images/airbyte_ethereum_sql.png){:width="100%"}{:.align-center}

---
---

# Source Code

* [Personal Airbyte repo](https://github.com/leodonethat/airbyte)
* [Pull request with changes for new connector](https://github.com/leodonethat/airbyte/pull/1)
* [Directory for ethereum api](https://github.com/leodonethat/airbyte/tree/master/airbyte-integrations/connectors/source-ethereum-api)
* [File with sync logic (source.py)](https://github.com/leodonethat/airbyte/blob/master/airbyte-integrations/connectors/source-ethereum-api/source_ethereum_api/source.py)
