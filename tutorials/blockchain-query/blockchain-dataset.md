---
title: Query the Bitcoin blockchain - set up dataset
excerpt: Set up a dataset so you can query the Bitcoin blockchain
products: [cloud]
keywords: [beginner, crypto, blockchain, Bitcoin, finance, analytics]
layout_components: [next_prev_large]
content_group: Query the Bitcoin blockchain
---

import CreateAndConnect from "versionContent/_partials/_cloud-create-connect-tutorials.mdx";
import CreateHypertableBlockchain from "versionContent/_partials/_create-hypertable-blockchain.mdx";
import AddDataBlockchain from "versionContent/_partials/_add-data-blockchain.mdx";

# Set up the database

This tutorial uses a dataset that contains Bitcoin blockchain data for
the past five days, in a hypertable named `transactions`.

<Collapsible heading="Create a Timescale service and connect to your service" defaultExpanded={false}>

<CreateAndConnect/>

</Collapsible>

<Collapsible heading="The dataset" defaultExpanded={false}>

The dataset contains around 1.5 million Bitcoin transactions, the trades for five days. It includes 
information about each transaction, along with the value in [satoshi][satoshi-def]. It also states if a 
trade is a [coinbase][coinbase-def] transaction, and the reward a coin miner receives for mining the coin.

<CreateHypertableBlockchain />

<AddDataBlockchain />

</Collapsible>

[satoshi-def]: https://www.pcmag.com/encyclopedia/term/satoshi
[coinbase-def]: https://www.pcmag.com/encyclopedia/term/coinbase-transaction
