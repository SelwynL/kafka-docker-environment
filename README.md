# Kafka Environment (Docker)
Different Docker Compose environments to run kafka

## Environments
The table belows shows the different kafka setups to be launched with Docker Compose. Each coincides to a separate `.yml` file in this directory

| Compose File    | Description |
|-----------------|-------------|
| compose-all.yml | All Confluent Kafka services: <br/> - zookeeper <br/> - kafka-broker <br/> - schema-registry <br/> - connect <br/> - control-center <br/> - ksql-server <br/> - ksql-cli <br/> - ksql-datagen <br/> - rest-proxy <br/><br/> Confluent's Control Center will be available at: http://localhost:9021 |
| compose-core.yml | Core Kafka services: <br/> - zookeeper <br/> - kafka-broker <br/> - schema-registry |
| compose-core-ssl.yml | **[NOT YET IMPLEMENTED]**<br/> Core Kafka services using SSL: <br/> - zookeeper <br/> - kafka-broker <br/> - schema-registry |

### Starting an environment
Determine the desired environment from the `Environments` table then launch Docker Compose with reference to that file (note it may take a mintute or two for all the services to come up)
```
K_ENV=compose-all.yml
docker-compose -f ${K_ENV} up -d --build
```

Check if the services are running
```
docker ps
```

### Killing an environment
To take down the Docker network and services created with `up` above, use `down`
```
K_ENV=compose-all.yml
docker-compose -f ${K_ENV} down
```

## Kafka CLI
Kafka is running in Docker, but can be interacted with using the regular Kafka CLI tools. These are included when installing [Confluent](https://docs.confluent.io/current/installation/installing_cp/dev-cli.html#installing-cp). This can also be done with Homebrew on Mac:
```
brew install confluent-oss
```

### CLI: Kafka Topic
Create a basic Kafka topic
```
TOPIC_NAME=test
TOPIC_PARTITIONS=1
TOPIC_REPLICATION=1

/usr/local/bin/kafka-topics --create --if-not-exists --zookeeper localhost:2181 --replication-factor ${TOPIC_REPLICATION} --partitions ${TOPIC_PARTITIONS} --topic ${TOPIC_NAME}
```

List the topics that exist
```
kafka-topics --list --zookeeper localhost:2181
```

### CLI: Topic Schema
Create an avro-schema associated with a topic. Note you'll need to install (jq)[https://stedolan.github.io/jq/] to easily escape the file needed for uploading Avro schemas to Schema Registry

```
brew install jq
```

Use `curl` and `jq` to add a schema associated with the `users` topic
```
TOPIC_NAME=test
SCHEMA_FILE_PATH=src/test/resources/user-schema.avsc

curl -X POST -H "Content-Type: application/json" --data "$(jq '{"schema": . | tostring }' $SCHEMA_FILE_PATH)" http://localhost:8081/subjects/$TOPIC_NAME-value/versions
```

### CLI: Consume Topic
```
TOPIC_NAME=test
kafka-console-consumer --from-beginning --bootstrap-server localhost:9092 --property print.key=true --property print.value=true --topic ${TOPIC_NAME}
```

### CLI: PRODUCE to Topic
Using the following
```
TOPIC_NAME=test
kafka-console-producer --broker-list localhost:9092 --property "parse.key=true" --property "key.separator=:" --topic ${TOPIC_NAME}
```

Send messages with `key:value` pairs
```
> key1:hello
```
