---
title: Clustering and Khepri
---

# Clustering and Khepri

When you cluster RabbitMQ, what really happens is that RabbitMQ calls the
metadata store backend to create or expand its cluster. This is the same with
both Mnesia and Khepri.

Therefore, creating a Khepri-based RabbitMQ cluster is done the same way as
with Mnesia.

## Creating a cluster

You can create a cluster using the regular methods:
* using the CLI
* using peer discovery

See the [clustering guide](../clustering) for a complete description.

Here is an example using the CLI:

```
rabbitmqctl join_cluster rabbit@remote-host
```

Khepri can be enabled before creating the cluster. In this case joining nodes
that don't have Khepri enabled will automatically enable it in the process.

Khepri can be enabled after creating the cluster. Here, the Khepri cluster
will be initialized from whateven members are part of the Mnesia cluster. The
synchronization is handled by
[`khepri_mnesia_migration`](https://rabbitmq.github.io/khepri_mnesia_migration/).

## Removing a node from a cluster

Again, the regular methods can be used to remove a node from a cluster. Here
is an example with the CLI:

```
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl start_app
```

## Caveats

Because of the use of the Raft consensus algorithm, changing the cluster with
Khepri has some constraints to keep in mind:

* There must be a quorum number of nodes available to validate the change
* A single node can be added or removed at a time

Example of a node joining a cluster with stopped nodes

Example of a node leaving a clister with stopped nodes
