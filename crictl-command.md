### 🔹 What is the `crictl` Command?
`crictl` is the CLI for the Kubernetes Container Runtime Interface (CRI). It talks directly to the node’s container runtime (e.g., containerd or CRI-O) via the CRI socket—not to the Kubernetes API.

- Think of it as the “docker CLI replacement” on Kubernetes nodes after dockershim removal (K8s ≥ 1.24).
- It’s perfect when kubectl isn’t enough, the kubelet is unhealthy, or you need low-level runtime visibility.

---

### When to use it
- A Pod is CrashLoopBackOff / ContainerCreating and you want node-level details.
- kubelet is down but containers are running—you still need logs/inspect.
- Image pulls failing; you want to inspect/pull/prune images at the runtime level.
- Network/cgroup/sandbox issues (Pod sandbox = CRI “pod”).

### How it fits
```markdown
kubectl  ──► kube-apiserver ──► kubelet ──► (CRI) ──► containerd / CRI-O
                                           ▲
                                         crictl
```
### Setup (once per node)
Create a config that points to your runtime socket:
#### containerd (most common)
```bash
sudo tee /etc/crictl.yaml >/dev/null <<'EOF'
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 10
debug: false
EOF

```
#### CRI-O

```bash
sudo tee /etc/crictl.yaml >/dev/null <<'EOF'
runtime-endpoint: unix:///var/run/crio/crio.sock
image-endpoint: unix:///var/run/crio/crio.sock
timeout: 10
debug: false
EOF

```
Tip: Paths can differ by distro/cloud image. If unsure, check `ps aux | grep -E 'containerd|crio'` or kubelet flags (`--container-runtime-endpoint=`).
### Everyday commands (the 80/20 you’ll use constantly)
#### Inspect what’s running
```
sudo crictl ps             # running containers
sudo crictl ps -a          # all containers (incl. exited/creating)
sudo crictl pods           # Pod sandboxes (CRI concept of a Kubernetes Pod)
sudo crictl images         # images in the runtime
sudo crictl info           # runtime info (cgroup driver, etc.)
sudo crictl version        # client/server versions
```
#### Logs & details
```bash
# From a Kubernetes Pod -> get the container ID, then use crictl
kubectl get pod myapp -n default -o jsonpath='{.status.containerStatuses[0].containerID}'
# e.g. "containerd://b1f3..."; strip the "containerd://" prefix:

CID=b1f3...
sudo crictl inspect $CID         # deep JSON inspect
sudo crictl logs --tail 200 $CID # container logs (even if kubelet is sad)
```
#### Images (pull/clean)
```bash
sudo crictl pull docker.io/library/alpine:3.19
sudo crictl rmi IMAGE_ID_OR_REF
sudo crictl rmi -a                 # remove all (careful on prod!)
```
#### Stop/remove containers/sandboxes (use with caution)

```bash
sudo crictl stop  $CID
sudo crictl rm    $CID

# For Pod sandbox (CRI "pod"):
sudo crictl pods                    # find sandbox ID (SID)
sudo crictl stopp $SID              # stop sandbox
sudo crictl rmp   $SID              # remove sandbox
```
Prefer `kubectl delete pod ...` for normal ops. Use `crictl` to surgically fix broken states or when the control plane/kubelet path is unhealthy.

### Troubleshooting recipes (copy/paste friendly)
#### Pod stuck in `ContainerCreating`
```bash
# See sandbox and container state
sudo crictl pods
sudo crictl ps -a

# Inspect container for events/reasons
sudo crictl inspect $CID | jq '.info,.status'
```
Look for image pull errors, permission/volume mount failures, or cgroup issues.
#### CrashLoopBackOff but kubelet feels fine
```bash
# Grab container ID and check recent logs
CID=$(kubectl get pod myapp -o jsonpath='{.status.containerStatuses[0].containerID}' | sed 's@.*/@@')
sudo crictl logs --tail 500 $CID
sudo crictl inspect $CID | jq '.status.exitCode,.status.reason,.status.message'
```
#### Image won’t pull
```bash
sudo crictl pull my.registry.local/app:tag
sudo crictl images | grep app
# If auth issues, confirm node has proper creds (imagePullSecrets/kubelet creds) or try docker-style creds if CRI supports it.
```

