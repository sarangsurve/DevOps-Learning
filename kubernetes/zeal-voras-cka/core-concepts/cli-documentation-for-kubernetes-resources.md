# CLI Documentation for Kubernetes Resources

### 🔹 1. Kubernetes API Documentation

* Official API reference docs:\
  👉 [Kubernetes API Reference](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.33/#pod-v1-core)
* Provides detailed descriptions of **resource fields, types, and usage**.
* Example: For Pods, you’ll find `spec`, `metadata`, `status`, etc., each with nested fields.

***

### 🔹 2. CLI-Based Documentation (`kubectl explain`)

Instead of browsing docs, you can use `kubectl explain` to explore resources and their fields directly from the terminal.

#### Example: Top-Level Pod Documentation

```bash
$ kubectl explain pods
KIND:       Pod
VERSION:    v1

DESCRIPTION:
    Pod is a collection of containers that can run on a host. This resource is
    created by clients and scheduled onto hosts.

FIELDS:
  apiVersion    <string>
    APIVersion defines the versioned schema of this representation of an object.
    Servers should convert recognized schemas to the latest internal value, and
    may reject unrecognized values. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources

  kind  <string>
    Kind is a string value representing the REST resource this object
    represents. Servers may infer this from the endpoint the client submits
    requests to. Cannot be updated. In CamelCase. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds

  metadata      <ObjectMeta>
    Standard object's metadata. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata

  spec  <PodSpec>
    Specification of the desired behavior of the pod. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status

  status        <PodStatus>
    Most recently observed status of the pod. This data may not be up to date.
    Populated by the system. Read-only. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
```

👉 This gives:

* Resource **kind** & **version**.
* High-level description.
* Top-level fields (`apiVersion`, `kind`, `metadata`, `spec`, `status`).

***

### 🔹 3. Navigating Nested Fields

You can **drill down** into fields, similar to how you would browse the API reference docs.

#### Example: Inspect Pod Metadata

```bash
$ kubectl explain pods.metadata
KIND:       Pod
VERSION:    v1

FIELD: metadata <ObjectMeta>


DESCRIPTION:
    Standard object's metadata. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata
    ObjectMeta is metadata that all persisted resources must have, which
    includes all objects users must create.

FIELDS:
  annotations   <map[string]string>
    Annotations is an unstructured key value map stored with a resource that may
    be set by external tools to store and retrieve arbitrary metadata. They are
    not queryable and should be preserved when modifying objects. More info:
    https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations

  creationTimestamp     <string>
  ...
```

#### Example: Inspect Pod Spec → Containers

```bash
$ kubectl explain pods.spec.containers
KIND:       Pod
VERSION:    v1

FIELD: containers <[]Container>


DESCRIPTION:
    List of containers belonging to the pod. Containers cannot currently be
    added or removed. There must be at least one container in a Pod. Cannot be
    updated.
    A single application container that you want to run within a pod.

FIELDS:
  args  <[]string>
    Arguments to the entrypoint. The container image's CMD is used if this is
    not provided. Variable references $(VAR_NAME) are expanded using the
    container's environment. If a variable cannot be resolved, the reference in
    the input string will be unchanged. Double $$ are reduced to a single $,
    which allows for escaping the $(VAR_NAME) syntax: i.e. "$$(VAR_NAME)" will
    produce the string literal "$(VAR_NAME)". Escaped references will never be
    expanded, regardless of whether the variable exists or not. Cannot be
    updated. More info:
    https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/#running-a-command-in-a-shell

  command       <[]string>
  ...
```

👉 This is the **same information** you’d find in the API reference — but immediately available via CLI.

***

### 🔹 4. Practical Tips

* Use `kubectl explain <resource>` to get **overview of fields**.
* Use `kubectl explain <resource>.<field>` to **go deeper into nested objects**.
* Use `kubectl explain --recursive <resource>` to see **entire schema in one go** (but it can be very long).

#### Example: Full Pod Spec Schema

```bash
kubectl explain pod.spec --recursive
```

***

### 🔹 5. Why This Matters

* Great for **YAML authoring** (no need to memorize field names).
* Prevents **typos or wrong nesting** in manifest files.
* Useful for **offline learning** when docs aren’t handy.

***

## 🚀 Quick Summary

* **Docs site** → Full reference with explanations.
* **kubectl explain** → CLI equivalent to explore resources.
* Drill down into fields with `.` notation (`pods.spec.containers`).
* Use `--recursive` to see full nested schema.
* Extremely useful for writing manifest files correctly.
