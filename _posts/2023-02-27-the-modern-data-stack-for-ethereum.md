---
title:  "The Modern Data Stack for Ethereum"
date:   2023-02-27 13:00:00 +0200
toc: true
toc_sticky: true
categories: blockchain
tags: blockchain ethereum development data stack
published: true

---

I've been having a blast exploring some amazing places and indulging in my hobbies during my free time. Learning Swahili, practicing self-defense, and scuba diving have been some of the highlights of my adventures so far. But I wanted to take on a fun technical challenge as well, so I decided to build a blockchain analytics platform using modern open-source tools.

My goal was to create a proof of concept for an end-to-end data pipeline using _The Modern Data Stack_, an umbrella term for a set of data tools launched in recent years that enable us to implement a data pipeline. If you're interested in learning more about it, you can check out this [blog post by Fivetran](https://www.fivetran.com/blog/what-is-the-modern-data-stack) or [this one by dbt](https://www.getdbt.com/blog/future-of-the-modern-data-stack/). There are many advantages to a more traditional approach, but what sets The Modern Data Stack apart for me is its modularity. Each step of the pipeline can be implemented by a different tool depending on our specific requirements, whether it's open-source, on-premise, cloud-based, third-party, or in-house.

For this project, I focused on using open-source and user-friendly tools. In the past, it was difficult to find a combination of these two qualities, but I was pleasantly surprised to find that newer tools offer an excellent user experience. I suppose it doesn't hurt that there are well-funded companies offering managed versions of their products!

# The big picture
Without further ado, our Ethereum data stack:

![mds_ethereum]({{ site.url }}{{ site.baseurl }}/assets/images/mds_ethereum.png){:width="100%"}{:.align-center}

Following the data from left to right, we:
* [Setup our own Ethereum node with Erigon]({% post_url 2022-11-14-running_our_own_ethereum_node %}) to serve as the data source. 
* [Use Airbyte to extract the data from Erigon and load it into Postgres]({% post_url 2023-01-22-extracting_data_from_ethereum_with_airbyte %}).
* [Transform the raw data with dbt]({% post_url 2023-02-13-transforming_ethereum_data_with_dbt %}).
* [Visualize our neatly transformed data with Metabase]({% post_url 2023-02-20-visualizing_ethereum_data_with_metabase %}).

# Lessons learned
* Setting up the Ethereum node proved to be the most difficult part of the project. I can see why node providers like [Alchemy](https://www.alchemy.com/), [Infura](https://www.infura.io/), and [Quicknode](https://www.quicknode.com/) are so popular! (and have seen their valuations skyrocket).
* The developer experience for [Airbyte](https://airbyte.com/), [dbt](https://www.getdbt.com/), and [Metabase](https://www.metabase.com/) was nothing short of _delightful_. I am thrilled to see how much progress has been made in the past few years and are excited to see what the future holds.

# Next steps
* Productionize our proof-of-concept code to ensure that all steps work efficiently when a new block is produced every twelve seconds.
* Add new data sources, such as downloading the price of ETH and tokens in dollars and other currencies.
* Deploy on the cloud and replace the local Postgres database with [BigQuery](https://cloud.google.com/bigquery) or [Snowflake](https://www.snowflake.com/) to improve scalability (another big selling point of the modern data stack).
* Add an orchestration engine like [Airflow](https://airflow.apache.org/) or [Dagster](https://dagster.io/) to handle an increasing number of tasks.
* Add a data catalog like [DataHub](https://datahubproject.io/) to serve as a metadata management system and keep track of a growing number of data artifacts. While not open-source, a newer tool like [metaphor](https://metaphor.io/) has amazing features like column-level lineage and social annotations.
* Ensure data quality, with a tool like [great expectations](https://greatexpectations.io/).


# Competition

What would be the world without friendly competition?! If you can't wait to learn more and are eager to dive in I have great news: there are folks out there working tirelessly to provide free blockchain data in a user-friendly format. I highly recommend checking out [Dune Analytics](https://dune.com/home) and [Flipside Crypto](https://flipsidecrypto.xyz/) for a `sql` interface. If you're looking for more curated tools that are perfect for specific use cases, you might want to explore [Nansen](https://nansen.ai/), [Glassnode](https://glassnode.com/), or [Messari](https://messari.io/). These tools are all excellent resources for anyone looking to learn more about blockchain data.