#### Node disk pressure from images/old sandboxes
```bash
sudo crictl ps -a | wc -l
sudo crictl pods | wc -l
sudo crictl rmi -a        # Dangerous blanket cleanup; prefer targeted `crictl rmi <id>`
```
#### Map Kubernetes ↔ runtime
```bash
# From Pod -> container ID:
kubectl get pod p -o jsonpath='{.status.containerStatuses[*].containerID}'

# From runtime -> back to Pod name/namespace:
sudo crictl inspect $CID | jq '.status.labels,"podSandboxId"'
sudo crictl inspectp $SID | jq '.status.metadata'   # shows pod name/namespace
```
### Advanced: run a container purely with `crictl`
You usually won’t do this in Kubernetes, but it’s handy to understand CRI primitives.

#### Minimal pod sandbox config (`pod.json`)
```json
{
  "metadata": { "name": "demo", "uid": "demo-uid", "namespace": "default" },
  "log_directory": "/var/log/pods/demo",
  "linux": {}
}
```
#### Minimal container config (`ctr.json`)
```json
{
  "metadata": { "name": "alpine" },
  "image": { "image": "docker.io/library/alpine:3.19" },
  "command": ["sh", "-c", "echo hello-from-crictl; sleep 3600"],
  "log_path": "alpine.0.log",
  "linux": { "resources": {} }
}
```
#### Create & start
```bash
SID=$(sudo crictl runp pod.json)                    # create sandbox (pod)
CID=$(sudo crictl create $SID ctr.json pod.json)    # create container in sandbox
sudo crictl start $CID
sudo crictl ps
sudo crictl logs $CID
# Cleanup
sudo crictl stop $CID && sudo crictl rm $CID
sudo crictl stopp $SID && sudo crictl rmp $SID
```
#### `crictl` vs `kubectl` vs `ctr/nerdctl` (quick mental model)
- kubectl: Cluster-level (API). What Kubernetes intends to run.
- crictl: Node-level (CRI). What the runtime is actually doing.
- ctr (containerd) / nerdctl: Runtime-native CLIs (bypass CRI). Use only if you know the runtime internals; `crictl` is the Kubernetes-aligned tool.

### Safety tips
- On production nodes, prefer read-only ops (`ps`, `pods`, `inspect`, `logs`) first.
- Avoid `rmi -a`, `rm`, `rmp` unless you’re certain—Kubernetes will recreate, but you can cause avoidable churn.
- Always confirm you’re targeting the right socket in `/etc/crictl.yaml`.

### Mini cheat-sheet
```bash
# List
crictl ps [-a]          # containers
crictl pods             # pod sandboxes
crictl images           # images

# Inspect/logs
crictl inspect  <CID>
crictl inspectp <SID>
crictl logs --tail 200 <CID>

# Control
crictl stop <CID>; crictl rm <CID>
crictl stopp <SID>; crictl rmp <SID>

# Images
crictl pull <ref>
crictl rmi <img-or-id>
```








This sends `SIGTERM` to process with PID `1234`.

### 🔹 Commonly Used Signals
| Signal Name | Number | Meaning                               | Example           |
| ----------- | ------ | ------------------------------------- | ----------------- |
| SIGTERM     | 15     | Graceful termination (default)        | `kill 1234`       |
| SIGKILL     | 9      | Force kill (cannot be caught/ignored) | `kill -9 1234`    |
| SIGSTOP     | 19     | Stop/suspend process (like pause)     | `kill -STOP 1234` |
| SIGCONT     | 18     | Resume a stopped process              | `kill -CONT 1234` |
| SIGHUP      | 1      | Reload config / restart process       | `kill -HUP 1234`  |

### 🔹 Why Use kill?

- Stop a program that’s misbehaving.
- Restart daemons by sending SIGHUP.
- Pause/resume long-running tasks.  
- Clean up stuck processes.














