+++
weight = 10
+++

{{% section %}}

## How did we end up here?

---

### Problems we're trying to solve

- High Availability
- Scalability

---

### How are we (trying to) solving those?

- Distributed systems
- Make a set of host act like a large one

---

### In the context of application hosting

- Containers reduced the friction of running apps
- They make the workload more flexible,
  - it's "easy" to move on application to another hosts
- So why not delegate workload management to a computer?

---

### Hello Orchestrators

They manage automatically the execution of a given workload on set of hosts

---

### Existing Implementations

- Docker Swarm
- Nomad
- ~~Fleet~~
- Kubernetes (k8s)

---

### Disclaimer

- Kubernetes is a **very** complex beast.
- I will <small>(voluntarily or by plain ignorance)</small> omit lot of aspects of it.
- I'm not at ease giving talks, please be kind :-).

{{% /section %}}
