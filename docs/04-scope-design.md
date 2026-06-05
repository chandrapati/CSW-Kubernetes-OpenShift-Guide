# Scope design for Kubernetes

> **Cisco source.** [Secure Workload and Kubernetes Security — Deep Dive](https://secure.cisco.com/secure-workload/docs/secure-workload-and-k8s),
> Use Case #2, Step 1 (Figures 21–22).

A **scope** groups a set of workloads so you can manage their policy lifecycle
independently — and delegate RBAC to the owning team without affecting other
apps. For Kubernetes there are two broad approaches.

---

## Single-scope design

Use when you want to manage policy for **an entire cluster** (or a group of
similar clusters) **together**.

```
   Scope: k8s-cluster-prod
   ┌───────────────────────────────────────────────┐
   │  all nodes + all pods + all services           │
   │  one policy workspace, one owner team          │
   └───────────────────────────────────────────────┘
```

*(Recreation of Figure 21.)*

- Simplest model; one workspace, one RBAC boundary.
- Best when one team owns the whole cluster, or clusters are single-tenant
  (one app/BU per cluster).

---

## Split-scope design

Use when **multiple applications share a cluster** and each needs **independent**
policy management.

```
   Parent scope: k8s-cluster-prod   ← cluster inventory (nodes, cluster-wide svcs)
   ├── Child scope: ns=payments     ← pods/services in the payments namespace
   ├── Child scope: ns=orders       ← pods/services in the orders namespace
   └── Child scope: ns=checkout     ← pods/services in the checkout namespace
```

*(Recreation of Figure 22.)*

- Map the **cluster inventory to a parent scope**.
- Map each application's pods/services to a **child scope** — commonly **one
  child scope per Kubernetes namespace** (if each app lives in its own namespace).
- Each child scope gets its own workspace, owner, and policy lifecycle.

---

## Choosing

| Question | Single scope | Split scope |
|---|---|---|
| Who owns the cluster? | One team | Multiple app teams |
| Tenancy | Single app / BU per cluster | Multi-app / multi-tenant cluster |
| RBAC delegation needed? | No | Yes — per namespace/app |
| Policy blast radius | Whole cluster | Per app/namespace |
| Operational overhead | Lower | Higher (more workspaces) |

---

## The split-scope enforcement gotcha (read this)

When nodes and app pods/services land in **different scopes** (parent = nodes,
child = app), some flows are enforced **across the scope boundary** — most
notably **NodePort/LoadBalancer ingress** and **node-sourced health checks**,
which involve **both** a node rule and a pod rule.

> **Rule:** for those patterns you must add the allow rule on **both** the parent
> (node) scope and the child (app) scope. Miss one half and the flow is blocked
> even though "the policy looks right." See Pattern 3 and Pattern 4 in
> [`docs/03-enforcement-translation.md`](./03-enforcement-translation.md).

```
   External ──► NodePort ──► pod
                 │            │
            node scope     app/child scope
            (allow on      (allow on
             parent)        child)        ← BOTH required
```

---

## See also

- [`docs/03-enforcement-translation.md`](./03-enforcement-translation.md) — why
  some flows need rules in two scopes
- [`docs/01-architecture.md`](./01-architecture.md) — the connector that supplies
  the labels you scope on
- Cisco: [Manage Inventory / Scopes](https://www.cisco.com/c/en/us/td/docs/security/workload_security/secure_workload/user-guide/4_0/cisco-secure-workload-user-guide-on-prem-v40/manage-inventory-for-secure-workload.html)
