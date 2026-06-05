# Install prerequisites (Kubernetes / OpenShift)

> **Cisco source.** [Deploy Software Agents — Install Kubernetes or OpenShift Agents (4.0)](https://www.cisco.com/c/en/us/td/docs/security/workload_security/secure_workload/user-guide/4_0/cisco-secure-workload-user-guide-on-prem-v40/deploy-software-agents.html).

Check these before running the Agent Script Installer.

---

## Access & tooling

| Requirement | Detail |
|---|---|
| **kubectl/oc access** | Run the installer on a Linux host with access to the cluster's **kube-apiserver** and a `kubeconfig` with **admin** privileges as the **default context/cluster/user**. |
| **Admin credentials** | The script needs Kubernetes/OpenShift **administrator** credentials to start **privileged** agent pods on nodes. |
| **kubeconfig path** | Defaults to `~/.kube/config`; override with `--kubeconfig <path>`. |
| **Tenant (on-prem only)** | Choose the tenant during install. **Not required** for SaaS clusters. |

---

## Connectivity (this trips people up)

```
   Installer host ────────────► kube-apiserver           (to create namespace/RBAC/DaemonSet)
   Every cluster NODE ────────► CSW cluster CFG-SERVER:443 (to PULL the agent Docker image)
   Agent pods ────────────────► CSW cluster              (telemetry / control channel)
```

- The installer script **does not contain the agent software**. Each node
  **pulls the agent Docker image from the Secure Workload cluster**
  (`CFG-SERVER-IP:443`) at pod startup via the container runtime's image fetch.
- **HTTP proxy caveat:** the proxy you set on the installer page only controls how
  **agents reach the CSW cluster** — it does **not** affect how **nodes pull
  Docker images**. Node image pulls use the **container runtime's own proxy
  config**. Air-gapped/proxied clusters must configure the runtime proxy or
  mirror the image to an internal registry.

---

## Platform support

| Item | Support |
|---|---|
| **Node OS** | Linux **and** Windows nodes (see Compatibility Matrix for exact list) |
| **Kubernetes** | 1.27 and later (per 4.0 User Guide — confirm for your release) |
| **CNI** | CNI-agnostic: Calico, Cilium, Weave, Azure CNI, AWS VPC CNI, etc. |
| **Container runtime** | containerd, Docker, CRI-O |
| **OpenShift networking** | **SDN supported; OVN not yet.** **IPVS-based kube-proxy is not supported for OpenShift.** |
| **Service mesh** | Istio supported for visibility + enforcement |

> **Calico (CSW 4.0).** Supported with Calico **3.13** using one of these Felix
> configs: `ChainInsertMode: Append, IptablesRefreshInterval: 0` **or**
> `ChainInsertMode: Insert, IptablesFilterAllowAction: Return,
> IptablesMangleAllowAction: Return, IptablesRefreshInterval: 0`. Differing
> versions/configs → validate with Cisco TAC. See
> [`operations/01-cni-coexistence.md`](../operations/01-cni-coexistence.md).

---

## Cluster security policy (privileged pods)

The agent pod runs **privileged** with `hostNetwork: true`, `hostPID: true`, and
host-path mounts (`/proc`, `/sys`, `/var/log`). Make sure your cluster admission
policy allows that **in the `tetration` namespace**:

| Control | Action |
|---|---|
| **Pod Security Admission (PSA)** | Default `restricted` profile blocks privileged pods — allow privileged for the `tetration` namespace per your policy |
| **OpenShift SCC** | Bind the `privileged` SCC to the agent's ServiceAccount — see [`install/04-openshift.md`](./04-openshift.md) |
| **Control-plane nodes** | To run on control-plane nodes, pass a toleration with `--toleration` (typically the `NoSchedule` toleration) |

---

## Namespace

Secure Workload entities are created in the **`tetration`** namespace. Don't
rename it unless your generated installer output says otherwise.

---

## See also

- [`install/02-connector-orchestrator.md`](./02-connector-orchestrator.md) —
  cluster metadata (do this so flows are label-enriched)
- [`install/03-agent-script-installer.md`](./03-agent-script-installer.md) — the
  install itself
- [`install/04-openshift.md`](./04-openshift.md) — SCC and OpenShift specifics