---
### 🔹 Use of `kill -9 PID`?
- kill is a Linux command used to send signals to processes.
- 9 means you’re sending the SIGKILL signal.
- PID is the Process ID of the program you want to kill.
So, kill -9 PID = “Forcefully terminate the process with this PID immediately, no cleanup allowed.”

### 🔹 Signals in Linux (Background)
Every process in Linux can receive signals (messages from the kernel or user). Examples:
- SIGTERM (15) → default signal (kill PID) – asks the process to terminate gracefully (cleanup, save work, close files).
- SIGKILL (9) → cannot be ignored or handled, kills process instantly.
- SIGSTOP (19) → pause process (like Ctrl+Z).
- SIGCONT → continue a stopped process.

👉 Check all signals:
```
kill -l
```
---
### 🔹 Why use kill -9?
- When a process is hung/frozen and doesn’t respond to kill PID (SIGTERM).
- When a process ignores other signals.
- When you need to force-stop quickly (like zombie processes, stuck apps).

### 🔹 Problems with kill -9
- Process does not clean up (temporary files, open sockets, locks remain).
- If it’s writing to a database/file → may cause corruption.
- Should be last resort.

### 🔹 Step-by-Step Examples
Find PID of a process
```
ps aux | grep nginx
```
(or)
```
pidof nginx
```
Try graceful kill first
```
kill PID   # Sends SIGTERM (15)
```
If it doesn’t stop, force kill
```
kill -9 PID
```
Check if process is gone
```
ps -p PID
```


### 🔹 Alternative to kill -9
#### Instead of kill -9, try:
Graceful stop with SIGTERM
```
kill -15 PID
```
(default signal)
• Stop then continue (debugging)
```
kill -STOP PID
kill -CONT PID
```
• Use pkill or killall (kill by process name instead of PID):
```
pkill -9 nginx
killall -9 nginx
```
### 🔹 Comparison: kill vs kill -9

| Command          | Signal       | Effect               | When to Use        |
| ---------------- | ------------ | -------------------- | ------------------ |
| `kill PID`       | 15 (SIGTERM) | Graceful shutdown    | Default, preferred |
| `kill -9 PID`    | 9 (SIGKILL)  | Immediate force kill | Last resort        |
| `kill -STOP PID` | 19 (SIGSTOP) | Pause process        | Debugging          |
| `kill -CONT PID` | SIGCONT      | Resume process       | After STOP         |

✅ Golden Rule: Always try kill PID (SIGTERM) first, use kill -9 PID only when absolutely necessary.

## 🔹 Most Common Signals Explained

