+++
weight = 30
+++

{{% section %}}

## Describing The Workload

---

### The Atom: The Pod

- It's a "virtual host"
- Has one or more container
- Has one or more volumes linked to it
- Has one IP address
- Provides a sh\*tload of options to control it's lifecycle

{{% note %}}
  => (probes, topology, resourceLimits, disruption budgets...)
{{% /note %}}

---

### Pod Lifecycle (1/2)

- Pod: Pending => Running => Succeeded OR Failed
- Container: Waiting => Running Or Terminated

---

### Pod Lifecycle (2/2)

- Succeeded OR Failed are final states.
  - If a pod fails another one needs to be created!

---

### Let's create a Pod

Pod Manifest

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: toolbox-runtime
spec:
  containers:
    - name: runtime
      image: giantswarm/tiny-tools
      command:
        - sleep
        - infinity
```

Applying the Manifest

```bash
kubectl apply -f resources/pod.yaml
```

---

### What happened here ?

- We created a new pod resource in the API server.
- The kube-scheduler picks this newly created pod.
  - Checks if there isn't already a pod running that satisfied this spec
  - Assign a node to this pod
  - Issues a pod spec to the selected node node kubelet.
  - Kubelets instructs the container runtime to create the pod, and we get a running pod.

---

### Workload Management 1/2

- When a node goes down, all pods scheduled to run on it are considered as failed.
- That's a bummer, we need to recreate our pods by hand!?

---

### Workload Management 2/2

- Fortunately no, k8s provides a set of built-in controllers to manage pods automatically.
- We have 4(6-ish) different types of resources to describe our workload.

---

### `Deployments` and `ReplicaSets`

- Designed to manage stateless pods
- `Deployments` manages one or more `ReplicaSets`
- `ReplicaSets` maintains a set pods at a given spec.

- This design allows to achieve goals like zero downtime updates, by maintainting multiple `ReplicaSets` in parallel.

{{% note %}}
If a pod fails, the replicate recreates a new one.
{{% /note %}}

---

### `Deployment Example`

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: super-app
  labels:
    app: super-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: super-app
  template:
    metadata:
      labels:
        app: super-app
    spec:
      containers:
      - name: app
        image: traefik/whoami:v1.6.0
        ports:
        - containerPort: 80
```

---

### `StatefulSet`

- Designed to manages Stateful applications (databases?)
- From docs
  - Stable, unique network identifiers.
  - Stable, persistent storage.
  - Ordered, graceful deployment and scaling.
  - Ordered, automated rolling updates.

---

### `DaemonSet`

- Ensures that all (or some) nodes run a copy of a pod.

---

### `Job` And `CronJobs`

- `Job` manage one shot executions of a pod and make sure they succeed
- `CronJob` creates `Job` instances in a regular maner.

{{% /section %}}
