# OpenShift specifics (SCC, OVN/IPVS, `oc`)

> **Cisco source.** [Deploy Software Agents (4.0)](https://www.cisco.com/c/en/us/td/docs/security/workload_security/secure_workload/user-guide/4_0/cisco-secure-workload-user-guide-on-prem-v40/deploy-software-agents.html)
> and the [K8s Deep Dive FAQ](https://secure.cisco.com/secure-workload/docs/secure-workload-and-k8s).

OpenShift uses the **same Agent Script Installer** as Kubernetes
([`03-agent-script-installer.md`](./03-agent-script-installer.md)), with a few
OpenShift-specific items.

---

## Networking support — read first

| Item | Support |
|---|---|
| **OpenShift SDN** | ✅ Supported |
| **OVN-Kubernetes** | ❌ **Not supported yet** |
| **IPVS-based kube-proxy** | ❌ **Not supported on OpenShift** |

> If your cluster runs **OVN** (the OpenShift 4.x default for newer installs),
> CSW enforcement is **not** supported as of the referenced releases. Confirm
> your SDN plugin (`oc get network.operator cluster -o yaml`) and validate with
> Cisco TAC before planning enforcement.

---

## Security Context Constraints (SCC)

The agent pod is **privileged** (`hostNetwork`, `hostPID`, host-path mounts).
OpenShift enforces **SCC** on top of Pod Security Admission, so the agent's
ServiceAccount needs the **`privileged`** SCC.

```bash
# Bind the privileged SCC to the agent ServiceAccount in the tetration namespace
oc adm policy add-scc-to-user privileged -z <agent-serviceaccount> -n tetration

# Verify
oc get scc privileged -o yaml | grep -A5 users
```

> Use the **exact ServiceAccount name** the installer creates (inspect with
> `oc get sa -n tetration`). Don't guess it.

---

## `oc` equivalents

| Task | Command |
|---|---|
| List agent pods | `oc get pod -n tetration` |
| Watch rollout | `oc rollout status ds/<daemonset> -n tetration` |
| Pod logs | `oc logs <agent-pod> -n tetration` |
| Restart an agent on a node | delete the pod (`oc delete pod <agent-pod> -n tetration`) — DaemonSet recreates it |
| Node SDN plugin | `oc get network.operator cluster -o yaml` |

---

## Control-plane / master nodes

To run the agent on control-plane (master) nodes, pass the appropriate
**toleration** via the installer's `--toleration` flag (typically the
`NoSchedule` toleration). See
[`03-agent-script-installer.md`](./03-agent-script-installer.md).

---

## Preserve Rules (required)

For OpenShift (and Kubernetes), enable **Preserve Rules** in the agent config so
CSW's filter rules coexist with the CNI's iptables rules. See
[`operations/01-cni-coexistence.md`](../operations/01-cni-coexistence.md).

---

## See also

- [`install/03-agent-script-installer.md`](./03-agent-script-installer.md) — the install
- [`install/05-verification.md`](./05-verification.md) — verify
- [`operations/01-cni-coexistence.md`](../operations/01-cni-coexistence.md) — Preserve Rules
- [`operations/03-faq.md`](../operations/03-faq.md) — OVN/SDN, runtimes, CNI FAQ
