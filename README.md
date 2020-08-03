Kafka Cluster Starter
============

A starter project for running a three node Kafka cluster under Docker. The Docker Compose file contains,

* Three [ZooKeeper](https://zookeeper.apache.org) nodes
* [ZooNavigator](https://zoonavigator.elkozmon.com/en/stable/index.html) - A UI for the ZooKeeper cluster
* Three [Kafka](https://kafka.apache.org) brokers using [this image](https://github.com/wurstmeister/kafka-docker)
* [Kafka Manager](https://hub.docker.com/r/hlebalbau/kafka-manager) - An open source UI by Yahoo for Kafka.
* [Kafka Topics UI](https://hub.docker.com/r/landoop/kafka-topics-ui) - An open source UI for examining messages
* [Kafka Connect UI](https://github.com/lensesio/kafka-connect-ui) - A UI for managing Kafka Connect clusters.
* A container, named kafka-tools, for executing commands if you don't want to install the tools locally
* [Kafka Rest Proxy](https://docs.confluent.io/current/kafka-rest/index.html) - A RESTful interface to a Kafka cluster,
  making it easy to produce and consume messages, view the state of the cluster, and perform administrative actions
  without using the native Kafka protocol or clients.

This setup allows you to experiment with a number of scenarios you may wish to test,

* Evaluating replication by taking a broker offline and deleting the data for that broker then bringing the broker back up
* Taking a Kafka Connect worker offline and observing the other workers pick up the orphaned partitions.


## Pre-Requisites

1. Docker

    + [Windows](https://docs.docker.com/docker-for-windows/install/)
    + [Mac](https://download.docker.com/mac/stable/Docker.dmg)

## Up & Running

Get started by running,

```
git clone git@github.com:aedenj/kafka-cluster-starter.git ~/projects/my-project
cd ~/projects/my-project;docker-compose up
```

### Kafka Admin Setup
Once the cluster is up, which will be indicated by the last line of the console output being,
```
kafka-manager_1  | 2020-08-02 15:13:14,912 - [INFO] k.m.a.KafkaManagerActor - Updating internal state...
```

we can now setup the cluster in the Kafka Admin tool. [Navigate to the UI](http://localhost:9000/addCluster) and at a minimum fill out the name
and the ZooKeeper hosts with `zk1:2181,zk2:2182,zk3:2183`

### ZooKeeper Setup

[Navigate to the UI](http://localhost:9001/) and at a minimum fill out the name
and the ZooKeeper hosts with `zk1:2181,zk2:2182,zk3:2183`


### Kafka Topics UI

There's not setup required. [Start browsing data](http://localhost::9002/) from Kafka Topics.


## Common Tasks

For common tasks with Kafka you have one of two options,

* Perform the task through the [Kafka Admin UI](http://localhost:9000/)
* Perform the task in the kafka-tools container, which you can access by running `docker exec -it kafka-tools bash`


### Create a Topic
[Navigate to the UI](http://localhost:9000) and use the UI or execute the following in the kafka-tools container,

```
kafka-topics.sh --create --bootstrap-server broker-1:19092,broker-2:19093,broker-3:19094 --replication-factor 3 --partitions 9 --topic messages
```

### Produce Messages

In the kafka-tools container run,

```
kafka-console-producer.sh --broker-list broker-1:19092,broker-2:19093,broker-3:19094 --topic messages --property "parse.key=true" --property "key.separator=:"
```
At the prompt enter each line,

```
1:Wash dishes
2:Clean bathroom
3:Mop living room
```


### Consume Messages

In the kafka-tools container run,
```
kafka-console-consumer.sh --bootstrap-server broker-1:19092,broker-2:19093,broker-3:19094 --from-beginning --topic messages --property "print.key=true"
```

If you entered the messages in the previous section you should see those messages in the console.

## Other Options

[Conduktor](https://www.conduktor.io) is a commercially licensed admin interface for Kafka, which means it has more features than CMAK.
