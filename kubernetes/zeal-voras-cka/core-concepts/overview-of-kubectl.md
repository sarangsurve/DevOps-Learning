# Overview of kubectl

**kubectl** is the **command-line tool** for interacting with Kubernetes clusters. It allows administrators and developers to:

* Deploy and manage applications.
* Inspect and troubleshoot cluster resources.
* View cluster state and logs.
* Execute commands inside containers.

In short, **kubectl is the primary interface between you and the Kubernetes API Server**.

***

### 🔑 Authentication with kubectl

Before kubectl can communicate with the Kubernetes cluster, it needs two critical pieces of information:

1. **Cluster Address** → The DNS name or IP address of the **Control Plane (API Server)**.
2. **Authentication Credentials** → Token, certificate, or other authentication data to securely connect.

These details are stored in a configuration file called **kubeconfig** (named `config`, without an extension).

kubectl automatically uses the **kubeconfig file** to determine:

* Which cluster to connect to.
* Which user credentials to use.
* Which namespace and context are active.

***

### 📂 Default Path of Kubeconfig File

By default, kubectl looks for the kubeconfig file in your home directory under `.kube/config`:

*   **Linux / macOS / RHEL:**

    ```
    ~/.kube/config
    ```
*   **Windows:**

    ```
    %USERPROFILE%/.kube/config
    ```

    (typically `C:/Users/<logged-in-user>/.kube/config`)

***

### ⚙️ Using a Custom Kubeconfig File

You can specify a **custom kubeconfig file** instead of the default one by using the `--kubeconfig` flag.

**Example:**

```bash
$ kubectl get nodes --kubeconfig custom-kubeconfig
```

This is useful when managing **multiple clusters** or working with **different credentials**.

***

## 🚀 Summary

* **kubectl** → Command-line tool to interact with Kubernetes clusters.
* **Authentication** → Uses cluster address + credentials stored in a **kubeconfig** file.
* **Default location** → `~/.kube/config` (Linux/macOS), `%USERPROFILE%/.kube/config` (Windows).
* **Custom config** → Use `--kubeconfig` flag to reference another config file.

👉 For everyday Kubernetes operations, **kubectl + kubeconfig** is the standard workflow.
