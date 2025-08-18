# Kubernetes Commands and Arguments

### 🔹 Docker Background

When building a Docker image, you can define default process behavior using:

* **ENTRYPOINT** → The main executable that always runs when the container starts.
* **CMD** → Default arguments (or fallback command) passed to the ENTRYPOINT.

Example:

```dockerfile
FROM nginx:1.10.1-alpine
COPY index.html /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

* ENTRYPOINT not defined → container starts with CMD.
*   When `docker run nginx`, the process executed is:

    ```bash
    nginx -g "daemon off;"
    ```

#### ⚡ Example – Short-lived Process

```dockerfile
FROM busybox:latest
CMD ["ping", "-c", "3", "google.com"]
```

* Runs `ping` three times, then exits.
* No long-running process → container stops immediately.

#### ENTRYPOINT vs CMD

```dockerfile
FROM ubuntu
ENTRYPOINT ["/bin/echo"]
CMD ["hello", "world"]
```

* `docker run demo` → `hello world`
* `docker run demo how are you` → `how are you`

👉 ENTRYPOINT is fixed, CMD can be overridden at runtime.

***

### 🔹 Kubernetes Pod: Commands and Args

In Kubernetes:

* `command` → overrides **ENTRYPOINT**.
* `args` → overrides **CMD**.

Example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: command-demo
spec:
  containers:
  - name: demo
    image: busybox
    command: ["/bin/echo"]
    args: ["hello", "world"]
```

When this Pod runs:

```bash
hello world
```

***

### 🔹 Defining Commands in kubectl

**General Syntax:**

```bash
kubectl run <pod-name> --image=<image> --command -- <command> <args>
```

**Example:**

```bash
$ kubectl run my-echo-pod --image=busybox:latest --command -- echo "Hello from kubectl run!"
pod/my-echo-pod created

$ kubectl logs my-echo-pod
Hello from kubectl run!
```

* Pod status: `Completed` (since echo is a short-lived command).

***

### 🔹 Defining in Manifest File

Both **array notation** and **multi-line YAML list** are supported:

**1. Array Notation (compact, inline):**

```yaml
command: ["/bin/sh", "-c", "echo Hello Kubernetes!"]
```

**2. Multi-line YAML List (readable for long commands):**

```yaml
command:
- "/bin/sh"
- "-c"
- "echo Hello Kubernetes!"
```

***

### High-Level Comparison

| **Feature**  | **Array Notation `[]`**    | **Multi-line YAML `-`**    |
| ------------ | -------------------------- | -------------------------- |
| Syntax Style | JSON-style array (compact) | YAML-style list (indented) |
| Readability  | Good for short commands    | Clearer for long scripts   |
| Best Use     | Inline, small commands     | Complex/long commands      |

👉 Recommendation:

* Use **array notation** for short commands.
* Use **multi-line YAML** for readability in complex cases.

***

### 🔹 Best Practices & Notes

* Keep **command** = the main executable, **args** = parameters.
* If image defines ENTRYPOINT and CMD:
  * Kubernetes uses them **unless overridden** in the manifest.
* Using separate `command` and `args` improves clarity and maintainability.
* If your Pod is exiting immediately, check if the command is **short-lived** (e.g., `echo`, `ping -c 3`).

***

## 🚀 Quick Summary

* **ENTRYPOINT vs CMD in Docker** → ENTRYPOINT = main process, CMD = default args.
* **Kubernetes mapping** → `command` = ENTRYPOINT, `args` = CMD.
* **kubectl run** allows defining inline commands.
* **Manifest files** provide structured ways (array or YAML list).
* Keep `command` and `args` separate for readability and flexibility.
