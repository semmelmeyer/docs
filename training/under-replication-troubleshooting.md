---
title: Under-Replication Troubleshooting
toc: false
toc_not_nested: true
sidebar_data: sidebar-data-training.json
---

In this lab, you'll cause all ranges in a 3-node cluster to lose 1 of 3 replicas. This will make the cluster under-replicated and in a vulnerable state. You'll then troubleshoot and resolve the problem.

<style>
  #toc ul:before {
    content: "Hands-on Lab"
  }
</style>
<div id="toc"></div>

## Before You Begin

In this lab, you'll start with a fresh cluster, so make sure you've stopped and cleaned up the cluster from the previous labs.

## Step 1. Start a 3-node cluster

1. In a new terminal, start node 1:

    {% include copy-clipboard.html %}
    ~~~ shell
    $ cockroach start \
    --certs-dir=certs \
    --locality=datacenter=us-east-1 \
    --store=node1 \
    --host=localhost \
    --port=26257 \
    --http-port=8080 \
    --join=localhost:26257,localhost:26258,localhost:26259
    ~~~~

2. In another terminal, start node 2:

    {% include copy-clipboard.html %}
    ~~~ shell
    $ cockroach start \
    --certs-dir=certs \
    --locality=datacenter=us-east-1 \
    --store=node2 \
    --host=localhost \
    --port=26258 \
    --http-port=8081 \
    --join=localhost:26257,localhost:26258,localhost:26259
    ~~~

3. In another terminal, start node 3:

    {% include copy-clipboard.html %}
    ~~~ shell
    $ cockroach start \
    --certs-dir=certs \
    --locality=datacenter=us-east-1 \
    --store=node3 \
    --host=localhost \
    --port=26259 \
    --http-port=8082 \
    --join=localhost:26257,localhost:26258,localhost:26259
    ~~~

4. In another terminal, perform a one-time initialization of the cluster:

    {% include copy-clipboard.html %}
    ~~~ shell
    $ cockroach init --certs-dir=certs
    ~~~

## Step 2. Simulate the problem

1. In the same terminal, reduce the amount of time the cluster waits before considering a node dead to just 1 minute:

    {% include copy-clipboard.html %}
    ~~~ shell
    $ cockroach sql \
    --certs-dir=certs \
    --execute="SET CLUSTER SETTING server.time_until_store_dead = '1m0s';"
    ~~~

2. In the terminal where node 3 is running, press **CTRL + C** to stop the node.

## Step 3. Troubleshoot the problem

1. Open the Admin UI at <a href="https://localhost:8080" data-proofer-ignore>https://localhost:8080</a>.

2. Select the **Replication** dashboard.

3. Hover over the **Ranges** graph:

    <img src="{{ 'images/training-11.png' | relative_url }}" alt="CockroachDB Admin UI" style="border:1px solid #eee;max-width:100%" />

    You'll see that there are 16 ranges total, and 16 ranges are under-replication, which means that every range in the cluster is missing 1 of 3 replicas. This is a vulnerable state because, if another node were to go offline, all ranges would lose consensus, and the entire cluster would become unavailable.

## Step 4. Resolve the problem

To bring the cluster back to a safe state, you need to either restart the down node or add a new node.

1. In the terminal where node 3 was running, restart the node:

    {% include copy-clipboard.html %}
    ~~~ shell
    $ cockroach start \
    --certs-dir=certs \
    --locality=datacenter=us-east-1 \
    --store=node3 \
    --host=localhost \
    --port=26259 \
    --http-port=8082 \
    --join=localhost:26257,localhost:26258,localhost:26259
    ~~~

3. Hover over the **Ranges** graph:

    <img src="{{ 'images/training-12.png' | relative_url }}" alt="CockroachDB Admin UI" style="border:1px solid #eee;max-width:100%" />

    Soon, you'll see that there no longer any under-replicated ranges.

## What's Next?

[Cluster Unavailability Troubleshooting](cluster-unavailability-troubleshooting.html)