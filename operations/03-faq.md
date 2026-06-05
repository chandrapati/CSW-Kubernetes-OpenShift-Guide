# FAQ

> **Cisco source.** [Secure Workload and Kubernetes Security — Deep Dive, FAQs](https://secure.cisco.com/secure-workload/docs/secure-workload-and-k8s).
> Reproduced and lightly annotated.

---

**Q: What node operating systems are supported?**
Both **Linux and Windows** node OS are supported. For the exact list, see the
[Compatibility Matrix](../docs/00-official-references.md).

---

**Q: Which CNI is supported by the Secure Workload agent?**
Flow visibility and policy enforcement **do not depend on any specific CNI**. All
popular CNIs — **Calico, Weave, Cilium, Azure CNI, AWS VPC CNI**, etc. — are
supported. *(Note: the CNI **mode** affects how many flow records you see — see
[`docs/02-flow-visibility.md`](../docs/02-flow-visibility.md).)*

---

**Q: Can I enforce CNI-based policies along with CSW policies?**
**Yes.** Secure Workload filter rules take **priority**, and the iptables rules
created by CNIs are **preserved**. **Note:** the **Preserve Rules** function in
the agent config must be **enabled** for Kubernetes and OpenShift deployments.
See [`operations/01-cni-coexistence.md`](./01-cni-coexistence.md).

---

**Q: Does Secure Workload support OpenShift SDN or OVN?**
**SDN is supported. OVN is not supported yet.** Also note **IPVS-based kube-proxy
is not supported on OpenShift**. See [`install/04-openshift.md`](../install/04-openshift.md).

---

**Q: What container runtimes are supported?**
All leading container runtimes: **containerd, Docker, CRI-O**.

---

**Q: Does Secure Workload support provisioning policies from a CI/CD pipeline?**
**Yes.** Use the **Terraform or Ansible** providers to write policy playbooks and
push updates from CI/CD pipelines.

---

## Extra practitioner Q&A

**Q: Does CSW inject a sidecar into every pod?**
**No.** One privileged **DaemonSet** agent per node captures flows at the pod and
node interfaces and attributes them via cluster metadata. No per-pod sidecar.
See [`docs/01-architecture.md`](../docs/01-architecture.md).

**Q: Conversation mode vs detailed mode?**
**Conversation** = summarized observations reported every **15s**. **Detailed** =
every packet captured (node + pod namespaces), reported every **1s**. Detailed is
heavier; use it when you need packet-level fidelity.

**Q: Why do I see a UDP node→node flow next to my pod→pod flow?**
Overlay CNI encapsulation (VXLAN/Geneve = UDP; IPIP = TCP). It's the tunnel, not
a separate connection. Don't write policy against it.

**Q: My NodePort policy is right but traffic is blocked.**
If nodes and apps are in **different scopes**, add the allow rule on **both**
parent (node) and child (app) scopes. See [`docs/04-scope-design.md`](../docs/04-scope-design.md).

**Q: Is Windows container CVE scanning supported?**
The whitepaper notes **Linux container CVE scanning** at time of publishing —
confirm Windows support for your release. See [`docs/05-vulnerability-scanning.md`](../docs/05-vulnerability-scanning.md).

---

## See also

- [`docs/00-official-references.md`](../docs/00-official-references.md)
- [`operations/02-troubleshooting.md`](./02-troubleshooting.md)
