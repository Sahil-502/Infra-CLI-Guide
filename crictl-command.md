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
