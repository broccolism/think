## Book
https://www.oreilly.com/library/view/cloud-native-devops/9781492040750/

Cloud Native DevOps with Kubernetes
by John Arundel, Justin Domingus
OREILLY, March 2019

### Reading
- 2026.02.01 - (WIP)

# Questions
#### 5.1.3 Resource Limits

Q. `Kubernetes allows resource overcommit. Overcommit means that the sum of all resource limits of containers in a node can exceed the total resources of that node. It is essentially a gamble.` --> Then it seems like only the `resources.requests` value is used to decide which node to place a pod on, and `limits` is not used for scheduling. Is `limits` only used for forcefully terminating pods?

A. It is correct that only `requests` values are used for scheduling. However, the role of `limits` is not limited to forced termination. The behavior differs between CPU and memory.
- **When CPU limits are exceeded**: The pod is not terminated but **throttled**. The CFS (Completely Fair Scheduler) restricts CPU usage time, so the pod simply slows down.
- **When memory limits are exceeded**: The OOM Killer kicks in and the container is **forcefully terminated**.

Additionally, the combination of `requests` and `limits` settings determines the pod's **QoS (Quality of Service) class**.
- **Guaranteed** (all containers have requests = limits): Last to be evicted when the node runs low on resources
- **Burstable** (requests < limits, or only partially set): Medium priority
- **BestEffort** (neither requests nor limits are set): First to be evicted

When a node experiences resource pressure (node pressure), the kubelet decides which pods to evict first based on their QoS class.

In summary, `limits` is used for three purposes: (1) CPU throttling, (2) OOM Kill when memory is exceeded, and (3) eviction priority through QoS class determination.

# Newly Learned

- It is best to keep container images as small as possible. The reasons are: smaller containers build faster, take up less image storage space, pull faster, and have fewer security vulnerabilities. - 5.1.4 Keep Your Containers Small
- There are commands that can wipe out all resources under a namespace. In the current version of Kubernetes, there is **no way to protect resources like namespaces from being deleted** (however, this feature is being discussed on the Kubernetes GitHub issues page). - 5.3.2 Which Namespaces Should I Use?
  - Still nothing!

- Service resources provide permanent IP addresses and DNS addresses that are automatically routed to pods. Service DNS names always follow this format: `SERVICE.NAMESPACE.svc.cluster.local`
  - `.svc.cluster.local` is optional, and the namespace is also optional. If you want to communicate with a `demo` service in the `prod` namespace, you can use: `prod.demo` - 5.3.3 Service Addresses
- You can also choose the type of nodes in a cluster (e.g., GPU-enabled nodes) and scale them up/down.
- Because Kubernetes can be configured in so many ways, it has a set of tests to verify that a cluster meets the core requirements for a specific version and is conformant. There are official certifications, as well as tools for running your own conformance tests. - 6.2 Conformance Checking
- `kubectl get pods --watch` — you can use the watch option. This seems useful in corporate environments where using external open-source tools like kubespy is frowned upon! (I'm actually using Lens, but this looks more fun.) - 7.1.9 Watching Objects
- Imperative commands are useful for quick tests or validating ideas, but they have the problem of lacking a reliable single source of truth. With imperative commands, there is no way to know when or who ran them, or what the results were. - 7.2.2 When Not to Use Imperative Commands
  - TIP) In production clusters, you should not use imperative `kubectl` commands like `create` or `edit`. It is recommended to manage resources with version-controlled YAML manifests and apply them with `kubectl apply` (or Helm charts).
