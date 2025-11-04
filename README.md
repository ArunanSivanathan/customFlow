# Custom Flow – A Comprehensive Network Traffic Representation

## Abstract
This paper presents a comprehensive dataset of **custom flow** representations derived from raw network traffic, designed to capture detailed behavioral characteristics of IoT communications. Each custom flow encapsulates comprehensive network behavior in a vectorized format, including flow-level metadata, the sequence of individual packets within the flow, and a subset of payloads. Flows are uniquely identified by a five-tuple—device IP address, remote IP address, protocol, device port, and remote port—and have a fixed lifetime of one minute. To maintain consistent temporal granularity and computational efficiency, long-lived connections such as persistent IoT–cloud sessions are segmented into consecutive flow records sharing the same identifier.

The dataset was generated from 60 days of PCAP traces publicly available through the UNSW IoT Traffic Analytics platform. Two distinct variants are provided: 1) **Bidirectional custom flows**, capturing both upstream and downstream packets (~6 million flows), and, 2) **Unidirectional custom flows**, capturing only upstream packets from the device perspective (~3.5 million flows).

Each day’s data is stored as a separate **Parquet file**, compressed by direction to facilitate scalable analysis. This dataset provides a fine-grained yet computationally efficient representation of network behavior, enabling advanced research in traffic analysis, anomaly detection, and IoT device identification.


## A. Overview

Analyzing patterns in network traffic can be performed at both **micro** and **macro** levels.  
At the **micro level**, inspecting byte values within packet headers and payloads provides detailed behavioral insights but is often computationally expensive and limited in capturing broader communication context.  
At the **macro level**, flow-based aggregation offers a more scalable and cost-effective alternative. A *network flow* represents a sequence of packets sharing common properties such as source/destination IP addresses, source/destination port numbers, and protocol (e.g., TCP or UDP).  

While flow records are efficient for large-scale monitoring, they may lack the granularity needed for fine-grained classification of diverse devices and applications.  
To address this, we propose a **hybrid representation** that combines the strengths of both approaches—integrating flow metadata with selective packet-level information.

---

## B. Custom Bidirectional Flow Design

We introduce **custom bidirectional flows** to provide a comprehensive representation of network behaviors. Each custom flow aggregates metadata (key header fields), statistical summaries, packet timestamps and directions, and a subset of payload bytes from the first few packets in the flow.

A custom flow is uniquely identified by a **five-tuple**:  
*device IP address, remote IP address, protocol, device port, and remote port*,  
and has a **fixed lifetime of one minute**.

Long-running connections, such as persistent IoT–cloud sessions, are segmented into multiple consecutive flows to maintain computational efficiency and ensure consistent temporal granularity.  
When a flow exceeds one minute, a new flow record is created with the same five-tuple.

---

## C. Terminology and Direction Handling

To maintain generality, we replace the conventional *client* and *server* terminology with *device* and *remote*.  
This choice reflects the challenges of determining client–server roles in real time, particularly in **UDP** communications where connection states are ambiguous.  
Even in **TCP** flows, identifying session initiation and termination can be unreliable due to missing SYN/FIN packets.

For **device-to-device** communication within the monitored network, we generate **two mirrored flow records**, each reflecting the perspective of one device endpoint.  
Outgoing packets from one device correspond to incoming packets for the other, ensuring consistent bidirectional representation.  

For **device-to-cloud** traffic, only a single bidirectional flow is recorded and attributed to the local device.

---

## D. Flow Metadata and Generalization

While TCP headers can exhibit device-specific characteristics [5], we intentionally exclude them to preserve **model generalization**.  
Our design prioritizes capturing discriminative information from the **transport-layer payload**, which often embeds unique **application-layer** behaviors revealing manufacturer signatures or functionality traits.

Each custom flow includes the following metadata:

- `timestamp` (µs): Unix timestamp of the first packet  
- `remote IPv4 address`  
- `protocol` (transport layer)  
- `device-side port` and `remote-side port`  
- total byte count and total packet count  

