# Kafka Cluster

A 3 node Apache Kafka cluster using Docker that demonstrates pub-sub messaging, with a Zookeeper ensemble created to manage the Kafka cluster.

Created for CS3219 Software Engineering Principles and Patterns Own Time Own Target (OTOT) Task D.

## Getting Started

1. Install Docker by following the instructions [here](https://docs.docker.com/engine/install/).
1. Install Docker Compose by following the instructions [here](https://docs.docker.com/compose/install/).

## Running the Kafka Cluster

1. Navigate to the root directory of the repository where the `docker-compose.yml` file is specified.
1. Start the Docker containers.
   ```sh
   $ docker-compose up
   ```
   If you wish to run the containers in the background, you can append a `-d` or `--detach`.
   ```sh
   $ docker-compose up -d
   ```
   Depending on your system's configuration, you might need to run this command (along with all other Docker/Docker Compose commands) with root permissions.
1. There should now be 3 Zookeeper and 3 Kafka containers running.
   To verify this, you can run
   ```sh
   $ docker ps
   ```
1. (Optional) If you ran the containers in the background previously and wish to stop them, run the following command in the root directory of the repository:
   ```sh
   $ docker-compose down
   ```

## Pub-Sub Messaging

Note that which Kafka container (`kafka-1`, `kafka-2`, `kafka-3`) you connect to does not matter for the steps below.

1. Open a new terminal if Docker Compose is not running in detached mode.
1. Create a new Kafka topic to store events.
   ```sh
   $ docker exec -it kafka-1 kafka-topics.sh --create --bootstrap-server kafka-1:9092,kafka-2:9092,kafka-3:9092 --replication-factor 3 --partitions 3 --topic cs3219
   ```
1. Run the console producer client.
   ```sh
   $ docker exec -it kafka-1 kafka-console-producer.sh --broker-list kafka-1:9092,kafka-2:9092,kafka-3:9092 --topic cs3219
   ```
1. Write some events into the topic.
   Do not close the producer client.
   ```sh
   > Hello world
   > 1
   > 2
   > 3
   > CS3219 OTOT Task D
   ```
1. Open a new terminal and run the console consumer client.
   ```sh
   $ docker exec -it kafka-1 kafka-console-consumer.sh --from-beginning --bootstrap-server kafka-1:9092,kafka-2:9092,kafka-3:9092 --topic cs3219
   ```
1. You should see all of the events which were previously written by the producer.
   ```sh
   Hello world
   1
   2
   3
   CS3219 OTOT Task D
   ```
1. Write some more events using the console producer client.
   The events should also appear in the console consumer client.
1. To exit, stop the console producer client and console consumer client with `Ctrl-C`.

## Master Node Failure

1. Continue from step 7 of the [Pub-Sub Messaging section](#pub-sub-messaging) above.
1. Open a new terminal and check if `zookeeper-1` is the master node in our cluster.
   ```sh
   $ docker exec -it zookeeper-1 zkServer.sh status
   ```
   In the output, you will see that its `Mode` is either a `leader` or a `follower`.
1. Repeat the previous step for `zookeeper-2` and `zookeeper-3`.
   Out of the 3 Zookeeper nodes, one of them will be the leader.
1. Stop the Zookeeper node which is the leader.
   For example, if the container `zookeeper-1` contains the Zookeeper node that is the leader:
   ```sh
   $ docker stop zookeeper-1
   ```
   Change the container name in the command accordingly if the first Zookeeper node is not the leader.
1. Now, one of the other Zookeeper nodes should have taken over as the master node.
   We can verify this by running step 2 on both of the other containers.
1. Going back to the terminal which has the console producer client open, we can write new events.
   These events can be seen in the console consumer client, demonstrating that the pub-sub messaging functionality still works even after we killed the previous master node in the cluster since Zookeeper elected a new master.
1. To exit all of the Docker containers, use `Ctrl-C` on all of them.
