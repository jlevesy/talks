+++
weight = 40
+++
{{% section %}}

## Exposing workloads on the network

---

### Kubernetes Networking Model (1/3)

- Port Binding
  - Each Pod/Container binds a port on the host
    - ex: `:8080 goes to container app-instance-1`

  - Requires careful management and sometimes creates nasty problems.

---

### Kubernetes Networking Model (2/3)

- k8s uses another approach:
  - One Pod gets One IP
  - An pod can reach another pod without NAT, whether it is on the same node or a different one
  - Delegated to CNI (Container Network Interface) implementations.
    - k3s uses flannel, EKS runs a proprietary solution based on VPC.

---

### Kubernetes Networking Model (3/3)

- TLDR: I can hit a pod from another pod knowing it's IP.

---

### Hello Services (1/2)

- By itself model is not satisfying.
  - If my container goes down, its IP might change, how do I discover it again?
  - How do we deal with multiple replica and loadbalancing?

---

### Hello Services (2/2)

- k8s provides another resource, called `Services`, to handle this.
- A service allow to address a set of pods, matching a selector.
  - It gives a stable IP to this set of pods
  - Optionally, a DNS name if the clusters supports it.
  - Each connection to the VirtualIP gets balanced to one of the matched pods.
- It allows to decouple the client from the server itself
  - if the set of pod changes, the client isn't impacted

---

### Service Example

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: super-app
spec:
  selector:
    app: super-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

---

### What Happened Here? (1/2)

- We created a service that selects all the pods from our previous deployment based on their label
- The service gets automatically a VirtualIP
- It gets also a stable DNS name (because DNS is enabled)
  - `<svc>.<namespace>.svc.cluster.local`

---

### What Happened Here? (2/2)

- The endpoint controller picks up the service and populates an endpoint object for this service with the IPs of all the matched pods.
- Kube-proxy configures all the nodes so they know how to route to the "Virtual IP"
- My other pod can talk to the "service" through its virtual IP and Domain Name

---

### How Services Are Implemented?

- An `IP` that routes to 3 possible hosts? Whichcraft!
- There are 3 possible implementations:
  - User Space Proxy
  - IPTables
  - IPVS
- `kube-proxy` handles that on every workers


{{% note %}}
- User-Space proxy mode: Route all outgoing connections (via an iptable rule) to a local lvl4 (TCP/UDP) proxy process handles the connection then forward it to the right replica. More flexible, but the slowest and least reliable.
- iptables (default mode on common distros): kube-proxy alters the node iptable rules to handle the service IP and rewrite to one of the actual server pod IPs, based on the endpoints. No healthchecks, no advanced loadbalancing strategies.
- IPVS (default mode on k3s (not sure)?): another kernel implementation of transport level load balancing, provides retries, healthchecks, better scalability and diverse loadbalancing algorithms.
{{% /note  %}}

---

### Service Discovery?

- How to discover services from one pod?
  - Environment variables are automatically propagated on container startup.
  - DNS to provide deterministic domain name you can rely on.

---

### Exposing Services: Service Types

Cool story for internal services, but how can I expose my apps outside?

- Bind your pods to host ports: that's possible, but so 2014 :p
- Somehow leverage services to do this?
  - Hello services types!

---

#### ClusterIP

- Default mode, it creates a virtual IP reachable internally
- Doesn't expose the service externally.

---

#### NodePort

- Superset of ClusterIP, it gets a virtual IP for internal traffic,
- Also opens a port on the host and routes all external traffic coming to that port to the service.
- LoadBalancing to nodes is still left to handle by the user.

{{% note %}}
That's something we could use for Fleet to K8S communication, instead of a full blown LoadBalancer)
{{% /note %}}

---

#### NodePort example

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: super-app-nodeport
spec:
  type: NodePort
  selector:
    app: super-app-nodeport
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30001
```

---

#### LoadBalancers

- Superset of NodePort service, VIP + External Port Binding.
- Also starts and configures automatically an external load balancer to the cloud environment.
- You get one or multiple ExternalIP (IP or domain name) that are reachable externally (and you don't have to care about it).

---

- Implementation depends of your cluster environment
- On EKS each time you create a LoadBalancer service, AWS creates (and charges you for!) a CLB.
- k3s on Docker uses host containers IP as public IP.

{{% /section %}}
