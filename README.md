# Custom Flow

This repository contains **"customFlow"** data that comprehensively represents the behaviors exhibited by IoT devices through their network traffic. Each data record is structured as a fixed-size matrix, where each row corresponds to a flow (up to and including the past five flows) and includes metadata such as:

- Flow timing
- Transport and network headers
- Volume statistics
- A sequence summarizing packets by direction, timing, and raw payloads

Explicit demarcation is used to preserve traffic semantics in a well-organized and structured manner.

The data was constructed by analyzing a public dataset of PCAP traces from [UNSW IoT Analytics](https://iotanalytics.unsw.edu.au/iottraces.html), collected by researchers at UNSW Sydney.

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

# Display the first few rows
print(df.head())
```

## Cite Our Data
[1] A. Sivanathan, D. Mishra, S. Ruj, N. Fernandes, Q. Z. Sheng, . Luo, D. Coscia, G. Batista and H. Habibi Gharakaheili, "Leveraging Neural Networks to Decode IoT Network Traffic Patterns and Sequence Dynamics", under review at IEEE Transcations on Netwrok and Service Management, Nov 2024.

## License

This project is licensed under the MIT License. See the [LICENSE](./LICENSE) file for details.