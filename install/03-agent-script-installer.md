# Install the DaemonSet — Agent Script Installer (Cisco's documented method)

> **Cisco source.** [Deploy Software Agents — Install Kubernetes or OpenShift Agent using the Agent Script Installer Method (4.0)](https://www.cisco.com/c/en/us/td/docs/security/workload_security/secure_workload/user-guide/4_0/cisco-secure-workload-user-guide-on-prem-v40/deploy-software-agents.html).

This is **the Cisco-supported way** to deploy the Secure Workload agent
DaemonSet to a Kubernetes/OpenShift cluster. The script creates the namespace,
RBAC, configuration, and the DaemonSet; each node then **pulls the agent image
from the CSW cluster** and starts one privileged agent pod.

> The installer **automatically installs agents on nodes added later** — the
> DaemonSet covers new nodes as they join.

---

## Procedure

```
   CSW UI → Manage → Agents → Installer tab → Agent Script Installer
        │
        ├── Select Platform = Kubernetes   (click "Show Supported Platforms" to verify)
        ├── Choose tenant                  (on-prem only; skip on SaaS)
        ├── HTTP proxy?                    (only affects agent→CSW, NOT node image pulls)
        ├── Download the installer script
        ▼
   On a Linux host with kubectl admin access to the cluster:
        ./install_k8s_agent.sh   [--kubeconfig ~/.kube/config]  [--toleration <toleration>]
        ▼
   Script creates: tetration namespace · RBAC · ConfigMap/Secret · DaemonSet
        ▼
   Each node pulls the agent image from CFG-SERVER:443 and starts the agent pod
```

| Step | Action |
|---|---|
| 1 | In the CSW UI: **Manage → Agents → Installer** tab. *(First-time users: Quick Start → Install Agents.)* |
| 2 | Click **Agent Script Installer**. |
| 3 | **Select Platform → Kubernetes.** Click **Show Supported Platforms** to confirm your distro/version. |
| 4 | Choose the **tenant** to install into. *(Not required on SaaS.)* |
| 5 | If a proxy is needed for agents to reach CSW, choose **Yes** and enter a valid proxy URL. |
| 6 | **Download** the script to local disk. |
| 7 | Run it on a **Linux host with kubectl admin access** as the default context. Use `--kubeconfig` to point at a non-default config. |

The script prints **verification instructions** for the DaemonSet and pods on
completion — see [`install/05-verification.md`](./05-verification.md).

---

## Useful flags

| Flag | Purpose |
|---|---|
| `--kubeconfig <path>` | Use a kubeconfig other than `~/.kube/config` |
| `--toleration <toleration>` | Schedule agent pods on **control-plane** nodes (typically the `NoSchedule` toleration that otherwise keeps pods off control-plane nodes) |

---

## What the script creates

| Object | Notes |
|---|---|
| **Namespace** | `tetration` |
| **RBAC** | ServiceAccount + ClusterRole/Binding for the agent |
| **Config** | ConfigMap / Secret with cluster + activation config |
| **DaemonSet** | One privileged agent pod per node (covers new nodes automatically) |

> **Image pull, not bundled.** The script does **not** ship the agent binary.
> Nodes fetch the agent **Docker image from the CSW cluster** (`CFG-SERVER:443`)
> via the container runtime. Make sure nodes have that connectivity (or mirror
> the image — see [`install/01-prerequisites.md`](./01-prerequisites.md)).

---

## GitOps / Helm note

Cisco's **supported** path is this script. Helm-chart and raw-manifest patterns
exist as **community** approaches (see the
[CSW-Agent-Installation-Guide `kubernetes/` folder](https://github.com/chandrapati/CSW-Agent-Installation-Guide/tree/main/kubernetes)).
If you GitOps this, the safe pattern is: run the script **once**, capture what it
creates (`kubectl get all -n tetration -o yaml`), and make **that** your source
of truth — don't hand-roll Secret keys or DaemonSet specs.

---

## See also

- [`install/04-openshift.md`](./04-openshift.md) — SCC, `oc` equivalents, OVN/IPVS caveats
- [`install/05-verification.md`](./05-verification.md) — confirm the install worked
- [`install/06-uninstall.md`](./06-uninstall.md) — remove the agent
- [`docs/01-architecture.md`](../docs/01-architecture.md) — what the DaemonSet does