| Signal        | Number | Meaning                                                | Usage Example                                         |
| ------------- | ------ | ------------------------------------------------------ | ----------------------------------------------------- |
| **SIGHUP**    | 1      | Hangup – tells process terminal closed / reload config | `kill -1 nginx` → reload nginx config without restart |
| **SIGINT**    | 2      | Interrupt (Ctrl + C)                                   | Stop a running process interactively                  |
| **SIGQUIT**   | 3      | Quit + core dump (Ctrl + )                             | Debugging: quits and dumps memory                     |
| **SIGILL**    | 4      | Illegal instruction                                    | Sent when CPU finds invalid machine code              |
| **SIGTRAP**   | 5      | Trace/breakpoint trap                                  | Debuggers like `gdb` use this                         |
| **SIGABRT**   | 6      | Abort process                                          | Programs abort on fatal errors                        |
| **SIGBUS**    | 7      | Bus error (invalid memory access)                      | Hardware/memory issues                                |
| **SIGFPE**    | 8      | Floating point error                                   | Division by zero, bad math                            |
| **SIGKILL**   | 9      | Kill immediately (cannot be caught/ignored)            | `kill -9 PID` (force kill)                            |
| **SIGUSR1**   | 10     | User-defined signal 1                                  | Custom use by apps (e.g. Java, Nginx reload stats)    |
| **SIGSEGV**   | 11     | Segmentation fault                                     | Program accessed invalid memory                       |
| **SIGUSR2**   | 12     | User-defined signal 2                                  | Custom use                                            |
| **SIGPIPE**   | 13     | Broken pipe                                            | Writing to a closed pipe/socket                       |
| **SIGALRM**   | 14     | Alarm clock                                            | Triggered by `alarm()` syscalls                       |
| **SIGTERM**   | 15     | Terminate (default kill)                               | `kill PID` (polite termination)                       |
| **SIGSTKFLT** | 16     | Stack fault (rare)                                     | Hardware stack errors                                 |
| **SIGCHLD**   | 17     | Child process stopped/exited                           | Parent notified when child dies                       |
| **SIGCONT**   | 18     | Continue if stopped                                    | Resume process after `SIGSTOP`                        |
| **SIGSTOP**   | 19     | Stop (cannot be ignored)                               | Like pause button (Ctrl+Z but stronger)               |
| **SIGTSTP**   | 20     | Terminal stop (Ctrl+Z)                                 | Suspend process interactively                         |
| **SIGTTIN**   | 21     | Background process read from TTY                       | Sent when bg process tries to read input              |
| **SIGTTOU**   | 22     | Background process write to TTY                        | Sent when bg process writes to terminal               |
| **SIGURG**    | 23     | Urgent condition on socket                             | Used in networking (TCP urgent data)                  |
| **SIGXCPU**   | 24     | CPU time exceeded                                      | Sent when process uses too much CPU                   |
| **SIGXFSZ**   | 25     | File size exceeded                                     | Writing beyond max file size                          |
| **SIGVTALRM** | 26     | Virtual timer expired                                  | Used in profiling                                     |
| **SIGPROF**   | 27     | Profiling timer expired                                | Performance analysis                                  |
| **SIGWINCH**  | 28     | Window size change                                     | Resizing terminal updates apps like `top`             |
| **SIGIO**     | 29     | I/O is ready                                           | Async I/O events                                      |
| **SIGPWR**    | 30     | Power failure                                          | Sent on power-related events                          |
| **SIGSYS**    | 31     | Bad system call                                        | Invalid syscall attempted                             |


### 🔹 Realtime Signals (34–64)
- SIGRTMIN → SIGRTMAX are Realtime signals.
- They’re not fixed in meaning – used by apps and systemd.
- Example: `systemd` services use them for notifications.

### 🔹 Grouping Signals
👉 By Category
1. Process Control
- `SIGTERM (15)` – polite stop
- `SIGKILL (9)` – force stop
- `SIGSTOP (19)` – pause
- `SIGCONT (18)` – resume
2. User Interaction
- `SIGINT (2)` – Ctrl+C
- `SIGTSTP (20)` – Ctrl+Z
- `SIGHUP (1)` – terminal hangup
3. Errors / Faults
- `SIGSEGV (11)` – segfault
- `SIGFPE (8)` – math error
- `SIGILL (4)` – illegal instruction
- `SIGBUS (7)` – memory error
4. Resource Limits
- `SIGXCPU (24)` – too much CPU
- `SIGXFSZ (25)` – file too big
5. Timers
- `SIGALRM (14)` – alarm
- `SIGVTALRM (26)` – virtual timer
- `SIGPROF (27)` – profiling timer
6. Custom
- `SIGUSR1 (10)`
- `SIGUSR2 (12)`

### 🔹 Real Examples
1. Gracefully stop Nginx
```
kill -15 $(pidof nginx)
```
2. Force kill a hung process
```
kill -9 1234
```
3. Reload app config (SIGHUP)
```
kill -1 $(pidof sshd)
```
4. Pause and resume a process
```
kill -STOP 1234
kill -CONT 1234
```
5. Send custom signal to app
kill -USR1 1234

(used by apps like nginx to reopen logs)


✅ So:
- `kill -l` shows all available signals.
- The most used in daily DevOps work: SIGHUP (1), SIGINT (2), SIGTERM (15), SIGKILL (9), SIGSTOP (19), SIGCONT (18).
