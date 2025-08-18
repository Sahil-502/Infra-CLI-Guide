### What is Wireshark?
- A GUI-based (Graphical User Interface) packet analyzer.
- Open-source, runs on Linux, Windows, and macOS.
- Lets you see what’s happening inside your network at the packet level.
- Supports hundreds of protocols (HTTP, DNS, TLS, TCP, UDP, etc.).

Think of it like a CCTV camera for your network — it records all traffic and lets you play back and analyze.
### Key Features
- Live capture & offline analysis → Capture packets in real time or open a saved `.pcap` file.
- Powerful filtering → Apply display filters like:
- `ip.addr == 8.8.8.8` → only packets to/from Google DNS.
- `http` → only HTTP traffic.
- Deep packet inspection → Look inside packets (headers + payload).
- Color coding → Different protocols (e.g., TCP = green, DNS = blue).
- Statistics & Graphs → Packet counts, conversations, throughput, TCP streams.
- Follow TCP/UDP streams → Reconstructs sessions (like entire HTTP requests).
### Example Workflow
- Open Wireshark GUI.
- Select a network interface (e.g., `eth0`, Wi-Fi adapter).
- Start capture → packets flow in.
- Apply filters:
    - Example: `tcp.port == 443` → capture HTTPS traffic only.
- Click a packet → see layers (Ethernet → IP → TCP → HTTP).
- Save capture in `.pcap` format.
### Real-Life Use Cases
- Network troubleshooting (find latency, packet drops).
- Security analysis (detect malicious traffic, packet injection).
- Protocol learning (understand how TCP handshake works).
- Debugging applications (check if app sends proper requests).

## Wireshark vs tcpdump (Side-by-Side Comparison)
| Feature                  | **tcpdump**                                                  | **Wireshark**                                            |
| ------------------------ | ------------------------------------------------------------ | -------------------------------------------------------- |
| **Interface**            | Command-line tool (CLI)                                      | GUI-based tool                                           |
| **Ease of Use**          | Steeper learning curve                                       | Beginner-friendly (point & click)                        |
| **Filtering**            | BPF filters (`tcp port 80 and host 1.1.1.1`)                 | Display filters (`http and ip.addr == 1.1.1.1`)          |
| **Packet Capture**       | Captures raw packets                                         | Captures raw packets                                     |
| **Packet Analysis**      | Basic header inspection in CLI                               | Deep inspection (full headers + payload)                 |
| **Visualization**        | Text only                                                    | Graphs, color coding, flow diagrams                      |
| **Performance**          | Lightweight, fast, low resource                              | Heavier, requires GUI & more resources                   |
| **Scripting/Automation** | Easy to script in bash                                       | Not script-friendly (but has CLI `tshark`)               |
| **Where to Use?**        | Remote servers, headless environments, quick troubleshooting | Local analysis, deep dives, protocol learning, reporting |

### When to Use Which?
#### Use tcpdump when:
- You’re on a remote server (no GUI).
- You just need quick info (like capture packets on port 443).
- You want a lightweight, scriptable tool.
#### Use Wireshark when:
- You need detailed analysis (follow TCP stream, check TLS handshake).
- You want visuals, graphs, color coding.
- You’re debugging complex protocols or learning networking.
#### Tip: You can combine both:
Capture with `tcpdump` on a server:
```
sudo tcpdump -i eth0 -w capture.pcap
```
Transfer `capture.pcap` to your machine.
Open in Wireshark for detailed analysis.

## tcpdump Hands-On (CLI Packet Capture Tool)
### Capture packets on an interface
```
sudo tcpdump -i eth0
```
Captures packets on interface `eth0`. Replace with your NIC (`ens33`, `wlan0`, etc.).
### Save packets to a file (for later analysis in Wireshark)
```
sudo tcpdump -i eth0 -w capture.pcap
```
Creates a capture.pcap file. You can open this later in Wireshark.
### Read packets from a saved file
```
sudo tcpdump -r capture.pcap
```
Displays packets stored in `capture.pcap`.

### Filter by host
``` 
sudo tcpdump -i eth0 host 192.168.1.10
```
Capture only packets to/from `192.168.1.10`.

### Filter by port
```
sudo tcpdump -i eth0 port 80
```
Capture only HTTP traffic (port 80).
### Capture only TCP packets
```
sudo tcpdump -i eth0 tcp
```
### Combine filters
```
sudo tcpdump -i eth0 src 192.168.1.10 and port 443
```
Only packets from `192.168.1.10` on port `443`.

## Wireshark Hands-On (GUI Packet Analyzer)
More powerful, visual, used for deep inspection.

After opening Wireshark:
- Select the network interface (like `eth0`, `Wi-Fi`, etc.).
- Click Start Capturing.

### Apply filters in Wireshark
These are Display Filters (different syntax from tcpdump):  

#### Filter by IP
```
ip.addr == 192.168.1.10
```
#### Filter by Source IP
```
ip.src == 192.168.1.10
```
#### Filter by Destination IP
```
ip.dst == 8.8.8.8
```
#### Filter by Protocol
```nginx
tcp
udp
http
dns
```
#### Filter by Port
```
tcp.port == 80
udp.port == 53
```
#### Combine Filters
```
ip.src == 192.168.1.10 && tcp.port == 443
```

### Open tcpdump capture file in Wireshark
```
wireshark capture.pcap
```
Opens the same file created by tcpdump in Wireshark for GUI analysis.

## Side-by-Side Examples
| Task                        | tcpdump command                     | Wireshark filter                 |
| --------------------------- | ----------------------------------- | -------------------------------- |
| Capture packets from host   | `tcpdump -i eth0 host 192.168.1.10` | `ip.addr == 192.168.1.10`        |
| Capture only HTTP traffic   | `tcpdump -i eth0 port 80`           | `tcp.port == 80`                 |
| Capture DNS packets         | `tcpdump -i eth0 port 53`           | `udp.port == 53`                 |
| Show packets from source IP | `tcpdump -i eth0 src 10.0.0.5`      | `ip.src == 10.0.0.5`             |
| Save & analyze later        | `tcpdump -i eth0 -w capture.pcap`   | Open `capture.pcap` in Wireshark |

### When to Use Which?
#### tcpdump
- Lightweight, CLI-only (good for servers, SSH sessions).
- Quick captures without GUI overhead.
- Useful in production debugging.
#### Wireshark
- GUI-based, detailed packet inspection.
- Reassemble TCP streams (see HTTP requests, SSL handshakes, etc.).
- Great for deep forensic analysis.
#### workflow:
- Capture packets with tcpdump (especially on remote servers).
- Transfer .pcap file to local machine.
- Open in Wireshark for deep analysis.
