# Kubernetes connector / orchestrator (cluster metadata)

> **Cisco source.** [External Orchestrators / Connectors (4.0)](https://www.cisco.com/c/en/us/td/docs/security/workload_security/secure_workload/user-guide/4_0/cisco-secure-workload-user-guide-on-prem-v40/connectors.html)
> and the [K8s Deep Dive](https://secure.cisco.com/secure-workload/docs/secure-workload-and-k8s) (component #2).

The **connector** (a.k.a. external orchestrator) is **component #2** of the
architecture. It runs on the **management plane** and talks to your cluster's
Kubernetes API to pull **pod and service metadata** — pod IDs, labels,
annotations, services. This is what enriches raw flows so you can segment on
labels instead of ephemeral pod IPs.

> **Connector ≠ DaemonSet.** The connector provides **metadata**; the DaemonSet
> ([`03-agent-script-installer.md`](./03-agent-script-installer.md)) provides
> **flows + enforcement**. You typically deploy **both**. Order doesn't strictly
> matter, but doing the connector first means the very first flows you see are
> already label-enriched.

---

## What it connects to

| Cluster type | Connector talks to |
|---|---|
| **EKS** (AWS) | EKS cluster API |
| **AKS** (Azure) | AKS cluster API |
| **GKE** (Google) | GKE cluster API |
| **OpenShift** | OpenShift API |
| **Unmanaged Kubernetes** | kube-apiserver directly |

---

## High-level setup

```
   CSW UI → Manage → Connectors (External Orchestrators)
        │
        ├── Choose Kubernetes / OpenShift (or EKS/AKS/GKE)
        ├── Provide API endpoint + credentials (service account token / kubeconfig)
        │      → least-privilege RBAC: read pods, services, namespaces, nodes, endpoints
        ├── (Optional) CA cert for TLS to the API
        └── Save → connector begins syncing inventory in near real-time
```

1. In the CSW UI go to **Manage → Connectors** (External Orchestrators) and add a
   **Kubernetes/OpenShift** orchestrator (or the managed variant: EKS/AKS/GKE).
2. Provide the **API server endpoint** and **credentials** — a Kubernetes
   **ServiceAccount token** (or kubeconfig) scoped with **read** access to pods,
   services, namespaces, nodes, and endpoints.
3. Provide the API server **CA certificate** if TLS verification is required.
4. Save. The connector starts a **continuous, near-real-time sync** of inventory:
   new pods, new labels, annotation changes all reflect quickly in CSW.

> **Least privilege.** Create a dedicated ServiceAccount with a read-only
> ClusterRole for the connector rather than reusing a cluster-admin token. Pull
> only the metadata you need.

---

## What you get once it's syncing

- Per-pod and per-service **profiles** with full label/annotation metadata.
- **Inventory filters** built on pod metadata (e.g. `app=payments`,
  `namespace=prod`) — the basis for dynamic, label-based policy.
- Flows in the cluster automatically **enriched** with this metadata.

*(See [`docs/02-flow-visibility.md`](../docs/02-flow-visibility.md) for how
enrichment feeds the flow map, and Figures 3–5 in the deep dive.)*

---

## See also

- [`install/03-agent-script-installer.md`](./03-agent-script-installer.md) —
  deploy the DaemonSet for flows + enforcement
- [`docs/01-architecture.md`](../docs/01-architecture.md) — where the connector
  sits in the architecture
- [`docs/05-vulnerability-scanning.md`](../docs/05-vulnerability-scanning.md) —
  scanning is enabled on a **connector-onboarded** cluster
