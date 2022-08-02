# Model-Driven Telemetry

Model-driven Telemetry allows a network device to stream operational/configuration data to a collecter using a push model.
Telemetry data is pushed at a defined period or on-change.

Two main pub/sub schemes are available:
  - **Dial-in**: A dynamic approach where subscriber initiates a subcription, as long as session remain up network device will keep pushing the telemetry data. gNMI and NETCONF are dial-in interfaces.
  - **Dial-out**: A pre-configured approach where subscriptions are statically configured on the network device usin CLI,NETCONF etc. Network device starts to push telemetry data to the collector, if session goes down, device tries to open a new one.
  gRPC interface is available for this approach.


> **Note**
> This repo only demonstrate gRPC Dial-out telemetry.

## Setup GRPC Dial-out Telemetry Using TIG Stack 

**Telegraf**: is an server agent to help collect metrics, we are using its [cisco_telemetry_mdt](https://github.com/influxdata/telegraf/blob/release-1.23/plugins/inputs/cisco_telemetry_mdt/README.md)
plugin, which let telegraf understands GPB-KV (self-describing-gpb) encoding, sent by our Cisco device.

**InfluxDB**: an high performance, scalable Time-series database

**Grafana**: plantform to query,analyze and visualize the metrics , allows us to build dynamic dashboards and generate alerts


> **Note**
> all above tools are open-sourced

![Diagram](./diagram.png)


## Spin-up 

> docker-compose up -d

## Cisco Telemetry Support
    - IOSXE => 16.10
    - NXOS  => 9.3
