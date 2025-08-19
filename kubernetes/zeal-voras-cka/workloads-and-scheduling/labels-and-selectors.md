# Labels and Selectors

### Overview of Labels <a href="#overview-of-labels" id="overview-of-labels"></a>

* Labels are **key/value pairs** attached to Kubernetes objects (e.g., Pods, Services, Deployments).
* Purpose: Organize, categorize, and select subsets of objects.
* Example labels:
  * `type: Server`, `name: kplabs-gateway`, `env: production`
  * `type: Database`, `name: kp-db`, `env: dev`

***

### Overview of Selectors <a href="#overview-of-selectors" id="overview-of-selectors"></a>

* **Selectors** filter objects based on labels.
* Example:
  * `env=production` → selects all prod resources
  * `env=dev` → selects all dev resources

***

### Kubernetes Objects That Can Use Labels <a href="#kubernetes-objects-that-can-use-labels" id="kubernetes-objects-that-can-use-labels"></a>

* Pods
* Services
* Secrets
* Namespaces
* Deployments
* DaemonSets

***

### Example: Assigning Labels <a href="#example-assigning-labels" id="example-assigning-labels"></a>

6 Pods running with labels:

* POD 1 → `env=dev`
* POD 2 → `env=prod`
* POD 3 → `env=dev`
* POD 4 → `env=prod`
* POD 5 → `env=dev`
* POD 6 → `env=prod`

Selectors:

* `kubectl get pods -l env=dev` → lists PODs 1, 3, 5
* `kubectl get pods -l env=prod` → lists PODs 2, 4, 6

***

### Commands: Working with Labels <a href="#commands-working-with-labels" id="commands-working-with-labels"></a>

#### 1. Create Pods <a href="#id-1-create-pods" id="id-1-create-pods"></a>

```bash
$ kubectl run pod-1 --image=nginx
pod/pod-1 created
$ kubectl run pod-2 --image=nginx
pod/pod-2 created
$ kubectl run pod-3 --image=nginx
pod/pod-3 created
$ kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
pod-1   1/1     Running   0          20s
pod-2   1/1     Running   0          16s
pod-3   1/1     Running   0          11s
```

#### 2. Show Labels

Kubectl CLI Syntax: `kubectl get pods --show-labels`&#x20;

```bash
$ kubectl get pods --show-labels
NAME    READY   STATUS    RESTARTS   AGE    LABELS
pod-1   1/1     Running   0          80s    run=pod-1
pod-2   1/1     Running   0          76s    run=pod-2
pod-3   1/1     Running   0          71s    run=pod-3
```

#### 3. Add Labels

Kubectl CLI Syntax: `kubectl label pod <pod-name> <label-name>=<label-value>`&#x20;

```bash
$ kubectl label pod pod-1 env=dev
pod/pod-1 labeled
$ kubectl label pod pod-2 env=stage
pod/pod-2 labeled
$ kubectl label pod pod-3 env=prod
pod/pod-3 labeled
$ kubectl get pods --show-labels
NAME    READY   STATUS    RESTARTS   AGE     LABELS
pod-1   1/1     Running   0          3m22s   env=dev,run=pod-1
pod-2   1/1     Running   0          3m18s   env=stage,run=pod-2
pod-3   1/1     Running   0          3m13s   env=prod,run=pod-3
```

#### 4. Remove Labels

Kubectl CLI Syntax: `kubectl label pods <pod-name> <label-name>-`&#x20;

```bash
$ kubectl label pods pod-1 env-
pod/pod-1 unlabeled
$ kubectl get pods --show-labels
NAME    READY   STATUS    RESTARTS   AGE   LABELS
pod-1   1/1     Running   0          5m50s   run=pod-1
pod-2   1/1     Running   0          5m46s   env=stage,run=pod-2
pod-3   1/1     Running   0          5m41s   env=prod,run=pod-3
```

#### 5. Overwrite Labels

Kubectl CLI Syntax: `kubectl label --overwrite pods <pod-name> <label-name>=<label-value>`&#x20;

```bash
$ kubectl label --overwrite pods pod-2 env=test
pod/pod-2 labeled
$ kubectl get pods --show-labels
NAME    READY   STATUS    RESTARTS   AGE   LABELS
pod-1   1/1     Running   0          6m24s   run=pod-1
pod-2   1/1     Running   0          6m20s   env=test,run=pod-2
pod-3   1/1     Running   0          6m15s   env=prod,run=pod-3
```



Just adding label of dev environment to pod-1 again.

```bash
$ kubectl label pod pod-1 env=dev
pod/pod-1 labeled
$ kubectl get pods --show-labels
NAME    READY   STATUS    RESTARTS   AGE   LABELS
pod-1   1/1     Running   0          6m47s   env=dev,run=pod-1
pod-2   1/1     Running   0          6m43s   env=test,run=pod-2
pod-3   1/1     Running   0          6m38s   env=prod,run=pod-3
```

***

Selectors in Action

```bash
$ kubectl get pods -l env=dev      # list dev pods
NAME    READY   STATUS    RESTARTS   AGE
pod-1   1/1     Running   0          10m

$ kubectl get pods -l env=prod     # list prod pods
NAME    READY   STATUS    RESTARTS   AGE
pod-3   1/1     Running   0          10m

$ kubectl get pods -l env!=dev     # list all except dev
NAME    READY   STATUS    RESTARTS   AGE
pod-2   1/1     Running   0          11m
pod-3   1/1     Running   0          11m
```

***

### Labels in Manifest File

Lets label the environment of the pod in manifest file as dev.

```bash
$ kubectl run nginx --image=nginx --dry-run=client -o yaml > label-pod.yaml
```

After updating manifest file(`label-pod.yaml`) by adding label of `env: dev` as follows:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    run: nginx
    env: dev
spec:
  containers:
  - name: nginx
    image: nginx
```

Apply:

```bash
$ kubectl apply -f label-pod.yaml
pod/nginx created

$ kubectl get pods --show-labels
NAME    READY   STATUS    RESTARTS   AGE   LABELS
nginx   1/1     Running   0          29s   env=dev,run=nginx
pod-1   1/1     Running   0          14m   env=dev,run=pod-1
pod-2   1/1     Running   0          14m   env=test,run=pod-2
pod-3   1/1     Running   0          14m   env=prod,run=pod-3
```

***

### Cleanup

```bash
$ kubectl delete pods --all
pod "nginx" deleted
pod "pod-1" deleted
pod "pod-2" deleted
pod "pod-3" deleted
```

***

✅ **Key Takeaways**

* Labels = metadata for identification.
* Selectors = queries to filter objects.
* Widely used in Services, Deployments, NetworkPolicies, etc.
