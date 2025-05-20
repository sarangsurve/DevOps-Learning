# Kubernetes Manifest files

A **Kubernetes manifest file** is a YAML (or sometimes JSON) configuration file that **defines the desired state** of Kubernetes resources — like Pods, Deployments, Services, ConfigMaps, etc.

***

#### What It _Means_:

It's a **blueprint** or **instruction sheet** for Kubernetes. You describe _what_ you want (e.g., "I want 3 replicas of this containerized app"), and Kubernetes works to make that happen and keep it that way.

***

#### How It _Looks_:

Here's a simple example of a **Deployment** manifest:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app-container
        image: my-app-image:1.0
        ports:
        - containerPort: 8080
```

***

#### Why It’s _Important_:

1. **Declarative Configuration**: You declare _what you want_, not how to do it.
2. **Reproducibility**: You can version-control manifest files and deploy the same setup across dev, test, and prod.
3. **Automation**: Tools like `kubectl`, Helm, ArgoCD, or GitOps workflows all rely on manifest files.
4. **Scalability & Maintainability**: It's easier to manage hundreds of services when they are defined cleanly in manifests.

***

#### Conclusion:

A **manifest file** is your main interface to Kubernetes — it’s how you describe and manage everything from apps to policies.\
Without it, you’d be manually clicking around or issuing imperative commands all the time — not scalable.
