---
id: event-streams
title: Publishing Events towards Frontends
sidebar_label: Publishing Events
---

RIG supports various ways to publish an event towards frontends. The recommended way is to have RIG consume one or more Kafka topics, but RIG also support Kinesis as an alternative. For testing and low-traffic scenarios, RIG also provides an HTTP endpoint that can be used to send events to.

## HTTP

The HTTP endpoint is available on RIG's [internal port](rig-ops-guide). It supports JSON-encoded CloudEvents in [structured and binary modes](event-format#http-transport-binding) (Avro is currently not supported).

Example usage (taken from the [tutorial](tutorial#4-create-a-new-chatroom-message-event-backend)):

```bash
$ http post :4000/_rig/v1/events \
  specversion=0.2 \
  type=chatroom_message \
  id=first-event \
  source=tutorial
HTTP/1.1 202 Accepted
content-type: application/json; charset=utf-8
...

{
    "specversion": "0.2",
    "id": "first-event",
    "time": "2018-08-21T09:11:27.614970+00:00",
    "type": "chatroom_message",
    "source": "tutorial"
}

```

## Kafka

As described in the [Event Format](event-format#kafka-transport-binding) Section, the Kafka consumer supports both structured and binary modes, each with JSON as well as Avro encoding (with details described in the [advanced guide on Avro](avro)).

All RIG nodes participate in the same Kafka consumer group and support automatic partition re-balancing in case new nodes are started or existing nodes go away.

By default, there are no Kafka brokers configured. Look for Kafka related variables in the [Operator's Guide](./rig-ops-guide.md) to enable the Kafka consumer.

```bash
# Kafka disabled
docker run accenture/reactive-interaction-gateway

# Kafka enabled
docker run -e KAFKA_BROKERS=kafka:9092 accenture/reactive-interaction-gateway
```

> Note that defining one Kafka broker is sufficient as RIG will automatically discover any connected brokers in the Kafka cluster.

### SSL

SSL for Kafka is disabled by default. To enable it, set the corresponding environment variables, e.g.:

```bash
docker run \
-e KAFKA_BROKERS=kafka:9092 \
-e KAFKA_SSL_ENABLED=1 \
-e KAFKA_SSL_KEYFILE_PASS=abcdefgh \
accenture/reactive-interaction-gateway

# Change default paths for certificates
docker run \
-e KAFKA_BROKERS=kafka:9092 \
-e KAFKA_SSL_ENABLED=1 \
-e KAFKA_SSL_KEYFILE_PASS=abcdefgh \
-e KAFKA_SSL_CA_CERTFILE=my.ca.crt.pem \
-e KAFKA_SSL_CERTFILE=my.crt.pem \
-e KAFKA_SSL_KEYFILE=my.key.pem \
accenture/reactive-interaction-gateway
```

### SASL

SASL for Kafka is disabled by default as well. To enable it, again make sure the corresponding environment variable is defined, e.g.:

```bash
docker run \
-e KAFKA_BROKERS=kafka:9092 \
-e KAFKA_SASL=plain:myusername:mypassword \
accenture/reactive-interaction-gateway
```

## Kinesis

The Kinesis consumer supports JSON-encoded CloudEvents in structured mode only.

Internally, RIG uses Amazon's official Java client in order to support automatic re-balancing of shards in case of a changing network topology.

In order to enable Kinesis, please make sure you're using an `-aws` tagged Docker image and refer to the [Operator's Guide](./rig-ops-guide.md) for environment variables available to configure.

```bash
# Kinesis disabled
docker run accenture/reactive-interaction-gateway:aws

# Kinesis enabled
docker run -e KINESIS_ENABLED=1 accenture/reactive-interaction-gateway

# Configure AWS region
docker run -e KINESIS_ENABLED=1 -e KINESIS_AWS_REGION=eu-west-3 accenture/reactive-interaction-gateway
```

The used consumer stream and the app name can be changed as well:

```bash
docker run \
-e KINESIS_ENABLED=1 \
-e KINESIS_STREAM=my-stream \
-e KINESIS_APP_NAME=my-app_name \
accenture/reactive-interaction-gateway:aws
```

> The app name is used as the name for the corresponding DynamoDB table. The DynamoDB table is used by Kinesis to handle leases and consumer groups. It is similar to the Group ID in Kafka.
