---
title: Metadata store
---

import './index.module.css';

# Metadata store

## Role of the metadata store

The metadata store is the database where RabbitMQ records everything except
queue messages:
* internat users; "internal" as opposed to users defined externally, for
  instance LDAP
* virtual hosts
* topotogy: exchanges, queues, bindings
* runtime parameters and policies

In a cluster, the metadata store is responsible for replicating that across
all RabbitMQ nodes.

The metadata store subsystem relies on a backend library to provide the
database and its replication.

## Supported backends

RabbitMQ supports two different libraries that provide this database:
* Mnesia
* Khepri

Only one of them is used at a given time. Each one is described below.

### Mnesia

<figure style={{width: "120px", float: "right"}}>
![](https://www.erlang.org/assets/img/erlang-logo.svg)
<figcaption>Erlang/OTP logo</figcaption>
</figure>

Mnesia is the only backend used by RabbitMQ until RabbitMQ 3.13.x. This
library is part of Erlang/OTP’s standard distribution and is maintained by the
same people that maintain Erlang.

It is efficient, provides transactions and cluster replication, an API for
backup and restore and, being a native Erlang/OTP library, is perfectly
integrated in any Erlang application.

Unfortunately, the replication part does not help the Erlang application above
—&nbsp;RabbitMQ in this case&nbsp;— to deal with network issues very much.
RabbitMQ is on its own to solve conflicts in the data if two nodes could not
communicate for a while and the database was updated on one side (e.g. a queue
was declared).

To deal with this, [network partition strategies](./partitions) were introduced
in RabbitMQ. However they are fragile and are difficult to reason about, even
opque to application developers.

### Khepri

<figure style={{width: "120px", float: "right"}}>
![](https://raw.githubusercontent.com/rabbitmq/khepri/main/doc/khepri-logo.svg)
<figcaption>Khepri logo</figcaption>
</figure>

Khepri becomes an option in RabbitMQ 4.0.x. It is developed by the RabbitMQ
team and reuses the work done for [quorum queues](./quorum-queues) and
[streams](./streams).

Indeed all these components are based on the Raft algorithm. Therefore the
behavior is well defined in the case of a loss of connectivity and is way
easier to reason about. The behavior is also consistent across RabbitMQ
components and subsystems because they all use the same algorithm.

The goal is to ultimately switch to Khepri only and stop using Mnesia.
However, the use of Khepri is a breaking change compared to Mnesia, even
though it is an internal piece, because it affects various user-visible parts
behave when the cluster or the network have a bad day.

The few next pages will explain how to enable Khepri and will cover these
changes of behavior for various daily operations.
