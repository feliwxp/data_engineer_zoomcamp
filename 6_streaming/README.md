# 6_streaming

## Kafka

[Apache Kafka](https://kafka.apache.org/) is a ***message broker*** and ***stream processor***. Kafka is used to handle ***real-time data feeds***.

* ***Consumers*** are those that consume the data: web pages, micro services, apps, etc.
* ***Producers*** are those who supply the data to consumers.
* ***Messages*** are used by producers and consumers to share information in Kafka (consists of key, value, timestamp) 
* ***topic*** like a concept can be anything that makes sense in the context of the project, such as "sales data", "new members", "clicks on banner", etc.

A producer pushes a message to a topic, which is then consumed by a consumer subscribed to that topic.

* ***Kafka broker*** is a machine (physical or virtualized) on which Kafka is running.
* ***Kafka cluster*** is a collection of brokers (nodes) working together.

## Kafka Demo

Run `docker-compose up` Check the status of the deployment with `docker ps`. 

Access the control center GUI using `localhost:9021`.

Now, create a demo of a Kafka system with a producer and a consumer and see how messages are created and consumed.

1. In the Control Center GUI, select the `Cluster 1` cluster and in the topic section, create a new `demo_1` topic with 2 partitions and default settings.
1. Install dependencies with `requirements.txt` file
1. Run `producer.py`: This script registers to Kafka as a producer and sends a message each second until it sends 1000 messages. Messages in the Messages tab of the `demo_1` topic window in Control Center will show up.
1. Run `consumer.py`: This script registers to Kafka as a consumer and continuously reads messages from the topic, one message each second. You should see the consumer read the messages in sequential order. Kill the consumer and run it again to see how it resumes from the last read message.
1. With the `consumer.py` running, modify the script and change `group_id` to `'consumer.group.id.demo.2'`. Run the script on a separate terminal; you should now see how it consumes all messages starting from the beginning because `auto_offset_reset` is set to `earliest` and we now have 2 separate consumer groups accessing the same topic.
1. On yet another terminal, run the `consumer.py` script again. The consumer group `'consumer.group.id.demo.2'` should now have 2 consumers. If you check the terminals, you should now see how each consumer receives separate messages because the second consumer has been assigned a partition, so each consumer receives the messages for their partitions only.
1. Finally, run a 3rd consumer. You should see no activity for this consumer because the topic only has 2 partitions, so no partitions can be assigned to the idle consumer.

## Avro Demo

This introduces ***schema*** to the data so that producers can define the kind of data they're pushing and consumers can understand it.

The ***schema registry*** is a component that stores schemas and can be accessed by both producers and consumers to fetch them.

1. Run the `producer.py` script and on a separate terminal run the `consumer.py` script. You should see the messages printed in the consumer terminal with the schema we defined. Stop both scripts.
2. Modify the `taxi_ride_value.avsc` schema file and change a data type to a different one (for example, change `total_amount` from `float` to `string`). Save it.
3. Run the `producer.py` script again. You will see that it won't be able to create new messages because an exception is happening.

When `producer.py` first created the topic and provided a schema, the registry associated that schema with the topic. By changing the schema, when the producer tries to subscribe to the same topic, the registry detects an incompatiblity because the new schema contains a string, but the scripts explicitly uses a `float` in `total_amount`, so it cannot proceed.

## Kafka Streams

[Kafka Streams](https://kafka.apache.org/documentation/streams/) is a _client library_ for building applications and services whose input and output are stored in Kafka clusters. In other words: _Streams applications_ consume data from a Kafka topic and produce it back into another Kafka topic.

* ***Streams*** (aka ***KStreams***) are _individual messages_ that are read sequentially.
* ***State*** (aka ***KTable***) can be thought of as a _stream changelog_: essentially a table which contains a _view_ of the stream at a specific point of time.
    * KTables are also stored as topics in Kafka.

## Kafka Streams Demo (1)

The native language to develop for Kafka Streams is Scala; we will use the [Faust library](https://faust.readthedocs.io/en/latest/) instead because it allows us to create Streams apps with Python.

1. `producer_tax_json.py` ([link](../6_streaming/streams/producer_tax_json.py)) will be the main producer.
    * Instead of sending Avro messages, we will send simple JSON messages for simplicity.
    * We instantiate a `KafkaProducer` object, read from the CSV file used in the previous block, create a key with `numberId` matching the row of the CSV file and the value is a JSON object with the values in the row.
    * We post to the `datatalkclub.yellow_taxi_ride.json` topic.
        * You will need to create this topic in the Control Center.
    * One message is sent per second, as in the previous examples.
    
1. `stream.py` ([link](../6_streaming/streams/stream.py)) is the actual Faust application.
    * We first instantiate a `faust.App` object which declares the _app id_ (like the consumer group id) and the Kafka broker which we will talk to.
    * We also define a topic, which is `datatalkclub.yellow_taxi_ride.json`.
        * The `value_types` param defines the datatype of the message value; we've created a custom `TaxiRide` class for it which is available [in this `taxi_ride.py` file](../6_streaming/streams/taxi_rides.py)
    * We create a _stream processor_ called `start_reading()` using the `@app.agent()` decorator.
        * In Streams, and ***agent*** is a group of ***actors*** processing a stream, and an _actor_ is an individual instance.
        * We use `@app.agent(topic)` to point out that the stream processor will deal with our `topic` object.
        * `start_reading(records)` receives a stream named `records` and prints every message in the stream as it's received.
        * Finally, we call the `main()` method of our `faust.App` object as an entry point.
    * You will need to run this script as `python stream.py worker` .
1. `stream_count_vendor_trips.py` ([link](../6_streaming/streams/stream_count_vendor_trips.py)) is another Faust app that showcases creating a state from a stream:
    * Like the previous app, we instantiate an `app` object and a topic.
    * We also create a KTable with `app.Table()` in order to keep a state:
        * The `default=int` param ensures that whenever we access a missing key in the table, the value for that key will be initialized as such (since `int()` returns 0, the value will be initialized to 0).
    * We create a stream processor called `process()` which will read every message in `stream` and write to the KTable.
        * We use `group_by()` to _repartition the stream_ by `TaxiRide.vendorId`, so that every unique `vendorId` is delivered to the same agent instance.
        * We write to the KTable the number of messages belonging to each `vendorId`, increasing the count by one each time we read a message. By using `group_by` we make sure that the KTable that each agent handles contains the correct message count per each `vendorId`.
    * You will need to run this script as `python stream_count_vendor_trips.py worker` .
* `branch_price.py` ([link](../6_streaming/streams/branch_price.py)) is a Faust app that showcases ***branching***:
    * We start by instancing an app object and a _source_ topic which is, as before, `datatalkclub.yellow_taxi_ride.json`.
    * We also create 2 additional new topics: `datatalks.yellow_taxi_rides.high_amount` and `datatalks.yellow_taxi_rides.low_amount`
    * In our stream processor, we check the `total_amount` value of each message and ***branch***:
        * If the value is below the `40` threshold, the message is reposted to the `datatalks.yellow_taxi_rides.low_amount` topic.
        * Otherwise, the message is reposted to `datatalks.yellow_taxi_rides.high_amount`.
    * You will need to run this script as `python branch_price.py worker` .

## Kafka Streams demo (2) - windowing

Let's now see an example of windowing in action.

* `windowing.py` ([link](../6_streaming/streams/windowing.py)) is a very similar app to `stream_count_vendor_trips.py` but defines a ***tumbling window*** for the table.
    * The window will be of 1 minute in length.
    * When we run the app and check the window topic in Control Center, we will see that each key (one per window) has an attached time interval for the window it belongs to and the value will be the key for each received message during the window.
    * You will need to run this script as `python windowing.py worker` .
