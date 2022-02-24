

```mermaid
graph TD
    classDef hi fill:#f9f;
    A["Look up destination IP in routing table"] 
    A -->|"Matching route is<br/>directly connected (G=0)"| B["Check if destination IP<br/> is in ARP table"]
    A -->|"No matching route"| D["Return error:<br/> Network is unreachable"]
    A -->|"Matching route is<br/>via gateway (G=1)"| C["Check if gateway IP<br/> is in ARP table"]
    B -->|"In ARP table"| F["Send IP packet to <br/>destination IP with<br/> destination host's<br/> MAC address"]
    B -->|"Not in table"| E["Try to resolve<br/> destination IP by<br/> sending ARP request(s)"]
    E -->|"ARP reply"|F
    E -->|"ARP timeout"|I["Send ICMP to self:<br/>Destination Unreachable:<br/> Host Unreachable"]
    C -->|"Not in table"| G["Try to resolve<br/> gateway IP by<br/> sending ARP request(s)"]
    G -->|"ARP timeout"|I
    G -->|"ARP reply"|H["Send IP packet to <br/>destination IP<br/> with gateway's<br/> MAC address"]
    C -->|"In ARP table"| H
```