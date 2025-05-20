# Kubernetes Probes

In Kubernetes, **probes** are mechanisms used by the kubelet (the agent running on each node) to check the health and status of containers in a Pod. They help Kubernetes manage the lifecycle of containers by determining:

1. **Is the container running properly?**
2. **Is it ready to accept traffic?**

***

There are **three types of probes**:

#### 1. **Liveness Probe**

* **Purpose**: Checks if the container is still running.
* **If it fails**: Kubernetes restarts the container.
* **Use case**: Detecting when the application is stuck or deadlocked.

#### 2. **Readiness Probe**

* **Purpose**: Checks if the container is ready to serve traffic.
* **If it fails**: The Pod is removed from the Service's endpoints, so it stops receiving traffic.
* **Use case**: Ensuring traffic isn't sent to an app before it's fully initialized or ready.

#### 3. **Startup Probe**

* **Purpose**: Checks if the container has started up successfully.
* **If it fails**: Kubernetes restarts the container.
* **Use case**: Useful for slow-starting applications; prevents premature liveness probe failures.

***

#### How Probes Work

Each probe can use one of the following methods:

* **HTTP GET**: Makes an HTTP request to a specified endpoint.
* **TCP Socket**: Attempts to open a TCP connection on a specified port.
* **Exec**: Runs a command inside the container and checks the exit code.

***

#### Does Kubernetes Probes fall under Kubernetes Administrator or under Kubernetes Application Developer

**Kubernetes Probes** fall under **both roles**, but in slightly different ways:

* **Developers define probes**.
* **Administrators monitor and fine-tune them**.

***

#### **Kubernetes Application Developer**

**Primary responsibility**

* **Design and configure probes** in the Pod or Deployment YAML.
* Understand the app’s startup, readiness, and failure patterns.
* Set appropriate endpoints or commands for probes.

📌 _Why?_ Developers know how their app behaves and what “ready” or “healthy” actually means

***

#### **Kubernetes Administrator (DevOps / SRE)**

**Secondary responsibility**

* **Validate and tune** probe settings for cluster performance.
* Monitor probe-related restarts or failures via metrics and logs.
* Help developers choose sane defaults or cluster-wide standards.

📌 _Why?_ Admins are responsible for overall system stability and may catch probe misconfigurations affecting deployments.

***

#### A simple example of a Kubernetes Pod YAML with **liveness**, **readiness**, and **startup** probes configured for a container running a web app:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
  - name: my-container
    image: my-app-image:latest
    ports:
    - containerPort: 8080

    # Liveness Probe: restarts the container if it fails
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 10
      periodSeconds: 5
      failureThreshold: 3

    # Readiness Probe: removes pod from service if it fails
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5
      failureThreshold: 2

    # Startup Probe: waits longer before triggering liveness
    startupProbe:
      httpGet:
        path: /startup
        port: 8080
      initialDelaySeconds: 0
      periodSeconds: 5
      failureThreshold: 30

```

#### Quick Breakdown:

* **`/healthz`**: App's general health (used by liveness).
* **`/ready`**: Indicates if the app can serve traffic.
* **`/startup`**: Used only during startup to delay liveness checks.
* **`initialDelaySeconds`**: Wait time before the probe starts running.
* **`failureThreshold`**: Number of consecutive failures before action is taken.

You can customize these paths and timings based on how your app behaves.
