# Kafka Crash Course - Hands-On Project 
[source (youtube)](https://www.youtube.com/watch?v=B7CwU_tNYIE)

### TechWorld with Nana

### project

webstore named **StreamStore**

## used technologies

- Apache Kafka
- Python
- Docker Compose

### notes

## compose

### environment variables

- **KAFKA_KRAFT_MODE: 'true'**
    - enable **KRaft** to internally use
        - metadata management
        - cluster coordination

- **CLUSTER_ID: '[string]'**
- **KAFKA_NODE_ID: [integer]**
    
    we name the cluster and specify there's 1 broker

    -  nodes/brokers
        - kafka is designed to run as a cluster, a cluster has nodes)
    - a node in kafka is a 'broker'
    - brokers 
        - store & manager the data (events created & consumed)
        - organizing data into topics and partitions
        - handle requests from producers & consumers
    - a cluster has an id that the brokers share
    - 1 broker is **active controller**
        - tasks
            - manages state of cluster (cluster coordination)
            - tracks leader broker 
            - handles broker failure
            - administrative tasks
        - only 1 active controller at any given time
        - if it crashes another broker becomes active controller

- **KAFKA_PROCESS_ROLES: broker,controller**

    this node we're defining acts as both broker a controller

- **KAFKA_CONTROLLER_QUORUM_VOTERS: 1@kafka:9093**

    which controllers vote on cluster decisions

    - comma separated string of 
    
            [KAFKA_NODE_ID]@kafka[kafka-port]

- **KAFKA_OFFSET_TOPIC_REPLICATION_FACTOR: 1**

    - there is only 1 copy of the kafka metadata

    - the metadata keeps track of which messages each consumer has read , which event from which topic

    - typically 1 per node in the cluster

- **KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9092,CONTROLLER://0.0.0.0:9093**

    tells kafka to open 2 'doors'

    - 1 for regular data traffic on port 9092
        - plaintext protocol
         here producers and cosumers connect to kafka for data read write

    - 1 for controllers
        - special controller comminucation (management)

- **KAFKA_ADVERTISED_LISTENERS**

    - definess addresses for clients/brokers

- **KAFKA_CONTROLLER_LISTENERS_NAMES**

    - tells controllers to use an address
    - shorthanded to CONTROLLER as defined in KAFKA_LISTENERS property


- **KAFKA_LOG_DIRS**

    where to store (persist with volumes in compose root level)

    - data files
    - broker logs
    - controller metadata
    - ...


## troubleshooting with kafka CLI

    docker exec -it kafka kafka-topics --list --bootstrap-server localhost:9092
    

    docker exec -it kafka kafka-topics --bootstrap-server localhost:9092 --describe --topic orders


    docker exec -it kafka kafka-console-consumer --bootstrap-server localhost:9092 --topic orders --from-beginning

## on the code

### producer

- producer config
    - port from config (compose -> LISTENERS)

- producer.produce()
    - sets up the event on the topic
    - takes in
        - topic: will be created if not exists
        - value: payload/msg
        - set callback: this handles success / eror handling

- producer.flush()
    - sends gathered messages to topic


### consumer

- group.id 
    - group.id makes a group of instances from the current application, these all want to read/process the events in parallel, to logically be grouped together
    - the load can be distributed between the instances

- auto.offset.reset: "earliest" 
    - tells kafka consumer if it cannot find where it cannot find where it last left off reading messages
    - this case: start with the oldest

- polling
    - consumer ask the broker for topic's events
    - the events anre not pushed to the consumers
    - consumer dictates the actions

- cleanup on exit
    - when cosumer stops or errors 
        - **close the consumer**
