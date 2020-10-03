Kafka Cluster Starter
============

A starter project for running a three node Kafka cluster under Docker. The Docker Compose file contains,

* Three [ZooKeeper](https://zookeeper.apache.org) nodes
* [ZooNavigator](https://zoonavigator.elkozmon.com/en/stable/index.html) - A UI for the ZooKeeper cluster
* Three [Kafka](https://kafka.apache.org) brokers using [this image](https://github.com/wurstmeister/kafka-docker)
* [Kafka Manager](https://hub.docker.com/r/hlebalbau/kafka-manager) - An open source UI by Yahoo for Kafka.
* [Kafka Topics UI](https://github.com/lensesio/kafka-topics-ui) - An open source UI for examining messages
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

With Docker installed, we will need a an external docker network to run the containers on.
Run `docker network create kafak-net`.

Now let's clone the repo and fire up our cluster,

```
git clone git@github.com:aedenj/kafka-cluster-starter.git ~/projects/my-project
cd ~/projects/my-project;docker-compose up
```

### Kafka Admin Setup

Once the cluster is up, which will be indicated by the last line of the console output being,
```
kafka-manager_1  | 2020-08-02 15:13:14,912 - [INFO] k.m.a.KafkaManagerActor - Updating internal state...
```

we can now setup the cluster in the Kafka Admin tool. [Navigate to the UI](http://localhost:9000/addCluster) and at a minimum fill out the name and the ZooKeeper hosts with `zk1:2181,zk2:2182,zk3:2183`

### ZooKeeper Setup

[Navigate to the UI](http://localhost:9001/) and specify ZooKeeper hosts with `zk1:2181,zk2:2182,zk3:2183`


### Kafka Topics UI

There's no setup required. [Start browsing data](http://localhost::9002/) for the topics.


## Common Tasks

For common tasks with Kafka you have one of two options,

* Perform the task through the [Kafka Admin UI](http://localhost:9000/)
* Perform the task on the command line through a docker container.

If you want to perform commands via the commandline is helpful to have this alias in your shell profile.

```
alias kafkad='docker run --rm -i --network kafka-net wurstmeister/kafka:latest'
```

Name the alias what you like. I prefer to add the `d` on the end to indicate the command is being run through docker.

### Create a Topic
[Navigate to the UI](http://localhost:9000) and create the topic there or execute the following assuming the alias above,

```
kafkad kafka-topics.sh --create --bootstrap-server broker-1:19092,broker-2:19093,broker-3:19094 --replication-factor 3 --partitions 9 --topic messages
```

To make life a little easier let's add another alias

```
alias kafkacreatetopic='f() { kafkad kafka-topics.sh --create --bootstrap-server $1 --partitions $2 --replication-factor $3 --topic $4; unset -f f; }; f'
```

Of course we'll want to delete topics so here's an alias for that too,

```
alias kafkadeletetopic='f() { kafkad kafka-topics.sh --delete --bootstrap-server $1 --topic $2; unset -f f; }; f'
```

### Produce Messages

Assuming you set up the `kafkad` alias above run,

```
kafkad kafka-console-producer.sh --broker-list broker-1:19092,broker-2:19093,broker-3:19094 --topic messages --property "parse.key=true" --property "key.separator=:"
```

At the prompt enter each line,

```
1:Wash dishes
2:Clean bathroom
3:Mop living room
```


### Consume Messages

Assuming you set up the `kafkad` alias above run,

```
kafkad kafka-console-consumer.sh --bootstrap-server broker-1:19092,broker-2:19093,broker-3:19094 --from-beginning --topic messages --property "print.key=true"
```

If you entered the messages in the previous section you should see those messages in the console.

## Other Options

[Conduktor](https://www.conduktor.io) is a commercially licensed admin interface for Kafka, which means it has more features than CMAK.
