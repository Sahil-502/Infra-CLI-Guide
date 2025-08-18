# `tcpdump` Command in Linux

## 🔹 What is `tcpdump`?
- `tcpdump` is a packet analyzer (sniffer).
- It captures network packets going in/out of your system’s network interface.
- Very useful for debugging, troubleshooting, and analyzing network issues.

---
### 🔹 Basic Syntax
```
tcpdump [options] [expression]
```
- options = flags that modify how `tcpdump` runs.
- expression = filter (like `port 80`, `host 8.8.8.8`, etc.).


### Examples of Usage
#### Capture packets on default interface
```
sudo tcpdump
```
Shows live packets (raw format).
#### Capture packets on a specific interface
```
sudo tcpdump -i eth0
```
`-i` = choose interface (eth0, wlan0, etc.).

#### Show only first N packets
```
sudo tcpdump -c 10
```
`-c` = capture 10 packets only.

#### Capture packets from a specific host
```
sudo tcpdump host 8.8.8.8
```
Captures traffic only to/from Google DNS.

#### Capture packets on a specific port
```
sudo tcpdump port 80
```
Captures HTTP traffic.
#### Capture TCP or UDP traffic
```
sudo tcpdump tcp
sudo tcpdump udp
```
#### Save captured packets to a file
```
sudo tcpdump -w capture.pcap
```
`-w` = write to file.
Later analyze with Wireshark.
#### Read from a saved file
```
sudo tcpdump -r capture.pcap
```
#### Show packet contents in ASCII
```
sudo tcpdump -A
```
Useful for seeing raw HTTP requests.
#### More detailed packet info
```
sudo tcpdump -vvv
```
The more v’s, the more verbose.
### Real-Life Use Cases
- Debugging why a service (e.g., web server) isn’t reachable.
- Checking DNS request/response flow.
- Monitoring suspicious activity (security investigations).
- Saving traffic and analyzing later with Wireshark.
