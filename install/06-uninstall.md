# Uninstall the Kubernetes / OpenShift agent

> **Cisco source.** [Deploy Software Agents — Uninstall an Enforcement Kubernetes or OpenShift Agent (4.0)](https://www.cisco.com/c/en/us/td/docs/security/workload_security/secure_workload/user-guide/4_0/cisco-secure-workload-user-guide-on-prem-v40/deploy-software-agents.html).

Use the **installer script's uninstall path** rather than hand-deleting objects
— it removes the DaemonSet, RBAC, config, and namespace cleanly, and (for
enforcement agents) reverts the iptables rules the agent programmed.

---

## Recommended — use the installer script

The same installer script supports removal. Run it with the uninstall option on a
host with kubectl/oc admin access:

```bash
./install_k8s_agent.sh --uninstall   [--kubeconfig ~/.kube/config]
```

> **Windows DaemonSet caveat.** A Kubernetes agent upgraded to a newer version
> **automatically includes the Windows DaemonSet agent**, but an **older**
> installer script will **not** uninstall the Windows DaemonSet. **Download the
> latest installer script** to uninstall the Windows DaemonSet agent.

---

## Verify removal

```bash
kubectl get all -n tetration          # should be empty / namespace gone
kubectl get ns tetration              # NotFound once fully removed
oc get all -n tetration               # OpenShift
```

On OpenShift, also remove the SCC binding you added:

```bash
oc adm policy remove-scc-from-user privileged -z <agent-serviceaccount> -n tetration
```

---

## Enforcement agents — confirm rules reverted

For **enforcement** agents, confirm the node and pod iptables no longer contain
Secure Workload chains after uninstall (the CNI's own rules should remain intact
thanks to *Preserve Rules*):

```bash
# On a node (debug/host shell), spot-check that CSW chains are gone
iptables-save | grep -i ta_  || echo "no CSW chains found (expected after uninstall)"
```

> If you removed the agent **abruptly** (deleted pods/DaemonSet by hand) on an
> enforcement cluster, residual rules can linger. Prefer the script's uninstall,
> which reverts cleanly.

---

## See also

- [`install/03-agent-script-installer.md`](./03-agent-script-installer.md) — install
- [`operations/01-cni-coexistence.md`](../operations/01-cni-coexistence.md) — Preserve Rules / CNI coexistence
- [`operations/02-troubleshooting.md`](../operations/02-troubleshooting.md)
