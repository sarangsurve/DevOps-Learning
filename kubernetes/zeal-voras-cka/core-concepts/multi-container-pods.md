# Multi-Container Pods

### 🔹 Pod vs Container

* A **container** (Docker world) runs independently, with its own isolated networking and storage.
* A **Pod** (Kubernetes world) is the **smallest deployable unit**, and it can host **one or more containers**.
* Containers inside the same Pod:
  * Share the **same network namespace** (communicate via `localhost`).
  * Can share **storage volumes** if configured.

👉 This makes Pods ideal for **tightly coupled containers** that must work closely together.

***

### 🔹 Default Option with kubectl

* Using `kubectl run`, you can **only create single-container Pods**.
* To create **multi-container Pods**, you must use a **manifest file**.

***

### 🔹 Multi-Container Pod&#x20;

### Example

**multi-container-pods.yaml**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  containers:
  - name: nginx-container
    image: nginx
  - name: redis-container
    image: redis
```

**Apply the manifest:**

```bash
$ kubectl apply -f multi-container-pods.yaml
pod/multi-container-pod created
```

This Pod will run **two containers**:

* `nginx-container` (web server)
* `redis-container` (in-memory database)

***

### 🔹 Executing into Containers

By default, `kubectl exec` connects to the **first container** defined in the Pod.

**Example – Default exec:**

```bash
$ kubectl exec -it multi-container-pod -- bash
Defaulted container "nginx-container" out of: nginx-container, redis-container
root@multi-container-pod:/#
```

***

### 🔹 Executing into a Specific Container

To connect to a different container inside the Pod, use the `-c` flag with the container name.

**Example – Exec into redis-container:**

```bash
$ kubectl exec -it multi-container-pod -c redis-container -- bash
root@multi-container-pod:/data#
```

***

## 🚀 Quick Summary

* **Pods can host multiple containers** → useful for tightly coupled workloads.
* **kubectl run** → creates single-container Pods.
* **Manifest files** → required for multi-container Pods.
* **kubectl exec** → by default connects to the first container; use `-c` to target a specific container.

👉 Multi-container Pods are often used in patterns like **sidecar containers** (logging, monitoring, proxying) or **helper containers** working alongside the main application.
