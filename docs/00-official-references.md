# Official Cisco References

The canonical Cisco sources behind every page in this repo. When in doubt,
these win — this repo is a practitioner's distillation, not an official Cisco
publication.

---

## Primary sources

| Topic | Cisco document |
|---|---|
| **Kubernetes deep dive** (how it works — flows, enforcement, scopes, scanning) | [Secure Workload and Kubernetes Security — Deep Dive](https://secure.cisco.com/secure-workload/docs/secure-workload-and-k8s) |
| **Install K8s/OpenShift agents** (Agent Script Installer, DaemonSet, uninstall) | [Deploy Software Agents — Install Kubernetes or OpenShift Agents (4.0 On-Prem)](https://www.cisco.com/c/en/us/td/docs/security/workload_security/secure_workload/user-guide/4_0/cisco-secure-workload-user-guide-on-prem-v40/deploy-software-agents.html) |
| **Kubernetes connector / orchestrator** (metadata, RBAC) | [External Orchestrators / Connectors (4.0 On-Prem User Guide)](https://www.cisco.com/c/en/us/td/docs/security/workload_security/secure_workload/user-guide/4_0/cisco-secure-workload-user-guide-on-prem-v40/connectors.html) |
| **Policy lifecycle** (rank, inheritance, consumer/provider) | [Manage Policy Lifecycle in Secure Workload (4.0)](https://www.cisco.com/c/en/us/td/docs/security/workload_security/secure_workload/user-guide/4_0/cisco-secure-workload-user-guide-on-prem-v40/manage-policy-lifecycle-in-secure-workload.html) |
| **Inventory / labels / filters** | [Manage Inventory (4.0)](https://www.cisco.com/c/en/us/td/docs/security/workload_security/secure_workload/user-guide/4_0/cisco-secure-workload-user-guide-on-prem-v40/manage-inventory-for-secure-workload.html) |
| **Supported platforms** (node OS, K8s/OpenShift versions) | [Secure Workload Compatibility Matrix](https://www.cisco.com/c/en/us/td/docs/security/workload_security/secure_workload/user-guide/4_0/cisco-secure-workload-user-guide-on-prem-v40/deploy-software-agents.html) |

> **SaaS users:** the same chapters exist under the SaaS User Guide tree; the
> behavior is identical except where noted (e.g. SaaS does not require tenant
> selection during install).

---

## Key facts these sources establish (cited throughout)

- **No sidecar.** Telemetry is captured at the **node** by the DaemonSet pod and
  attributed to pods/services/namespaces via cluster metadata — CSW does not
  inject a sidecar into every workload pod. *(Deep Dive; 4.0 User Guide.)*
- **Namespace.** Secure Workload entities are created in the **`tetration`**
  namespace. *(4.0 User Guide — Install K8s/OpenShift Agents.)*
- **Image pull.** The installer script does **not** contain the agent software;
  every node pulls the agent Docker image from the **Secure Workload cluster**
  (`CFG-SERVER-IP:443`) at pod startup. *(4.0 User Guide.)*
- **Supported path.** The Cisco-documented install method is the **Agent Script
  Installer** (*Manage → Agents → Installer → Agent Script Installer →
  Platform = Kubernetes*). *(4.0 User Guide.)*
- **CNI-agnostic.** Visibility and enforcement do not depend on a specific CNI;
  Calico, Cilium, Weave, Azure CNI, AWS VPC CNI, etc. are supported, and CNI
  rules are **preserved** (requires *Preserve Rules* enabled). *(Deep Dive FAQ.)*
- **OpenShift SDN yes, OVN not yet; IPVS kube-proxy not supported on OpenShift.**
  *(Deep Dive FAQ; 4.0 User Guide.)*

---

## Versioning note

Cisco's deep-dive whitepaper was published under **Release 3.9 & older** and the
install steps here track the **4.0 On-Prem User Guide**. Behavior is stable
across these releases, but **always confirm against the release you run** —
especially supported K8s/OpenShift versions, CNI/Felix configurations, and
whether enforcement (vs. visibility-only) is supported for your node OS.
