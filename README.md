# Custom Flow

This repository contains **"customFlow"** data that comprehensively represents the behaviors exhibited by IoT devices through their network traffic. Each data record is structured as a fixed-size matrix, where each row corresponds to a flow (up to and including the past five flows) and includes metadata such as:

- Flow timing
- Transport and network headers
- Volume statistics
- A sequence summarizing packets by direction, timing, and raw payloads

Explicit demarcation is used to preserve traffic semantics in a well-organized and structured manner.

The data was constructed by analyzing a public dataset of PCAP traces from [UNSW IoT Analytics](https://iotanalytics.unsw.edu.au/iottraces.html), collected by researchers at UNSW Sydney.

## Download Data Files
[The dataset is hosted on Dryad.](https://doi.org/10.5061/dryad.6q573n6c1)

## Parsing Parquet Data Files

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
[Download files](https://doi.org/10.5061/dryad.6q573n6c1)

# Display the first few rows
print(df.head())
```

#### Data Representation

The parquet files contains following columns:

- **Device** - Device MAC Address
- **FirstSeen** - Flow timestamp
- **RemIP** - Remote IP
- **Proto** - Transport layer Protocol
- **DevPort** - Device side port number of flow
- **RemPort** - Remote side port number of flow
- **TotalFlowSize** - Total byte count of the flow
- **PacketCount** - Total packet count of the flow
- **P00_TO** - **P10_TO** - Time offset of first 10 packets
- **P00_PS** - **P10_PS** - Packet size of first 10 packets
- **P00_D** - **P10_D** - Direction of first 10 packets
- **C_000** - **C_2999** - Payload of upto 10 packets.  We use `-4` and `-8` as delimiters for the start and end of each packet's payload

## Cite Our Data
A. Sivanathan, D. Mishra, S. Ruj, N. Fernandes, Q. Z. Sheng, M. Tran, B. Luo, D. Coscia, G. Batista and H. Habibi Gharakheili, "Real-Time and Trustworthy Classification of IoT Traffic Using Lightweight Deep Learning", IEEE Transactions on Network Science and Engineering (TNSE), , 2025. DOI: 10.1109/TNSE.2025.3628913

