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
#If you want to capture only port 80 traffic (HTTP), use:
sudo tcpdump -i any port 80 -w capture.pcap
```
`-i any` → Capture on all interfaces (you can also specify one, like -i eth0).  
`port 80` → Only traffic to/from TCP/UDP port 80.  
`-w capture.pcap` → Write to file instead of printing on screen.  
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

# TCP Flags in tcpdump Output
| Flag Symbol | Full Form                             | Meaning / When It Appears                                                                   |
| ----------- | ------------------------------------- | ------------------------------------------------------------------------------------------- |
| **S**       | SYN (Synchronize)                     | Used to initiate a TCP connection (1st step of 3-way handshake).                            |
| **.** (dot) | ACK (Acknowledgment)                  | Confirms receipt of data or a SYN. Almost all packets after connection setup carry ACK.     |
| **F**       | FIN (Finish)                          | Graceful connection termination request.                                                    |
| **R**       | RST (Reset)                           | Forcefully resets a TCP connection (e.g., if port closed or process not listening).         |
| **P**       | PSH (Push)                            | Tells the receiver to push buffered data to the application immediately (low latency data). |
| **U**       | URG (Urgent)                          | Indicates urgent data in the packet (rarely used).                                          |
| **E**       | ECE (ECN-Echo)                        | Explicit Congestion Notification echo (used for congestion control, RFC 3168).              |
| **C**       | CWR (Congestion Window Reduced)       | Sent when sender reduces its congestion window (with ECN).                                  |
| **W**       | CWR (sometimes shown as W in tcpdump) | Same as above, rarely seen.                                                                 |
| **N**       | Nonce                                 | Used in ECN nonce (experimental, rare).                                                     |

## Common Combinations You’ll See
- [S] → SYN only → "Please start a connection"
- [S.] → SYN + ACK → "I acknowledge your SYN, here’s mine"
- [.] → ACK only → "I got your data"
- [P.] → PSH + ACK → "Here’s data, deliver to app immediately"
- [F.] → FIN + ACK → "I want to close connection"
- [R] or [R.] → RST (with/without ACK) → "Reset/abort the connection"

### Example tcpdump Output With Flags
```makefile
10:00:01 IP 192.168.1.5.12345 > 192.168.1.10.80: Flags [S], seq 123456, win 65535, length 0
10:00:01 IP 192.168.1.10.80 > 192.168.1.5.12345: Flags [S.], seq 987654, ack 123457, win 65535, length 0
10:00:01 IP 192.168.1.5.12345 > 192.168.1.10.80: Flags [.], ack 987655, win 65535, length 0
```
This is the 3-way handshake:
- SYN
- SYN+ACK
- ACK

---
Bonus filters:

* Only incoming HTTP:

  ```bash
  sudo tcpdump -i any dst port 80 -w capture.pcap
  ```
* Only outgoing HTTP:

  ```bash
  sudo tcpdump -i any src port 80 -w capture.pcap
  ```
* HTTP + HTTPS (ports 80 & 443):

  ```bash
  sudo tcpdump -i any port 80 or port 443 -w capture.pcap
  ```
---

### 1. Reading a `.pcap` file

If you already have `capture.pcap`, you can read it like this:

```bash
sudo tcpdump -r capture.pcap
```

* `-r` → read packets from a saved file instead of live traffic.
* By default, it prints a **summary** of each packet (protocol, IPs, ports, flags, etc.).

---

### 2. Filtering while reading

You can apply filters to the `.pcap` just like live capture:

🔹 Show only HTTP (port 80) traffic:

```bash
sudo tcpdump -r capture.pcap port 80
```

🔹 Show only traffic between two hosts:

```bash
sudo tcpdump -r capture.pcap host 192.168.1.10
```

🔹 Show only TCP SYN packets (new connections):

```bash
sudo tcpdump -r capture.pcap 'tcp[tcpflags] & tcp-syn != 0'
```

---

### 3. Show packet contents

By default, `tcpdump` only shows headers. You can add:

* `-X` → Print **hex + ASCII dump** of the packet.
* `-A` → Print packet contents in **ASCII** (useful for HTTP payloads).

Example:

```bash
sudo tcpdump -r capture.pcap -A port 80
```

---

### 4. Count packets

Just want statistics?

```bash
sudo tcpdump -r capture.pcap -q | wc -l
```

(or use `-q` for less verbose output).

---

### 5. Verbose details

* `-v`, `-vv`, `-vvv` → More details about TTL, window size, options, etc.
* Example:

```bash
sudo tcpdump -r capture.pcap -vvv port 80
```

---

### 6. Example Analysis Workflow

Let’s say you captured port 80 traffic:

1. **View summary of packets**

   ```bash
   tcpdump -r capture.pcap -n port 80
   ```

   (shows source/destination IPs, TCP flags, etc.)

2. **Check only SYN packets** (to see connections being initiated):

   ```bash
   tcpdump -r capture.pcap 'tcp[tcpflags] & tcp-syn != 0'
   ```

3. **Look for HTTP payloads**:

   ```bash
   tcpdump -r capture.pcap -A port 80
   ```

4. **Check packet size distribution**:

   ```bash
   tcpdump -r capture.pcap -n -tttt
   ```

   (with timestamps)

---

In short:

* Use `-r` to **read** the capture.
* Combine with **filters** (`port`, `host`, `tcp flags`).
* Use `-A` or `-X` to **see data payloads**.
* Use `-v/-vvv` to get **deep protocol details**.