Local private IP ranges (`10.0.0.0/8`, `192.168.0.0/16`, `172.16.0.0/12`) of the remote endpoint are anonymized as `0.0.0.0`.  
To support portability across deployments, we exclude environment-specific identifiers such as MAC addresses and local IPs.

---

## E. Fine-Grained Packet-Level Features

To capture behavioral fingerprints within the flow, we include fine-grained information from the **first *i* packets** of each flow.  
The total number of packets and payload sizes can vary substantially, so analyzing every packet is impractical.

Empirical studies [6] indicate that distinguishing features often appear in the **early bytes** of initial packets.  
Accordingly, for each of the first *i* packets, we record:

- time offset from the flow’s first-seen timestamp  
- total packet size  
- direction flag (1 = device → remote, 0 = remote → device)  
- up to *j* bytes of the transport-layer payload  

Parameters *i* and *j* are configurable based on system resources and network conditions.

In our dataset:
- **92%** of 1-minute flows contain **≤10 packets**, and  
- **96%** of packets are **≤1,000 bytes**.  

Therefore, we set *i = 10* and *j = 1,000*.

---

## F. Payload Considerations

We make **no assumptions** about payload contents—whether encrypted, encoded, or plaintext [7].  
Including payload bytes in custom flows enables learning models to discover both strong and weak behavioral patterns that characterize IoT device operations, even in encrypted traffic.

---

## G. Illustration of Custom Flow Structure

![Structure of custom flow records constructed from raw network traffic.](Fig/customflow.png)  
*Figure 1: Structure of custom flow records constructed from raw network traffic.*

---

## H. Dataset Description

The data was constructed by analyzing a public dataset of PCAP traces from [UNSW IoT Analytics](https://iotanalytics.unsw.edu.au/iottraces.html), collected by researchers at UNSW Sydney. It contains **60 days of traffic** from **22 consumer IoT device types**, including cameras, lightbulbs, power plugs, sensors, appliances, and health monitors.  
After processing, we extracted over **5.9 million custom flow records**.  

Table I summarizes the number of flows per device type.  
The activity levels vary widely between devices—for instance, **Amazon Echo**, **Insteon camera**, and **Belkin motion sensor** generate significantly more flows than low-activity devices like the **Withings scale**.

---

### Table I. Summary of Custom Flow Data

| **IoT Device Type**        | **# Custom Flows** |
|-----------------------------|--------------------|
| Amazon Echo                 | 777,587            |
| Belkin motion sensor        | 970,940            |
| Belkin power switch         | 642,451            |
| Dropcam                     | 171,929            |
| HP printer                  | 238,831            |
| iHome power plug            | 17,026             |
| Insteon camera              | 1,095,843          |
| LiFX lightbulb              | 190,632            |
| NEST Protect                | 989                |
| Netatmo camera              | 308,053            |
| Netatmo weather             | 76,972             |
| PIX-STAR photoframe         | 24,197             |
| Samsung camera              | 620,912            |
| Smart Things                | 195,540            |
| TP-Link camera              | 55,959             |
| TP-Link power plug          | 17,729             |
| Triby speaker               | 164,076            |
| Withings baby monitor       | 63,304             |
| Withings scale              | 805                |
| Withings sleep sensor       | 96,332             |
| IT (Android tablet)         | 235,311            |

---

### Parsing Parquet Data Files

To analyze the customFlow data in this repository, which is provided in Parquet format, follow these steps:

### Prerequisites

Ensure you have the necessary Python modules installed. Use `pip` to install them:

```bash
pip install pandas pyarrow
```

### Reading Parquet Files

Bidirectional and unidirectional customFlows are are located in the sub directories of flows/bidirectional and flows/unidirectional. To load and parse them in Python:

```python
import pandas as pd
import os

# Define the type of flow and the file name
flow_type = 'bidirectional'
file = '16-09-23.parquet'

# Load the Parquet file from the specified path
df = pd.read_parquet(os.path.join('./data/', flow_type, file), engine='pyarrow')

# Display the first few rows
print(df.head())
```