### What is `journalctl`?
- `journalctl` is a command-line tool used to query and display logs collected by the systemd-journald service.
- Unlike `/var/log/messages` or `/var/log/syslog` (traditional flat text log files), `systemd` logs are stored in binary format inside the systemd journal (`/run/log/journal` or `/var/log/journal`).
  - `journalctl` reads logs from the systemd journal (a central logging service).
  - It combines logs from:
    - Kernel
    - System services (daemons)
    - Applications (that log to `stdout/stderr` in systemd units)
    - Boot messages
- Advantages:
  - Structured log data (includes metadata: PID, UID, systemd unit, boot ID, etc.)
  - Can filter by service, user, priority, kernel messages, time, etc.
  - Supports paging, JSON output, and live following (like `tail -f`).

### How journalctl Works Internally

- The systemd-journald daemon collects logs from:
  - Kernel logs (like dmesg)
  - Syslog (applications writing via syslog API)
  - stdout/stderr from systemd managed services
  - Audit logs (if enabled)
- Logs are indexed with metadata → making filtering super fast.
```
journalctl
```
- Shows all logs since the beginning.
- Default format is time-stamped lines.

### Common Flags and Deep Usage
#### Basic Usage
```
journalctl                # Show all logs since the beginning
journalctl -e             # Jump to the end (latest logs)
journalctl -n 50          # Show last 50 logs
journalctl -f             # Follow logs in real-time (like tail -f)
```
#### Filtering by Time
```bash
journalctl --since "2025-08-18 08:00:00"
journalctl --since "yesterday"
journalctl --since "10 min ago" --until "now"
```
`--since` and `--until` filter logs by time.
Handy for investigating issues after a specific deployment or reboot.

#### Follow Logs (like tail -f)
```
journalctl -f
```
Keeps streaming logs live.

Equivalent to `tail -f /var/log/syslog`.
#### Show Logs from Current Boot
```
journalctl -b
```
- Shows logs only since last system boot.
- For previous boots:
```
journalctl -b -1   # Previous boot
journalctl -b -2   # Two boots ago
journalctl -b                 # Current boot
journalctl --list-boots       # List all boots with boot IDs

```

### Severity/Priority Filtering
Priority levels (from RFC5424 syslog):
- 0 = Emergency
- 1 = Alert
- 2 = Critical
- 3 = Error
- 4 = Warning
- 5 = Notice
- 6 = Info
- 7 = Debug
```
journalctl -p err              # Show errors only
journalctl -p warning..err     # Warnings + Errors
journalctl -p 0..3             # Critical logs only
```
### Output Formatting
```
journalctl -o short            # Default
journalctl -o verbose          # Detailed metadata
journalctl -o json             # JSON format (good for log shipping)
journalctl -o cat              # Message only (no metadata)
```
JSON mode is often used with log collectors (Fluentd, ELK, Loki, etc.).

#### Filtering by Unit/Service
journalctl -u nginx.service
journalctl -u docker -f
```
journalctl -u nginx.service
journalctl -u docker -f
```
- `-u` filters by systemd unit (service).
- Useful for debugging a single service.
#### Filtering by PID, UID, or Executable
```
journalctl _PID=1234
journalctl _UID=1001
journalctl _EXE=/usr/bin/python3
```
- `_PID` → logs from a specific process.
- `_UID` → logs from a specific user.
- `_EXE` → logs from a specific binary.


#### Show Kernel Messages
```
journalctl -k
```
- Equivalent to `dmesg`.
- For previous boots:

```
journalctl -k                 # Show only kernel logs
journalctl -k -b              # Kernel logs from current boot
journalctl -k -b -1           # Kernel logs from previous boot

```
#### Limit Number of Lines
```
journalctl -n 50   # Last 50 logs
journalctl -n 100 -f   # Tail 100 and follow
```
#### Output Formatting
```
journalctl -o json-pretty
journalctl -o verbose
journalctl -o cat
```
- `json-pretty` → structured logs, good for parsing.
- `verbose` → full metadata (PID, UID, systemd unit, etc.).
- `cat`→ only the log message, no metadata.

#### Disk Usage Management & Persistent Logs
By default, `journalctl` may only store logs in memory. To persist across reboots:
```
mkdir -p /var/log/journal
systemctl restart systemd-journald
```
Check size:

```
journalctl --disk-usage
```
Vacuum old logs:
```
journalctl --vacuum-size=500M   # Keep only 500MB of logs
journalctl --vacuum-time=7d     # Keep 7 days of logs
```

- Shows how much space journal logs use.
```
sudo journalctl --vacuum-size=500M
sudo journalctl --vacuum-time=7d
```
- `--vacuum-size` → shrink logs to under X size.
- `--vacuum-time` → keep logs for only X days.

#### Combining Filters
```
journalctl -u ssh -b --since "1 hour ago" -f
```
Logs from SSH service, current boot, last 1 hour, live follow.

