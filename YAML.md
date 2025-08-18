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

- Step 1: Write simple key-value YAML
- Step 2: Create lists and dictionaries
- Step 3: Mix lists + dictionaries (nested)
- Step 4: Try Docker Compose & Kubernetes YAML
- Step 5: Validate YAML using an online linter (like yamllint.com)

## CamelCase Concept
### What is CamelCase?
CamelCase is a naming style where:
- The first word starts lowercase
- Every next word starts with an uppercase letter
- No spaces or underscores between words

👉 It looks like the “humps of a camel” 🐪 — that’s why it’s called CamelCase.

### Examples
- firstName ✅
- lastName ✅
- myPhoneNumber ✅
#### ❌ Wrong ways:
- firstname (hard to read)
- FirstName (this is called PascalCase, not camelCase)
- first_name (this is called snake_case)

### Types of CamelCase
#### lowerCamelCase → first letter is lowercase
- Example: myVariableName
- Used in: JavaScript variables, Java methods, JSON keys

#### UpperCamelCase (PascalCase) → first letter is uppercase
- Example: MyVariableName
- Used in: Class names in Java, C#, Python

### Comparison with Other Naming Styles
| Style       | Example            | Used In                    |
| ----------- | ------------------ | -------------------------- |
| camelCase   | `myVariableName`   | JavaScript, Java methods   |
| PascalCase  | `MyVariableName`   | Class names                |
| snake\_case | `my_variable_name` | Python variables/functions |
| kebab-case  | `my-variable-name` | URLs, CSS classes          |

### Why use CamelCase?
- Improves readability
- Avoids spaces (not allowed in code identifiers)
- Follows coding conventions of many languages

Example in JavaScript:
```javascript
let firstName = "Sahil";   // camelCase
class UserProfile { }      // PascalCase
```

## Is YAML a Language or a Format?
### YAML is both a "data serialization language" and a "format".
- Officially, YAML is a data serialization language → meaning it is a way to represent structured data in text form.
- Practically, YAML acts as a format (like JSON, XML, CSV) used to store and exchange configuration data.

### Language Aspect
- YAML has syntax rules (indentation, key-value, lists, anchors, etc.).
- It allows you to describe structured data (maps, lists, scalars).
#### Example:
```yaml
person:
  name: Sahil
  age: 25
  skills:
    - Kubernetes
    - GCP
```
Here, YAML is acting like a mini-language for describing data.

### Format Aspect
- YAML is used as a file format for configuration in tools like:
  - Kubernetes (`deployment.yaml`)
  - Docker Compose (`docker-compose.yaml`)
  - Ansible Playbooks (`playbook.yaml`)
  - GitHub Actions (`workflow.yaml`)

It’s basically a human-friendly alternative to JSON or XML.

### Key Difference
- Programming Language (like Python, Java): executes logic, has loops, conditions, functions.
- Serialization Language / Format (like YAML, JSON, XML): only represents and structures data, no logic execution.
- So, YAML ≠ a programming language
- YAML = data serialization language + configuration file format
- YAML is like a recipe (ingredients + steps written clearly).
- Programming language is like the chef who actually cooks using that recipe.
- YAML is not a programming language.
- YAML is a data serialization language (human-readable) and used as a format for configuration files.