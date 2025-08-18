# Basics of Kubernetes Resource Types

### 🔹 Analogy – Hosting Provider Evolution

* **Early days:** Hosting providers offered just **servers / VMs** to run applications.
* **Today:** Cloud providers offer **hundreds of services** to cover the **entire application lifecycle** — from compute, networking, and storage to security and monitoring.

***

### 🔹 Kubernetes Perspective

Similarly in Kubernetes:

* **Pods** allow you to host applications inside containers.
* But Pods alone are not enough for real-world applications.
* Depending on the use case, Pods often require support from other **Kubernetes resource types** (e.g., Secrets, Volumes, Services).

Kubernetes provides a **broad set of resources** to cover application deployment, scaling, networking, configuration, and storage needs.

***

### 🔑 Common Kubernetes Resource Types

* **Pod** – Smallest deployable unit; represents one or more containers running together.
* **Deployment** – Manages multiple replicas of Pods; supports scaling and rolling updates.
* **Service** – Provides a stable network endpoint and load balances traffic to Pods.
* **ConfigMap** – Stores non-sensitive configuration data (e.g., app settings).
* **Secret** – Stores sensitive data (e.g., passwords, tokens, certificates) securely.
* **PersistentVolume (PV)** – Represents storage resources (e.g., disk, cloud storage).
* **PersistentVolumeClaim (PVC)** – Requests a specific amount of storage from a PV.
* **Ingress** – Manages external HTTP/HTTPS traffic and routes it to Services inside the cluster.
* **Namespace** – Logical partitioning to group and isolate resources in a cluster.

***

### 🖥️ Example – Listing Available Resources

You can check the full set of supported resources in your cluster using:

```bash
$ kubectl api-resources
NAME                SHORTNAMES   APIVERSION   NAMESPACED   KIND
configmaps          cm           v1           true         ConfigMap
secrets                          v1           true         Secret
persistentvolumes   pv           v1           false        PersistentVolume
services            svc          v1           true         Service
deployments         deploy       apps/v1      true         Deployment
pods                po           v1           true         Pod
..
```

This shows all available resource types, their short names, and whether they are **namespaced**.

***

## 🚀 Quick Summary

* Kubernetes goes **beyond Pods** → it provides a rich set of **resource types** for managing applications.
* Resources cover compute (**Pods, Deployments**), networking (**Services, Ingress**), storage (**PV, PVC**), and configuration/security (**ConfigMaps, Secrets**).
* Use `kubectl api-resources` to explore all supported resource types in your cluster.

👉 Just like cloud providers offer diverse services, Kubernetes offers diverse resources to support applications end-to-end.
