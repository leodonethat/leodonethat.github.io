---
title:  "Visualizing Ethereum data with Metabase"
date:   2023-02-20 13:00:00 +0200
toc: true
toc_sticky: true
categories: blockchain
tags: blockchain ethereum development data stack
published: true

gallery1:
  - url: /assets/images/metabase_2.png
    image_path: /assets/images/metabase_2.png
    alt: "Metabase"
    title: "Metabase"
  - url: /assets/images/metabase_3.png
    image_path: /assets/images/metabase_3.png
    alt: "Metabase"
    title: "Metabase"
  - url: /assets/images/metabase_4.png
    image_path: /assets/images/metabase_4.png
    alt: "Metabase"
    title: "Metabase"

gallery2:
  - url: /assets/images/metabase_transactions_1.png
    image_path: /assets/images/metabase_transactions_1.png
  - url: /assets/images/metabase_transactions_2.png
    image_path: /assets/images/metabase_transactions_2.png

---

In previous posts saw how to [setup our own Ethereum node](({% post_url 2022-11-14-running_our_own_ethereum_node %})), how to [extract data from it](({% post_url 2023-01-22-using_airbyte_to_extract_data_from_ethereum %})) and [how to transform this data](({% post_url 2023-02-13-using_dbt_to_transform_ethereum_data %})). In this post we will continue with one of the most fun parts of data analytics, data visualization.

There are many business intelligence tools we could use. Commercially, two of the most well known are [Tableau](https://www.tableau.com/) (acquired by Salesforce) and [Looker](https://www.looker.com/) (acquired by Google). I love how intuitive and user friendly Looker. Choosing a paid tool is a great option when we are looking for an extra level of support and user experience.

There are also great open-source options like [Redash](https://redash.io/), [Superset](https://superset.apache.org/) or [Metabase](https://www.metabase.com/). In this example, we will use Metabase as it has a really easy setup and a very intuitive user interface. Remember we use the data transformation step to perform complex logic and will avoid complicated queries within the visualization.

# Installing Metabase

The official commands to pull and run the docker container:

```bash
docker pull metabase/metabase:latest
docker run -d -p 3000:3000 --name metabase metabase/metabase
```

Unfortunately, there seems to an issue when running this image in Macs with the M1 chip. A workaround is building this image ourselves with the right configuration. For the purposes of this post I will use an image built by somebody else but if we were to use it in production we would have to build it from scratch.

```bash
docker pull bobblybook/metabase
docker run -d -p 3000:3000 --name metabase bobblybook/metabase
```

# Setting up Metabase

To setup our instance of Metabase the most important step is to connect it to the database. In our case, we will use the same instance of postgres we have been using in the other examples.

{% include gallery id="gallery1" %}

**Note:** we are using Metabase from a docker image. In order for it to connect to a service running in our laptop we have to use the docker equivalent for localhost: `host.docker.internal`.

# Creating our first dashboard

Metabase is indeed very intuitive. It only takes a few minutes to navigate the interface to create a first chart. There are great examples in the [official docs](https://www.metabase.com/learn/getting-started/getting-started).

In our case, we can follow the steps:
* Go to the home page
* Click on `+ New`
* Select question or sql query
* Choose the table
* Select the data fields, filters and summaries

The last step is Metabase's user interface, the equivalent of a sql query when we choose the `question` option.

{% include gallery id="gallery2" %}

With a little playing around we can quickly develop our first Ethereum dashboard! 🚀

![metabase_dashboard]({{ site.url }}{{ site.baseurl }}/assets/images/metabase_dashboard.png){:width="100%"}{:.align-center}

**Note:** in this example we extracted the first million blocks with Airbyte. The next step productionizing this data pipeline is to make sure we process data incrementally and are always up to date.