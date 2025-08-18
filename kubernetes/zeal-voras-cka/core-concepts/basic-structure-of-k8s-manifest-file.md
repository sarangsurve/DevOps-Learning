# Basic Structure of K8S Manifest File

In Kubernetes, most resources (Pods, Deployments, Services, etc.) are defined and created using a **manifest file**.\
Manifest files are usually written in **YAML** (sometimes JSON) and describe the **desired state** of the resource.

***

### 🔹 Example – Pod Manifest File

**pod.yaml**

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

This manifest tells Kubernetes:

* **apiVersion** – use `v1` API.
* **kind** – create a `Pod`.
* **metadata** – give it the name `nginx`.
* **spec** – inside this Pod, run a container named `nginx-container` using the `nginx` image.

***

### 🔹 Generic Manifest Structure

**manifest-structure.yaml**

```yaml
apiVersion: [API version]
kind: [Resource type]
metadata:
  name: [Resource name]
spec:
  [Resource-specific configuration]
```

***

### 🔹 Comparison (Generic vs Pod Example)

<table data-header-hidden><thead><tr><th width="243.4000244140625"></th><th width="358.0001220703125"></th><th width="272.800048828125"></th></tr></thead><tbody><tr><td><strong>Manifest Component</strong></td><td><strong>Generic Structure</strong></td><td><strong>Pod Example (pod.yaml)</strong></td></tr><tr><td>API version</td><td><pre class="language-yaml"><code class="lang-yaml">apiVersion: [API version]
</code></pre></td><td><pre class="language-yaml"><code class="lang-yaml">apiVersion: v1
</code></pre></td></tr><tr><td>Resource type (kind)</td><td><pre class="language-yaml"><code class="lang-yaml">kind: [Resource type]
</code></pre></td><td><pre class="language-yaml"><code class="lang-yaml">kind: Pod
</code></pre></td></tr><tr><td>Metadata (name, labels, etc.)</td><td><pre class="language-yaml"><code class="lang-yaml">metadata:
  name: [Resource name]
</code></pre></td><td><pre class="language-yaml"><code class="lang-yaml">metadata:
  name: nginx
</code></pre></td></tr><tr><td>Spec (details)</td><td><pre class="language-yaml"><code class="lang-yaml">spec:
  [Resource-specific configuration]
</code></pre></td><td><pre class="language-yaml"><code class="lang-yaml">spec:
  containers:
  - name: nginx-container
    image: nginx
</code></pre></td></tr></tbody></table>

***

### 🔑 Key Manifest Components

<table data-header-hidden><thead><tr><th width="135.79998779296875"></th><th></th></tr></thead><tbody><tr><td><strong>Component</strong></td><td><strong>Description</strong></td></tr><tr><td><strong>apiVersion</strong></td><td>Defines which version of the Kubernetes API to use for this resource (e.g., <code>v1</code>, <code>apps/v1</code>).</td></tr><tr><td><strong>kind</strong></td><td>The type of resource being created (e.g., <code>Pod</code>, <code>Deployment</code>, <code>Service</code>).</td></tr><tr><td><strong>metadata</strong></td><td>Identifying information about the resource (name, labels, annotations).</td></tr><tr><td><strong>spec</strong></td><td>The specification of the resource — defines the desired state (e.g., containers inside a Pod, replicas in a Deployment, ports in a Service).</td></tr></tbody></table>

***

## 🚀 Quick Summary

* A **manifest file** defines the **desired state** of a Kubernetes object.
* All manifests follow a **basic structure**:
  * `apiVersion`
  * `kind`
  * `metadata`
  * `spec`
* The `spec` section changes depending on the resource type.
* Example: a Pod’s spec defines **containers**, while a Deployment’s spec defines **replicas and Pod templates**.

👉 Think of a manifest file as the **blueprint** that Kubernetes follows to create and manage resources.

