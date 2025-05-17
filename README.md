# Elastiflow Grafana (KVLAG) Stack

A log stack to visulaize NetFlows that are processed and collected by Elastiflow in [Grafana](https://grafana.com/oss/grafana/). Uses [Kafka](https://github.com/apache/kafka) as a data ingester, [Grafana Alloy](https://grafana.com/oss/alloy) as a scraper, and [VictoriaLogs](https://victoriametrics.com/products/victorialogs/) as a log storage backend.

## Prerequisites

You will need an instance of [ElastiFlow NetObserv](https://docs.elastiflow.com/docs/flowcoll/introduction) that is configured to recieve NetFlow traffic. It must also be configured to use Kafka as an output source.

Below is an example config of the options you need to set on the collector to send data to Kafka.
```
EF_OUTPUT_KAFKA_ENABLE: "true" # Enables Kafka output
EF_OUTPUT_KAFKA_ECS_ENABLE: "true" # Sends data using the Elastic Common Schema
EF_OUTPUT_KAFKA_TIMESTAMP_SOURCE: start # Uses start of flow as timestamp
EF_OUTPUT_KAFKA_BROKERS: <YOUR_KAFKA_BROKER_HOST>:9092 # Set to where you are hosting Kafka
EF_OUTPUT_KAFKA_VERSION: 4.0.0 # Kafka version
EF_OUTPUT_KAFKA_TOPIC: elastiflow-flow-codex # Topic name to send data to
EF_OUTPUT_KAFKA_PRODUCER_COMPRESSION: 0 # Disables compresssion (required in order to clear data from Kafka)
```

Docker and Docker Compose is also required.

## Install

Clone the repo to your machine and set the repo as the working directory.
```
git clone https://github.com/dynomite567/elastiflow-grafana.git
cd elastiflow-grafana
```

Edit `.env` and change the `HOST_IP` value to the IP of the machine you are hosting the stack on (If this isnt set, the ElastiFlow collector wont be able to talk to Kafka).

You can also set the log retention (via `LOG_RETENTION`) to however long you want the collected netflow data to last. By default, this is set to `90d`.
```
HOST_IP=<IP>
LOG_RETENTION=90d
```

Once set, you can spin up the stack using Docker Compose:
```
docker compose up -d
```
If you are using docker-compose V1 or Podman, use this command instead:
```
docker-compose up -d
```
Navigate to http://localhost:3000/ and sign in with the username `admin` and password `admin`.

## Importing the Dashboards
There are some pre-made dashboards inside the `dashboards/` directory that you can import into Grafana to have views of all the data collected by ElastiFlow.

To import them navigate to the **Dashboards** section in Grafana, click **New > Import...** and upload the JSON file of the dashboard. Select the ``victorialogs`` data source and click **Import**.

Repeat for all the dashboards.

## Updating the Stack
If you want to update VictoriaLogs, you will need to specifiy the version you want to update to by changing the ``VMLOG_VERSION`` variable in ``.env``. You can check for a list of versions [here](https://hub.docker.com/r/victoriametrics/victoria-logs/tags).

Afterwards, run the following command:
```
docker compose pull && docker compose up -d
```

If you are using docker-compose V1 or Podman, use this command instead:
```
docker-compose pull && docker-compose up -d
```
## Screenshots
![](https://github.com/dynomite567/elastiflow-grafana/blob/assets/screenshot1.png)

![](https://github.com/dynomite567/elastiflow-grafana/blob/assets/screenshot2.png)

![](https://github.com/dynomite567/elastiflow-grafana/blob/assets/screenshot3.png)

## Motivation
I recently started using ElastiFlow on my home network using an ELK stack as a backend. It ran well and data visualizations were super responsive but the stack was using a lot of system resources, even while idling. I also enjoy using Grafana for metric and log visualization and I wanted an alternative way to use ElastiFlow without having to have a resource heavy backend.

Luckily ElastiFlow supported Kafka as a way to output data, so this gave me a way to experiment other ways to process and store NetFlow data on other backends.

Originally, I used Loki as a log storage backend as it was meant to be lightweight and could compress log data to smaller amounts than ElasticSearch. It worked, however when it came to processing queries that required reading log data with high cardinality (e.g. NetFlow records) it would consume all the resources on my host and either timeout or crash the host.

I then found VictoriaLogs, which worked similarly to Loki but only processes the data that was queried and not the whole log line. It also was 5x more responsive, could managed the high cardinality of NetFlow records and didn't take up much system resources.
