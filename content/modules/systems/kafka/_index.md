---
title: Kafka
weight: 1
bookCollapseSection: false
---

Kafka is a system that allows for messages to be passed between different systems.

## 1. Deploying Kafka

### 1.1 AWS Configuration

To deploy the email system, add `kafka` to a lab configuration that is being deployed or your `deployment_config.yml` file.

### 1.2 Exposed Endpoints

Below are the list of important things exposed by Kafka:

External Bootstrap Server: exposed on port `32000` - this is how you will be communicating with Kafka
Kafka UI: exposed on port `31000` - A UI for Kafka, so that you can easily verify your producers are working without having to use the binaries.

### 1.3 Playing with Kafka

There are a bunch of binaries you can download, but for the sake of development, I would advise that you skip to the following section.

If you're curious regarding the CLI, you can find the documentation [here](https://docs.confluent.io/kafka/operations-tools/kafka-tools.html)

I won't provide details on how to download the binaries, but if you get your hands on them, then here is how you can use them (with a VPN, if running from your local machine)

Below are some commands that might help you when playing around with Kafka.

```bash
kafka-topics.sh --create --topic test-topic --bootstrap-server server.kafka.lab:32000
```

Write events to the topic like so, which will prompt you to enter events on separate lines:
```bash
kafka-console-producer.sh --topic test-topic --bootstrap-server server.kafka.lab:32000
```

And read them like so:
```bash
kafka-console-consumer.sh --topic test-topic --from-beginning --bootstrap-server server.kafka.lab:32000
```

These changes are also reflected in the kafka ui which you can either use VNC to check for functionality or by tunneling it out of AWS.

## 2. Using Kafka with Python

### 2.1 Installation

You can install it manually by installing the dependencies listed below.

You will need the following packages:
```bash
python3
python3-pip
```
You will need to install `kafka-python` using:
```bash
pip install kafka-python
```

### 2.2 Example Usage

After installing the dependencies, you can interact with the kafka server by using python scripts.

`kafka-python` provides 3 main classes that are useful for interacting with the server:

`KafkaClient` - Allows for adding topics and in general monitoring the kafka server status - NOT LOGS
`KafkaProducer` - Sends messages to topics on the kafka server
`KafkaConsumer` - Reads messages from the kafka server

First, make a python file on a separate machine from the kafka server.
Then, put the following in to import the relevant classes, in addition to any other imports you need:

```python
from kafka import KafkaProducer, KafkaConsumer, KafkaClient
from time import sleep
```

You'll likely need to make topics if they don't already exist, and we can do this by creating a KafkaClient object.

```python
server = "server.kafka.lab:32000"

# give your client an appropriate name
client = KafkaClient(bootstrap_servers=server, client_id="client_name")
```

Checking the connection to the Kafka server is always a good idea, even if you think it should be up, before adding the topic.

```python
connected = client.bootstrap_connected()
max_retries=10

while !connected and max_retries>0:
  client.bootstrap_connected()
  max_retries -= 1
  sleep(10)

client.add_topic("foobar")
```

Now that the topics are added, we can start sending messages with the python script.
You MUST send your data as bytes, rather than as plaintext.
There are several ways we can achieve this, but the easiest way to do this is by doing the following:

```python
producer = KafkaProducer(bootstrap_servers=server, key_serializer=str.encode, value_serializer=str.encode)

producer.send(topic="foobar", key="key", value="value")
```

If you'd rather send JSON, then you should do the following instead:
```python
import json
json_producer = KafkaProducer(bootstrap_servers=server, key_serializer=str.encode, value_serializer=lambda v: json.dumps(v).encode("utf-8"))

json_producer.send(topic="foobar", key="something", value={"my":"json"})
```
The `key_serializer` and `value_serializer` parameters are not required when creating the `KafkaProducer` class, but `bootstrap_servers` is.

If you'd rather manually encode them because you intend to use both JSON and text, then you can just do:
```python
base_producer = KafkaProducer(bootstrap_servers=server)

base_producer.send(topic="foobar", key=b"manually", value=b"encoded")
```

For the `KafkaConsumer`, we simply need to define the bootstrap server - we can assign the topics later
```python
consumer = KafkaConsumer(bootstrap_servers=server)
consumer.subscribe(topics=["foobar"])

for message in consumer:
  topic = message.topic
  key = message.key
  value = message.value
  # offset = message.offset
  # partition = message.partition
```
The key and value get returned in bytes, so we need to decode them with `message.key.decode("utf-8")` or `message.value.decode("utf-8")`

Alternatively, we can set deserializers in the `KafkaConsumer` constructor.

```python
consumer = KafkaConsumer(bootstrap_servers=server, key_deserializer=str.decode, value_deserializer=str.decode)

for message in consumer:
  topic = message.topic
  key = message.key
  value = message.value
  # offset = message.offset
  # partition = message.partition
```

There is more documentation available at: [https://kafka-python.readthedocs.io/en/master/usage.html](https://kafka-python.readthedocs.io/en/master/usage.html)
