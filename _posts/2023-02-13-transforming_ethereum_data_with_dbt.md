---
title:  "Transforming Ethereum data with dbt"
date:   2023-02-13 13:00:00 +0200
toc: true
toc_sticky: true
categories: blockchain
tags: blockchain ethereum development data stack
published: true
---

In a previous post we saw how to [extract data from Ethereum with Airbyte]({% post_url 2023-01-22-extracting_data_from_ethereum_with_airbyte %}). Once we have our raw data in a database, we can transform it to suit our needs. In this case, we will use [dbt](https://www.getdbt.com/) for these transformations:

* [Getting started with dbt Core](https://docs.getdbt.com/docs/get-started/getting-started-dbt-core)
* [Install dbt with Docker](https://docs.getdbt.com/docs/get-started/docker-install)

# Creating our first dbt project

We will use docker as it will help us streamline deployment in the future.

First we pull the right dbt image

```bash
docker pull ghcr.io/dbt-labs/dbt-postgres
```

Then we run the docker equivalent of the `dbt init <project_name>` command. This will create the skeleton for us to get started:

```bash
mkdir dbt_projects
mkdir dbt_projects/profiles
cd dbt_projects
```

```bash
docker run -it \
--platform linux/amd64 \
--network=host \
--mount type=bind,source=/Users/leodonethat/dbt_projects,target=/usr/app \
--mount type=bind,source=/Users/leodonethat/dbt_projects/profiles/,target=/root/.dbt/ \
ghcr.io/dbt-labs/dbt-postgres \
init dbt_ethereum
```

```
10:47:01  Running with dbt=1.2.3
Which database would you like to use?
[1] postgres

(Don't see the one you want? https://docs.getdbt.com/docs/available-adapters)

Enter a number: 1
10:47:03  Profile dbt_ethereum written to /root/.dbt/profiles.yml using target's sample configuration. Once updated, you'll be able to start developing with dbt.
10:47:03
Your new dbt project "dbt_ethereum" was created!

For more information on how to configure the profiles.yml file,
please consult the dbt documentation here:

  https://docs.getdbt.com/docs/configure-your-profile

One more thing:

Need help? Don't hesitate to reach out to us via GitHub issues or on Slack:

  https://community.getdbt.com/

Happy modeling!
```

![dbt_profiles_yml]({{ site.url }}{{ site.baseurl }}/assets/images/dbt_profiles_yml.png){:width="100%"}{:.align-center}


We update `profiles.yml` with the credentials for our local postgres database

```yaml
dbt-ethereum:
  target: dev
  outputs:
    dev:
      type: postgres
      threads: 1
      host: 127.0.0.1
      port: 5432
      user: postgres
      pass: postgres
      dbname: eth
      schema: dbt
```

We can also update the file `dbt_project.yml` in the newly created `dbt_ethereum` directory.

![dbt_project_yml]({{ site.url }}{{ site.baseurl }}/assets/images/dbt_project_yml.png){:width="100%"}{:.align-center}


Note that there are two example models in the directory `my_first_dbt_model.sql` and `my_second_dbt_model.sql`. They don't do much but they are perfect to make sure we are heading in the right direction.

# Running our dbt project

The next step is to try the docker equivalent of `dbt run` (for our newly created project):

```bash
docker run -it \
--platform linux/amd64 \
--network=host \
--mount type=bind,source=/Users/leodonethat/dbt_projects,target=/usr/app \
--mount type=bind,source=/Users/leodonethat/dbt_projects/profiles/,target=/root/.dbt/ \
ghcr.io/dbt-labs/dbt-postgres \
run --project-dir /usr/app/dbt/dbt_ethereum
```

```
11:18:06  Running with dbt=1.2.3
11:18:06  Partial parse save file not found. Starting full parse.
11:18:10  Found 2 models, 4 tests, 0 snapshots, 0 analyses, 256 macros, 0 operations, 0 seed files, 0 sources, 0 exposures, 0 metrics
11:18:10
11:18:11  Concurrency: 1 threads (target='dev')
11:18:11
11:18:11  1 of 2 START table model dbt.my_first_dbt_model ................................ [RUN]
11:18:11  1 of 2 OK created table model dbt.my_first_dbt_model ........................... [SELECT 2 in 0.58s]
11:18:11  2 of 2 START table model dbt.my_second_dbt_model ............................... [RUN]
11:18:12  2 of 2 OK created table model dbt.my_second_dbt_model .......................... [SELECT 1 in 0.23s]
11:18:12
11:18:12  Finished running 2 table models in 0 hours 0 minutes and 1.75 seconds (1.75s).
11:18:12
11:18:12  Completed successfully
11:18:12
11:18:12  Done. PASS=2 WARN=0 ERROR=0 SKIP=0 TOTAL=2
```

Everything looks good!

If we go to `postico` to see what is in our postgres database we will two new tables in the dbt schema:

![dbt_postico_schema_png]({{ site.url }}{{ site.baseurl }}/assets/images/dbt_postico_schema.png){:width="50%"}{:.align-center}

<br>

This is excellent! our basic setup is working well and we are ready to do some real work.

# Updating the logic

Now it's time to take the [raw data we extracted with Airbyte]({% post_url 2023-01-22-extracting_data_from_ethereum_with_airbyte %}) and transform it. This is where dbt truly shines!

## Understanding data from Airbyte

Our first step is taking a look at the data we get from Airbyte

![dbt__airbyte_raw_ethereum_blocks]({{ site.url }}{{ site.baseurl }}/assets/images/dbt__airbyte_raw_ethereum_blocks.png){:width="100%"}{:.align-left}

Each row has only three columns, where the field that really interests us is `_airbyte_data`. It's a json string with the result of the [eth_getBlockByNumber](https://ethereum.org/en/developers/docs/apis/json-rpc/#eth_getblockbynumber) api call.

```sql
select _airbyte_data
from _airbyte_raw_ethereum_blocks
order by _airbyte_emitted_at asc
limit 1

```

```json
{
  "hash": "0xd4e56740f876aef8c010b86a40d5f56745a118d0906a34e69aec8c0db1cb8fa3",
  "size": "0x21c",
  "miner": "0x0000000000000000000000000000000000000000",
  "nonce": "0x0000000000000042",
  "number": "0x0",
  "uncles": [
  ],
  "gasUsed": "0x0",
  "mixHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "gasLimit": "0x1388",
  "extraData": "0x11bbe8db4e347b4e8c937c1c8370e4b5ed33adb3db69cbdb7a38e1e50b1b82fa",
  "logsBloom": "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
  "stateRoot": "0xd7f8974fb5ac78d9ac099b9ad5018bedc2ce0a72dad1827a1709da30580f0544",
  "timestamp": "0x0",
  "difficulty": "0x400000000",
  "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "sha3Uncles": "0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347",
  "receiptsRoot": "0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421",
  "transactions": [
  ],
  "totalDifficulty": "0x400000000",
  "transactionsRoot": "0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421"
}
```

## Transforming data from Airbyte

Our next step with `dbt` will be small (but significant!). We want to turn the json results in `_airbyte_data`into a table called `blocks`. Note we are also adding an extra field at the end with the number of transactions within this block.

Note: the operator `->` returns a json object while `->>` returns a string.

File `blocks.sql` in the directory `models`:

```sql
with 
blocks as (
    select
        _airbyte_data ->> 'number' as number,
        _airbyte_data ->> 'hash' as hash,
        _airbyte_data ->> 'parentHash' as parentHash,
        _airbyte_data ->> 'nonce' as nonce,
        _airbyte_data ->> 'sha3Uncles' as sha3Uncles,
        _airbyte_data ->> 'logsBloom' as logsBloom,
        _airbyte_data ->> 'transactionsRoot' as transactionsRoot,
        _airbyte_data ->> 'stateRoot' as stateRoot,
        _airbyte_data ->> 'receiptsRoot' as receiptsRoot,
        _airbyte_data ->> 'miner' as miner,
        _airbyte_data ->> 'difficulty' as difficulty,
        _airbyte_data ->> 'totalDifficulty' as totalDifficulty,
        _airbyte_data ->> 'extraData' as extraData,
        _airbyte_data ->> 'size' as size,
        _airbyte_data ->> 'gasLimit' as gasLimit,
        _airbyte_data ->> 'gasUsed' as gasUsed,
        _airbyte_data ->> 'mixHash' as mixHash,
        _airbyte_data ->> 'timestamp' as timestamp,
        _airbyte_data ->> 'uncles' as uncles,
        _airbyte_data ->> 'transactions' as transactions,
        json_array_length((_airbyte_data ->> 'transactions')::json) as numberTransactions
    from
        _airbyte_raw_ethereum_blocks
)
select
    *
from
    blocks
```

After running dbt again we should have the table `dbt.blocks` in our postgres database! 

To keep our momentum, we will add a model `transactions` where we will unpack the data from the blocks above:

```sql
with 
blocks_transactions as (
    select
        json_array_elements(transactions::json) as transaction
    from
        {{ "{{ ref('blocks') " }}}}
),
transactions as (
    select
        transaction ->> 'r' as r,
        transaction ->> 's' as s,
        transaction ->> 'v' as v,
        transaction ->> 'to' as to,
        transaction ->> 'gas' as gas,
        transaction ->> 'from' as from,
        transaction ->> 'hash' as hash,
        transaction ->> 'type' as type,
        transaction ->> 'input' as input,
        transaction ->> 'nonce' as nonce,
        transaction ->> 'value' as value,
        transaction ->> 'chainId' as chainId,
        transaction ->> 'gasPrice' as gasPrice,
        transaction ->> 'blockHash' as blockHash,
        transaction ->> 'blockNumber' as blockNumber,
        transaction ->> 'transactionIndex' as transactionIndex
    from
        blocks_transactions
)
select
    *
from
    transactions
```

There are two bits of new information here. The function `json_array_elements` that turns each one of the elements in a json array into its own row, plus the reference to the model `blocks` which is our first line of `dbt` syntax.

I will delete the example models and re-run our dbt project with a full refresh. We will use the same docker command as before but changing the last part to:

```bash
run --full-refresh --project-dir /usr/app/dbt/dbt_ethereum
```

```
06:44:57  Running with dbt=1.2.3
06:44:57  Found 2 models, 6 tests, 0 snapshots, 0 analyses, 256 macros, 0 operations, 0 seed files, 0 sources, 0 exposures, 0 metrics
06:44:57
06:44:58  Concurrency: 1 threads (target='dev')
06:44:58
06:44:58  1 of 2 START table model dbt.blocks ............................................ [RUN]
06:44:59  1 of 2 OK created table model dbt.blocks ....................................... [SELECT 10 in 0.69s]
06:44:59  2 of 2 START table model dbt.transactions ...................................... [RUN]
06:44:59  2 of 2 OK created table model dbt.transactions ................................. [SELECT 499 in 0.40s]
06:44:59
06:44:59  Finished running 2 table models in 0 hours 0 minutes and 1.88 seconds (1.88s).
06:44:59
06:44:59  Completed successfully
06:44:59
06:44:59  Done. PASS=2 WARN=0 ERROR=0 SKIP=0 TOTAL=2
```

## Writing our first data-shaping transformation

Now we have a nice table of blockchain transactions. We can let our imagination roam free and do all sorts of analysis. To take a first step in that direction let's create a transformation to calculate daily aggregations (for the number of transactions plus the total value of it).

Note we are using a custom user-defined function `hex_to_num` to make conversions easier. One idea is to use it in the previous transformations but for the purposes of this proof of concept we will keep the code as simple as possible.

```sql
select
    date(date_trunc('day', to_timestamp(hex_to_num(blk.timestamp)))) as date_ymd,
    count(distinct tx.hash) as number_transactions,
    count(distinct tx.blockNumber) as number_blocks,
    sum(hex_to_num(value)) as total_value_wei,
    sum(hex_to_num(value) * (1/pow(10,18))) as total_value_eth
from
    {{ "{{ ref('transactions') " }}}}  as tx
    left join {{ "{{ ref('blocks') " }}}} as blk on (tx.blocknumber = blk.number)
group by
    date_ymd
```

We then run the docker version of `dbt run` again and take a look at our database with a quick query

```sql
select *
from dbt.transactions_value_daily
order by date_ymd asc
```

![dbt_postico_value_daily]({{ site.url }}{{ site.baseurl }}/assets/images/dbt_postico_value_daily.png){:width="100%"}{:.align-left}

Fantastic! 🚀

We now have a table easy to visualize with exactly what we need. We could access it with something like 
JupyterHub if we want to use notebooks for data analysis or we can use a visualization tool like Metabase to build dashboards on top of it.

**Note**: for the purposes of this proof of concept, we assumed the underlying data is static and we process it all at once. That use case is
fine for analytical purposes (when we want to study past behavior) but it doesn't paint the full picture of a blockchain. In order to productionize
this pipeline, we have to process new blocks efficiently. We have to setup Airbyte to extract blocks from where it was last time it ran and we have to [update our dbt models to process only the data incrementally](https://docs.getdbt.com/docs/build/incremental-models).

---
---

# Source Code

* [dbt_ethereum repo](https://github.com/leodonethat/dbt_ethereum)
* [commit with changes for the first transformation](https://github.com/leodonethat/dbt_ethereum/commit/e7b35f27ab55f97da7df2ef6acfb0125ec4a64ab)
