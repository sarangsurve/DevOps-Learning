# ReplicaSet

### Setting the Base <a href="#setting-the-base" id="setting-the-base"></a>

Requirement: Run **3 Pods** based on the `nginx` image at all times.

Manual approach:

```bash
$ kubectl run nginx-pod-1 --image=nginx
$ kubectl run nginx-pod-2 --image=nginx
$ kubectl run nginx-pod-3 --image=nginx
```

Result → 3 running Pods:

* `nginx-pod-1`
* `nginx-pod-2`
* `nginx-pod-3`

***

### Problem with Manual Pods

#### ❌ Pod Failure (Issue 1)

* If a Pod fails (node crash, process kill, etc.), **Kubernetes won’t recreate it**.
* Example: `nginx-pod-1` and `nginx-pod-2` stop → user must manually restart.

#### ❌ Scalability (Issue 2)

* Scaling = manually creating/deleting multiple Pod manifests.
* Example: Requirement → 20 Pods. You’d need **20 `kubectl run` commands** → time-consuming.

***

### Introducing ReplicaSet

ReplicaSet ensures desired number of Pods are running at all times.

Example: Desired = `3` Pods of `nginx`.

* Kubernetes guarantees 3 Pods alive (`nginx-pod-1`, `nginx-pod-2`, `nginx-pod-3`).
* If one fails → auto-recreated.

***

### Solutions by ReplicaSet

* ✅ **Failure Handling** → Auto-recreates failed Pods.
* ✅ **Scaling** → Change replica count in spec → auto scale Pods.

***

### ReplicaSet Practical

⚠️ Note: ReplicaSets are created only via **YAML manifests**, not `kubectl run`.

#### Manifest File: `replicaset.yaml`

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: us-docker.pkg.dev/google-samples/containers/gke/gb-frontend:v5
```

#### Apply

```bash
$ kubectl apply -f replicaset.yaml
replicaset.apps/frontend-replicaset created
```

#### Verify

```bash
$ kubectl get replicaset
NAME                  DESIRED   CURRENT   READY   AGE
frontend-replicaset   3         3         3       30s

$ kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
frontend-replicaset-c7kqt   1/1     Running   0          37s
frontend-replicaset-llxl2   1/1     Running   0          37s
frontend-replicaset-pz2km   1/1     Running   0          37s
```

***

### Scaling ReplicaSet

#### Scale Up (3 → 5)

```bash
$ kubectl scale --replicas=5 rs/frontend-replicaset
replicaset.apps/frontend-replicaset scaled

$ kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
frontend-replicaset-b8cnf   1/1     Running   0          7s
frontend-replicaset-c7kqt   1/1     Running   0          2m13s
frontend-replicaset-llxl2   1/1     Running   0          2m13s
frontend-replicaset-pz2km   1/1     Running   0          2m13s
frontend-replicaset-z9wmj   1/1     Running   0          7s
```

#### Scale Down (5 → 1)

```bash
$ kubectl scale --replicas=1 rs/frontend-replicaset
replicaset.apps/frontend-replicaset scaled

$ kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
frontend-replicaset-pz2km   1/1     Running   0          2m33s
```

***

### Delete ReplicaSet

```bash
$ kubectl delete -f replicaset.yaml
replicaset.apps "frontend-replicaset" deleted
```

***

### Challenges with ReplicaSets

#### ⚠️ Challenge 1: Updating Container Image

* Updating Pod template (e.g., `nginx → httpd`) **doesn’t update existing Pods**.
* Only new Pods will have the updated image.

**Example**

Base ReplicaSet Manifest File Used (`rs.yaml`)

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: webserver-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webserver
  template:
    metadata:
      labels:
        app: webserver
    spec:
      containers:
      - name: nginx-container
        image: nginx:latest
```

Apply&#x20;

```bash
$ kubectl apply -f rs.yaml
replicaset.apps/webserver-replicaset created
```

Verify

