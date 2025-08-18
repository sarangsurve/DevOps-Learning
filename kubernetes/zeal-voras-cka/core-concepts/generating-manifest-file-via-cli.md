# Generating Manifest File via CLI

In Kubernetes, there are two common ways to create resources:

* **CLI commands (imperative):** Quick and easy for testing.
* **Manifest files (declarative):** Preferred in organizations for collaboration, version control, and reproducibility.

A smart approach is to **generate manifest files using CLI commands** — combining the simplicity of CLI with the flexibility of manifests.

***

### 🔹 Basics of Dry Runs

The `--dry-run=client` option allows you to **validate a resource definition without actually creating it** in the cluster.

**Example:**

```bash
$ kubectl run nginx --image=nginx --dry-run=client
pod/nginx created (dry run)
```

Here, Kubernetes confirms what would be created, but **no Pod is actually launched**.

***

### 🔹 Default Output of kubectl

Normally, if you create a resource, kubectl just prints a confirmation message.

**Example:**

```bash
$ kubectl run nginx --image=nginx
pod/nginx created
```

This is convenient, but does not provide details of the resource definition.

***

### 🔹 Generating YAML Output

By adding the `-o yaml` flag, kubectl **prints the full manifest** (YAML) of the resource it is about to create.

**Example:**

```bash
$ kubectl run nginx --image=nginx -o yaml
```

**Output snippet:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: default
spec:
  containers:
  - image: nginx
    name: nginx
```

This is extremely useful for understanding what Kubernetes is creating under the hood.

***

### 🔹 Combining Dry Run + YAML Output

The most powerful option is combining `--dry-run=client` with `-o yaml`.\
This lets you **generate a manifest file directly from a kubectl command**, without actually deploying anything.

**Example:**

```bash
$ kubectl run nginx --image=nginx --dry-run=client -o yaml
```

**Output snippet:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

👉 You can now **save this YAML to a file** (e.g., `pod.yaml`) and reuse it for deployments with:

```bash
$ kubectl apply -f pod.yaml
```

***

## 🚀 Quick Summary

* **`--dry-run=client`** → Validates resource definition, no creation.
* **`-o yaml`** → Outputs full YAML manifest instead of a confirmation.
* **Combined usage** → Best practice for generating reusable **manifest files** from CLI commands.

This approach bridges **imperative** (quick) and **declarative** (collaborative) styles of working with Kubernetes.