### Why `journalctl` is Powerful?
- Structured logs (key=value pairs) instead of plain text.
- Easier to filter by service, PID, user, boot session.
- Handles binary logs with indexing → faster than grepping `/var/log/*`.
#### In real-world DevOps/GCP/Azure:
- Use `journalctl -u docker -f` → debug container runtime.
- Use `journalctl -b -1` → find crash reason in previous boot.
- Use `journalctl -k` → check kernel networking/disk issues.
- Use `journalctl --vacuum-time=14d` → manage log storage in production.

### Real-World Troubleshooting Scenarios
#### Web server crash
```
journalctl -u nginx --since "30 min ago"
```
#### Debugging a failed service startup
```
systemctl status docker
journalctl -u docker -b
```
#### Check kernel OOM (Out of Memory) kills
```
journalctl -k | grep -i "killed process"
```
#### SSH brute force attempts
```
journalctl -u ssh -p warning --since "24 hours ago"
```
#### Check logs after reboot failure
```
journalctl -b -1 -p err
```
### Best Practices
- Use `journalctl -f -u <service>` during service debugging.
- Combine time + priority filters to zero in quickly.
- Always configure persistent storage if logs are important (`/var/log/journal`).
- Export logs for deeper analysis:
```
journalctl -u nginx -o json > nginx-logs.json
```
Rotate/vacuum old logs to prevent disk filling.
---
- journalctl is systemd’s Swiss Army knife for logs.
- It combines functionality of `tail`, `grep`, `dmesg`, and `/var/log/*` reading in one tool.
- With flags like `-u`, `-p`, `-b`, and `--since`, you can slice & dice logs however you need.

### Commonly Used journalctl Flags
| Flag                  | Usage                                                      | Example                                                            |
| --------------------- | ---------------------------------------------------------- | ------------------------------------------------------------------ |
| `-f`                  | Follow logs in real-time (like `tail -f`)                  | `journalctl -f`                                                    |
| `-u <unit>`           | Show logs for a systemd service                            | `journalctl -u nginx.service`                                      |
| `-b`                  | Logs since the current boot                                | `journalctl -b`                                                    |
| `-b -1`               | Logs from previous boot                                    | `journalctl -b -1`                                                 |
| `--since` / `--until` | Time filtering                                             | `journalctl --since "2025-08-18 10:00" --until "2025-08-18 11:00"` |
| `-p <priority>`       | Filter by log level (0-7 or names like err, warning, info) | `journalctl -p err`                                                |
| `-k`                  | Show only kernel logs (dmesg equivalent)                   | `journalctl -k`                                                    |
| `-n <N>`              | Show last N log entries                                    | `journalctl -n 50`                                                 |
| `-o <format>`         | Change output format (short, json, cat, etc.)              | `journalctl -o json-pretty`                                        |
| `-g <pattern>`        | Grep-like search within logs                               | `journalctl -g "error"`                                            |

### Journalctl vs Syslog Tools (tail, less, grep)
| Feature            | `journalctl` (systemd-journald)                         | Classic Syslog Tools (`tail`, `less`, `grep`)             |
| ------------------ | ------------------------------------------------------- | --------------------------------------------------------- |
| **Storage format** | Binary structured                                       | Plain text files (`/var/log/syslog`, `/var/log/messages`) |
| **Metadata**       | Rich: PID, UID, boot ID, unit, capabilities             | Only plain log lines                                      |
| **Filtering**      | Built-in (`-u`, `-p`, `--since`)                        | External (`grep`, `awk`, `cut`)                           |
| **Performance**    | Indexed, fast queries                                   | Slower for large logs (must scan file)                    |
| **Persistence**    | Needs `/var/log/journal/` (else logs lost after reboot) | Text logs are persistent by default                       |
| **Compatibility**  | Works only with `systemd` systems                       | Works everywhere (Linux, Unix, containers)                |
| **Real-time**      | `journalctl -f` (structured follow)                     | `tail -f /var/log/syslog`                                 |
| **Output formats** | Multiple: short, json, export                           | Plain text only                                           |
| **Security**       | Tamper-evident (optional)                               | Easy to tamper with                                       |
| **Learning curve** | Higher (many options)                                   | Simpler, familiar to admins                               |


### When to Use Which?
#### Use `journalctl` when:
- You’re on a `systemd` system (modern Linux distros).
- You need service-specific logs (`-u <unit>`).
- You want structured queries (by time, priority, PID, etc.).
- You need fast filtering of large log sets.
- You want JSON output for integration with log management tools.

#### Use Syslog tools when:
- You’re on a non-systemd system (e.g., older Linux, BSD, Solaris).
- You just want to quickly check raw logs (`tail -f /var/log/syslog`).
- You’re troubleshooting inside containers where journald isn’t available.
- You prefer UNIX philosophy tools (`grep`, `awk`, `sed`) for text processing.