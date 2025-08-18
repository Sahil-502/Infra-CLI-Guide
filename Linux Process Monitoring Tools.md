# Linux Process Monitoring Tools

## 1. `ps aux`
One-time snapshot of all running processes.
```
ps aux
```
### Example output:
```
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.1 168512  9100 ?        Ss   Aug16   0:05 /sbin/init
root       723  0.1  1.2 372564 50200 ?        Ssl  Aug16   3:15 /usr/bin/dockerd
sahil     1502  2.0  5.3 982456 210000 pts/0   Sl+  10:00   1:05 /usr/bin/python3 script.py
```
### Use Cases:
- Quick check of running processes.
- Search for a process:

```
ps aux | grep nginx
```
### Sort by memory or CPU:
```
ps aux --sort=-%mem | head -n 10
```

## 2. `top`
Real-time, interactive view of system processes.
```
top
```
### Features:
- Updates live every few seconds.
- Shows CPU%, memory%, PID, command.
- Interactive keys:
    - `M` → sort by memory
    - `P` → sort by CPU
    - `k` → kill a process
    - `q` → quit

### Use Cases:
- Watch system usage live.
- Identify which process is hogging CPU/memory in real time.
- Useful for servers when debugging spikes.

## 3. htop
Advanced, colorful, user-friendly version of `top`.
```
htop
```
### Features:
- Color-coded CPU, memory, and swap usage bars.
- Easier navigation (scroll up/down, left/right).
- Supports mouse interaction.
- Tree view (`F5`) → shows processes like `pstree`.

### Use Cases:
- Quickly find and kill processes (press `F9`).
- Better visualization than `top`.
- Ideal for DevOps monitoring when debugging Docker/Kubernetes nodes.

## 4. Comparison Table
| Feature         | `ps aux`                 | `top`                   | `htop`                          |
| --------------- | ------------------------ | ----------------------- | ------------------------------- |
| **Type**        | Snapshot (static)        | Real-time (interactive) | Real-time (interactive + UI)    |
| **Updates**     | No                       | Yes (every few secs)    | Yes (smooth scrolling)          |
| **Sorting**     | Manual (`--sort`)        | Keys (`M`, `P`)         | Interactive (mouse/keys)        |
| **Tree View**   | No                       | No                      | Yes (`F5`)                      |
| **Ease of Use** | Medium (script-friendly) | Medium                  | Easy (color + mouse support)    |
| **Best For**    | Scripts, one-time check  | Live server monitoring  | DevOps debugging, visualization |

## 5. DevOps Real Example
### Scenario: A Kubernetes pod is consuming high CPU.
#### Steps you might take:
##### Find the pod process:
```
ps aux | grep kube
```
##### Watch live usage:
```
top
```
Sort by CPU (`P`) or Memory (`M`).
##### Visualize clearly and kill if needed:
```
htop
```
→ Press F9 to kill the stuck process.

### Summary
- Use `ps aux` for quick process search or scripts.
- Use `top` for real-time performance monitoring.
- Use `htop` for an interactive, DevOps-friendly view.