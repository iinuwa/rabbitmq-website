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

Because of the use of the Raft consensus algorithm, you need to keep in mind
that all operations that involve an update to the metadata store — and
sometimes even queries — require that a quorum number of nodes are available.

### Restarting a cluster member

When a cluster member is restarted or stopped, the remaining nodes may not
form a quorum anymore. This may affect the ability to start a node.

For example, in a cluster of e.g. 5 nodes where all nodes are stopped, the
first two starting nodes will wait for a third node to start before completing
their boot and start serving messages. That’s because the metadata store needs
at least 3 node in this example to elect a leader and finich to initialize its
state to a known good one. In the meantime the first two nodes wait and may
time out if the third one does not appear.

### Adding or removing a cluster member

Likewise, there must be a Raft leader and thus a quorum number of nodes to
validate and commit any change to the cluster membership, whether a member is
added or removed. Again, the operation will time out if that condition is not
met.

Here is an example of a node joining a 4-node cluster with 3 stopped nodes:
```
# rabbitmqctl -n rabbit@host-5 join_cluster rabbit@host-4

Error:
Khepri has timed out on node rabbit@host-5.
Khepri cluster could be in minority.
```

