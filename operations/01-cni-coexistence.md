# CNI coexistence — Preserve Rules

> **Cisco source.** [K8s Deep Dive FAQ](https://secure.cisco.com/secure-workload/docs/secure-workload-and-k8s)
> and [Deploy Software Agents (4.0)](https://www.cisco.com/c/en/us/td/docs/security/workload_security/secure_workload/user-guide/4_0/cisco-secure-workload-user-guide-on-prem-v40/deploy-software-agents.html).

Secure Workload enforcement on Kubernetes/OpenShift programs iptables on both the
node and pod namespaces. Your **CNI** (Calico, Cilium, Weave, etc.) also programs
iptables. They have to coexist.

---

## The rule

> **Secure Workload filter rules take priority; the iptables rules created by
> CNIs are preserved.** This requires the **Preserve Rules** function in the
> agent config to be **enabled** for Kubernetes and OpenShift deployments.

```
   Node iptables
   ┌───────────────────────────────────────────────┐
   │  Secure Workload chains  ← evaluated first      │
   │  (allow/deny per policy intent)                 │
   ├───────────────────────────────────────────────┤
   │  CNI chains (Calico/Cilium/etc.)  ← PRESERVED   │
   │  (routing, NAT, network policy)                 │
   └───────────────────────────────────────────────┘
```

- **CSW rules take priority** for segmentation decisions.
- **CNI rules are kept intact** — routing/NAT/network policy keep working.
- You can run **CNI-based network policies alongside CSW policies**; just keep
  *Preserve Rules* on so CSW doesn't clobber the CNI chains.

> **Enable it.** *Preserve Rules* is a per-agent-config setting. Turn it on for
> the agent config/profile that applies to your K8s/OpenShift nodes **before**
> enabling enforcement.

---

## Calico (CSW 4.0)

Supported with **Calico 3.13** using **one** of these Felix configurations:

| Option | Felix config |
|---|---|
| A | `ChainInsertMode: Append`, `IptablesRefreshInterval: 0` |
| B | `ChainInsertMode: Insert`, `IptablesFilterAllowAction: Return`, `IptablesMangleAllowAction: Return`, `IptablesRefreshInterval: 0` |

```bash
# Inspect current Felix config
kubectl get felixconfiguration default -o yaml
```

> Different Calico version or Felix config → **validate with Cisco TAC** before
> relying on CSW enforcement on that cluster. `IptablesRefreshInterval: 0` matters
> — a non-zero refresh can have Felix periodically rewrite chains and fight the
> agent.

---

## CNI-agnostic, with caveats

Visibility and enforcement don't depend on specific CNI functions — Calico,
Cilium, Weave, Azure CNI, AWS VPC CNI, etc. are supported. **But** the *flow
records you see* depend on CNI **mode** (direct routing vs. overlay
VXLAN/Geneve/IPIP) — see [`docs/02-flow-visibility.md`](../docs/02-flow-visibility.md).

---

## See also

- [`docs/03-enforcement-translation.md`](../docs/03-enforcement-translation.md) — node vs pod rules
- [`install/04-openshift.md`](../install/04-openshift.md) — SDN/OVN/IPVS support
- [`operations/03-faq.md`](./03-faq.md) — CNI / runtime / OVN FAQ
