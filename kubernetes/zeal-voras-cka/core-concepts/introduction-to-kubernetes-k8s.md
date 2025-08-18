# Introduction to Kubernetes (K8s)

**Kubernetes (K8s)** is an **open-source container orchestration engine** originally designed by **Google**, and now maintained by the **Cloud Native Computing Foundation (CNCF)**.

It provides a powerful platform to **deploy, scale, and manage containerized applications** automatically, based on the same principles that enable Google to run **billions of containers per week**.

***

## 🏗️ Kubernetes Architecture

A **Kubernetes cluster** is made up of two main components:

#### 1. **Control Plane (Master)**

The **Control Plane** manages the cluster. It is responsible for making **decisions** about scheduling, scaling, and maintaining the desired state of the system.

Key components of the Control Plane:

* **API Server** – Acts as the front-end of Kubernetes; all requests (from users, CLI, or other components) go through it.
* **etcd** – A distributed key-value store used as Kubernetes’ **source of truth** (stores cluster state and configuration).
* **Controller Manager** – Watches the cluster state and makes necessary changes to bring it back to the desired state (e.g., ensuring the correct number of Pods are running).
* **Scheduler** – Assigns containers (Pods) to appropriate worker nodes based on resource availability and policies.

#### 2. **Worker Nodes**

Worker nodes are the **machines (physical or virtual)** where the **containers actually run**. Each node is managed by the control plane and contains:

* **Kubelet** – An agent that runs on every node; it communicates with the control plane and ensures containers are running as instructed.
* **Kube-Proxy** – Handles **networking** on each node; enables communication between containers and external services.
* **Container Runtime** – The software responsible for running containers (e.g., Docker, containerd, CRI-O).

👉 In simple terms:

* **Control Plane = the brain** of Kubernetes.
* **Nodes = the workers** that actually run the containers.

**Example:**\
Imagine a cluster with **1 control plane server** and **3 worker nodes**. The control plane decides what runs where, and the worker nodes host the running containers.

***

## 🔄 Basic Kubernetes Workflow

The workflow in Kubernetes follows a **desired state model**:

1. The **user tells Kubernetes what they want** (e.g., “Run 2 Nginx containers”).
2. The **Control Plane decides how to achieve that state** (which node, how many replicas, networking, etc.).
3. The **Worker Nodes execute** the instructions and run the containers.
4. Kubernetes continuously **monitors and corrects** the system to ensure the desired state is always met.

***

#### Example 1 – Multiple Requirements

User instructs the Control Plane:

1. Run **2 containers of Nginx**.
2. Run **app container on the largest node**.

➡️ Kubernetes decides placement:

* Deploys 2 Nginx containers on the **small node**.
* Deploys the app container on the **large node**.

***

#### Example 2 – Single Container

User requests:

```bash
Run 1 container of Apache
```

➡️ Kubernetes schedules it on one of the worker nodes.\
➡️ If the Apache container crashes, Kubernetes will automatically restart or replace it.

This highlights the **self-healing** ability of Kubernetes.

***

## 🚀 Key Features of Kubernetes

Kubernetes offers a rich set of features that make it the industry standard for container orchestration:

* **Pod Auto-Scaling** – Adjusts the number of Pods based on CPU/memory usage or custom metrics.
* **Service Discovery & Load Balancing** – Provides built-in service discovery and distributes traffic across Pods.
* **Self-Healing** – Restarts failed containers, reschedules them on healthy nodes, and ensures high availability.
* **Secret & Config Management** – Securely stores sensitive data like passwords, tokens, and configuration.
* **Automated Rollouts & Rollbacks** – Gradually updates applications with zero downtime, and rolls back automatically if something fails.

***

📖 **In summary**:\
Kubernetes is the **standard container orchestration platform**. Its **control plane + worker node architecture** ensures applications are deployed, scaled, and kept healthy without manual intervention. With features like **auto-scaling, self-healing, and automated deployments**, Kubernetes provides the reliability and flexibility needed to run applications at scale.
