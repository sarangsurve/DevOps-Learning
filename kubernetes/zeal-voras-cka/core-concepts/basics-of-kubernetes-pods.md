# Basics of Kubernetes Pods

### 🔹 Setting the Base – Docker Analogy

When you first install Docker, the simplest action is to run a container:

```bash
docker run nginx
```

This directly launches an **nginx container** on your system.

***

### 🔹 Setting the Base – Kubernetes Analogy

When you set up a Kubernetes cluster, the equivalent first action is:

```bash
kubectl run nginx --image=nginx
```

This command creates a **Pod** in Kubernetes that runs an **nginx container**.

***

### 🔑 Difference Between Containers and Pods

* **Container (in Docker environment):** Runs in isolation with its own networking and storage.
* **Pod (in Kubernetes environment):** The smallest deployable unit in Kubernetes, which can **wrap one or more containers**.

👉 Key point:

* A **Docker container** runs individually.
* A **Kubernetes Pod** can contain **multiple containers** (e.g., `nginx-container` + `app-container`) that:
  * Share the same **network namespace** (communicate via `localhost`).
  * Share **storage volumes** if defined.

This allows containers within a Pod to work closely together, like parts of a single application.



***

### Comparison Table - Docker vs Kubernetes Commands

<table><thead><tr><th width="267.20001220703125">Docker Command</th><th width="292">Kubernetes Command</th><th width="258.800048828125">Purpose</th></tr></thead><tbody><tr><td>docker run nginx </td><td>kubectl run nginx --image=nginx </td><td>Create and run a container/pod</td></tr><tr><td>docker ps </td><td>kubectl get pods </td><td>List running containers/pods</td></tr><tr><td>docker logs &#x3C;container> </td><td>kubectl logs &#x3C;pod> </td><td>View logs</td></tr><tr><td>docker inspect &#x3C;container> </td><td>kubectl describe pod &#x3C;pod-name> </td><td>Get detailed info</td></tr><tr><td>docker exec -it &#x3C;container> bash </td><td>kubectl exec -it &#x3C;pod-name> -- bash </td><td>Execute interactive shell</td></tr><tr><td>docker rm &#x3C;container> </td><td>kubectl delete pod &#x3C;pod-name> </td><td>Remove container/pod</td></tr></tbody></table>

***

### 📍 Points to Note – Kubernetes Pod

* A **Pod always runs on a Node**.
* A **Node** = worker machine in Kubernetes (VM or physical server).
* Each Node is managed by the **Control Plane**.
* A **Node can host multiple Pods**.
* Pods are **ephemeral** → if a Pod fails, Kubernetes may create a replacement Pod (with a new identity).

***

## 🚀 Quick Summary

* **Pod = Smallest deployable unit in Kubernetes.**
* **Analogy:** `docker run` launches a container, `kubectl run` launches a Pod.
* **Pods can hold multiple tightly-coupled containers**, sharing network and storage.
* **Pods run on Nodes**, and Nodes are managed by the Control Plane.
