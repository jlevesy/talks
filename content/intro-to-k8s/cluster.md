+++
weight = 20
+++

{{% section %}}

## Hello Kubernetes

---

### Cluster And Nodes

- In k8s, a host is actually called a **Node**.
- A group of node is called a **Cluster**

---

### Two Types of Nodes

- Data Plane: execute the workload
- Control Plane: execute the management logic

<small>(Sometimes one node can be both)</small>

---

### Data Plane (Worker Nodes)

- Execute the workload
- They run:
  - A kubelet agent
  - A container runtime
  - A kube-proxy agent

{{% note %}}
Kubelet agent: listen to pod specs and drives the container runtime so it guarantees that thoses specs are executed. In other word it manages the workload execution.

Container runtime: implementation of "containers", could be Containerd, CRI-O, Docker (now deprecated)...

Kube-proxy agent: reflects the network topology on the node. More on this in a little while.

{{% /note %}}

---

### Control Plane

Makes sure that the cluster state is the same than the desired state

- They run:
  - A storage backend (`etcd` in many cases)
  - `kube-apiserver`
  - `kube-scheduler`
  - `kube-controller-manager`
  - `cloud-controller-manager`


{{% note %}}
`etcd` as main storage (or some storage layers in other distributions)
`kube-apiserver` serves the kubernetes API.
`kube-scheduler` dispatches newly created pods to nodes.
`kube-controller-manager` runs k8s main control loops.
`cloud-controller-manager` control loops that interacts with the cloud provider environment.
{{% /note %}}

---

### Control Loops?

- k8s is a declarative system (!= imperative)
  - User declares a desired state and the system takes actions to converge to this new state.
- Desired state is described through objects.
  - Pod, Deployment, Service and so on...

---

- Control plane executes a set of loops to control the subsystems of the cluster.
  - PodScheduler, DeploymentController ...
- They watch a subset of resources and take necessary actions to make the system converge to the desired state.

- A good analogy is a thermostat in a room.

---

- Highly extensible design:
  - You can add as many controllers as you want
  - You can define new resource types

---

### Example

Let's create a k8s cluster based on [k3s](https://rancher.com/docs/k3s/latest/en/)

```bash
k3d cluster create --agents=2 -p '8080:30001@agent[0]'

# [...]

kubectl config use-context k3d-k3s-default
k get nodes
```

{{% note %}}
- k3s is a lightweight distribution of k8s, packed in a single library.
- k3d is an helper tool that allows to create and manipulate k3s in docker clusters in a breeze
- On OSX, docker4Mac provides a built-in kubernetes support.
{{% /note %}}

{{% /section %}}