```bash
$ kubectl get rs
NAME                   DESIRED   CURRENT   READY   AGE
webserver-replicaset   3         3         3       11s

$ kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
webserver-replicaset-p9wk5   1/1     Running   0          28s
webserver-replicaset-qzjtn   1/1     Running   0          28s
webserver-replicaset-rwmm9   1/1     Running   0          28s
```

Changing the Image of ReplicaSet to Observe Effects (`rs.yaml`)

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: webserver-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webserver
  template:
    metadata:
      labels:
        app: webserver
    spec:
      containers:
      - name: nginx-container
        image: httpd:latest
```

Apply&#x20;

```bash
$ kubectl apply -f rs.yaml
replicaset.apps/webserver-replicaset configured
```

Verify

```bash
$ kubectl get pods # No update on running pods, age is not reset.
NAME                         READY   STATUS    RESTARTS   AGE
webserver-replicaset-p9wk5   1/1     Running   0          81s
webserver-replicaset-qzjtn   1/1     Running   0          81s
webserver-replicaset-rwmm9   1/1     Running   0          81s

$ kubectl describe rs webserver-replicaset # ReplicaSets updated to httpd:latest
Name:         webserver-replicaset
Namespace:    default
Selector:     app=webserver
Labels:       <none>
Annotations:  <none>
Replicas:     3 current / 3 desired
Pods Status:  3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=webserver
  Containers:
   nginx-container:
    Image:         httpd:latest
...

$ kubectl describe pod webserver-replicaset-qzjtn # Existing Pods still runs nginx.
Name:             webserver-replicaset-qzjtn
Namespace:        default
Priority:         0
Service Account:  default
Node:             worker-pool-2m1aq/10.122.0.3
Start Time:       Tue, 19 Aug 2025 18:50:01 +0530
Labels:           app=webserver
Annotations:      <none>
Status:           Running
IP:               10.244.0.39
IPs:
  IP:           10.244.0.39
Controlled By:  ReplicaSet/webserver-replicaset
Containers:
  nginx-container:
    Container ID:   containerd://bf81ce2dd5d7123e1c4ce76c6dcd1deab13c6b4d0359905518f54f610d523cb8
    Image:          nginx:latest
...
```

Deleting one pod of ReplicaSets

```bash
$ kubectl delete pod webserver-replicaset-qzjtn
pod "webserver-replicaset-qzjtn" deleted
```

Verify

```bash
$ kubectl get pods # Age is reset for new created pod.
NAME                         READY   STATUS    RESTARTS   AGE
webserver-replicaset-p9wk5   1/1     Running   0          3m23s
webserver-replicaset-rwmm9   1/1     Running   0          3m23s
webserver-replicaset-ww854   1/1     Running   0          13s

$ kubectl describe pod webserver-replicaset-ww854 # Newly created Pod after deletion runs httpd.
Name:             webserver-replicaset-ww854
Namespace:        default
Priority:         0
Service Account:  default
Node:             worker-pool-2m1aq/10.122.0.3
Start Time:       Tue, 19 Aug 2025 18:53:11 +0530
Labels:           app=webserver
Annotations:      <none>
Status:           Running
IP:               10.244.0.44
IPs:
  IP:           10.244.0.44
Controlled By:  ReplicaSet/webserver-replicaset
Containers:
  nginx-container:
    Container ID:   containerd://aed825dd6b5d495a28e48c4aa0a618f6ff46907afb53183e1b640ef9a61dc7fd
    Image:          httpd:latest
...
```

Workaround, not feasible for production workloads:

```bash
$ kubectl scale rs/webserver-replicaset --replicas=0
replicaset.apps/webserver-replicaset scaled

$ kubectl get pods
No resources found in default namespace.

$ kubectl scale rs/webserver-replicaset --replicas=3
replicaset.apps/webserver-replicaset scaled

$ kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
webserver-replicaset-7dtcq   1/1     Running   0          5s
webserver-replicaset-7ppff   1/1     Running   0          5s
webserver-replicaset-f624v   1/1     Running   0          5s
```

Delete ReplicaSet

```bash
$ kubectl delete -f rs.yaml
replicaset.apps "webserver-replicaset" deleted
```

***

#### ⚠️ Challenge 2: No Built-In Rollback Mechanism

* ReplicaSets lack rollback mechanism.
* If new config breaks Pods → no way to auto-revert.

***

#### ⚠️ Challenge 3: Label Collision with ReplicaSet Selectors

* ReplicaSets controls Pods using **selectors**.
* If **unrelated Pod has same label**, ReplicaSets mistakenly adopts it.

Example: Initially, there is an another unrelated running pod labeled as app=test-app. Now, if user mention ReplicaSet by 3 desired pods for a particular image with label as app=test-app. Kubernetes ReplicaSet will create only 2 pods and mistakenly consider the unrelated pod as a part of its managed replicaset since the label selector is app=test-app.

***

### Label Collision Use-Case

Manually creating an external pod with label app=webserver

```bash
$ kubectl run external-pod --image=nginx
pod/external-pod created

$ kubectl label pod external-pod app=webserver
pod/external-pod labeled

$ kubectl get pods --show-labels
NAME           READY   STATUS    RESTARTS   AGE   LABELS
external-pod   1/1     Running   0          10s   app=webserver,run=external-pod
```

Applying ReplicaSets(`rs.yaml`)&#x20;

```bash
$ kubectl apply -f rs.yaml
replicaset.apps/webserver-replicaset created

$ kubectl get pods # Only 2 new pods are created under webserver-replicaset
NAME                         READY   STATUS    RESTARTS   AGE
external-pod                 1/1     Running   0          29s
webserver-replicaset-kjr44   1/1     Running   0          5s
webserver-replicaset-xs2mj   1/1     Running   0          5s

$ kubectl get rs # Only 2 new pods are created but current state is 3
NAME                   DESIRED   CURRENT   READY   AGE
webserver-replicaset   3         3         3       9s
```

Deleting ReplicaSets(`rs.yaml`)&#x20;

```bash
$ kubectl delete -f rs.yaml
replicaset.apps "webserver-replicaset" deleted

$ kubectl get pods
No resources found in default namespace.

$ # The external-pod is also deleted
```

***

### Avoid Label Collision Use-Case

To avoid Label Collision, Add another label which is not present in the external running pod.

Manually creating an external pod with label app=webserver

```bash
$ kubectl run external-pod --image=nginx
pod/external-pod created

$ kubectl label pod external-pod app=webserver
pod/external-pod labeled

$ kubectl get pods --show-labels
NAME           READY   STATUS    RESTARTS   AGE   LABELS
external-pod   1/1     Running   0          9s    app=webserver,run=external-pod
```

Updating the Image of ReplicaSet with Label env=prod(rs.yaml)

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: webserver-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webserver
      env: prod
  template:
    metadata:
      labels:
        app: webserver
        env: prod
    spec:
      containers:
      - name: nginx-container 
```

Apply

```bash
$ kubectl apply -f rs.yaml
replicaset.apps/webserver-replicaset created

$ kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
external-pod                 1/1     Running   0          36s
webserver-replicaset-c544d   1/1     Running   0          4s
webserver-replicaset-g448x   1/1     Running   0          4s
webserver-replicaset-x8r66   1/1     Running   0          4s

$ kubectl get rs
NAME                   DESIRED   CURRENT   READY   AGE
webserver-replicaset   3         3         3       8s
```

Deleting ReplicaSets(`rs.yaml`)&#x20;

```bash
$ kubectl delete -f rs.yaml
replicaset.apps "webserver-replicaset" deleted

$ kubectl get pods
NAME           READY   STATUS    RESTARTS   AGE
external-pod   1/1     Running   0          59s

$ # The external-pod is not deleted
```

***

### Key Takeaways

* **ReplicaSet ensures a fixed number of Pods are running.**
* Great for **self-healing** and **scaling**.
* Not designed for **rolling updates / rollbacks** → use **Deployments** for that.
* Always define **unique selectors** to avoid label collision.
