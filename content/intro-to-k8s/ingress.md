+++
weight = 50
+++

{{% section %}}

### Routing Ingress Traffic?

---

### The Problem

How to expose externally multiple applications on my cluster ?

- For instance:
  - workflow.myapp.co
  - ecommerce.myapp.co

---

### Solutions?

- Create one NodePort service per app, and handle external routing manually.
  - Doesn't really leverages k8s capabilities.
  - Hard to keep up to date?
- Create one LoadBalancer service per app.
  - Not super efficient ...and super expensive!
- Maybe we could add an aditional level of routing when a request arrives to the cluster?

---

### Hello Ingress Controllers

- Controllers in charge of routing ingress traffic to backend pods.
- Run as workload on the k8s cluster (often deployments).
- Exposed externally by a `NodePort` or `LoadBalancer` service.
- Allows to multiplex multiple services behind the same LoadBalancer.

---

- Brings cools features on top
  - HTTP routing (Host, Path, Headers...)
  - HTTP Middlewares (Caching, Rate Limiting, Identifier Injection)
  - TLS termination.

- Under the hood,
  - Good old lvl4 or lvl7 reverse proxies that are dynamically configured by a set of kubernetes objects.

- Some known actors: Nginx ingress (we're using this), HAProxy, Traefik (default on k3s) ....

---

### Configuring Ingress Controllers

- Ingress Controllers can consume various kind of resources
  - Most common one is `Ingress`, became stable in 1.18
  - [ServiceAPI](https://github.com/kubernetes-sigs/gateway-api) for more complex routing
  - Some providers also defines their own CRDs.

---

### Ingress Example

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: super-app
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /superapp
        pathType: Prefix
        backend:
          service:
            name: super-app
            port:
              number: 80
```

---

### What Happened Here?

- We created an Ingress resource that describes a rule to match all requests with the prefix `/superapp`
  and forward them to any endpoint of the service super-app on port 80.
- The ingress controller, Traefik in our case, picked this rule and configured itself to forward an request with path `/superapp` to one of the backends pods.

---

### Bounce Bounce Bounce

Let's sum up the life on an HTTP Request on our EKS cluster.

- Request hits AWS CLB, connection gets forwarded to one cluster node
- Request hits the node and the Nginx loadbalancer service port, gets bounced to one ingress controller instance (maybe not on the same node).
- Request hits the ingress controller, gets bounced to one on the backends (maybe not on the same node, again!).

---

TLDR; at worst, you get two network bounces to reach one instance of your application.

{{% /section %}}
