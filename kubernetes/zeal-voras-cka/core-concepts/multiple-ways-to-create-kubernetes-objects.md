# Multiple Ways to Create Kubernetes Objects

In Kubernetes, there are **two primary approaches** to creating objects (like Pods, Deployments, or Services):

1. **Imperative Commands (kubectl run)**
2. **Declarative Manifests (kubectl apply -f)**

***

### 🔹 Approach 1 – Imperative (kubectl command)

The simplest way is to use direct **kubectl commands** to create resources.

**Example:**

```bash
$ kubectl run nginx --image=nginx
pod/nginx created
```

* Quick and useful for **testing or one-off tasks**.
* The downside: no record of the command exists for versioning or future reuse.

***

### 🔹 Approach 2 – Declarative (Manifest File)

A **Kubernetes manifest file** is a YAML or JSON file that defines the **desired state** of a resource.

You apply it using:

```bash
$ kubectl apply -f manifest.yaml
```

**Example – Pod manifest (`pod.yaml`):**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx-container
    image: nginx
```

This instructs Kubernetes to create a Pod named `nginx` running the `nginx` image.

***

### 🎯 Benefits of Using Manifest Files

1. **Version Control**
   * Manifests can be stored in **Git or other VCS**, enabling infrastructure-as-code.
   * Easy rollback to previous configurations.
2. **Multiple Resources in One File**
   * You can define multiple dependent objects together, ensuring consistency.

**Example – Pod + Service (`demo.yaml`):**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
    - name: nginx-container
      image: nginx:latest
      ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30000
```

* Creates both a **Pod** (`nginx-pod`) and a **Service** (`nginx-service`) in a single step.
* Ensures the Pod and Service are always deployed together.

***

## 🚀 Quick Summary

* **kubectl run** → Fast, imperative commands, good for quick testing.
* **kubectl apply -f manifest.yaml** → Declarative, reproducible, and version-controlled approach.
* **Manifests are the standard way** to manage Kubernetes objects in production (infrastructure as code).

👉 In practice: use **imperative commands** for quick experiments, but prefer **manifest files** for real-world deployments and team collaboration.
