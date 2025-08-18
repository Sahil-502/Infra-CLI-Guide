# The `base64` Command in Linux

### What is Base64?
- Base64 = a way to encode binary data (files, images, passwords) into text using only 64 characters:
`A–Z, a–z, 0–9, +, /` and `=` (for padding).
- Useful when you need to transfer data over systems that only handle text safely (like YAML, JSON, HTTP, email).

### Basic Usage
Encode a string:
```
echo "sahil" | base64
```
Output:
```
c2FoaWwK
```
Decode back:
```
echo "c2FoaWwK" | base64 --decode
```
Output:
```
sahil
```
👉 Notice the --decode (or -d) flag.
### Encode/Decode Files
Encode a file:
```
base64 myfile.txt > encoded.txt
```
Decode back:
```
base64 --decode encoded.txt > original.txt
```

### Common Options
| Option           | Description                                                                 |
| ---------------- | --------------------------------------------------------------------------- |
| `base64 file`    | Encode file                                                                 |
| `base64 -d file` | Decode file                                                                 |
| `-w N`           | Wrap output at every N characters (default 76). Use `-w 0` for no wrapping. |

Example:
```
echo "sahil" | base64 -w 0
```
### Real DevOps Use Cases
#### Kubernetes Secrets
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
data:
  username: c2FoaWw=
  password: cGFzc3dvcmQ=
```
Encode values before storing:
```
echo -n "sahil" | base64   # c2FoaWw=
echo -n "password" | base64  # cGFzc3dvcmQ=
```
Decode to check:
```
echo "c2FoaWw=" | base64 --decode
```

### Email Attachments
Email (SMTP) only supports text → attachments get converted to Base64.

### API/CI-CD Pipelines
Some APIs require credentials in Base64. Example:
```
echo -n "username:password" | base64
```
Used in HTTP headers:
```makefile
Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=
```
### Storing Binary Data in YAML/JSON
If you need to include a file (like a cert or SSH key) inside a YAML:
```
cat id_rsa | base64 -w 0
```
Then paste it into YAML.
### Quick Security Note
- Base64 is not encryption ❌ — it’s just encoding.
- Anyone can decode Base64 back to original.
- Use it only for safe transport, not for protection.

### Summary
- `base64` = encode/decode data as text.
- Common in Kubernetes, APIs, YAML configs, email.
- Encode: `echo "text" | base64`
- Decode: `echo "encoded" | base64 -d`
