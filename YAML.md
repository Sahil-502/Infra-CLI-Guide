# YAML
## Introduction to YAML
- YAML = YAML Ain’t Markup Language
- It’s used for configuration files in tools like Kubernetes, Ansible, GitHub Actions, Docker Compose, etc.
- Human-readable (like JSON, but cleaner).
- File extension: `.yaml` or `.yml`.

## YAML Basics
Rules you must remember:
- Indentation = spaces only (no tabs).
- Key-value pairs are written as:
```yaml
name: Sahil
role: GCP Engineer
```
Strings don’t always need quotes:
```yaml
city: Delhi
designation: "Cloud Engineer"
```
Lists (arrays):
```yaml
fruits:
  - Apple
  - Banana
  - Mango
```
## Data Types in YAML
Scalars: string, int, float, bool, null
```yaml
age: 25
height: 5.9
isAdmin: true
nickname: null
```
Lists:
```yaml
skills:
  - GCP
  - Kubernetes
  - DevOps
```
Dictionaries (maps):
```yaml
person:
  name: Sahil
  job: Engineer
  location: India
```
## Advanced YAML Features
### Nested structures:
```yaml
server:
  name: app-server
  ip: 192.168.1.10
  ports:
    - 80
    - 443
```
### Anchors (&) and Aliases (*) (reusability):
```yaml
default: &default
  retries: 3
  timeout: 30s

service1:
  <<: *default
  name: frontend
```
#### YAML Anchors (&) and Aliases (*)
##### What are Anchors and Aliases?
- Anchor (`&`) = Save a block of configuration with a name (like a variable).
- Alias (`*`) = Reuse that saved block anywhere else in the YAML.

Think of it like copy-paste without writing everything again.

##### Basic Example
```yaml
default: &default_values
  retries: 3
  timeout: 30s

service1:
  <<: *default_values
  name: frontend

service2:
  <<: *default_values
  name: backend
```
##### Explanation:
- `&default_values` = anchor (saved config).
- `*default_values` = alias (reuses the config).
- `<<:` = merge the reused config into the current object.

Result after YAML expands:
```yaml
service1:
  retries: 3
  timeout: 30s
  name: frontend

service2:
  retries: 3
  timeout: 30s
  name: backend
```
Without Anchors (Repetition Problem)
```yaml
service1:
  retries: 3
  timeout: 30s
  name: frontend

service2:
  retries: 3
  timeout: 30s
  name: backend
```
Here you had to repeat retries and timeout.
With anchors → you write it once, reuse multiple times.

##### Real Example: Kubernetes Pod
```yaml
container_defaults: &container_defaults
  image: nginx
  ports:
    - containerPort: 80

pod1:
  containers:
    - <<: *container_defaults
      name: frontend

pod2:
  containers:
    - <<: *container_defaults
      name: backend
```
Both `pod1` and `pod2` reuse the same container config, only the `name` changes.

#### When to Use Anchors & Aliases?  
- When you have repeated configs (timeouts, ports, labels, env vars).
- Makes YAML cleaner, shorter, and easy to update.
  - Change one place → it reflects everywhere.

In simple words:

`&` = save something
`*` = use it again anywhere


### Multiline strings:
```yaml
description: |
  This is a long text
  that spans multiple lines.
```
### Comparison: YAML vs JSON
JSON:
```json
{
  "name": "Sahil",
  "skills": ["GCP", "Kubernetes"]
}
```
YAML: 
```yaml
name: Sahil
skills:
  - GCP
  - Kubernetes
```
YAML is shorter and more readable.
### YAML in Real Tools
Docker Compose:
```yaml
version: "3"
services:
  web:
    image: nginx
    ports:
      - "80:80"
```
Kubernetes Pod:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 80
```
### Practice Plan

🔹 Step 1: Write simple key-value YAML
🔹 Step 2: Create lists and dictionaries
🔹 Step 3: Mix lists + dictionaries (nested)
🔹 Step 4: Try Docker Compose & Kubernetes YAML
🔹 Step 5: Validate YAML using an online linter (like yamllint.com)
