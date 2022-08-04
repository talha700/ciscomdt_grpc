# Model-Driven Telemetry

Model-driven Telemetry allows a network device to stream operational/configuration data to a collecter using a push model.
Telemetry data is pushed at a defined period or on-change.

Two main pub/sub schemes are available:
  - **Dial-in**: A dynamic approach where subscriber initiates a subcription, as long as session remain up network device will keep pushing the telemetry data. gNMI and NETCONF are dial-in interfaces.
  - **Dial-out**: A pre-configured approach where subscriptions are statically configured on the network device usin CLI,NETCONF etc. Network device starts to push telemetry data to the collector, if session goes down, device tries to open a new one.
  gRPC interface is available for this approach.


> **Note**
> This repo only demonstrate gRPC Dial-out telemetry.

## Issues with SNMP

Well, for the past 3 decades SNMP has been doing a great job for network monitoring, built with simplicity in mind, but as our applications/services have become more & more network critical. As result, we try to poll our devices aggressively which let alone have scalability issues. Let me mention some key points, explaining, why SNMP is not up to the mark for our modern networks.


- Upon performing a _GetBulk_ against a network device, _GetNext_ operation fetches all the columns of a given table. Device returns as many columns as can fit into a packet. After the first _GetBulk_ packet is filled up, the SNMP agent on the router
continues its lexicographical walk and fills up another packet.
When the poller detects that the last received OID is not from the next table, it stops sending _GetBulk_ operations.
Well, this is too bad on the device/network resources.
- When the device is being polled from multiple NMS, things get worse as there is no way to synchronize polling, as result, the device has to process each request individually.
- Does not provide Single model for configuration and monitoring.
- No relation between data (only textual reference exists in MIBS)


On the other hand, Model-driven Telemetry addresses these concerns. It provides a single YANG-based model/schema for the configuration and monitoring.
Telemetry is way more resource efficeint and creates less mess on the wire.
The device fetches the data in a single dip from itself, moreover, if there are multiple collectors, a network device can do one of the fundamental jobs i.e replicate the packet,as metrics are being pushed instead of being polled asynchronously.

## Configuring gRPC Dial-out Telemetry on Cisco IOSXE

1. Select enconding scheme
2. What to push ? here in example i have choose boot-time from open-config-system YANG model 
3. Select streaming method
4. When to push [periodic/on-change]
5. Collector Address and transport

```
configure terminal
telemetry ietf subscription 107
 encoding encode-kvgpb                                         
 filter xpath /oc-sys:system/state/boot-time 
 stream yang-push
 update-policy periodic 5000
 receiver ip address 192.168.100.242 57555 protocol grpc-tcp
```

> **Note**:
> Not all models are on-change valid,here is the list on on-change models [ON_CHANGE_MODELS](https://github.com/YangModels/yang/blob/main/vendor/cisco/xe/1711/ON_CHANGE_MODELS/ON_CHANGE_MODELS.MD)

### Supported Cisco Platforms
  - IOSXE => 16.10
  - NXOS  => 9.3
  - IOSXR

## Setup TIG Stack 


**Telegraf**: is an server agent to help collect metrics, we are using its [cisco_telemetry_mdt](https://github.com/influxdata/telegraf/blob/release-1.23/plugins/inputs/cisco_telemetry_mdt/README.md)
plugin, which let telegraf understands GPB-KV (self-describing-gpb) encoding, sent by our Cisco device.

**InfluxDB**: an high performance, scalable Time-series database.

**Grafana**: plantform to query,analyze and visualize the metrics , allows us to build dynamic dashboards and generate alerts.


> **Note**
> all above tools are open-sourced


![Diagram](./diagram.png)

Repo includes a docker compose file to deploy the stack

### Spin-up 

> docker-compose up -d

## Dashboard Samples

## Resources

- [Cisco Programmability Configuration Guide](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/prog/configuration/1612/b_1612_programmability_cg/model_driven_telemetry.html)
- [Network Programmability with YANG: The Structure of Network Automation with YANG, NETCONF, RESTCONF, and gNMI](https://www.amazon.com/Network-Programmability-YANG-Modeling-driven-Management/dp/0135180392)
- [The limits of SNMP](https://blogs.cisco.com/sp/the-limits-of-snmp)
- [YangModel Github Repo](https://github.com/YangModels/yang)
- 


